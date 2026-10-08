---
name: lipila
description: >-
  Use when integrating, configuring, or troubleshooting the Lipila payment gateway for mobile money (MTN, Airtel, Zamtel),
  card payments (Visa, Mastercard), and bank disbursements in Zambia. Covers sandbox/live environment setup,
  initiating collections (proximity push USSD prompt & hosted card redirect), polling transaction status,
  Zambian MSISDN telephone normalization & telco detection (097/077/057 Airtel, 096/076/056 MTN, 095/075/055 Zamtel),
  and robust HMAC-SHA256 webhook signature verification with replay attack protection.
metadata:
  author: lenga
  version: "1.0.0"
---

# Lipila Payment Gateway Integration

Lipila is a Zambian payment gateway providing collection and disbursement APIs for Mobile Money (MTN, Airtel, Zamtel), Cards (Visa, Mastercard), and Bank Transfers.

---

## 1. Quick Reference & Environment Config

### Base URLs & Portals

| Environment | Dashboard | API Base URL |
|---|---|---|
| **Sandbox** | `https://dashboard.lipila.dev` | `https://api.lipila.dev/api/v1` |
| **Live / Production** | `https://dashboard.lipila.io` | `https://blz.lipila.io/api/v1` |

### Environment Variables (.env)

```env
# Lipila Credentials
LIPILA_ENV=sandbox # or "live"
LIPILA_BASE_URL=https://api.lipila.dev/api/v1 # or https://blz.lipila.io/api/v1
LIPILA_API_KEY=your_api_key_here
LIPILA_WEBHOOK_SECRET=your_base64_signing_secret_here

# App Callbacks
NEXT_PUBLIC_APP_URL=https://yourdomain.com
LIPILA_CALLBACK_URL=https://yourdomain.com/api/webhooks/lipila
```

### Common Request Headers

Every API request requires:
- `accept: application/json`
- `Content-Type: application/json` (for POST)
- `x-api-key: <YOUR_API_KEY>`
- `callbackUrl: <YOUR_WEBHOOK_URL>` (Optional, receives async status changes)

---

## 2. Zambian Phone Number (MSISDN) Normalization

Lipila automatically infers the telecom operator (MTN, Airtel, Zamtel) from the `accountNumber` phone number. However, the number **must be formatted in international format without the leading `+`** (e.g. `260971234567`).

### Network Prefix Mapping (Zambia)

| Provider | Prefixes | Lipila `paymentType` |
|---|---|---|
| **Airtel** | `097`, `077`, `057` | `AirtelMoney` |
| **MTN** | `096`, `076`, `056` | `MtnMoney` |
| **Zamtel** | `095`, `075`, `055` | `ZamtelKwacha` |

### TypeScript Normalizer & Detector

```typescript
export type LipilaProvider = "AIRTEL" | "MTN" | "ZAMTEL" | "UNKNOWN";

export function normalizeZambianNumber(input: string): {
  isValid: boolean;
  international: string; // e.g. "260971234567"
  provider: LipilaProvider;
  formattedLocal: string; // e.g. "0971234567"
} {
  // Strip spaces, dashes, parentheses, plus signs
  let clean = input.replace(/[\s\-\(\)\+]/g, "");

  // Convert local 09x/07x/05x to 260...
  if (clean.startsWith("0") && clean.length === 10) {
    clean = "260" + clean.slice(1);
  } else if (/^(9|7|5)/.test(clean) && clean.length === 9) {
    clean = "260" + clean;
  }

  // Must be 12 digits starting with 260
  const isMatch = /^260(97|77|57|96|76|56|95|75|55)\d{6}$/.test(clean);
  if (!isMatch) {
    return { isValid: false, international: clean, provider: "UNKNOWN", formattedLocal: clean };
  }

  const prefix = clean.substring(3, 5); // 97, 77, 57, 96, etc.
  let provider: LipilaProvider = "UNKNOWN";

  if (["97", "77", "57"].includes(prefix)) provider = "AIRTEL";
  else if (["96", "76", "56"].includes(prefix)) provider = "MTN";
  else if (["95", "75", "55"].includes(prefix)) provider = "ZAMTEL";

  return {
    isValid: true,
    international: clean,
    provider,
    formattedLocal: "0" + clean.slice(3),
  };
}
```

---

## 3. Collections (Accepting Customer Payments)

### 3.1 Mobile Money Push Prompt (USSD Push)

Pushes an instant USSD PIN entry prompt to the customer's mobile phone.

- **Endpoint:** `POST /collections/mobile-money`
- **Request Body:**
```json
{
  "referenceId": "ORDER-987123",
  "amount": 5.00,
  "currency": "ZMW",
  "narration": "Book purchase: Moby Dick",
  "accountNumber": "260971234567",
  "email": "customer@example.com"
}
```

> **UX Note:** For MTN users, if the push prompt is delayed, advise the customer they can dial `*115#` on their handset to approve pending approval requests under approvals/my wallet.

### 3.2 Card Payment (Visa / Mastercard)

Initiates a 3D-Secure hosted checkout flow for card payments.

- **Endpoint:** `POST /collections/card`
- **Request Body:**
```json
{
  "customerInfo": {
    "firstName": "Chipo",
    "lastName": "Tembo",
    "phoneNumber": "260971234567",
    "city": "Lusaka",
    "country": "ZM",
    "address": "Plot 104, Great East Road",
    "zip": "10101",
    "email": "chipo@example.com"
  },
  "collectionRequest": {
    "referenceId": "ORDER-987124",
    "amount": 25.00,
    "currency": "ZMW",
    "narration": "Lenga Plus Subscription",
    "accountNumber": "260971234567",
    "backUrl": "https://yourapp.com/payment/cancelled",
    "redirectUrl": "https://yourapp.com/payment/callback"
  }
}
```
- **Response:** Look for `cardRedirectionUrl` in response and redirect the client to that URL.

### 3.3 Check Collection Status (Polling)

- **Endpoint:** `GET /collections/check-status?referenceId=<referenceId>`
- **Response Status Values:**
  - `Pending`: Customer has not yet entered their PIN or bank authorization is ongoing.
  - `Successful`: Payment received and credited to merchant wallet.
  - `Failed`: Transaction declined, customer timed out, insufficient balance, or cancelled.

---

## 4. Disbursements (Sending Money / Payouts)

Disbursements do **not** require the beneficiary to enter a PIN.

### 4.1 Mobile Money Payout
- **Endpoint:** `POST /disbursements/mobile-money`
- **Payload:** `{ "referenceId": "PAYOUT-001", "amount": 100, "currency": "ZMW", "accountNumber": "260961234567", "narration": "Author Royalty" }`

### 4.2 Bank Payout
- **Endpoint:** `POST /disbursements/bank`
- **Payload:**
```json
{
  "referenceId": "PAYOUT-002",
  "amount": 500,
  "currency": "ZMW",
  "narration": "Publisher Monthly Settlement",
  "accountNumber": "001122334455",
  "swiftCode": "ZNCOZMLU",
  "firstName": "Mwansa",
  "lastName": "Banda",
  "accountHolderName": "Mwansa Banda",
  "phoneNumber": "260971234567"
}
```

### 4.3 Check Disbursement Status
- **Endpoint:** `GET /disbursements/check-status?referenceId=<referenceId>`

---

## 5. Webhook Implementation & Signature Verification

Lipila signs webhook notifications using **HMAC-SHA256**. The signing secret is a base64-encoded 32-byte string found in your dashboard under **Settings → Webhooks**.

### Required Verification Steps
1. **Extract Headers:**
   - `webhook-id`: Unique message ID (idempotency key).
   - `webhook-timestamp`: Unix timestamp in seconds.
   - `webhook-signature`: Space-delimited signatures prefixed with `v1,`.
2. **Replay Attack Protection:** Reject requests where `abs(now - webhook-timestamp) > 300` (5 minutes).
3. **Raw Body Bytes:** Construct signed payload: `${webhook-id}.${webhook-timestamp}.${raw_body}`.
4. **Timing-Safe Comparison:** Use `crypto.timingSafeEqual`.

### Next.js App Router Webhook Route (`app/api/webhooks/lipila/route.ts`)

```typescript
import { NextRequest, NextResponse } from "next/server";
import crypto from "crypto";

export async function POST(req: NextRequest) {
  try {
    const rawBody = await req.text(); // Raw payload string
    const webhookId = req.headers.get("webhook-id");
    const webhookTimestamp = req.headers.get("webhook-timestamp");
    const webhookSignature = req.headers.get("webhook-signature");
    const secret = process.env.LIPILA_WEBHOOK_SECRET;

    if (!webhookId || !webhookTimestamp || !webhookSignature || !secret) {
      return NextResponse.json({ error: "Missing webhook headers" }, { status: 400 });
    }

    // 1. Replay prevention (5 minute tolerance)
    const now = Math.floor(Date.now() / 1000);
    const sentTime = parseInt(webhookTimestamp, 10);
    if (isNaN(sentTime) || Math.abs(now - sentTime) > 300) {
      return NextResponse.json({ error: "Timestamp expired" }, { status: 400 });
    }

    // 2. Decode base64 secret to raw bytes
    const key = Buffer.from(secret, "base64");
    const signedPayload = `${webhookId}.${webhookTimestamp}.${rawBody}`;

    const computedSignature = "v1," + crypto
      .createHmac("sha256", key)
      .update(signedPayload, "utf8")
      .digest("base64");

    // 3. Compare signatures with constant-time equality
    const signatures = webhookSignature.split(" ");
    const isValid = signatures.some((sig) => {
      const a = Buffer.from(sig.trim());
      const b = Buffer.from(computedSignature);
      return a.length === b.length && crypto.timingSafeEqual(a, b);
    });

    if (!isValid) {
      return NextResponse.json({ error: "Invalid signature" }, { status: 401 });
    }

    // 4. Parse payload and process idempotently
    const event = JSON.parse(rawBody);
    const { referenceId, status, paymentType, amount } = event;

    // TODO: Update order / user wallet / book license in database

    return NextResponse.json({ received: true });
  } catch (error: any) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
}
```

---

## 6. Integration Checklist

- [ ] Merchant account created on `dashboard.lipila.dev` (sandbox) and live.
- [ ] Wallet and till number generated.
- [ ] `LIPILA_API_KEY` and `LIPILA_WEBHOOK_SECRET` stored in `.env.local` / environment secrets.
- [ ] Phone numbers validated and normalized to `260XXXXXXXXX` before calling `/collections/mobile-money`.
- [ ] Network indicator UI (Airtel / MTN / Zamtel) auto-detects from prefixes (`097/077/057`, `096/076/056`, `095/075/055`).
- [ ] Unique `referenceId` generated for every charge and disbursement.
- [ ] Webhook endpoint verifies HMAC-SHA256 signature with raw request body and 300-second timestamp window.
- [ ] Fallback polling implemented on the client/server via `/collections/check-status`.
- [ ] Error handling captures provider-specific failure codes (e.g. `User didn't enter the pin`, `LOW_BALANCE_OR_PAYEE_LIMIT_REACHED_OR_NOT_ALLOWED`).
