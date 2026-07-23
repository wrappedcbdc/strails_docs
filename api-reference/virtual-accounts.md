---
title: "Virtual Accounts API"
sidebarTitle: "Virtual Accounts"
description: "API endpoints for retrieving and managing virtual accounts for NGN funding operations"
---

This section documents endpoints fintechs use to **retrieve and manage virtual accounts** for funding and payment operations.

<Info>
**New to Virtual Accounts?** See the [Virtual Accounts Guide](/virtual-accounts-guide) first for an overview of account types, deposit flows, and best practices.
</Info>

---

## Virtual Accounts API — Endpoints Overview

| Section                         | Endpoint                       | Method | Description                                                                                        |
| ------------------------------- | ------------------------------ | ------ | -------------------------------------------------------------------------------------------------- |
| [**Get Fintech Virtual Account**](#retrieve-the-ngn-virtual-account-assigned-to-your-fintech) | `/getfintechvirtualaccount` | GET    | Retrieve the NGN virtual account created for a fintech during onboarding. Used for onramp funding. |
| [**Get Virtual Account**](#retrieve-a-specific-virtual-account-with-details-about-amounts-and-fees)         | `/getvirtualaccount`        | POST   | Retrieve details of a specific virtual account, including fees, total amount, and status.          |

---

### Retrieve the NGN virtual account assigned to your fintech

<Tabs>

<Tab title="Request (Fintech Account)">

```http
GET {{BASE_URL}}/getfintechvirtualaccount
```

</Tab>

<Tab title="Response (Fintech Account)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Virtual account retrieved successfully",
  "data": {
    "virtualAccounts": [
    {
      "accountNumber": "1234567890",
      "accountName": "Example Fintech Ltd",
      "bankName": "ALP Bank",
      "provider": "nutmeg",
      "status": "active",
      "createdAt": "2026-01-04T05:26:39.804Z"
    }
  ]
  }
}
```

</Tab>

</Tabs>

---

### Retrieve a specific virtual account with details about amounts and fees

<Tabs>

<Tab title="Request (Get Account)">

```http
POST {{BASE_URL}}/getvirtualaccount
```

</Tab>

<Tab title="Body (Get Account)">

```json
{
  "requestId": "a9f706ad-6861-4669-904a-5d4e0d9963c3"
}
```

</Tab>

<Tab title="Response (Get Account)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Virtual account retrieved successfully. Virtual account amount is before gateway fee. User must pay total amount including all fees.",
  "data": {
    "requestId": "a1b2c3d4-5678-90ab-cdef-111213141516",
    "virtualAccount": {
      "accountNumber": "0123456789",
      "bankName": "Example Bank",
      "accountName": "Company Name/John Doe Smith",
      "amount": 528.84,
      "createdAt": "2026-06-09T08:33:17.614Z",
      "baseAmount": 520,
      "feeAmount": 8.84,
      "totalAmountWithFee": 528.84,
      "feeBreakdown": {
          "userRequestedAmount": 520,
          "fintechFeeAmount": 0,
          "fintechFeePercentage": 0,
          "fintechFeeCapped": false,
          "strailsFeeAmount": 8.84,
          "strailsFeePercentage": 1.7,
          "strailsFeeCapped": false,
          "totalFeeAmount": 8.84,
          "finalAmount": 528.84,
          "amountToWallet": 520
      }
    },
    "status": "created",
    "walletAddress": "0xAbC123dEf456GhI789jKl012MnO345pQrS678tUv",
    "version": "1.0.0"
  }
}
```

</Tab>

<Tab title="Error (Get Account)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "Virtual account not found",
  "error": "No virtual account found with the provided requestId"
}
```

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Virtual account expired",
  "error": "This virtual account has expired (30 min timeout)"
}
```

</Tab>

</Tabs>

---

<Info>
**Looking for conceptual documentation?** See [Virtual Accounts Guide](/virtual-accounts-guide) for details on account types, deposit flows, BVN validation, and best practices.
</Info>