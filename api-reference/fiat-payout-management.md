---
title: "Bank Account Management API"
sidebarTitle: "Bank Accounts"
description: "API endpoints for managing Nigerian bank accounts for offramp and fiat payout operations"
---

This section documents endpoints fintechs use to **manage Nigerian bank accounts** for **offramp (fiat payout and settlement) operations**.

---

## Endpoints Overview

| Section             | Endpoint                | Method | Description                                                               |
| ------------------- | ----------------------- | ------ | ------------------------------------------------------------------------- |
| [**Bank Codes**](#retrieve-supported-banks-nigeria)          | `/getbankscode`      | POST   | Retrieve supported Nigerian banks.                                        |
| [**Add Bank Account**](#add-a-new-nigerian-bank-account-for-fintech-payout)    | `/addbankaccount`    | POST   | Add and verify a new bank account for your fintech account.               |
| [**Update Bank Account**](#update-bank-account) | `/updatebankaccount` | PUT    | Update details of an existing bank account.                               |
| [**Delete Bank Account**](#delete-bank-account) | `/deletebankaccount` | DELETE | Remove a bank account from your fintech configuration.                    |
| [**Get Single Bank Account**](#retrieve-single-bank-account)   | `/getbankaccounts`   | GET    | Retrieve a single bank account configured for your fintech account. |
| [**Get Bank Accounts**](#retrieve-all-bank-accounts)   | `/listbankaccounts`   | GET    | Retrieve all bank accounts currently configured for your fintech account. |

---

### Retrieve Supported Banks (Nigeria)

<Tabs>

<Tab title="Request (All Banks)">

```http
POST {{BASE_URL}}/getbankscode
```

</Tab>

<Tab title="Response (All Banks)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Banks retrieved successfully",
  "data": {
    "countryCode": "NG",
    "banks": [
      { "bank_name": "AB MICROFINANCE BANK", "bank_code": "090270" },
      { "bank_name": "HACKMAN MICROFINANCE BANK", "bank_code": "090147" }
    ]
  }
}
```

</Tab>

</Tabs>

---

### Add a New Nigerian Bank Account for Fintech Payout

<Info>
Accounts are automatically verified against bank records before activation.
</Info>

<Tabs>

<Tab title="Request (Add Bank)">

```http
POST {{BASE_URL}}/addbankaccount
```

</Tab>

<Tab title="Body (Add Bank)">

```json
{
  "accountNumber": "1234567890",
  "bankCode": "044",
  "isDefault": false
}
```

### Parameters

| Parameter       | Type    | Required | Description |
|-----------------|---------|----------|-------------|
| `accountNumber` | string  | Yes   | Bank account number |
| `bankCode`      | string  | Yes   | Bank identifier code (e.g. NIBSS bank code like `044`) |
| `isDefault`     | boolean | No    | Marks this bank account as the default payout account |

</Tab>

<Tab title="Response (Add Bank)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Bank account added and verified successfully",
  "data": {
    "accountId": "uuid-account-id",
    "verifiedAccountName": "JOHN DOE",
    "accountNumber": "0123456789",
    "bankName": "NoAccess Bank",
    "bankCode": "044",
    "isValidated": true,
    "message": "Bank account added successfully"
  }
}
```

</Tab>

<Tab title="Error (Add Bank)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Account number must be 10 digits"
}
```

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Bank account verification failed",
  "error": "Account name does not match or account is invalid"
}
```

</Tab>

</Tabs>

---

### Update Bank Account

<Tabs>

<Tab title="Request (Update Bank)">

```http
PUT {{BASE_URL}}/updatebankaccount
```

</Tab>

<Tab title="Body (Update Bank)">

```json
{
  "accountId": "uuid-account-id",
  "isDefault": true
}
```

### Parameters

| Parameter     | Type    | Required | Description |
|--------------|---------|----------|-------------|
| `accountId`   | string  | Yes   | Unique identifier (UUID) of the bank account to update |
| `isDefault`   | boolean | No    | Sets this bank account as the default payout account |


</Tab>

<Tab title="Response (Update Bank)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Bank account updated successfully"
}
```

</Tab>

<Tab title="Error (Update Bank)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "Bank account not found",
  "error": "No bank account found with the provided accountId"
}
```

</Tab>

</Tabs>

---

### Delete Bank Account
<Info>
Accounts with **pending offramp transactions cannot be deleted**.
</Info>

<Tabs>

<Tab title="Request (Delete Bank)">

```http
DELETE {{BASE_URL}}/deletebankaccount?accountId=uuid-account-id
```

</Tab>

<Tab title="Response (Delete Bank)">

```json
{
   "status": "Success",
  "response_code": "00",
  "message": "Bank account deleted successfully"
}
```

</Tab>

<Tab title="Error (Delete Bank)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Cannot delete account with pending offramp transactions",
  "error": "PENDING_TRANSACTIONS"
}
```

</Tab>

</Tabs>

---


### Retrieve Single Bank Account

<Tabs>

<Tab title="Request (Account)">

```http
GET {{BASE_URL}}/getbankaccount?accountId=uuid-account-id
```
### Query Parameters

| Parameter   | Type   | Required | Description                                      |
| ----------- | ------ | -------- | ------------------------------------------------ |
| `accountId` | string | Yes   | Unique identifier (UUID) of the added bank account |

</Tab>

<Tab title="Response (Account)">

```json
{
  "status": "00",
  "message": "Bank accounts retrieved successfully",
  "data": {
    "accounts": [
      {
        "accountId": "account-uuid",
        "fintechId": "fintech-uuid-here",
        "accountName": "JOHN DOE",
        "accountNumber": "0123456789",
        "bankName": "Access Bank",
        "bankCode": "044",
        "isDefault": true,
        "isValidated": true,
        "validationDetails": {
          "validatedAt": "2026-02-04T10:30:00.000Z",
          "accountNameMatch": true
        },
        "createdAt": "2026-02-04T10:30:00.000Z",
        "updatedAt": "2026-02-04T10:30:00.000Z"
      }
    ]
  }
}
```

</Tab>

</Tabs>

---

### Retrieve All Bank Accounts

<Tabs>

<Tab title="Request (Accounts)">

```http
GET {{BASE_URL}}/listbankaccounts
```

</Tab>

<Tab title="Response (Accounts)">

```json
{
  "status": "00",
  "message": "Bank accounts retrieved successfully",
  "data": {
    {
  "accounts": [
    {
      "accountId": "a1b2c3d4-...",
      "fintechId": "fintech-uuid",
      "accountName": "JOHN DOE",
      "accountNumber": "0123456789",
      "bankName": "Access Bank",
      "bankCode": "044",
      "isDefault": true,
      "isValidated": true,
      "validationDetails": {
                    "validatedAt": "2026-04-27T08:32:24.917Z",
                    "accountNameMatch": true
                },
      "createdAt": "2026-02-04T10:30:00.000Z",
      "updatedAt": "2026-02-04T10:30:00.000Z"
    },
    {
      "accountId": "b2c3d4e5-...",
      "fintechId": "fintech-uuid",
      "accountName": "JOHN DOE ENTERPRISES",
      "accountNumber": "9876543210",
      "bankName": "GTBank",
      "bankCode": "058",
      "isDefault": false,
      "isValidated": true,
      "validationDetails": {
                    "validatedAt": "2026-04-27T08:32:24.917Z",
                    "accountNameMatch": true
                },
      "createdAt": "2026-02-03T08:00:00.000Z",
      "updatedAt": "2026-02-03T08:00:00.000Z"
    }
  ],
  "total": 2,
  "defaultAccount": {
    "accountId": "a1b2c3d4-...",
    "accountName": "JOHN DOE",
    ...
  }
}
  }
}
```

</Tab>

</Tabs>