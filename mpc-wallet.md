---
title: "MPC Wallet"
description: Connect and manage non-custodial MPC wallet
---

## Overview

The MPC Wallet (also called MPC Vault) is a fintech-controlled custody wallet used for stablecoin settlement in FX trading. It uses threshold signing so transactions can be authorized without exposing a single reusable private key.

In the StRails FX model:
- MPC Vault is the custody source for stablecoins such as USDC and USDT.
- Smart Wallet is the custody source for cNGN.
- The side sending stablecoins from MPC Vault is the side that must sign.

## Key Characteristics

| Feature | Description |
|---------|-------------|
| Control | Fintech-controlled transaction authorization |
| Security | Threshold signing with separated key material |
| Primary assets | USDC, USDT (and other enabled quote tokens) |
| Primary network | Base |
| Trading role | Stablecoin custody and settlement leg for FX orderbook |

## What MPC Means Here

MPC (Multi-Party Computation) allows a signature to be produced through distributed key operations instead of using one exposed private key in one location.

Practical outcomes:
- No single key holder can unilaterally move funds.
- Signing can be policy-gated (manual approval or auto-signing rules).
- Key material is not handled like a plain private key workflow.

## FX Settlement Model

All LP-to-LP FX settlements involving stablecoin transfer use MPC signing on the stablecoin leg.

### Who Signs

| Order On Book | Stablecoin Sender | Who Signs |
|---------------|-------------------|-----------|
| BUY order | Maker's MPC Vault | Maker |
| SELL order | Taker's MPC Vault | Taker |

Rule of thumb: whoever is sending stablecoins from MPC Vault signs.

### Wallet Roles

| Asset Leg | Source Wallet |
|-----------|---------------|
| cNGN leg | Smart Wallet |
| USDC/USDT leg | MPC Vault |

## Setup Flow

The MPC setup flow for FX has three core steps:
1. Register MPC Vault configuration.
2. Enable auto-signing.
3. Set auto-signing threshold.

### 1. Register MPC Vault

Register your MPC vault credentials and token wallet mappings so your fintech can participate in FX settlement. This step initializes your MPC configuration and places it in approval flow before activation.

Before registration, create and enable your MPCVault API user so you can generate the API token used as `mpcVaultApiKey` in your configuration.

MPCVault API setup guide: [How to Enable API - Step 1: Create the API User](https://docs.mpcvault.com/guides/18-how-to-enable-api#step-1-create-the-api-user)

To obtain `callbackClientSignerPublicKey` and `mpcClientSignerPrivateKey`, follow steps 1, 2, and 3 in the MPCVault client signer guide:
[How to Enable API - Client Signer](https://docs.mpcvault.com/guides/18-how-to-enable-api/client-signer)

See: [Onchain FX Settings - Register MPC Vault](/api-reference/fx-setting#register-mpc-vault)

### 2. Enable Auto-Signing

Enable automatic signing for eligible FX transactions so lower-risk trades can proceed without manual approval.

See: [Onchain FX Settings - Enable Auto-Signing](/api-reference/fx-setting#enable-auto-signing)

### 3. Set Auto-Signing Threshold

Define the USD value limit for auto-signing. Transactions at or below this threshold can be auto-signed (when auto-signing is enabled), while higher-value transactions stay on manual approval.

See: [Onchain FX Settings - Set Auto-Signing Threshold](/api-reference/fx-setting#set-auto-signing-threshold)

### 4. Verify Configuration

Retrieve your FX configuration to confirm vault status, auto-signing state, threshold, and signer infrastructure readiness.

See: [Onchain FX Settings - Get FX Settings](/api-reference/fx-setting#get-fx-settings)
See response fields: [Onchain FX Settings - Response Fields](/api-reference/fx-setting#response-fields)

<Warning>
**Critical Whitelist Requirement**

After StRails admin approval, use the returned `natExternalIp` value from Get FX Settings as the IP address to whitelist in MPCVault.

Add this IP in both places:
- API User IP Whitelist
- API Client Signer IP Whitelist

Auto-signing will not work correctly until this whitelist step is completed.
</Warning>

## Liquidity Requirements

When placing orders, liquidity is required in the correct wallet:

| Order Type (on book) | Required Liquidity | Wallet |
|----------------------|--------------------|--------|
| BUY order | cNGN | Smart Wallet |
| SELL order | USDC/USDT | MPC Vault |

Important:
- Active orders lock liquidity.
- Available balance must be treated as total balance minus locked balance.

## Operational Lifecycle

Typical lifecycle for signed settlements:
1. Trade is locked.
2. Signing request is created.
3. Status is tracked by polling.
4. On-chain transfer is verified.
5. Settlement state is finalized.

This means completion is based on verified state transitions, not only request submission.

## Security and Control Boundaries

What you control:
- MPC Vault credentials and authorization posture.
- Manual approval process (where applicable).
- Auto-signing thresholds and operational enablement.

What platform enforces:
- API auth and request validation.
- Trade-state orchestration and settlement checks.
- Signing workflow integration and status tracking.

## Wallet Comparison

| Aspect | MPC Vault | Smart Wallet | Multi-Chain Wallet |
|--------|-----------|--------------|-------------------|
| Primary control | Fintech authorization | System-controlled smart contract flows | System-managed execution wallet |
| Primary role in FX | Stablecoin custody and signing | cNGN custody/transfer leg | Network expansion and operational routing |
| Signing model | Threshold signing | Contract-based execution | Managed key infrastructure |

## Related Documentation

- [Smart Wallet](/smart-wallet)
- [Multi-Chain Wallet](/multi-chain-wallet)
- [Onchain FX Settings](/api-reference/fx-setting)
- [Trading Management](/api-reference/trading-management)
