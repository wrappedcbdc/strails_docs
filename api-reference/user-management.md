---
title: "User Management API"
sidebarTitle: "User Management"
description: "API endpoints for user onboarding with BVN verification, status management, and user details retrieval"
---

This section documents endpoints fintechs use to onboard users, verify identity, retrieve user details, and manage user status.

---

## User Management API — Endpoints Overview

| Section                     | Endpoint                       | Method | Description                                                                                 |
| --------------------------- | ------------------------------ | ------ | ------------------------------------------------------------------------------------------- |
| [**Onboard User**](#onboard-a-new-user-with-bvn-verification)            | `/onboarduser`              | POST   | Onboard a new user with BVN verification.                                                   |
| [**Check Onboarding Status**](#check-the-current-status-of-a-users-onboarding-request) | `/onboardstatus`            | POST   | Check the current status of a user onboarding request.                                      |
| [**Get User Details**](#retrieve-detailed-information-about-a-user-account)        | `/getuserdetails`           | POST   | Retrieve detailed information about a user account, including wallets and virtual accounts. |
| [**Manage User Status**](#activate-or-deactivate-a-user-account)      | `/manageuserstatus` | POST   | Activate or deactivate a user account with a reason for status change.                      |
| [**List Fintech Users**](#list-all-fintech-users)      | `/listfintechusers` | POST   | Retrieve a paginated list of all users belonging to a fintech.                              |

---

### Onboard a new user with BVN verification

<Tabs>

<Tab title="Request (Onboard)">

```http
POST {{BASE_URL}}/onboarduser
```

</Tab>

<Tab title="Body (Onboard)">

```json
{
  "bvn": "12345678901"
}
```

</Tab>

<Tab title="Response (Onboard)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "User registration initiated",
  "data": {
    "requestId": "7898f29e-10a6-4b2b-9496-f4ee50549fd4",
    "status": "processing",
    "userHash": "user_uniqueID",
    "message": "User registration request has been submitted for processing",
    "estimatedCompletionTime": "2-5 minutes"
  }
}
```

</Tab>

<Tab title="Error (Onboard)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Invalid BVN format. BVN must be 11 digits"
}
```

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "User already exists",
  "error": "A user with this BVN is already registered"
}
```

```json
{
  "status": "Error",
  "response_code": "05",
  "message": "BVN verification failed",
  "error": "Unable to verify BVN with identity provider"
}
```

</Tab>

</Tabs>

---

### Check the current status of a user's onboarding request

<Tabs>

<Tab title="Request  (User Status)">

```http
POST {{BASE_URL}}/onboardstatus
```

</Tab>

<Tab title="Body (User Status)">

```json
{
  "requestId": "7898f29e-10a6-4b2b-9496-f4ee50549fd4"
}
```

</Tab>

<Tab title="Response (User Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "User registration completed successfully",
  "data": {
    "type": "user_registration",
    "userId": "user_uniqueID",
    "status": "completed",
    "createdAt": "2026-01-18T19:33:39.945Z",
    "message": "User registration completed successfully"
  }
}
```

</Tab>

<Tab title="Error (User Status)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "Request not found",
  "error": "No onboarding request found with the provided requestId"
}
```

</Tab>

</Tabs>

---

### Retrieve detailed information about a user account

<Tabs>

<Tab title="Request (Get User)">

```http
POST {{BASE_URL}}/getuserdetails
```

</Tab>

<Tab title="Body (Get User)">

```json
{
  "userId": "user_uniqueID"
}
```

</Tab>

<Tab title="Response (Get User)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "User details retrieved successfully",
  "data": {
    "userId": "user_uniqueID",
    "isActive": true,
    "personalDetails": {
      "firstName": "John",
      "middleName": "Doe",
      "lastName": "Smith"
    },
    "walletDetails": {
      "evmWallet": "0x2KCC051A79dE65d7068542cB5D9D03754cc22F60",
      "bantuWallet": "GB63QBTTWQXRO5BNO3Z4RRAGBPQFDGAGWNHKHTGSJOZF3V2NKG7V4XTY",
      "solanaWallet": "HaXjvARhAxjKZRojmer86G31kXm52Nd4FZP4jrg4H2hf",
      "walletCreatedAt": "2025-08-20T10:15:00.000Z"
    },
      "virtualAccounts": [
          {
              "accountNumber": "1234567890",
              "accountName": "John Doe",
              "bankName": "Optimus Bank",
              "bankCode": "0000",
              "provider": "novac",
              "accountType": "user_permanent",
              "reference": "userId",
              "createdAt": "2026-05-29T12:42:45.034Z"
          }
      ]
  }
}
```

</Tab>

<Tab title="Error (Get User)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "User not found",
  "error": "No verified user found with the provided userId"
}
```

</Tab>

</Tabs>

---

### Activate or deactivate a user account

<Tabs>

<Tab title="Request (Manage User)">

```http
POST {{BASE_URL}}/manageuserstatus
```

</Tab>

<Tab title="Body (Manage User)">

```json
{
  "userId": "user_uniqueID",
  "active": false,
  "reason": "Account suspended by customer request"
}
```

</Tab>

<Tab title="Response (Manage User)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "User deactivated successfully",
  "data": {
    "userId": "user_uniqueID",
    "active": false,
    "changedAt": "2025-12-16T09:17:33.658Z"
  }
}
```

</Tab>

<Tab title="Error (Manage User)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "User not found",
  "error": "No verified user found with the provided userId"
}
```

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Reason is required when deactivating a user"
}
```

</Tab>

</Tabs>

---

### List all fintech users

<Tabs>

<Tab title="Request (List Users)">

```http
POST {{BASE_URL}}/listfintechusers
```

</Tab>

<Tab title="Body (List Users)">

```json
{
  "limit": 20,
  "offset": 0,
  "status": "all",
  "orderBy": "verifiedAt",
  "orderDirection": "desc"
}
```

| Field            | Type   | Required | Default      | Description                              |
| ---------------- | ------ | -------- | ------------ | ---------------------------------------- |
| `limit`          | number | No    | 20           | Number of users to return (1-100)        |
| `offset`         | number | No    | 0            | Number of users to skip                  |
| `status`         | string | No    | "all"        | Filter by status: "active", "inactive", "all" |
| `orderBy`        | string | No    | "verifiedAt" | Sort field: "createdAt" or "verifiedAt"  |
| `orderDirection` | string | No    | "desc"       | Sort direction: "asc" or "desc"          |

</Tab>

<Tab title="Response (List Users)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Users retrieved successfully",
  "data": {
    "users": [
      {
        "userId": "abc123...",
        "firstName": "John",
        "lastName": "Doe",
        "isActive": true,
        "hasWallet": true,
        "wallets": {
          "evm": "0x...",
          "solana": null,
          "bantu": null
        },
        "virtualAccount": {
          "accountNumber": "1234567890",
          "accountName": "John Doe",
          "bankName": "Fidelity Bank",
          "bankCode": "070",
          "provider": "redbiller"
        },
        "verifiedAt": "2026-05-10T12:00:00.000Z",
        "createdAt": "2026-05-10T12:00:00.000Z"
      }
    ],
    "pagination": {
      "total": 150,
      "limit": 20,
      "offset": 0,
      "currentPage": 1,
      "totalPages": 8,
      "hasNextPage": true,
      "hasPreviousPage": false
    }
  }
}
```

</Tab>

</Tabs>
