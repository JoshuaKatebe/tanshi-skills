# Zambian Telecoms & MSISDN Format Guide

Understanding Zambian mobile money networks is essential for correct usage with Lipila.

---

## 1. Network Operators & Prefixes

Zambia has three primary Mobile Network Operators (MNOs):

| Operator | Country Code | National Destination Codes (Prefixes) | Brand Name | Lipila `paymentType` |
|---|---|---|---|---|
| **Airtel Zambia** | `+260` | `097`, `077`, `057` | Airtel Money | `AirtelMoney` |
| **MTN Zambia** | `+260` | `096`, `076`, `056` | MTN Mobile Money (MoMo) | `MtnMoney` |
| **Zamtel** | `+260` | `095`, `075`, `055` | Zamtel Kwacha | `ZamtelKwacha` |

---

## 2. Phone Number Formats

| Format | Representation | Length | Example |
|---|---|---|---|
| **International (Standard)** | `260` + 9 digits | 12 digits | `260971234567` |
| **International (with +)** | `+260` + 9 digits | 13 chars | `+260971234567` |
| **National / Local** | `0` + 9 digits | 10 digits | `0971234567` |
| **Short / Bare** | 9 digits without 0 or 260 | 9 digits | `971234567` |

> **LIPILA REQUIREMENT:** All API requests sending phone numbers (`accountNumber` or `phoneNumber`) must be converted to **International format without the leading plus**: `260XXXXXXXXX` (exactly 12 digits).

---

## 3. Fallback USSD Codes

- **MTN Manual Approval:** If the push prompt does not immediately pop up on an MTN subscriber's handset, they can dial **`*115#`** -> Select `My Approvals` or `Pending Approvals` to enter their PIN and finalize payment.
- **Lipila Merchant USSD:** For customers without active data/smartphones, merchants can advise customers to dial:
  ```
  *488*<till_number>*<amount>#
  ```
  Works on MTN, Airtel, and Zamtel.
