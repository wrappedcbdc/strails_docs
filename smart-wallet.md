---
title: "Smart Wallet"
description: Generate disposable wallet for transaction privacy
---

## Overview

StRails uses **Smart Wallets** (also known as Smart Contract Wallets) to provide secure, programmable token custody for fintechs and their users. Unlike traditional Externally Owned Accounts (EOAs) controlled directly by private keys, Smart Wallets are smart contracts that can execute arbitrary transactions on behalf of their owners.

## Smart Wallet vs Account Abstraction (ERC-4337)

These terms are often confused but refer to different things:

| Term | Definition |
|------|------------|
| **Smart Wallet** | Any wallet implemented as a smart contract rather than an EOA. Generic term for contract-based wallets. |
| **Account Abstraction (ERC-4337)** | A specific Ethereum standard that adds advanced features like gas sponsorship, bundlers, and paymasters to smart wallets. |

### What StRails Uses

StRails implements a **minimal smart contract wallet** — not a full ERC-4337 Account Abstraction wallet. Our approach:

- **Simple & Gas-Efficient**: Uses EIP-1167 minimal proxy pattern for cheap deployments
- **Owner-Controlled**: Your EOA directly calls `execute()` on the wallet
- **No External Dependencies**: No bundlers, paymasters, or EntryPoint contracts required

### What We Don't Have (ERC-4337 Features)

| ERC-4337 Feature | Description | StRails |
|------------------|-------------|---------|
| UserOperations | Meta-transactions bundled off-chain | ❌ Not supported |
| Bundlers | Off-chain actors that submit transactions | ❌ Not needed |
| Paymasters | Contracts that sponsor gas fees | ❌ Owner pays gas directly |
| EntryPoint | Singleton contract for AA validation | ❌ Not used |

**Why this design?** Simplicity and lower gas costs. Your EOA signs transactions directly to the wallet contract, avoiding the overhead of the ERC-4337 infrastructure.

## Smart Wallet Architecture

### WalletFactory Contract

The `WalletFactory` deploys new wallet instances using the **Clones** library (EIP-1167 minimal proxy pattern). This approach:

- Deploys wallets at **deterministic addresses** based on a salt
- Reduces deployment gas costs by ~90% compared to full contract deployment
- All wallets share the same implementation logic

```solidity
// Get the predicted address before deployment
function getAddress(bytes32 _salt) external view returns (address);

// Deploy a new wallet for an owner
function createAccount(address _owner, bytes32 _salt) external returns (address wallet);
```

### Wallet Contract

Each deployed wallet is a minimal proxy that delegates to the implementation contract:

| Function | Description |
|----------|-------------|
| `owner` | Returns the wallet owner's EOA address |
| `nonce` | Transaction counter for replay protection |
| `initialize(address _owner)` | One-time setup (called by factory) |
| `execute(address to, uint256 value, bytes data)` | Execute any transaction |
| `changeOwner(address newOwner)` | Transfer wallet ownership |

**Key Security Features:**
- Only the owner can call `execute()` and `changeOwner()`
- Each wallet can only be initialized once
- Nonce increments on every execution for replay protection

## Withdrawing Tokens from Your Smart Wallet

Since your EOA owns the Smart Wallet, you call `execute()` on the wallet contract to make it transfer tokens. The wallet then calls the token's `transfer()` function internally.

### Method 1: Via Block Explorer (No Code Required)

1. Navigate to your wallet contract on [Basescan](https://basescan.org) or [Etherscan](https://etherscan.io)
2. Go to **Contract** → **Write Contract**
3. Connect your MetaMask wallet (must be the owner EOA)
4. Find the `execute` function and fill in:

| Parameter | Value |
|-----------|-------|
| `to` | Token contract address (e.g., cNGN: `0x...`) |
| `value` | `0` (we're not sending native tokens) |
| `data` | Encoded `transfer(address,uint256)` call (see below) |

### Building the `data` Parameter

The `data` parameter is the ABI-encoded function call for the token's `transfer` function:

```javascript
// Example: Transfer 1000 cNGN (6 decimals) to a recipient
const { ethers } = require("ethers");

const data = new ethers.Interface([
  "function transfer(address to, uint256 amount)"
]).encodeFunctionData("transfer", [
  "0xRecipientAddressHere",
  ethers.parseUnits("1000", 6)  // 1000 tokens with 6 decimals
]);

console.log(data);
// Output: 0xa9059cbb000000000000000000000000...
// Use this hex string as the 'data' parameter in execute()
```

**Manual Encoding Format:**
```
0xa9059cbb                                                       // transfer function selector
000000000000000000000000RECIPIENT_ADDRESS_WITHOUT_0x              // recipient (32 bytes, padded)
0000000000000000000000000000000000000000000000000000000000AMOUNT   // amount in hex (32 bytes)
```

### Method 2: Using ethers.js Script

```javascript
const { ethers } = require("ethers");

// Setup provider and signer
const provider = new ethers.JsonRpcProvider("https://mainnet.base.org");
const signer = new ethers.Wallet("YOUR_PRIVATE_KEY", provider);

// Your Smart Wallet address (generated by factory)
const walletAddress = "0xYourSmartWalletAddress";
const walletABI = [
  "function execute(address to, uint256 value, bytes data) returns (bytes)"
];
const wallet = new ethers.Contract(walletAddress, walletABI, signer);

// Token details
const tokenAddress = "0xTokenContractAddress"; // e.g., cNGN
const recipient = "0xYourPersonalWallet";
const amount = ethers.parseUnits("1000", 6); // 1000 tokens (6 decimals)

// Encode the transfer call
const transferData = new ethers.Interface([
  "function transfer(address to, uint256 amount)"
]).encodeFunctionData("transfer", [recipient, amount]);

// Execute the withdrawal
const tx = await wallet.execute(tokenAddress, 0, transferData);
await tx.wait();

console.log("Withdrawal successful:", tx.hash);
```

### Method 3: Using viem (TypeScript)

```typescript
import { createWalletClient, http, encodeFunctionData, parseUnits } from 'viem';
import { base } from 'viem/chains';
import { privateKeyToAccount } from 'viem/accounts';

const account = privateKeyToAccount('0xYOUR_PRIVATE_KEY');

const client = createWalletClient({
  account,
  chain: base,
  transport: http(),
});

// Encode the ERC20 transfer
const transferData = encodeFunctionData({
  abi: [{ 
    name: 'transfer', 
    type: 'function',
    inputs: [
      { name: 'to', type: 'address' },
      { name: 'amount', type: 'uint256' }
    ],
    outputs: [{ type: 'bool' }]
  }],
  functionName: 'transfer',
  args: ['0xRecipientAddress', parseUnits('1000', 6)],
});

// Call execute on your Smart Wallet
const hash = await client.writeContract({
  address: '0xYourSmartWalletAddress',
  abi: [{
    name: 'execute',
    type: 'function',
    inputs: [
      { name: 'to', type: 'address' },
      { name: 'value', type: 'uint256' },
      { name: 'data', type: 'bytes' }
    ],
    outputs: [{ type: 'bytes' }]
  }],
  functionName: 'execute',
  args: ['0xTokenContractAddress', 0n, transferData],
});

console.log('Transaction hash:', hash);
```

## Important Notes

| Topic | Details |
|-------|---------|
| **Gas Fees** | The owner EOA pays gas in the native token (ETH on Base/Ethereum) |
| **Token Decimals** | cNGN uses 6 decimals; always verify the token's decimals before encoding amounts |
| **Ownership** | Only the wallet owner can execute transactions; verify ownership before attempting withdrawals |
| **Liquidity Locks** | If the wallet has active FX orders, some liquidity may be locked and unavailable for withdrawal |

## Related Documentation

- [Wallet Management](/api-reference/wallet) - API endpoints for wallet operations
- [Transactions Management](/api-reference/transactions) - Transaction history and status
