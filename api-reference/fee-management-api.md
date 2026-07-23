---
title: "Fee Management API"
sidebarTitle: "Fee Management"
description: "API endpoints for configuring fees, tracking accumulated fees, and processing fee withdrawals"
---

This section documents endpoints fintechs use to **configure fee structures**, **track accumulated fees**, and **process fee withdrawals** to linked bank accounts.

---

## Fee Management API - Endpoints Overview

| Section                    | Endpoint                       | Method | Description                                                                                     |
| -------------------------- | ------------------------------ | ------ | ----------------------------------------------------------------------------------------------- |
| [**Get Withdrawal History**](#retrieve-fintech-fee-withdrawal-history) | `/getwithdrawalhistory`     | GET    | Retrieve paginated history of all fintech fee withdrawals.                                      |
| [**Request Fee Withdrawal**](#request-withdrawal-of-collected-fintech-fees-to-a-destination-bank-account) | `/feewithdrawal` | POST   | Request withdrawal of accumulated fintech fees to a linked bank account.                        |
| [**Get Accumulated Fees**](#get-a-summary-of-accumulated-fees-and-available-balance-for-withdrawal)   | `/getaccumulatedfees`       | GET    | Retrieve a summary of all accumulated fees, including available and pending amounts.            |
| [**Manage Fees**](#configure-your-fee-settings)            | `/managefees`               | PUT    | Configure onramp/offramp fees for your fintech account, including percentages, caps, and rules. |
| [**Get Fees**](#retrieve-the-current-fee-configuration-and-an-example-calculation-for-typical-amounts)               | `/getfees`                  | GET    | Retrieve the currently configured fees and example calculations for transparency.               |
| [**Preview Strails Fee**](#preview-strails-fee-for-onramp-or-offramp-transactions) | `/fees/strails/preview`    | POST   | Calculate the Strails platform fee for a given amount and transaction type.                     |
| [**Verify Withdrawal**](#verify-the-status-or-outcome-of-a-withdrawal-previously-requested-via-fintechrequestwithdrawal)      | `/verifywithdrawal`        | POST   | Verify the status or outcome of a previously requested fee withdrawal.                          |

---

### Retrieve fintech fee withdrawal history
<Info>
Supports query parameters for pagination and filtering by `status`
</Info>
<Tabs>
<Tab title="Request (Get History)">

```http
GET {{BASE_URL}}/getwithdrawalhistory?limit=20&status=completed
```

</Tab>

<Tab title="Response (Get History)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Withdrawal history retrieved successfully",
  "data": {
    "withdrawals": [
      {
        "withdrawalId": "FTW_your_fintech_id_1725444600000",
        "amount": 50000,
        "bankAccountNumber": "123456****",
        "bankCode": "058",
        "accountName": "YOUR COMPANY NAME",
        "narration": "Fee withdrawal for August 2025",
        "status": "completed",
        "createdAt": "2025-09-04T10:30:00.000Z",
        "processedAt": "2025-09-04T12:15:00.000Z",
        "failureReason": null
      }
    ],
    "pagination": { "count": 1, "limit": 20, "hasMore": false }
  }
}
```

</Tab>
</Tabs>


### Request withdrawal of collected fintech fees to a destination bank account

<Tabs>
<Tab title="Request (Withdrawal)">

```http
POST {{BASE_URL}}/feewithdrawal
```
</Tab>

<Tab title="Body (Withdrawal)">

```json
{
  "bankAccountNumber": "1234567890",
  "bankCode": "058",
  "accountName": "YOUR COMPANY NAME",
  "amount": 50000,
  "narration": "Fee withdrawal for August 2025",
  "metadata": {
    "reference": "FEE_WITHDRAWAL_AUG_2025",
    "notes": "Monthly fee withdrawal"
  }
}
```

### Parameters

| Parameter          | Type    | Required | Description |
|-------------------|---------|----------|-------------|
| `bankAccountNumber`| string  | Yes   | Destination bank account number |
| `bankCode`         | string  | Yes   | Bank identifier code (e.g., NIBSS code `058`) |
| `accountName`      | string  | Yes   | Name of the account holder (e.g., your company name) |
| `amount`           | number  | Yes   | Amount to withdraw (in the smallest currency unit) |
| `narration`        | string  | No    | Optional description for the transaction (e.g., “Fee withdrawal for August 2025”) |
| `metadata`         | object  | No    | Additional info related to the withdrawal |

#### `metadata` Object

| Field       | Type   | Required | Description |
|------------|--------|----------|-------------|
| `reference`| string | No    | Unique reference ID for the transaction |
| `notes`    | string | No    | Optional notes or comments about the withdrawal |


</Tab>

<Tab title="Response (Withdrawal)">
```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Withdrawal request submitted successfully",
  "data": {
    "withdrawalId": "FTW_your_fintech_id_1725444600000",
    "amount": 50000,
    "status": "pending",
    "estimatedProcessingTime": "1-3 business days"
  }
}
```
</Tab>

<Tab title="Error (Withdrawal)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Amount exceeds available balance"
}
```

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Amount is below minimum withdrawal threshold"
}
```

</Tab>

</Tabs>


### Get a summary of accumulated fees and available balance for withdrawal

<Tabs>
<Tab title="Request (Get Fees)">

```http
GET {{BASE_URL}}/getaccumulatedfees
```

</Tab>


<Tab title="Response (Get Fees)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Accumulated fees retrieved successfully",
  "data": {
    "summary": {
      "totalAccumulatedFees": 125000,
      "onrampFees": 100000,
      "offrampFees": 25000,
      "collectedFees": 120000,
      "pendingFees": 5000,
      "transactionCount": 150,
      "pendingWithdrawals": 20000,
      "availableForWithdrawal": 100000
    },
    "details": {
      "lastUpdated": "2025-09-04T10:30:00.000Z",
      "currency": "NGN",
      "minimumWithdrawal": 1000,
      "maximumWithdrawal": 10000000
    }
  }
}
```

</Tab>

</Tabs>


### Configure your fee settings
<Info>
Both `onrampFee` and `offrampFee` are optional; omit fields you do not want to change
</Info>
<Tabs>
<Tab title="Request (Manage Fee)">

```http
PUT {{BASE_URL}}/managefees
```
</Tab>


<Tab title="Body (Manage Fee)">

```json
{
  "onrampFee": {
    "percentageFee": 1.5,
    "capFee": 500,
    "enabled": true
  },
  "offrampFee": {
    "percentageFee": 2.0,
    "capFee": 1000,
    "enabled": true
  },
  "metadata": {
    "description": "Updated fee configuration for Q4 2025",
    "notes": "Reduced onramp fees, increased offramp fees",
    "lastUpdatedBy": "admin@yourfintech.com"
  }
}
```

### Parameters

| Parameter     | Type    | Required | Description |
|---------------|---------|----------|-------------|
| `onrampFee`   | object  | Yes   | Configuration for onramp (funding) fees |
| `offrampFee`  | object  | Yes   | Configuration for offramp (withdrawal) fees |
| `metadata`    | object  | No    | Additional information about the fee update |

#### `onrampFee` & `offrampFee` Object

| Field           | Type    | Required | Description |
|-----------------|---------|----------|-------------|
| `percentageFee` | number  | Yes   | Fee percentage applied to the transaction (e.g., 1.5 for 1.5%) |
| `capFee`        | number  | Yes   | Maximum fee amount that can be charged |
| `enabled`       | boolean | Yes   | Enable or disable this fee |

#### `metadata` Object

| Field          | Type    | Required | Description |
|----------------|---------|----------|-------------|
| `description`  | string  | No    | Short description of the fee change |
| `notes`        | string  | No    | Additional notes or rationale for the update |
| `lastUpdatedBy`| string  | No    | Email or identifier of the admin who made the change |

</Tab>

<Tab title="Response (Manage Fee)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Fee configuration updated successfully",
  "data": {
    "fintechId": "your_fintech_id",
    "operation": "updated",
    "feeConfiguration": { /* ... */ },
    "exampleCalculation": { /* ... */ }
  }
}
```

</Tab>
</Tabs>

### Retrieve the current fee configuration and an example calculation for typical amounts

<Tabs>
<Tab title="Request (Get Fees)">

```http
GET {{BASE_URL}}/getfees
```

</Tab>

<Tab title="Response (Get Fees)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Fee configuration retrieved successfully",
  "data": {
    "fintechId": "your_fintech_id",
    "hasConfiguration": true,
    "feeConfiguration": {
      "onrampFee": { "percentageFee": 1.5, "capFee": 500, "enabled": true },
      "metadata": { "description": "Onramp fee configuration", "lastUpdatedBy": "your_fintech_id" }
    },
    "timestamps": {
      "createdAt": "2025-09-10T11:02:11.539Z",
      "updatedAt": "2026-04-29T05:47:38.653Z"
        },
      "version": "1.0.0"
  }
}
```

</Tab>
</Tabs>

### Preview Strails fee for onramp or offramp transactions
<Info>
This endpoint calculates the Strails platform fee (separate from fintech fees) based on the transaction amount and type. Useful for displaying fee breakdowns to end users before confirming a transaction.
</Info>
<Tabs>
<Tab title="Request (Preview)">

```http
POST {{BASE_URL}}/fees/strails/preview
```

</Tab>

<Tab title="Body (Preview)">

```json
{
  "amount": 150000,
  "transactionType": "offramp"
}
```

### Parameters

| Parameter         | Type   | Required | Description |
|-------------------|--------|----------|-------------|
| `amount`          | number | Yes      | Transaction amount (in the smallest currency unit) |
| `transactionType` | string | Yes      | Type of transaction: `onramp` or `offramp` |

</Tab>

<Tab title="Response (Preview)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Strails fee calculated",
  "data": {
    "input": {
      "amount": 150000,
      "transactionType": "offramp",
      "fintechId": "fintech-123"
    },
    "result": {
      "amount": 150000,
      "feePercentage": 0.4,
      "calculatedFee": 600,
      "fixedFee": 0,
      "finalFee": 600,
      "wasCapped": false,
      "appliedCap": null,
      "tierUsed": {
        "minAmount": 100000,
        "maxAmount": 500000
      },
      "transactionType": "offramp",
      "usedFintechOverride": false
    }
  }
}
```

### Response Fields

| Field                  | Type    | Description |
|------------------------|---------|-------------|
| `input.amount`         | number  | The input amount provided |
| `input.transactionType`| string  | The transaction type (`onramp` or `offramp`) |
| `input.fintechId`      | string  | The authenticated fintech's ID |
| `result.feePercentage` | number  | The percentage fee applied |
| `result.calculatedFee` | number  | Fee calculated from percentage |
| `result.fixedFee`      | number  | Any fixed fee component |
| `result.finalFee`      | number  | Total fee to be charged |
| `result.wasCapped`     | boolean | Whether the fee was capped |
| `result.appliedCap`    | number  | The cap amount if applied, otherwise `null` |
| `result.tierUsed`      | object  | The fee tier that matched the amount |
| `result.usedFintechOverride` | boolean | Whether a fintech-specific override was applied |

</Tab>

</Tabs>

### Verify the status or outcome of a withdrawal previously requested via `/fintechrequestwithdrawal`
<Info>
The exact request/response payload for verification may include your `withdrawalId` and will return the latest status
</Info>
<Tabs>
<Tab title="Request (Verify)">

```http
POST {{BASE_URL}}/verifywithdrawal
```

</Tab>

<Tab title="Body (Verify)">

```json
{
  "transactionReference": "Withdrawal_ID"
}
```

</Tab>

<Tab title="Response (Verify)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Withdrawal status retrieved",
  "data": {
    "withdrawalId": "Withdrawal_ID",
    "status": "completed",
    "processedAt": "2025-09-04T12:15:00.000Z",
    "failureReason": null
  }
}
```

</Tab>
</Tabs>
