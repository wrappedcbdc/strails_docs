---
title: "Webhook Events"
description: "Webhook event payloads for virtual accounts, payments, wallet funding, swaps, deposits, asset transfers, and offramp operations"
---

Real-time notifications for all critical operations including virtual account creation, payment confirmations, wallet funding, swaps, deposits, asset transfers, and offramp transactions.

## Create a webhook URL

Your webhook URL is a public POST endpoint that accepts JSON. Keep the handler fast: acknowledge receipt immediately (HTTP 200) and perform any heavy work asynchronously.

Example (minimal):

```http
Content-Type: application/json
X-Strails-Signature: <hex-hmac>
X-Strails-Timestamp: 2026-06-09T14:50:26.934Z
X-Webhook-ID: <uuid>
X-Strails-Event: <event>
```

Node (minimal acknowledge):

```javascript
// Express (ack only)
app.post('/webhook', express.json(), (req, res) => {
  // read and enqueue work, or verify + process
  res.sendStatus(200)
})
```

## Supported webhook events (index)

* [user.onboarded](#user-onboarded)
* [virtual.account.created](#virtual-account-created)
* [payments.confirmed](#payments-confirmed)
* [wallet.funding.completed](#wallet-funding-completed)
* [swap.completed](#swap-completed)
* [swap.failed](#swap-failed)
* [fintech.virtual_account.deposit.received](#fintech-virtual_account-deposit-received)
* [fintech.user.deposit.received](#fintech-user-deposit-received)
* [fintech.user.deposit.funding.completed](#fintech-user-deposit-funding-completed)
* [fintech.user.deposit.refunded](#fintech-user-deposit-refunded)
* [fintech.asset.transfer.completed](#fintech-asset-transfer-completed)
* [fintech.asset.transfer.failed](#fintech-asset-transfer-failed)
* [fintech.user.asset.transfer.completed](#fintech-user-asset-transfer-completed)
* [fintech.user.asset.transfer.failed](#fintech-user-asset-transfer-failed)
* [vault.return.transfer.confirmed](#vault-return-transfer-confirmed)
* [vault.return.payout.completed](#vault-return-payout-completed)
* [vault.return.payout.failed](#vault-return-payout-failed)
* [fintech.offramp.initiated](#fintech-offramp-initiated)
* [fintech.offramp.transfer.completed](#fintech-offramp-transfer-completed)
* [fintech.offramp.payout.initiated](#fintech-offramp-payout-initiated)
* [fintech.offramp.completed](#fintech-offramp-completed)
* [fintech.offramp.failed](#fintech-offramp-failed)

---

## user-onboarded

Triggered when a user has been successfully onboarded.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid-event-id",
  "eventType": "user.onboarded",
  "timestamp": "2026-07-02T10:30:00.000Z",
  "requestId": "req-12345",
  "fintechId": "fintech-uuid",
  "version": "1.0.0",
  "userId": "user-hash-12345",
  "payload": {
    "firstName": "John",
    "lastName": "Doe",
    "onboardedAt": "2026-07-02T10:30:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

## virtual-account-created

Triggered when a reserved or checkout virtual account is created for a user.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "ba092127-3e96-4439-bb30-b9c44142afa7",
  "eventType": "virtual.account.created",
  "timestamp": "2026-05-07T15:24:14.635Z",
  "requestId": "req-12345",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "payload": {
    "vaId": "mF50aVg9GLc2xyup0blf",
    "accountNumber": "9800003872",
    "bankName": "FAME MFB",
    "accountName": "Alphgeek Technologies/UDE",
    "createdAt": "2026-05-07T15:24:14.635Z"
  }
}
```

</Tab>
</Tabs>

---

## payments-confirmed

Triggered when a payment to a virtual account is received from payment gateway.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "143f72e3-d47c-4339-8fe0-a59a78781db8",
  "eventType": "payments.confirmed",
  "timestamp": "2026-05-07T15:27:01.717Z",
  "requestId": "req-12345",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "userId": "user-hash-12345",
  "payload": {
    "txRef": "0x095a6e55efd0c17ab0f717e367db386aa6d06a27dfa838b90037453e6e6c35bf",
    "reference": "0x095a6e55efd0c17ab0f717e367db386aa6d06a27dfa838b90037453e6e6c35bf",
    "amount": 101.5,
    "currency": "NGN",
    "status": "confirmed",
    "confirmedAt": "2026-05-07T15:27:01.717Z",
    "paymentTimestamp": "2026-05-07T15:27:01.717Z",
    "metadata": {
      "vaId": "mF50aVg9GLc2xyup0blf",
      "provider": "novac",
      "walletAddress": "0x82Cc3B1035D363B02B09B787B4325a8C740658b1"
    }
  }
}
```

</Tab>
</Tabs>

---

## wallet-funding-completed

Triggered when wallet funding (cNGN minting) is completed after a confirmed payment.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "178f0abc-2db7-4a78-bf89-510abfdd615b",
  "eventType": "wallet.funding.completed",
  "timestamp": "2026-05-07T15:28:20.832Z",
  "requestId": "req-12345",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "userId": "user-hash-12345",
  "payload": {
    "walletAddress": "0x82Cc3B1035D363B02B09B787B4325a8C740658b1",
    "amount": "100",
    "transactionHash": "0x7d832e32bb5128bf8ac19e0ad4d18c92026eec14a1b0d9a60a4df95945afd1f4",
    "completedAt": "2026-05-07T15:28:20.832Z"
  }
}
```

</Tab>
</Tabs>

---

## swap-completed

Triggered when a token swap is successfully completed.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "ba092127-3e96-4439-bb30-b9c44142afa7",
  "eventType": "swap.completed",
  "timestamp": "2026-05-07T15:30:00.000Z",
  "requestId": "req-12345",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "userId": "user-hash-12345",
  "payload": {
    "walletAddress": "0x82Cc3B1035D363B02B09B787B4325a8C740658b1",
    "owner": "0x1234567890abcdef1234567890abcdef12345678",
    "sellToken": "0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9",
    "buyToken": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "amountIn": "1000.00",
    "amountOut": "0.75",
    "swapTxHash": "0x7d832e32bb5128bf8ac19e0ad4d18c92026eec14a1b0d9a60a4df95945afd1f4",
    "transferTxHash": "0x095a6e55efd0c17ab0f717e367db386aa6d06a27dfa838b90037453e6e6c35bf",
    "completedAt": "2026-05-07T15:30:00.000Z",
    "smartWalletContext": {
      "smartWalletAddress": "0x82Cc3B1035D363B02B09B787B4325a8C740658b1",
      "managedWalletAddress": "0xabc123..."
    },
    "swapMetrics": {
      "executionTime": 2500,
      "gasUsed": "185000",
      "slippage": "0.5"
    }
  }
}
```

</Tab>
</Tabs>

---

## swap-failed

Triggered when a token swap fails.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "ba092127-3e96-4439-bb30-b9c44142afa8",
  "eventType": "swap.failed",
  "timestamp": "2026-05-07T15:30:00.000Z",
  "requestId": "req-12346",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "userId": "user-hash-12345",
  "payload": {
    "walletAddress": "0x82Cc3B1035D363B02B09B787B4325a8C740658b1",
    "owner": "0x1234567890abcdef1234567890abcdef12345678",
    "sellToken": "0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9",
    "buyToken": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "amountIn": "1000.00",
    "failedAt": "2026-05-07T15:30:00.000Z",
    "error": {
      "code": "INSUFFICIENT_LIQUIDITY",
      "message": "Not enough liquidity in pool for swap"
    },
    "retryable": true,
    "metadata": {
      "attemptNumber": 1,
      "pool": "0xabc..."
    }
  }
}
```

</Tab>
</Tabs>

---

## fintech-virtual_account-deposit-received

Triggered when a deposit is received on a fintech's reserved virtual account.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "f8e2a1b3-4c5d-6e7f-8a9b-0c1d2e3f4a5b",
  "eventType": "fintech.virtual_account.deposit.received",
  "timestamp": "2025-08-27T10:00:00Z",
  "requestId": "dep-67890",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "payload": {
    "depositId": "dep-67890",
    "virtualAccount": {
      "accountNumber": "1234567890",
      "accountName": "Fintech System Account"
    },
    "deposit": {
      "amount": 50000.00,
      "currency": "NGN",
      "reference": "TRX123456789"
    },
    "depositor": {
      "name": "JOHN DOE SMITH",
      "accountNumber": "0123456789",
      "bankName": "Access Bank",
      "bankCode": "044"
    },
    "metadata": {
      "provider": "novac",
      "receivedAt": "2025-08-27T09:58:30Z"
    }
  }
}
```

</Tab>
</Tabs>

---

## fintech-user-deposit-received

Triggered when a deposit is received on a fintech user's reserved virtual account (before wallet funding).

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid",
  "eventType": "fintech.user.deposit.received",
  "timestamp": "2026-06-10T12:00:00.000Z",
  "requestId": "deposit-id",
  "fintechId": "your-fintech-id",
  "version": "1.0",
  "userId": "user-hash-string",
  "payload": {
    "depositId": "abc123",
    "virtualAccount": {
      "accountNumber": "9800003872",
      "accountName": "Alphgeek Technologies/UDE"
    },
    "deposit": {
      "amount": 50000.00,
      "currency": "NGN",
      "reference": "NOVAC-REF-123"
    },
    "depositor": {
      "name": "JOHN DOE SMITH",
      "accountNumber": "0123456789",
      "bankName": "Access Bank",
      "bankCode": "044"
    },
    "metadata": {
      "provider": "novac",
      "receivedAt": "2026-06-10T11:58:30.000Z"
    }
  }
}
```

</Tab>
</Tabs>

---

## fintech-user-deposit-funding-completed

Triggered after a fintech user successfully deposits funds into their permanent virtual account and cNGN is sent to their Strails-controlled HSM wallet.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid",
  "eventType": "fintech.user.deposit.funding.completed",
  "timestamp": "2026-05-30T12:00:00.000Z",
  "requestId": "deposit-id",
  "fintechId": "your-fintech-id",
  "version": "1.0",
  "userId": "user-hash-string",
  "payload": {
    "depositId": "abc123",
    "amount": "1000.00",
    "currency": "NGN",
    "transactionReference": "NOVAC-REF-123",
    "smartWalletAddress": "0x...",
    "transactionHash": "0x...",
    "depositor": {
      "accountNumber": "1234567890",
      "accountName": "John Doe",
      "bankCode": "123456",
      "bankName": "Test Bank"
    },
    "bvnVerified": true,
    "completedAt": "2026-05-30T12:00:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

## fintech-user-deposit-refunded

Triggered when a deposit to a fintech user's permanent virtual account is refunded due to BVN name mismatch.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "eventType": "fintech.user.deposit.refunded",
  "timestamp": "2026-06-10T12:00:00.000Z",
  "requestId": "deposit-id-123",
  "fintechId": "fintech-uuid",
  "version": "1.0.0",
  "userId": "user-hash-string",
  "payload": {
    "depositId": "deposit-id",
    "amount": 5000.00,
    "currency": "NGN",
    "transactionReference": "tx-ref-123",
    "refundReference": "refund-ref-456",
    "refundReason": "bvn_name_mismatch",
    "accountType": "checkout",
    "depositor": {
      "accountNumber": "0123456789",
      "accountName": "John Doe",
      "bankCode": "044",
      "bankName": "Access Bank"
    },
    "bvnVerification": {
      "isMatch": false,
      "confidenceScore": 0.45,
      "rejectionReasons": ["name_mismatch", "low_confidence"]
    },
    "refundedAt": "2026-06-10T12:00:00.000Z",
    "metadata": {}
  }
}
```

</Tab>
</Tabs>

---

## fintech-asset-transfer-completed

Triggered when a fintech successfully transfers assets from their smart wallet to a registered external wallet.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid",
  "eventType": "fintech.asset.transfer.completed",
  "timestamp": "2026-06-10T12:00:00.000Z",
  "requestId": "transfer-request-id",
  "fintechId": "your-fintech-id",
  "version": "1.0",
  "payload": {
    "smartWalletAddress": "0x82Cc3B1035D363B02B09B787B4325a8C740658b1",
    "destinationAddress": "0x1234567890abcdef1234567890abcdef12345678",
    "destinationLabel": "Treasury Wallet",
    "amount": "1000.00",
    "ticker": "cNGN",
    "tokenAddress": "0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9",
    "transactionHash": "0x7d832e32bb5128bf8ac19e0ad4d18c92026eec14a1b0d9a60a4df95945afd1f4",
    "blockNumber": 12345678,
    "gasUsed": "85000",
    "commitmentsSafeguarded": {
      "activeOrders": 2,
      "activeEscrows": 1,
      "totalCommitted": "5000000000",
      "remainingBalance": "95000000000"
    },
    "note": "Monthly treasury transfer",
    "completedAt": "2026-06-10T12:00:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

## fintech-asset-transfer-failed

Triggered when a fintech asset transfer fails.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid",
  "eventType": "fintech.asset.transfer.failed",
  "timestamp": "2026-06-10T12:00:00.000Z",
  "requestId": "transfer-request-id",
  "fintechId": "your-fintech-id",
  "version": "1.0",
  "payload": {
    "error": "Insufficient balance after accounting for FX orderbook commitments",
    "failedAt": "2026-06-10T12:00:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

## fintech-user-asset-transfer-completed

Triggered when a fintech user successfully transfers assets from their smart wallet to an external wallet.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
  "eventType": "fintech.user.asset.transfer.completed",
  "timestamp": "2026-06-10T12:00:00.000Z",
  "requestId": "0x...",
  "fintechId": "fintech-uuid",
  "version": "1.0.0",
  "userId": "user-hash",
  "payload": {
    "smartWalletAddress": "0x...",
    "destinationAddress": "0x...",
    "amount": "1000.00",
    "ticker": "CNGN",
    "tokenAddress": "0x...",
    "network": "base",
    "transactionHash": "0x...",
    "blockNumber": "12345678",
    "gasUsed": "21000",
    "completedAt": "2026-06-10T12:00:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

## fintech-user-asset-transfer-failed

Triggered when a fintech user asset transfer fails.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "c3d4e5f6-a7b8-9012-cdef-345678901234",
  "eventType": "fintech.user.asset.transfer.failed",
  "timestamp": "2026-06-10T12:00:00.000Z",
  "requestId": "failed-1718020800000",
  "fintechId": "fintech-uuid",
  "version": "1.0.0",
  "userId": "user-hash",
  "payload": {
    "smartWalletAddress": "0x...",
    "destinationAddress": "0x...",
    "amount": "1000.00",
    "ticker": "CNGN",
    "tokenAddress": "0x...",
    "network": "base",
    "error": "Insufficient balance for transfer",
    "failedAt": "2026-06-10T12:00:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

## vault-return-transfer-confirmed

Triggered when a fintech user asset is sent from their smart wallet to Strails Vault for offramp.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid",
  "eventType": "vault.return.transfer.confirmed",
  "timestamp": "2026-06-23T10:00:00.000Z",
  "requestId": "request-uuid",
  "fintechId": "fintech-uuid",
  "version": "1.0.0",
  "userId": "user-hash",
  "payload": {
    "transferId": "...",
    "vaultReturnId": "...",
    "amount": "100.50",
    "tokenAddress": "0x...",
    "transactionHash": "0x...",
    "confirmedAt": "...",
    "blockNumber": 12345,
    "metadata": {}
  }
}
```

</Tab>
</Tabs>

---

## vault-return-payout-completed

Triggered after the token is received and payout to the fintech user's fiat bank account is confirmed.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid",
  "eventType": "vault.return.payout.completed",
  "timestamp": "2026-06-23T10:00:00.000Z",
  "requestId": "request-uuid",
  "fintechId": "fintech-uuid",
  "version": "1.0.0",
  "userId": "user-hash",
  "payload": {
    "payoutId": "...",
    "vaultReturnId": "...",
    "amount": "15000",
    "recipientAccountNumber": "0123456789",
    "recipientBankCode": "044",
    "transactionReference": "...",
    "completedAt": "...",
    "metadata": {}
  }
}
```

</Tab>
</Tabs>

---

## vault-return-payout-failed

Triggered when payout to the fintech user's fiat bank account fails.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "uuid",
  "eventType": "vault.return.payout.failed",
  "timestamp": "2026-06-23T10:00:00.000Z",
  "requestId": "request-uuid",
  "fintechId": "fintech-uuid",
  "version": "1.0.0",
  "userId": "user-hash",
  "payload": {
    "payoutId": "...",
    "vaultReturnId": "...",
    "amount": "15000",
    "recipientAccountNumber": "0123456789",
    "recipientBankCode": "044",
    "failedAt": "...",
    "error": {
      "message": "...",
      "code": "..."
    },
    "retryable": true,
    "metadata": {}
  }
}
```

</Tab>
</Tabs>

---

## fintech-offramp-initiated

Triggered when a fintech initiates an offramp or payout in fiat(cNGN to NGN).

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "d4e5f6a7-b8c9-0123-def4-567890123456",
  "eventType": "fintech.offramp.initiated",
  "timestamp": "2025-08-27T10:00:00Z",
  "requestId": "offramp-req-67890",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "payload": {
    "amount": 100000.00,
    "currency": "NGN",
    "status": "pending",
    "bankAccount": {
      "accountNumber": "0123456789",
      "accountName": "John Doe",
      "bankName": "GTBank"
    },
    "wallet": {
      "source": "system_wallet",
      "address": "0x1234...5678",
      "network": "base"
    },
    "metadata": {
      "description": "Withdrawal request",
      "initiatedBy": "api"
    }
  }
}
```

</Tab>
</Tabs>

---

## fintech-offramp-transfer-completed

Triggered when the on-chain cNGN transfer to Strails settlement wallet is confirmed (before bank payout).

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "e5f6a7b8-c9d0-1234-ef56-789012345678",
  "eventType": "fintech.offramp.transfer.completed",
  "timestamp": "2025-08-27T10:02:00Z",
  "requestId": "offramp-req-67890",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "payload": {
    "amount": 100000.00,
    "currency": "NGN",
    "status": "initiating_payout",
    "burnDetails": {
      "txHash": "0xabc123...def456",
      "status": "confirmed",
      "cNgnAmount": 100000000000000000000,
      "blockNumber": 12345678,
      "confirmations": 12
    },
    "metadata": {
      "gasUsed": "85000",
      "transferredAt": "2025-08-27T10:02:00Z"
    }
  }
}
```

</Tab>
</Tabs>

---

## fintech-offramp-payout-initiated

Triggered when bank payout is initiated after token is returned to strails wallet.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "f6a7b8c9-d0e1-2345-f678-901234567890",
  "eventType": "fintech.offramp.payout.initiated",
  "timestamp": "2025-08-27T10:03:00Z",
  "requestId": "offramp-req-67890",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "payload": {
    "amount": 100000.00,
    "currency": "NGN",
    "status": "payout_processing",
    "bankAccount": {
      "accountNumber": "0123456789",
      "accountName": "John Doe",
      "bankName": "GTBank",
      "bankCode": "058"
    },
    "payoutDetails": {
      "reference": "PAYOUT-123456",
      "provider": "novac",
      "status": "processing"
    },
    "metadata": {
      "payoutInitiatedAt": "2025-08-27T10:03:00Z"
    }
  }
}
```

</Tab>
</Tabs>

---

## fintech-offramp-completed

Triggered when an offramp payout is successfully completed.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "a7b8c9d0-e1f2-3456-7890-123456789abc",
  "eventType": "fintech.offramp.completed",
  "timestamp": "2025-08-27T10:05:00Z",
  "requestId": "offramp-req-67890",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "payload": {
    "amount": 100000.00,
    "currency": "NGN",
    "status": "completed",
    "bankAccount": {
      "accountNumber": "0123456789",
      "accountName": "John Doe",
      "bankName": "GTBank"
    },
    "wallet": {
      "source": "system_wallet",
      "address": "0x1234...5678",
      "network": "base"
    },
    "burnDetails": {
      "txHash": "0xabc123...def456",
      "status": "confirmed",
      "cNgnAmount": 100000000000000000000
    },
    "payoutDetails": {
      "reference": "PAYOUT-123456",
      "provider": "novac",
      "status": "completed"
    },
    "completedAt": "2025-08-27T10:05:00Z",
    "metadata": {
      "processingTime": 300000
    }
  }
}
```

</Tab>
</Tabs>

---

## fintech-offramp-failed

Triggered when an offramp payout fails at any stage.

<Tabs>
<Tab title="JSON">

```json
{
  "eventId": "b8c9d0e1-f2a3-4567-8901-23456789abcd",
  "eventType": "fintech.offramp.failed",
  "requestId": "offramp-req-67890",
  "fintechId": "fintech-12345",
  "version": "1.0.0",
  "payload": {
    "amount": 100000.00,
    "currency": "NGN",
    "status": "failed",
    "bankAccount": {
      "accountNumber": "0123456789",
      "accountName": "John Doe",
      "bankName": "GTBank"
    },
    "wallet": {
      "source": "system_wallet",
      "address": "0x1234...5678",
      "network": "base"
    },
    "burnDetails": {
      "txHash": "0xabc123...def456",
      "status": "confirmed",
      "cNgnAmount": 100000000000000000000
    },
    "error": {
      "message": "Bank payout failed: Invalid account number",
      "code": "PAYOUT_FAILED"
    },
    "timestamp": "2025-08-27T10:05:00Z",
    "metadata": {
      "failedAt": "2025-08-27T10:05:00Z",
      "retryable": false,
      "attemptNumber": 1
    }
  }
}
```

</Tab>
</Tabs>