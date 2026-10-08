# Lipila API Reference

This reference provides full request/response schemas, field types, and HTTP error codes for the Lipila payment gateway.

## Environments

- **Sandbox:** `https://api.lipila.dev/api/v1`
- **Live:** `https://blz.lipila.io/api/v1`

---

## 1. POST `/collections/mobile-money`

Initiates an interactive mobile money push payment (USSD prompt). The telecom operator is automatically resolved from `accountNumber`.

### Headers
- `accept: application/json`
- `Content-Type: application/json`
- `x-api-key: <string>` (Required)
- `callbackUrl: <url>` (Optional)

### Body Parameters
| Field | Type | Required | Description |
|---|---|---|---|
| `referenceId` | string | Yes | Unique merchant-generated transaction reference |
| `amount` | number | Yes | Transaction amount (e.g. `5` or `120.50`) |
| `currency` | string | Yes | Supported: `"ZMW"`, `"USD"` |
| `narration` | string | Yes | Brief memo or description of the charge |
| `accountNumber` | string | Yes | 12-digit Zambian phone number without `+` (e.g. `"260971234567"`) |
| `email` | string | No | Optional customer email address |

### Success Response (`200 OK`)
```json
{
  "currency": "ZMW",
  "amount": 5.0,
  "accountNumber": "260971234567",
  "status": "Pending",
  "paymentType": "AirtelMoney",
  "ipAddress": "::1",
  "cardRedirectionUrl": null,
  "createdAt": "2026-10-06T10:00:00.000000+00:00",
  "referenceId": "ORDER-12345",
  "identifier": "LPLXC-20261006-1234",
  "message": "Transaction Successful"
}
```

---

## 2. POST `/collections/card`

Initiates a 3D-Secure card checkout session.

### Headers
- `accept: application/json`
- `Content-Type: application/json`
- `x-api-key: <string>` (Required)
- `callbackUrl: <url>` (Optional)

### Body Parameters
```json
{
  "customerInfo": {
    "firstName": "string",
    "lastName": "string",
    "phoneNumber": "string (e.g. 260971234567)",
    "city": "string",
    "country": "string (e.g. ZM)",
    "address": "string",
    "zip": "string",
    "email": "string"
  },
  "collectionRequest": {
    "referenceId": "string",
    "amount": 100.0,
    "narration": "string",
    "accountNumber": "string",
    "currency": "ZMW",
    "backUrl": "https://yourapp.com/payment/cancel",
    "redirectUrl": "https://yourapp.com/payment/return"
  }
}
```

### Success Response (`200 OK`)
```json
{
  "currency": "ZMW",
  "amount": 100.0,
  "accountNumber": "customer@example.com",
  "status": "Pending",
  "paymentType": "Card",
  "cardRedirectionUrl": "https://secure.3gdirectpay.com/payv2.php?ID=...",
  "createdAt": "2026-10-06T10:00:00.000000+00:00",
  "referenceId": "ORDER-CARD-1234",
  "identifier": "LPLXC-20261006-5678"
}
```

---

## 3. GET `/collections/check-status`

Queries the current settlement state of a collection transaction.

### Query Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `referenceId` | string | Yes | The merchant transaction reference used at creation |

### Response (`200 OK`)
```json
{
  "referenceId": "ORDER-12345",
  "currency": "ZMW",
  "amount": 5.0,
  "accountNumber": "260971234567",
  "status": "Successful",
  "paymentType": "AirtelMoney",
  "type": "Collection",
  "ipAddress": "::1",
  "identifier": "LPLTXNCAM-20261006120000-B4AB61CD8A8F",
  "externalId": "MP261006.1200.C7324",
  "message": "Transaction Successful"
}
```

---

## 4. POST `/disbursements/mobile-money`

Sends funds directly from merchant balance to a mobile money recipient without recipient PIN entry.

### Body Parameters
```json
{
  "referenceId": "PAYOUT-001",
  "amount": 150.0,
  "currency": "ZMW",
  "accountNumber": "260961234567",
  "narration": "Royalty Payout"
}
```

---

## 5. POST `/disbursements/bank`

Disburses funds directly into a recipient's bank account via SWIFT/BIC.

### Body Parameters
```json
{
  "referenceId": "BANK-001",
  "amount": 500.0,
  "currency": "ZMW",
  "narration": "Monthly Settlement",
  "accountNumber": "001122334455",
  "swiftCode": "ZNCOZMLU",
  "firstName": "Mwansa",
  "lastName": "Banda",
  "accountHolderName": "Mwansa Banda",
  "phoneNumber": "260971234567",
  "email": "mwansa@example.com"
}
```

---

## 6. HTTP Error Codes

| Status | Code | Meaning |
|---|---|---|
| `400` | `BAD REQUEST` | Missing required parameters or malformed JSON |
| `401` | `UNAUTHORIZED` | Invalid or missing `x-api-key` |
| `403` | `FORBIDDEN` | Merchant inactive or suspended |
| `404` | `NOT FOUND` | No transaction exists matching `referenceId` |
| `429` | `TOO MANY REQUESTS` | Rate limit throttled |
| `500` | `SERVER_ERROR` | Internal error upstream or mobile money telco failure |
