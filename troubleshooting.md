---
title: "API Status & Error Reference"
sidebarTitle: "Troubleshooting"
description: "Complete reference for all API response codes, status values, error codes, and state transitions"
---

Complete reference for all statuses, response codes, and state transitions across Strails API endpoints.

---

## Response Codes

All API responses include a `response_code` field indicating the result:

| Code | Status | HTTP Code | Description |
|------|--------|-----------|-------------|
| `00` | SUCCESS | 200 | Operation completed successfully |
| `01` | VALIDATION_ERROR | 400 | Invalid request parameters |
| `02` | NOT_FOUND | 404 | Resource not found |
| `03` | UNAUTHORIZED | 401/403 | Authentication failed or access denied |
| `04` | RATE_LIMITED | 429 | Too many requests |
| `05` | INTERNAL_ERROR | 500 | Server error |

### Response Structure

**Success Response:**
```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Operation completed successfully",
  "data": { }
}
```

**Error Response:**
```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Validation error: Invalid amount",
  "error": { "field": "amount", "reason": "Must be greater than 0" }
}
```

---

## Transaction Statuses

### General Transaction Status

| Status | Description |
|--------|-------------|
| `pending` | Transaction initiated, awaiting processing |
| `processing` | Transaction is being processed |
| `completed` | Transaction completed successfully |
| `failed` | Transaction failed |
| `confirmed` | Payment confirmed (used for payments) |
| `cancelled` | Transaction was cancelled |

### Transaction Status Lifecycle

```
pending -> processing -> completed
                    \-> failed
```

---

## User Onramp Statuses (cngnonramp / getvirtualaccount)

### Wallet/Onramp Status Lifecycle

| Status | Description |
|--------|-------------|
| `requested` | Wallet creation requested, awaiting virtual account |
| `virtual_account_created` | Virtual account created |
| `pending` | Virtual account created, awaiting payment |
| `processing` | Payment received, processing cNGN minting |
| `funded` | Wallet funded with cNGN successfully |
| `swap_queued` | cNGN funded, auto-swap to USDC/USDT queued |
| `swap_processing` | Swap is being executed |
| `completed` | Full flow complete (funding + optional swap) |
| `failed` | Funding or swap failed |

### Onramp Status Flow

```
requested -> pending -> processing -> funded -> completed
                            v           v
                          failed      failed
                                        v
                          (if autoSwap) -> swap_queued -> swap_processing -> completed
                                                                 v
                                                              failed
```

### Virtual Account Status

| Status | Description |
|--------|-------------|
| `active` | Virtual account is active and ready to receive payments |
| `expired` | Virtual account has expired (30 min timeout) |
| `used` | Virtual account has been used for a payment |

---

## User Offramp Statuses (cngnofframp)

### Vault Return Status Lifecycle

| Status | Description |
|--------|-------------|
| `pending` | Offramp request initiated |
| `processing` | Processing the offramp request |
| `bank_verification` | Verifying bank account details |
| `transfer_pending` | cNGN transfer to burn address pending |
| `transfer_confirmed` | cNGN transfer confirmed on-chain |
| `payout_pending` | Bank payout pending |
| `completed` | Funds sent to user's bank account |
| `failed` | Offramp failed |
| `cancelled` | Offramp cancelled |

### User Offramp Status Flow

```
pending -> processing -> bank_verification -> transfer_pending -> transfer_confirmed -> payout_pending -> completed
    v          v               v                    v                   v                   v
 cancelled   failed          failed               failed              failed              failed
```

---

## Fintech Offramp Statuses (initiateofframp)

### Fintech Offramp Status Lifecycle

| Status | Description |
|--------|-------------|
| `pending` | Offramp request initiated, awaiting processing |
| `processing` | Request is being processed |
| `transferring_cngn` | cNGN is being transferred to burn address |
| `initiating_payout` | On-chain transfer confirmed, initiating bank payout |
| `payout_processing` | Bank payout is being processed |
| `completed` | Funds successfully sent to bank account |
| `failed` | Offramp failed (burn failed or payout failed) |
| `cancelled` | Offramp was cancelled before completion |
| `reversed` | Offramp was reversed after completion |
| `pending_manual_review` | Requires manual review before proceeding |

### Offramp Status Flow

```
pending -> processing -> transferring_cngn -> initiating_payout -> payout_processing -> completed
    v          v               v                     v                   v
 cancelled   failed          failed                failed              failed
                                                                         v
                                                                      reversed
```

### Offramp Sub-Statuses

#### Transfer Transaction Status (`transferTxStatus`)

| Status | Description |
|--------|-------------|
| `pending` | Blockchain transaction submitted |
| `confirmed` | Transaction confirmed on-chain |
| `failed` | Transaction failed |

#### Payout Status (`payoutStatus`)

| Status | Description |
|--------|-------------|
| `pending` | Payout request created |
| `processing` | Payout being processed by payment gateway |
| `completed` | Funds delivered to bank account |
| `failed` | Payout failed |

---

## FX Orderbook Statuses

### Order Status

| Status | Description |
|--------|-------------|
| `active` | Order is live and can be matched |
| `paused` | Order is temporarily paused |
| `deleted` | Order has been permanently removed |

### Order Status Transitions

```
active <-> paused
   v          v
 deleted    deleted
```

- `active` -> `paused` (via update)
- `paused` -> `active` (via update)
- `active` -> `deleted` (via delete)
- `paused` -> `deleted` (via delete)

---

## FX Trade Statuses

### Trade Status Lifecycle

| Status | Description |
|--------|-------------|
| `pending` | Trade created, validating parameters and liquidity |
| `locked` | Funds locked in escrow, 5-minute countdown active |
| `signing` | Awaiting MPC signature for transaction |
| `settling` | Blockchain transaction execution in progress |
| `completed` | Trade executed successfully |
| `expired` | Lock timeout - trade not completed within 5 minutes |
| `failed` | Settlement or execution failed |

### Trade Status Flow

```
pending -> locked -> signing -> settling -> completed
                        v           v
         locked -> expired       failed
         signing -> expired (only after MPC signing completes/fails)
```

---

## Fintech Deposit Statuses

### Fintech Virtual Account Deposit Status

| Status | Description |
|--------|-------------|
| `pending` | Deposit received, awaiting processing |
| `processing` | Deposit is being processed |
| `completed` | Deposit completed and funds credited |
| `failed` | Deposit processing failed |

### Fintech User Deposit Status (with refund support)

| Status | Description |
|--------|-------------|
| `pending` | Deposit received, awaiting BVN verification |
| `processing` | Processing deposit after verification |
| `completed` | Deposit completed, cNGN sent to user wallet |
| `failed` | Deposit processing failed |
| `refund_initiated` | Refund initiated (BVN mismatch) |
| `refund_processing` | Refund being processed |
| `refunded` | Deposit refunded to sender |
| `refund_failed` | Refund failed |

### Deposit Funding Status (`fundingStatus`)

| Status | Description |
|--------|-------------|
| `pending` | Awaiting cNGN minting |
| `requested` | Minting request sent to MPC vault |
| `completed` | cNGN successfully minted to wallet |
| `failed` | Minting failed |

---

## Fintech Transfer Status (fintechtransfer)

Fintech transfers are synchronous and complete immediately. The response indicates success or failure directly.

| Result | Description |
|--------|-------------|
| `success` | Transfer completed successfully in the response |
| `error` | Transfer failed with error details in response |

---

## Asset Withdrawal Statuses (withdrawasset)

### Withdrawal Status Lifecycle

| Status | Description |
|--------|-------------|
| `pending` | Withdrawal initiated, transaction submitted to blockchain |
| `confirmed` | Withdrawal confirmed on blockchain (met required confirmations) |
| `failed` | Withdrawal failed (insufficient balance, gas issues, or blockchain error) |

### Withdrawal Status Flow

```
pending -> confirmed
    \-> failed
```

---

## Swap Statuses

| Status | Description |
|--------|-------------|
| `pending` | Swap queued for processing |
| `requested` | Swap request submitted |
| `queued` | Swap is queued for execution |
| `processing` | Swap execution in progress |
| `completed` | Swap executed successfully |
| `failed` | Swap failed |

---

## Fee Withdrawal Statuses

| Status | Description |
|--------|-------------|
| `pending` | Withdrawal request submitted |
| `processing` | Withdrawal is being processed |
| `completed` | Withdrawal sent to bank account |
| `failed` | Withdrawal failed |

---

## User Statuses

### User Onboarding Status

| Status | Description |
|--------|-------------|
| `pending` | Onboarding in progress |
| `processing` | BVN verification in progress |
| `completed` | User is fully onboarded |
| `active` | User account is active |
| `inactive` | User account is deactivated |
| `failed` | Onboarding failed |

### BVN Verification Status

| Status | Description |
|--------|-------------|
| `verified` | BVN verification successful |
| `pending` | BVN verification in progress |
| `failed` | BVN verification failed |

---

## Wallet Statuses

### External Wallet Status

| Status | Description |
|--------|-------------|
| `active` | Wallet is whitelisted and can be used for transfers |
| `inactive` | Wallet is disabled |

---

## Virtual Account Statuses

| Status | Description |
|--------|-------------|
| `active` | Virtual account is active and can receive payments |
| `created` | Virtual account has been created |
| `expired` | Virtual account has expired |

---

## Common Error Codes

| Code | Description |
|------|-------------|
| `VALIDATION_ERROR` | Request validation failed |
| `AUTHENTICATION_FAILED` | Invalid or missing API key |
| `INSUFFICIENT_BALANCE` | Wallet has insufficient balance |
| `INSUFFICIENT_LIQUIDITY` | Not enough liquidity in pool |
| `ORDER_NOT_FOUND` | Orderbook order not found |
| `TRADE_NOT_FOUND` | Trade not found |
| `QUOTE_EXPIRED` | FX quote has expired (5 minutes) |
| `TRADE_EXPIRED` | Trade lock expired (5 minutes) |
| `NO_MATCHING_ORDERS` | No orders available to match |
| `SELF_TRADE_NOT_ALLOWED` | Cannot trade against your own orders |
| `PAYOUT_FAILED` | Bank payout failed |
| `BVN_MISMATCH` | BVN name verification failed |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `INVALID_WALLET_ADDRESS` | Wallet address is invalid or not whitelisted |
| `USER_NOT_FOUND` | User not found |
| `WALLET_NOT_FOUND` | Wallet not found |
| `BANK_ACCOUNT_NOT_FOUND` | Bank account not found |

### Swap Error Codes

| Code | Description |
|------|-------------|
| `INSUFFICIENT_LIQUIDITY` | Not enough liquidity in pool for swap |
| `SLIPPAGE_EXCEEDED` | Price moved beyond slippage tolerance |
| `GAS_ESTIMATION_FAILED` | Failed to estimate gas for transaction |

---

## Filter Values Reference

### Deposit Filter Values

| Filter | Values |
|--------|--------|
| `type` | `fintech_deposit`, `user_onramp` |
| `status` | `pending`, `processing`, `completed`, `failed`, `cancelled` |

### Payout Filter Values

| Filter | Values |
|--------|--------|
| `type` | `fintech_offramp`, `user_offramp` |
| `status` | `pending`, `processing`, `completed`, `failed`, `cancelled` |

### Transaction Filter Values

| Filter | Values |
|--------|--------|
| `direction` | `in`, `out` |
| `type` | `fintech_deposit`, `user_onramp`, `fintech_offramp`, `user_offramp` |
| `status` | `pending`, `processing`, `completed`, `failed`, `cancelled` |

### FX Order Filter Values

| Filter | Values |
|--------|--------|
| `pair` | `CNGN-USDC`, `CNGN-USDT` |
| `side` | `buy`, `sell` |
| `status` | `active`, `paused`, `deleted` |

### FX Trade Filter Values

| Filter | Values |
|--------|--------|
| `pair` | `CNGN-USDC`, `CNGN-USDT` |
| `side` | `buy`, `sell` |
| `status` | `pending`, `locked`, `signing`, `settling`, `completed`, `expired`, `failed` |

### Withdrawal Filter Values

| Filter | Values |
|--------|--------|
| `tokenType` | `CNGN`, `USDC`, `USDT` |
| `status` | `pending`, `confirmed`, `failed` |
| `sortOrder` | `asc`, `desc` |

---

## Important Notes

### Asynchronous Processing

All blockchain operations return a `requestId` immediately and process in the background:

```json
{
  "requestId": "req_abc123",
  "status": "processing"
}
```

Track status via:
1. **Webhooks** - Real-time notifications to your configured URL
2. **Polling** - Query status endpoints with the `requestId`

### cNGN Token Decimals

**cNGN uses 6 decimals** (same as USDC/USDT):
- `500000000` = 500 cNGN
- `1000000` = 1 cNGN

### Amount Formats

| Operation Type | Format | Example |
|---------------|--------|---------|
| Fiat (Onramp/Offramp) | Main currency unit | `5000` = ₦5,000 |
| Token (Blockchain) | Smallest unit (wei) | `500000000` = 500 cNGN |

---

*Last updated: June 2026*
*API Version: v1.0*
