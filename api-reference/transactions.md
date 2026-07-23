---
title: "Transactions API"
sidebarTitle: "Transactions"
description: "Transactions API endpoints for processing swaps, funding, withdrawal and payouts"
---

The Transactions API handles all financial operations including **onramp (funding)**, **offramp (payout)**, **token swaps**, asset transfers, and transaction tracking.

---

## Transactions API - Endpoints Overview

| Section                | Endpoint                       | Method | Description                                                       |
| ---------------------- | ------------------------------ | ------ | ----------------------------------------------------------------- |
| [**User Onramp**](#user-onramp-user-wallet-funding)            | `/cngnonramp`               | POST   | Fund a user wallet by converting fiat to cNGN or supported tokens |
| [**User Offramp**](#user-offramp-payout-to-user-bank-account)           | `/cngnofframp`              | POST   | Returns cNGN in user's wallet to payout fiat to user bank account |
| [**Fintech Onramp**](#fintech-onramp)         | `/getfintechvirtualaccount` | GET    | Returns the bank account attached to a fintech                    |
| [**Fintech Offramp**](#fintech-offramping)        | `/initiateofframp`          | POST   | Returns cNGN in fintech's wallet to payout fiat                   |
| [**User Token Withdrawal**](#user-token-withdrawal)       | `/withdrawasset`            | POST   | Multi-asset support. Withdraw assets from user wallet to external wallet                                |
| [**Fintech Token Withdrawal**](#fintech-token-withdrawal)       | `/fintechtransfer`            | POST   | Multi-asset support. Withdraw assets from fintech smart wallet to external wallet                                |
| [**cNGN Onramp Status**](#cngn-onramp-status)     | `/cngnonrampstatus`         | POST   | Check funding request status                                      |
| [**cNGN Offramp Status**](#cngn-offramp-status)    | `/cngnofframpstatus`        | POST   | Check offramp transaction status                                  |
| [**Fintech Offramp Status**](#fintech-offramp-status) | `/getofframpstatus`         | GET    | Check fintech offramp transaction status                          |
| [**List Deposits**](#list-deposits) | `/deposits`         | GET    | Retrieve deposit transactions with filters                          |
| [**List Payouts**](#list-payouts) | `/payouts`         | GET    | Retrieve payout transactions with filters                          |
| [**List Transactions**](#list-transactions) | `/transactions`         | GET    | Retrieve all transactions (deposits & payouts) with filters                          |
| [**Swap Trigger**](#swap-trigger)          | `/swaptrigger`        | POST   | Trigger an asynchronous token swap (between cNGN and USDC/USDT)             |
| [**User Swap**](#user-swap)                | `/swap`               | POST   | Swap tokens on user's default smart wallet (USDC -> cNGN)          |
| [**Swap Status**](#swap-status)            | `/swapstatus`         | GET    | Get the status of an asynchronous swap request                    |

---

### User Onramp (user wallet funding)

<Info>
Use the [`/getvirtualaccount`](/api-reference/virtual-accounts#retrieve-a-specific-virtual-account-with-details-about-amounts-and-fees) endpoint to get the account to make payment. Bank account is only active for 30mins after creation.
</Info>

<Warning>
Payment into the generated account must be from the bank account of the verified Fintech user. The generated wallet is a [Smart Wallet](/smart-wallet) which can be controlled/owned by your external EOA wallet. Also, kindly note that OWNER parameter is not compulsory. 
</Warning>

<Tabs>

<Tab title="Request (Onramp)">

```http
POST {{BASE_URL}}/cngnonramp
```

</Tab>

<Tab title="Body (Onramp)">

```json
{
  "owner": "",
  "amount": 500,
  "assetSwap": "USDC",
  "autoSwap": true,
  "userId": "user_uniqueID",
  "sweepToOfframp": true,
  "destinationAssetSwap": "0x..."
}
```
### Parameters

| Parameter          | Type    | Required | Description |
|-------------------|---------|----------|-------------|
| `owner`            | string  | No    | Your EOA wallet address to control the generated [Smart Wallet](/smart-wallet). When set, you can [withdraw tokens directly](/smart-wallet#withdrawing-tokens-from-your-smart-wallet) using your EOA. **Note:** StRails cannot withdraw funds from wallets with a custom owner. |
| `amount`           | number  | Yes   | Amount to fund in Naira (e.g., `500` = ₦500) |
| `assetSwap`        | string  | No    | Asset to swap into after funding (e.g. `USDC`) |
| `autoSwap`         | boolean | No    | Automatically swap funded amount to `assetSwap` |
| `userId`           | string  | Yes   | Unique identifier (hash/UUID) of the user |
| `sweepToOfframp`   | boolean | No    | Automatically route funds to the user's default wallet after funding |
| `destinationAssetSwap` | string | No | Destination wallet address for swapped assets. Only used when `autoSwap=true` and `sweepToOfframp=false`. |

### Behavior Matrix

| `autoSwap` | `sweepToOfframp` | `destinationAssetSwap` | Result |
|------------|------------------|------------------------|--------|
| `false`    | `false`          | -                      | cNGN stays in generated wallet |
| `false`    | `true`           | ignored                | cNGN -> user's default wallet |
| `true`     | `false`          | not set                | cNGN->USDC, stays in generated wallet |
| `true`     | `false`          | set                    | cNGN->USDC -> `destinationAssetSwap` |
| `true`     | `true`           | ignored                | cNGN->USDC -> user's default wallet |

> **Note:** When `sweepToOfframp=true`, the destination is automatically resolved to the user's default smart wallet (`verifiedUsers.smartWalletAddress`). You don't need to set `destinationAssetSwap`.

</Tab>

<Tab title="Response (Onramp)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "cNGN request submitted successfully",
  "data": {
    "requestId": "uuid",
    "walletAddress": "0x..",
    "status": "requested",
    "version": "1.0.0",
    "message": "Ensure you pay 528.84 naira (520 + 8.84 total fees: 0.00 fintech + 8.84 strails) to the bank account generated. Wallet created successfully and Bank account generation in progress. Bank account is only active for 30mins after creation. Use requestId to get the bank account details for payment.",
    "autoSwapEnabled": false,
    "sweepToOfframpEnabled": false,
    "targetAsset": "USDC",
    "feeBreakdown": {
      "baseAmount": 520,
      "fintechFee": 0,
      "strailsFee": 8.84,
      "totalFee": 8.84,
      "totalAmount": 528.84
    }
  }
}
```

</Tab>

<Tab title="Error (Onramp)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Amount must be greater than 0"
}
```

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

### User Offramp (payout to user bank account)

<Tabs>

<Tab title="Request (Offramp)">

```http
POST {{BASE_URL}}/cngnofframp
```

</Tab>

<Tab title="Body (Offramp)">

```json
{
  "userId": "user_uniqueID",
  "amount": 5000,
  "accountNumber": "0123456789",
  "bankCode": "058",
  "ticker": "CNGN"
}
```
### Parameters

| Parameter        | Type   | Required | Description |
|------------------|--------|----------|-------------|
| `userId`         | string | Yes   | Unique identifier (hash/UUID) of the user initiating the offramp |
| `amount`         | number | Yes   | Amount to convert and transfer to bank in Naira (e.g., `5000` = ₦5,000) |
| `accountNumber`  | string | Yes   | Destination bank account number |
| `bankCode`       | string | Yes   | Bank identifier code (e.g. NIBSS bank code such as `058`) |
| `ticker`         | string | Yes   | Asset symbol being off-ramped (e.g. `CNGN`) |


</Tab>

<Tab title="Response (Offramp)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Vault return token transfer completed, payout processing initiated",
  "data": {
    "requestId": "uuid",
    "status":"payout_pending",
    "stage":"payout",
    "tokenAddress":"0x...",
    "vaultAddress":"0x..."
  }
}
```

</Tab>

<Tab title="Error (Offramp)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Invalid bank code format"
}
```

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
  "response_code": "05",
  "message": "Insufficient balance",
  "error": "User wallet has insufficient cNGN balance for this offramp"
}
```

</Tab>

</Tabs>

---

### Fintech Onramp

<Info>
Use the [`/getfintechwallet`](/api-reference/wallet#retrieve-the-fintech-smart-wallet-and-all-external-wallets) endpoint to get the fintech default wallet where cNGN is sent to after payment is confirmed.
</Info>

<Tabs>

<Tab title="Request (Fintech Onramp)">

```http
GET {{BASE_URL}}/getfintechvirtualaccount
```

</Tab>

<Tab title="Response (Fintech Onramp)">

```json
{
  "status":"Success",
  "response_code":"00",
  "message":"Virtual account retrieved successfully",
  "data":{
    "virtualAccount":{
      "accountNumber":"9800000094",
      "accountName":"Example Fintech",
      "bankName":"FAME Bank",
      "provider":"Sample Provider",
      "status":"active",
      "createdAt":"2026-01-28T13:06:38.867Z"
      }
    }
}
```

</Tab>

</Tabs>

---

### Fintech Offramping

<Tabs>

<Tab title="Request (Wallet->Bank)">

```http
POST {{BASE_URL}}/initiateofframp
```

</Tab>


<Tab title="Body (Wallet->Bank)">

```json
{
  "amount": 1000000,
  "bankAccountId": "uuid"
}
```

### Parameters

| Parameter          | Type   | Required | Description |
|-------------------|--------|----------|-------------|
| `amount`           | number | Yes   | Amount to transfer from wallet to bank (in lowest currency unit) |
| `bankAccountId`    | string | Yes   | Unique identifier (UUID) of the beneficiary bank account |

</Tab>

<Tab title="Response (Wallet->Bank)">

```json
{
    "status": "Success",
    "response_code": "00",
    "message": "Offramp initiated successfully",
    "data": {
        "requestId": "uuid",
        "amount": 170,
        "walletSource": "system_wallet",
        "status": "pending",
        "message": "Your offramp request is being processed. cNGN will be transferred from your smart wallet. You will receive a webhook notification when completed."
    }
}
```

</Tab>

<Tab title="Error (Wallet->Bank)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Amount must be greater than 0"
}
```

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "Bank account not found",
  "error": "No bank account found with the provided bankAccountId"
}
```

```json
{
  "status": "Error",
  "response_code": "05",
  "message": "Insufficient balance",
  "error": "Smart wallet has insufficient cNGN balance"
}
```

</Tab>

</Tabs>

---

### User Token Withdrawal

<Tabs>

<Tab title="Request (Withdrawal)">

```http
POST {{BASE_URL}}/withdrawasset
```

</Tab>

<Tab title="Body (Withdrawal)">

```json
{
  "userId": "user_uniqueID",
  "internalWallet": "0xInternal",
  "destinationWallet": "0xExternal",
  "amount": 100,
  "ticker": "CNGN",
  "network": "base"
}
```
### Parameters

| Parameter            | Type   | Required | Description |
|---------------------|--------|----------|-------------|
| `userId`            | string | Yes   | Unique identifier (hash/UUID) of the user initiating the withdrawal |
| `internalWallet`    | string | Yes   | User's internal wallet address on the platform (source wallet) |
| `destinationWallet` | string | Yes   | External wallet or bank-linked wallet address to receive funds |
| `amount`            | number | Yes   | Amount to withdraw (must be greater than 0) |
| `ticker`            | string | Yes   | Asset symbol to withdraw (e.g. `CNGN`) |
| `network`           | string | No    | When set, uses a bridge to withdraw the token via the specified network (e.g., `base`, `bsc`, `sol`, `eth`, `xbn`, `asc`, `arc`, `lisk`). Defaults to Base if left empty |

</Tab>


<Tab title="Response (Withdrawal)">

```json
{
  "status": "Success",
    "response_code": "00",
    "message": "Withdrawal executed successfully",
    "data": {
        "transactionHash": "tranx_hash",
        "smartWalletAddress": "0xInternal",
        "requestId": "req_uuid_12345",
        "executedAt": "2026-06-11T08:47:59.653Z",
        "gasUsed": "85397",
        "effectiveGasPrice": "1010000000",
        "blockNumber": "47189167",
        "duration": 8641,
        "userId": "user_uniqueID",
        "amount": "200",
        "destinationWallet": "0xExternal",
        "tokenAddress": "0x46C85152bFe9f96829aA94755D9f915F9B10EF5F",
        "tokenSymbol": "CNGN",
        "network": "base",
        "networkDisplayName": "Base",
        "estimatedConfirmationTime": "2-5 minutes",
        "warnings": [
            "High gas fees relative to withdrawal amount"
        ],
        "feeBreakdown": {
            "requestedAmount": "200",
            "totalFee": "5.965",
            "netAmount": "194.035",
            "usedFintechOverride": false
        }
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
  "error": "Amount must be greater than 0"
}
```

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
  "response_code": "05",
  "message": "Insufficient balance",
  "error": "Wallet has insufficient balance for this withdrawal"
}
```

</Tab>

</Tabs>

---

### Fintech Token Withdrawal

<Info>
Ensure you whitelist the address you are about to transfer to before calling /fintechtransfer endpoint. Use the [`/addexternalwallet`](/api-reference/wallet#add-an-external-wallet) endpoint to whitelist an address.
</Info>

<Tabs>

<Tab title="Request (Fintech Withdrawal)">

```http
POST {{BASE_URL}}/fintechtransfer
```

</Tab>

<Tab title="Body (Fintech Withdrawal)">

```json
{
  "destinationAddress": "0xExternal",
  "amount": 5000000,
  "ticker": "CNGN",
  "note": "Withdrawal to cold storage"
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `destinationAddress` | string | Yes | Ethereum address (must be a registered external wallet) |
| `amount` | number | Yes | Amount in smallest unit (e.g., 5000000 = 5.0 cNGN) |
| `ticker` | string | No | Token symbol: CNGN (default), USDC, USDT |
| `note` | string | No | Optional note (max 500 characters) |

</Tab>

<Tab title="Response (Fintech Withdrawal)">

```json
{
  "code": "00",
  "message": "Transfer completed successfully",
  "data": {
    "requestId": "req_uuid_12345",
    "transactionHash": "0xabc...def",
    "smartWalletAddress": "0x123...456",
    "destinationAddress": "0x742...c87",
    "destinationLabel": "My Cold Wallet",
    "amount": "5000000",
    "ticker": "CNGN",
    "blockNumber": "12345678",
    "gasUsed": "150000",
    "commitmentsSafeguarded": {
      "activeOrders": 3,
      "activeEscrows": 1,
      "totalCommitted": "2000000",
      "remainingBalance": "3000000"
    },
    "executedAt": "2026-03-21T10:30:00Z",
    "version": "1.0.0"
  }
}
```

</Tab>

<Tab title="Error (Fintech Withdrawal)">

```json
{
  "code": "01",
  "message": "Validation error",
  "error": "Destination address is not a registered external wallet"
}
```

```json
{
  "code": "05",
  "message": "Insufficient balance",
  "error": "Smart wallet has insufficient balance after accounting for active orders and escrows"
}
```

</Tab>

</Tabs>

---

### cNGN OnRamp Status

<Tabs>

<Tab title="Request (OnRamp Status)">

```http
POST {{BASE_URL}}/cngnonrampstatus
```

</Tab>

<Tab title="Body (OnRamp Status)">

```json
{
  "walletAddress": "0x.."
}
```

</Tab>

<Tab title="Response (OnRamp Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Wallet status retrieved successfully",
  "data": {
    "status": "funded",
    "requestId": "uuid",
    "version": "1.0.0",
    "wallet": {
      "walletAddress": "0x.....",
      "owner": "0x.....",
      "tokenBuy": "USDC",
      "autoSwap": false,
      "sweepToOfframp": true,
      "amount": "200",
      "createdAt": "2026-01-28T11:17:47.513Z",
      "fundingRequestedAt": "2026-01-28T11:21:20.170Z",
      "fundedAt": "2026-01-28T11:22:01.527Z",
      "transactionHash": "0x......",
      "virtualAccountDetails": {
        "accountNumber": "xxxx",
        "bankName": "XXXX Bank",
        "accountName": "Okaformbah",
        "amount": 200,
        "createdAt": "2026-01-28T11:18:09.353Z"
      },
      "virtualAccountStatus": "created"
    },
    "fxQuote": {
      "available": true,
      "pair": "CNGN-USDC",
      "cngnAmount": "1000",
      "tokenAmount": "0.689655",
      "averagePrice": "1450.00",
      "averageSpread": 0.5,
      "expiresAt": "2026-04-04T08:58:36.072Z",
      "note": "This quote is valid for 5 minutes"
    }
  }
}
```

</Tab>

</Tabs>

---

### cNGN OffRamp Status

<Tabs>

<Tab title="Request (OffRamp Status)">

```http
POST {{BASE_URL}}/cngnofframpstatus
```

</Tab>


<Tab title="Body (OffRamp Status)">

```json
{
  "requestId": "uuid"
}
```

</Tab>

<Tab title="Response (OffRamp Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Vault return status retrieved successfully",
  "data": {
    "requestId": "vr_1769599852415_oosd",
    "status": "completed",
    "stage": "completed",
    "createdAt": "2026-01-28T12:30:54.921Z",
    "updatedAt": "2026-01-28T12:31:10.690Z",
    "tokenAddress": "0x....",
    "amount": "200",
    "tokenTransfer": {
      "status": "confirmed",
      "transactionHash": "0x....",
      "blockNumber": "41405258",
      "timestamp": "2026-01-28T12:31:02.112Z"
    },
    "payout": {
      "status": "completed",
      "reference": "0x....",
      "transactionId": "0x....",
      "timestamp": "2026-01-28T12:31:10.690Z",
      "attempts": 1
    }
  }
}
```

</Tab>

</Tabs>

---

### Fintech Offramp Status

<Tabs>

<Tab title="Request (Fintech Offramp Status)">

```http
GET {{BASE_URL}}/getofframpstatus?requestId=uuid
```

### Query Parameters

| Parameter   | Type   | Required | Description                                      |
| ----------- | ------ | -------- | ------------------------------------------------ |
| `requestId` | string | Yes   | Unique identifier (UUID) of the offramp request |

</Tab>

<Tab title="Response (Fintech Offramp Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Offramp status retrieved successfully",
  "data": {
    "requestId": "uuid",
    "status": "completed",
    "amount": 17000,
    "currency": "NGN",
    "bankAccount": {
      "accountNumber": "123456789",
      "accountName": "Bidemi Chioma Hauwa",
      "bankName": "UNITED BANK"
    },
    "wallet": {
      "source": "system_wallet",
      "address": "0x...."
    },
    "transferDetails": {
      "txHash": "Ox-transactionHash",
      "status": "confirmed"
    },
    "payoutDetails": {
      "reference": "uuid",
      "provider": "novac",
      "status": "completed"
    },
    "initiatedAt": "2026-01-29T03:56:20.198Z",
    "completedAt": "2026-01-29T03:57:07.012Z"
  }
}
```

</Tab>

</Tabs>

---

### List Deposits

Retrieve deposit transactions with optional filters for type, status, date range, and pagination.

<Tabs>

<Tab title="Request (List Deposits)">

```http
GET {{BASE_URL}}/deposits?type=fintech_deposit&offset=0
```

### Query Parameters

| Filter      | Type   | Values |
|-------------|--------|--------|
| `type`      | string | `fintech_deposit`, `user_onramp` |
| `userId`    | string | User hash (for `user_onramp` only) |
| `status`    | string | `pending`, `processing`, `completed`, `failed`, `cancelled` |
| `startDate` | ISO date | e.g., `2026-01-01T00:00:00Z` |
| `endDate`   | ISO date | e.g., `2026-01-31T23:59:59Z` |
| `limit`     | number | Max results (default: 20, max: 100) |
| `offset`    | number | Pagination offset |

</Tab>

<Tab title="Response (List Deposits)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Deposits retrieved successfully",
  "data": {
    "deposits": [
      {
        "id": "6653603d-74fd-4a1f-80a1-d9b84bc0173a",
        "type": "fintech_deposit",
        "direction": "in",
        "fintechId": "c8fe1911-d292-4f84-aa88-e34ceeea5b65",
        "amount": 500,
        "currency": "NGN",
        "virtualAccountNumber": "9800000094",
        "virtualAccountName": "Example Fintech",
        "depositor": {
          "name": "JOHN DOE",
          "accountNumber": "1234567890",
          "bankName": "SAMPLE BANK",
          "bankCode": "000001"
        },
        "transactionReference": "000015260129043400000000099569",
        "provider": "novac",
        "status": "completed",
        "walletAddress": "0x3af904Ff6FA747A3da92E7ce21d312de4b2Ef967",
        "mintTxHash": "0x59a8094c6dce96396d50e67083eba7d7d4ac7cd4393e976c7e73e46a67164077",
        "createdAt": "2026-01-29T03:34:35.582Z",
        "completedAt": "2026-01-29T03:35:12.198Z"
      }
    ],
    "total": 1,
    "limit": 20,
    "offset": 0,
    "hasMore": false,
    "filters": {
      "type": "fintech_deposit"
    }
  }
}
```

</Tab>

</Tabs>

---

### List Payouts

Retrieve payout transactions with optional filters for type, status, date range, and pagination.

<Tabs>

<Tab title="Request (List Payouts)">

```http
GET {{BASE_URL}}/payouts?type=fintech_offramp&status=completed&limit=10
```

### Query Parameters

| Filter      | Type   | Values |
|-------------|--------|--------|
| `type`      | string | `fintech_offramp`, `user_offramp` |
| `userId`    | string | User hash (for `user_offramp` only) |
| `status`    | string | `pending`, `processing`, `completed`, `failed`, `cancelled` |
| `startDate` | ISO date | e.g., `2026-01-01T00:00:00Z` |
| `endDate`   | ISO date | e.g., `2026-01-31T23:59:59Z` |
| `limit`     | number | Max results (default: 20, max: 100) |
| `offset`    | number | Pagination offset |

</Tab>

<Tab title="Response (List Payouts)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Payouts retrieved successfully",
  "data": {
    "payouts": [
      {
        "id": "6653603d-74fd-4a1f-80a1-d9b84bc0173a",
        "type": "fintech_offramp",
        "direction": "out",
        "fintechId": "c8fe1911-d292-4f84-aa88-e34ceeea5b65",
        "amount": 5000,
        "currency": "NGN",
        "sourceWalletAddress": "0x3af904Ff6FA747A3da92E7ce21d312de4b2Ef967",
        "sourceNetwork": "base",
        "bankAccount": {
          "accountNumber": "0123456789",
          "accountName": "JANE DOE",
          "bankName": "UNITED BANK FOR AFRICA",
          "bankCode": "000004"
        },
        "status": "completed",
        "transferTxHash": "0x59a8094c6dce96396d50e67083eba7d7d4ac7cd4393e976c7e73e46a67164077",
        "payoutReference": "6653603d-74fd-4a1f-80a1-d9b84bc0173a",
        "payoutProvider": "novac",
        "payoutStatus": "completed",
        "processingFee": 75.00,
        "totalDeducted": 5075.00,
        "createdAt": "2026-01-29T03:56:20.198Z",
        "initiatedAt": "2026-01-29T03:56:20.198Z",
        "completedAt": "2026-01-29T03:57:07.012Z",
        "updatedAt": "2026-01-29T03:57:07.012Z"
      }
    ],
    "total": 1,
    "limit": 10,
    "offset": 0,
    "hasMore": false,
    "filters": {
      "type": "fintech_offramp",
      "status": "completed"
    }
  }
}
```

</Tab>

</Tabs>

---

### List Transactions

Retrieve all transactions (both deposits and payouts) with optional filters for direction, type, status, date range, and pagination.

<Tabs>

<Tab title="Request (List Transactions)">

```http
GET {{BASE_URL}}/transactions?direction=in&status=completed&limit=50
```

### Query Parameters

| Filter      | Type   | Values |
|-------------|--------|--------|
| `direction` | string | `in` (deposits), `out` (payouts) |
| `type`      | string | `fintech_deposit`, `user_onramp`, `fintech_offramp`, `user_offramp` |
| `userId`    | string | User hash |
| `status`    | string | `pending`, `processing`, `completed`, `failed`, `cancelled` |
| `startDate` | ISO date | e.g., `2026-01-01T00:00:00Z` |
| `endDate`   | ISO date | e.g., `2026-01-31T23:59:59Z` |
| `limit`     | number | Max results (default: 20, max: 100) |
| `offset`    | number | Pagination offset |

</Tab>

<Tab title="Response (List Transactions)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Transactions retrieved successfully",
  "data": {
    "transactions": [
      {
        "id": "000015260129043400000000099569",
        "type": "fintech_deposit",
        "direction": "in",
        "fintechId": "c8fe1911-d292-4f84-aa88-e34ceeea5b65",
        "amount": 500,
        "currency": "NGN",
        "virtualAccountNumber": "9800000094",
        "virtualAccountName": "Example Fintech",
        "depositor": {
          "name": "JOHN DOE",
          "accountNumber": "1234567890",
          "bankName": "SAMPLE BANK",
          "bankCode": "000001"
        },
        "transactionReference": "000015260129043400000000099569",
        "provider": "novac",
        "status": "completed",
        "walletAddress": "0x3af904Ff6FA747A3da92E7ce21d312de4b2Ef967",
        "createdAt": "2026-01-29T03:34:35.582Z",
        "completedAt": "2026-01-29T03:35:12.198Z"
      },
      {
        "id": "000015260128161759000004122348",
        "type": "fintech_deposit",
        "direction": "in",
        "fintechId": "c8fe1911-d292-4f84-aa88-e34ceeea5b65",
        "amount": 1000,
        "currency": "NGN",
        "virtualAccountNumber": "9800000094",
        "virtualAccountName": "Example Fintech",
        "depositor": {
          "name": "JOHN DOE",
          "accountNumber": "1234567890",
          "bankName": "SAMPLE BANK",
          "bankCode": "000001"
        },
        "transactionReference": "000015260128161759000004122348",
        "provider": "novac",
        "status": "completed",
        "walletAddress": "0x3af904Ff6FA747A3da92E7ce21d312de4b2Ef967",
        "createdAt": "2026-01-28T15:18:20.327Z",
        "completedAt": "2026-01-28T15:19:05.198Z"
      }
    ],
    "total": 2,
    "limit": 50,
    "offset": 0,
    "hasMore": false,
    "summary": {
      "totalDeposits": 2,
      "totalPayouts": 0,
      "depositVolume": "1500.00",
      "payoutVolume": "0.00"
    },
    "filters": {
      "direction": "in",
      "status": "completed"
    }
  }
}
```

</Tab>

</Tabs>

---

### Swap Trigger

Trigger an asynchronous token swap between cNGN and USDC/USDT. The swap is queued and processed in the background.

<Tabs>

<Tab title="Request (Swap Trigger)">

```http
POST {{BASE_URL}}/swaptrigger
```

</Tab>

<Tab title="Body (Swap Trigger)">

```json
{
  "walletAddress": "0x...",
  "sellToken": "CNGN",
  "buyToken": "USDC",
  "amount": 500,
  "slippage": 2
}
```

### Parameters

| Parameter       | Type   | Required | Description                                                          |
|-----------------|--------|----------|----------------------------------------------------------------------|
| `walletAddress` | string | Yes      | Wallet address generated during `/cngnonramp` request                |
| `sellToken`     | string | Yes      | Token to sell (`CNGN`, `USDC`, or `USDT`)                            |
| `buyToken`      | string | No       | Token to buy (`CNGN`, `USDC`, or `USDT`). Falls back to wallet's `tokenBuy` if not provided |
| `amount`        | number | No       | Fiat amount of `sellToken` to swap (human-readable format). Falls back to funded amount if not provided |
| `slippage`      | number | No       | Maximum slippage percentage (default: 5%)                            |

</Tab>

<Tab title="Response (Swap Trigger)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Swap trigger submitted successfully",
  "data": {
    "requestId": "req_a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "status": "queued",
    "message": "Your swap request has been queued and will be processed asynchronously",
    "swapConfig": {
      "sellToken": "CNGN",
      "buyToken": "USDT",
      "amount": "500.000000",
      "walletAddress": "0x..."
    },
    "destinationConfig": {
      "destinationWallet": "fintech_smart_wallet",
      "sweepToOfframp": false
    }
  }
}
```

</Tab>

<Tab title="Error (Swap Trigger)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Invalid wallet address format"
}
```

```json
{
  "status": "Error",
  "response_code": "05",
  "message": "Insufficient liquidity",
  "error": "Not enough liquidity in pool for swap"
}
```

</Tab>

</Tabs>

---

### User Swap

Request a token swap on the user's default smart wallet address. Primarily used for USDC -> cNGN conversions.

<Tabs>

<Tab title="Request (User Swap)">

```http
POST {{BASE_URL}}/swap
```

</Tab>

<Tab title="Body (User Swap)">

```json
{
  "sellToken": "USDC",
  "buyToken": "CNGN",
  "amount": 100.50,
  "slippage": 2,
  "userId": "user_hash",
  "smartWalletAddress": "0x...",
  "destinationAssetSwap": "0x..."
}
```

### Parameters

| Parameter              | Type   | Required | Description                                                          |
|------------------------|--------|----------|----------------------------------------------------------------------|
| `sellToken`            | string | Yes      | Token to sell (`USDC`, `USDT`, or `CNGN`)                            |
| `buyToken`             | string | Yes      | Token to buy (`USDC`, `USDT`, or `CNGN`)                             |
| `amount`               | number | Yes      | Human-readable amount to swap (e.g., `100.50` = 100.50 USDC)         |
| `slippage`             | number | No       | Maximum slippage percentage (default: 2%)                            |
| `userId`               | string | Yes      | User identifier (verified user hash)                                 |
| `smartWalletAddress`   | string | No       | User's smart wallet address. If not provided, uses user's default smart wallet |
| `destinationAssetSwap` | string | No       | Alternative destination wallet for output tokens. If not provided, tokens go to user's default smart wallet |

</Tab>

<Tab title="Response (User Swap)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Swap request submitted successfully",
  "data": {
    "requestId": "swap-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "status": "requested",
    "message": "Your swap request has been submitted and will be processed asynchronously",
    "smartWalletExecutionDetails": {
      "smartWalletAddress": "0x...",
      "executionMethod": "smart_wallet_execute",
      "validationSummary": {
        "balanceCheck": "passed",
        "allowanceCheck": "passed"
      }
    }
  }
}
```

</Tab>

<Tab title="Error (User Swap)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Amount must be greater than 0"
}
```

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
  "response_code": "05",
  "message": "Insufficient balance",
  "error": "Wallet has insufficient balance for this swap"
}
```

</Tab>

</Tabs>

---

### Swap Status

Get the status of an asynchronous swap request. Use the `requestId` returned from `/swaptrigger` or `/swap` to track progress.

<Tabs>

<Tab title="Request (Swap Status)">

```http
GET {{BASE_URL}}/swapstatus?requestId=swap-xxx
```

### Query Parameters

| Parameter   | Type   | Required | Description                                      |
|-------------|--------|----------|--------------------------------------------------|
| `requestId` | string | Yes      | Request ID from `/swaptrigger` response          |

</Tab>

<Tab title="Response (Swap Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Operation status retrieved successfully",
  "data": {
    "requestId": "swap-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "type": "swap_trigger",
    "status": "completed",
    "createdAt": "2026-05-26T10:30:00.000Z",
    "updatedAt": "2026-05-26T10:30:45.000Z",
    "txHash": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "amountOut": "775000000",
    "executionRate": "1550.25",
    "executionMethod": "smart_wallet_execute",
    "smartWalletAddress": "0x625A4612e3aAa7ce62C43EdDf5C81d7443418E4f"
  }
}
```

</Tab>

<Tab title="Error (Swap Status)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "Not found",
  "error": "No swap request found with the provided requestId"
}
```

</Tab>

</Tabs>