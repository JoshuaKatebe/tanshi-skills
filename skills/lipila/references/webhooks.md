# Lipila Webhook Architecture & Security

Lipila sends asynchronous HTTP `POST` notifications whenever a transaction transitions state (e.g. `Pending` -> `Successful` or `Failed`).

## Webhook Headers
Lipila attaches three standard headers to every callback:
1. `webhook-id`: Unique message ID (e.g., `3f2a1b4c-5d6e-7f8a-9b0c-1d2e3f4a5b6c`). Guaranteed stable across retries.
2. `webhook-timestamp`: Unix epoch time in seconds (e.g., `1728243600`).
3. `webhook-signature`: Space-delimited signatures prefixed with `v1,`.

---

## Cryptographic Specification

- **Algorithm:** HMAC-SHA256
- **Signing Key:** Base64-encoded 32-byte secret (found in Lipila Dashboard under **Settings -> Webhooks**)
- **Data to sign:** `"{webhook-id}.{webhook-timestamp}.{raw_body}"`
- **Output:** `"v1," + Base64(HMAC_SHA256(decoded_key, data))`

> **CRITICAL REQUIREMENT:** You must use the raw string / byte buffer of the request body. Never parse JSON and re-stringify it before checking the signature, as spacing and key ordering differences invalidate cryptographic hashes.

---

## Verification Implementations

### Node.js / Next.js (TypeScript)

```typescript
import crypto from "crypto";

export function verifyLipilaWebhook({
  rawBody,
  webhookId,
  webhookTimestamp,
  webhookSignature,
  base64Secret,
  maxAgeSeconds = 300,
}: {
  rawBody: string;
  webhookId: string;
  webhookTimestamp: string;
  webhookSignature: string;
  base64Secret: string;
  maxAgeSeconds?: number;
}): { valid: boolean; reason?: string } {
  // 1. Replay Attack Prevention
  const timestampNum = parseInt(webhookTimestamp, 10);
  if (isNaN(timestampNum)) {
    return { valid: false, reason: "Invalid timestamp" };
  }
  const currentEpoch = Math.floor(Date.now() / 1000);
  if (Math.abs(currentEpoch - timestampNum) > maxAgeSeconds) {
    return { valid: false, reason: "Timestamp expired" };
  }

  // 2. Decode Secret
  const keyBuffer = Buffer.from(base64Secret, "base64");

  // 3. Compute Expected Signature
  const signedPayload = `${webhookId}.${webhookTimestamp}.${rawBody}`;
  const computedHash = crypto
    .createHmac("sha256", keyBuffer)
    .update(signedPayload, "utf8")
    .digest("base64");
  const expectedSignature = `v1,${computedHash}`;

  // 4. Timing-safe comparison against signatures (supports key rotation)
  const incomingSignatures = webhookSignature.split(" ").map((s) => s.trim());
  const expectedBuffer = Buffer.from(expectedSignature);

  const matched = incomingSignatures.some((candidate) => {
    const candidateBuffer = Buffer.from(candidate);
    if (candidateBuffer.length !== expectedBuffer.length) return false;
    return crypto.timingSafeEqual(candidateBuffer, expectedBuffer);
  });

  return { valid: matched, reason: matched ? undefined : "Signature mismatch" };
}
```

### Python (FastAPI / Flask / Django)

```python
import base64
import hashlib
import hmac
import time

def verify_lipila_webhook(raw_body: bytes, webhook_id: str, webhook_timestamp: str, webhook_signature: str, base64_secret: str, max_age_seconds: int = 300) -> bool:
    try:
        sent_time = int(webhook_timestamp)
    except ValueError:
        return False
        
    # Replay protection
    if abs(time.time() - sent_time) > max_age_seconds:
        return False

    key = base64.b64decode(base64_secret)
    signed_payload = f"{webhook_id}.{webhook_timestamp}.".encode("utf-8") + raw_body
    expected_sig = "v1," + base64.b64encode(
        hmac.new(key, signed_payload, hashlib.sha256).digest()
    ).decode("utf-8")

    signatures = [s.strip() for s in webhook_signature.split(" ")]
    return any(hmac.compare_digest(expected_sig, s) for s in signatures)
```

---

## Idempotency Best Practice

Always track `webhook-id` in your database or cache (e.g. Redis with a 24-48 hour TTL).
When a webhook arrives:
1. Verify signature.
2. Check if `webhook-id` has already been recorded.
3. If already recorded, return `200 OK` immediately without re-processing side-effects.
4. If new, record `webhook-id`, execute fulfillment logic in a transaction, and return `200 OK`.
