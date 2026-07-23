---
title: "Multi-Chain Wallet"
description: "Strails wallets for different supported blockchains"
---

## Overview

The **Multi-Chain Wallet** (also called **Managed Wallet**) is a Strails-controlled wallet secured by a Hardware Security Module (HSM) for ECDSA and EdDSA cryptographic signature schemes and aligns with [SLIP-0044 standard](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) HD wallet derivation. StRails uses this wallet to execute transactions on behalf of fintechs across multiple blockchain networks.

## Key Characteristics

| Feature | Description |
|---------|-------------|
| **Control** | Strails-controlled - StRails manages all transactions |
| **Security** | HSM-backed key management |
| **Assets** | cNGN, USDC, USDT |
| **Networks** | Base (primary), Ethereum, BNB Chain, Solana, Bantu |
| **Purpose** | Transaction execution, gas fee payment, cross-chain operations |

## How It Works

The Multi-Chain Wallet acts as the **execution layer** for StRails operations:

```
User Request -> StRails API -> Multi-Chain Wallet -> Blockchain
```

1. **You call an API endpoint** (e.g., `/withdrawasset`, `/swap`)
2. **StRails validates** the request and checks balances
3. **The Managed Wallet signs** the transaction using HSM
4. **Transaction is broadcast** to the appropriate blockchain

## What You Can Do

### Token Transfers
The Multi-Chain Wallet executes token transfers when you call:

- [`/withdrawasset`](/api-reference/transactions#user-token-withdrawal) - Withdraw user tokens to external wallets
- [`/fintechtransfer`](/api-reference/transactions#fintech-token-withdrawal) - Transfer fintech tokens to registered external wallets

### Swaps
Token swaps are executed through the Managed Wallet:

- [`/swap`](/api-reference/transactions#user-swap) - Swap tokens on user's smart wallet
- [`/swaptrigger`](/api-reference/transactions#swap-trigger) - Trigger async swaps (between cNGN and USDC/USDT)

### Cross-Chain Bridging
For multi-chain operations, the Managed Wallet coordinates:

- Source chain token lock/burn
- Destination chain mint/release
- Status callbacks to your webhook

## Security Model

```
+-------------------------------------+
|          AWS KMS (HSM)              |
|  +-----------------------------+    |
|  |   Private Key (never leaves) |    |
|  +-----------------------------+    |
|              |                      |
|              v                      |
|  +-----------------------------+    |
|  |   Sign Transaction Request   |    |
|  +-----------------------------+    |
+-------------------------------------+
              |
              v
     Signed Transaction -> Blockchain
```

**Key Security Features:**
- Private keys never leave the HSM
- All signing requests are audited
- No direct private key access for anyone (including StRails staff)
- Automatic key rotation policies

## Gas Fee Handling

The Multi-Chain Wallet pays gas fees for all system-initiated transactions:

- **User withdrawals**: Gas paid by StRails
- **Swaps**: Gas included in swap execution
- **Bridging**: Gas covered on both source and destination chains

You don't need to maintain native token balances - StRails handles all gas costs.

## Differences from Smart Wallet

| Aspect | Multi-Chain Wallet | Smart Wallet |
|--------|-------------------|--------------|
| Control | Strails-controlled | Fintech or Strails-controlled |
| Type | EOA (HSM-backed) | Smart Contract |
| Direct Access | No | Yes (if you set owner) |
| Use Case | Backend operations | User-facing custody |
| Self-Withdrawal | No | Yes (via `execute()`) |

## Related Documentation

- [Smart Wallet](/smart-wallet) - User-facing smart contract wallets
- [MPC Wallet](/mpc-wallet) - Fintech-controlled custody with MPC signing
- [Wallet Management API](/api-reference/wallet) - API endpoints for wallet operations
- [Transactions API](/api-reference/transactions) - Transaction execution endpoints
