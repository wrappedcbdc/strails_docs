---
title: "Going Live"
description: "Checklist before switching your Strails integration to production"
---

Before moving from the Strails test environment to production, complete the checks below to avoid integration failures, security gaps, or unexpected fees.

---

## 1. Credentials & environment

- [ ] You have received a **production API key** from Strails.
- [ ] You have switched your base URL from `https://beta.stablesrail.io/v1/` to `https://api.strails.io/v1/`.
- [ ] Your production API key is stored securely in a secret manager, not in source control or environment files committed to git.
- [ ] You have regenerated your API key if it was ever exposed or shared.

## 2. IP allowlisting

- [ ] You have retrieved the public IP address(es) of your production servers.
- [ ] You have added those IPs to your Strails production allowlist via [`/manageipallowlist`](/api-reference/management-api#manage-ip-addresses-allowed-to-access-your-api) or through Strails support.
- [ ] You have tested a request from production to confirm allowlisting works.

## 3. Payload encryption

- [ ] You have generated a production Ed25519 key pair.
- [ ] You have shared your production public key with Strails support.
- [ ] You have received the production Strails public key.
- [ ] You have verified encryption/decryption round-trips in the test environment.
- [ ] You are using the production keys when sending encrypted production payloads.

See [Encryption](/encryption) for details.

## 4. Webhooks

- [ ] You have configured a production HTTPS webhook URL via [`/setwebhook`](/api-reference/management-api#configure-webhook-urls).
- [ ] Your webhook handler returns `200 OK` immediately and processes events asynchronously.
- [ ] You have configured a strong webhook secret.
- [ ] Your handler verifies `X-Strails-Signature` using HMAC-SHA256.
- [ ] You reject old `X-Strails-Timestamp` values to prevent replay attacks.
- [ ] You have handled all critical event types your integration depends on.

See [HMAC Signatures](/hmac-signatures) and [Webhook Events](/api-reference/webhook-events) for details.

## 5. Wallets & liquidity

- [ ] You have retrieved your production fintech wallet via [`/getfintechwallet`](/api-reference/wallet).
- [ ] You have verified MPC Vault registration and auto-signing configuration for FX trading.
- [ ] You have whitelisted the `natExternalIp` from [`/fx/settings`](/api-reference/fx-setting#get-fx-settings) in MPCVault.
- [ ] You have confirmed sufficient cNGN and USDC/USDT liquidity for expected volumes.
- [ ] You understand how active FX orders lock wallet balances.

## 6. Fees & bank accounts

- [ ] You have reviewed your fintech fee configuration via [`/getfees`](/api-reference/fee-management-api#retrieve-the-current-fee-configuration-and-an-example-calculation-for-typical-amounts).
- [ ] You have configured your fee structure via [`/managefees`](/api-reference/fee-management-api#configure-your-fee-settings).
- [ ] You have added and verified your production settlement bank account via [`/addbankaccount`](/api-reference/fiat-payout-management#add-a-new-nigerian-bank-account-for-fintech-payout).
- [ ] You have tested a small-value offramp to confirm settlement works.

## 7. End-to-end testing

- [ ] User onboarding with BVN works end to end.
- [ ] Virtual account creation and deposit flow works end to end.
- [ ] Wallet funding (onramp) and payout (offramp) complete successfully.
- [ ] Token swap and withdrawal flows are tested with expected amounts.
- [ ] FX quote, trade execution, and trade status tracking are verified.

## 8. Monitoring & support

- [ ] You have logging and alerting in place for failed API requests and webhook deliveries.
- [ ] You have a runbook for handling rate limits (`429` responses).
- [ ] You know how to contact Strails support: [support@strails.io](mailto:support@strails.io).

---

## Still blocked?

If any item above is unchecked or unclear, contact [support@strails.io](mailto:support@strails.io) before switching to production.
