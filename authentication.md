---
title: "API Overview & Authentication"
sidebarTitle: "Authentication"
description: "API authentication, security, and best practices for Strails"
---

This section covers base URLs, authentication, rate limiting, error handling, and security best practices.

---

## Base URL

All API requests should be made to the following base URL:

**Test Environment:**
```
https://beta.stablesrail.io/v1/
```

**Production Environment:**
```
https://api.strails.io/v1/
```

---

## Authentication Methods

### API Key Authentication

All API requests require a valid API key included in the request headers.

<Tabs>
<Tab title="HTTP Headers">
```http
x-api-key: YOUR_API_KEY
Content-Type: application/json
```
</Tab>
</Tabs>

### Obtaining API Keys

**If you haven't received your API key yet,** please contact our team to complete your onboarding process. You will receive an email with detailed steps on how to obtain your `x-api-key`.

## Rate Limiting

Each API endpoint type has specific rate limits to ensure fair usage and system stability. Refer to the table below for the allowed request rates per endpoint category.

<Info>
If your backend encounters rate limits often, consider implementing a queue to process requests asynchronously. This approach allows your system to handle actions in an eventually consistent manner, reducing the likelihood of hitting limits.

If your use case requires higher rate limits, contact our team with details about your application and traffic patterns. We will review your request and discuss possible adjustments.
</Info>

### Rate Limits

#### Notes

- Rates below were extracted from the functions code (withRateLimit options) and converted to a human-friendly "requests per minute (rpm)" where needed. For endpoints that do not set a per-endpoint limit the service default is used: window = 60,000 ms, maxPerWindow = 30 -> 30 rpm.
- Where the configured window is longer than 60s we show both the configured limit (max requests per window) and the equivalent rpm in the table.

#### User Management & Authentication

| Endpoint | Configured limit | Equivalent (rpm) |
|---|---:|---:|
| [`/onboarduser`](/api-reference/user-management#onboard-a-new-user-with-bvn-verification) (user registration) | 100 requests / 60,000 ms | 100 rpm |
| [`/onboardstatus`](/api-reference/user-management#check-the-current-status-of-a-users-onboarding-request) (user status) | 200 requests / 60,000 ms | 200 rpm |
| [`/getuserdetails`](/api-reference/user-management#retrieve-detailed-information-about-a-user-account) | 100 requests / 60,000 ms | 100 rpm |
| [`/manageuserstatus`](/api-reference/user-management#activate-or-deactivate-a-user-account) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/listfintechusers`](/api-reference/user-management#list-all-fintech-users) | 100 requests / 60,000 ms | 100 rpm |

#### Virtual Accounts & Wallet Operations

| Endpoint | Configured limit | Equivalent (rpm) | Notes |
|---|---:|---:|---|
| [`/getvirtualaccount`](/api-reference/virtual-accounts#retrieve-a-specific-virtual-account-with-details-about-amounts-and-fees) | 100 requests / 600,000 ms | 10 rpm | configured as 100 requests per 10 minutes |
| [`/getfintechvirtualaccount`](/api-reference/virtual-accounts#retrieve-the-ngn-virtual-account-assigned-to-your-fintech) | default -> 30 requests / 60,000 ms | 30 rpm | fintech onramp account retrieval |

#### Wallet Management

| Endpoint | Configured limit | Equivalent (rpm) |
|---|---:|---:|
| [`/getfintechwallet`](/api-reference/wallet#retrieve-the-fintech-smart-wallet-and-all-external-wallets) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/addexternalwallet`](/api-reference/wallet#add-an-external-wallet) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/updateexternalwalletstatus`](/api-reference/wallet#update-external-wallet-status) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/removeexternalwallet`](/api-reference/wallet#remove-an-external-wallet) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/listuserwallets`](/api-reference/wallet#retrieve-user-wallets) | 100 requests / 60,000 ms | 100 rpm |
| [`/migrateuserwallets`](/api-reference/wallet#migrate-user-wallets) | default -> 30 requests / 60,000 ms | 30 rpm |

#### Transactions & Asset Management

| Endpoint | Configured limit | Equivalent (rpm) | Notes |
|---|---:|---:|---|
| [`/cngnonramp`](/api-reference/transactions#user-onramp-user-wallet-funding) | 1000 requests / 600,000 ms | 100 rpm | large batch/funding flows often use 10 minute windows |
| [`/cngnofframp`](/api-reference/transactions#user-offramp-payout-to-user-bank-account) | 1000 requests / 600,000 ms | 100 rpm |
| [`/cngnonrampstatus`](/api-reference/transactions#cngn-onramp-status) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/cngnofframpstatus`](/api-reference/transactions#cngn-offramp-status) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/withdrawasset`](/api-reference/transactions#user-token-withdrawal) | 30 requests / 60,000 ms | 30 rpm |
| [`/fintechtransfer`](/api-reference/transactions#fintech-token-withdrawal) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/initiateofframp`](/api-reference/transactions#fintech-offramping) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/getofframpstatus`](/api-reference/transactions#fintech-offramp-status) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/deposits`](/api-reference/transactions#list-deposits) | 100 requests / 60,000 ms | 100 rpm | list deposit transactions |
| [`/payouts`](/api-reference/transactions#list-payouts) | 100 requests / 60,000 ms | 100 rpm | list payout transactions |
| [`/transactions`](/api-reference/transactions#list-transactions) | 100 requests / 60,000 ms | 100 rpm | list all transactions |

#### Fintech Fee Management

The following endpoints allow fintechs to manage and retrieve accumulated fees, request withdrawals of collected fees, and configure fee rates for onramp/offramp flows.

| Endpoint | Configured limit | Equivalent (rpm) |
|---|---:|---:|
| [`/getwithdrawalhistory`](/api-reference/fee-management-api#retrieve-fintech-fee-withdrawal-history) (fintech-withdrawalhistory) | 100 requests / 60,000 ms | 100 rpm |
| [`/feewithdrawal`](/api-reference/fee-management-api#request-withdrawal-of-collected-fintech-fees-to-a-destination-bank-account) (fintech-requestwithdrawal) | 30 requests / 60,000 ms | 30 rpm |
| [`/getaccumulatedfees`](/api-reference/fee-management-api#get-a-summary-of-accumulated-fees-and-available-balance-for-withdrawal) (accumulated-fintech-fees) | 60 requests / 60,000 ms | 60 rpm |
| [`/managefees`](/api-reference/fee-management-api#configure-your-fee-settings) (manage-fees) | 30 requests / 60,000 ms | 30 rpm |
| [`/getfees`](/api-reference/fee-management-api#retrieve-the-current-fee-configuration-and-an-example-calculation-for-typical-amounts) (get-fees) | 60 requests / 60,000 ms | 60 rpm |
| [`/fees/strails/preview`](/api-reference/fee-management-api#preview-strails-fee-for-onramp-or-offramp-transactions) (strails-fee-preview) | 60 requests / 60,000 ms | 60 rpm |
| [`/verify-withdrawal`](/api-reference/fee-management-api#verify-the-status-or-outcome-of-a-withdrawal-previously-requested-via-fintechrequestwithdrawal) (verify-withdrawal) | 30 requests / 60,000 ms | 30 rpm |

#### Bank Account Management

The following endpoints allow fintechs to manage Nigerian bank accounts for offramp (fiat payout) operations.

| Endpoint | Configured limit | Equivalent (rpm) |
|---|---:|---:|
| [`/getbankscode`](/api-reference/fiat-payout-management#retrieve-supported-banks-nigeria) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/addbankaccount`](/api-reference/fiat-payout-management#add-a-new-nigerian-bank-account-for-fintech-payout) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/updatebankaccount`](/api-reference/fiat-payout-management#update-bank-account) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/deletebankaccount`](/api-reference/fiat-payout-management#delete-bank-account) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/getbankaccounts`](/api-reference/fiat-payout-management#retrieve-single-bank-account) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/listbankaccounts`](/api-reference/fiat-payout-management#retrieve-all-bank-accounts) | default -> 30 requests / 60,000 ms | 30 rpm |

#### FX Orderbook & Trading Management

The following endpoints allow fintechs to create liquidity orders and execute FX trades.

| Endpoint | Configured limit | Equivalent (rpm) | Notes |
|---|---:|---:|---|
| [`/fx/orders`](/api-reference/orderbook-management#create-limit-order) | 30 requests / 60,000 ms | 30 rpm | create orderbook order |
| [`/fx/orders/:orderId`](/api-reference/orderbook-management#update-limit-order) | 30 requests / 60,000 ms | 30 rpm | update orderbook order |
| [`/fx/orders/:orderId`](/api-reference/orderbook-management#delete-limit-order) | 30 requests / 60,000 ms | 30 rpm | delete orderbook order |
| [`/fx/orders/my-orders`](/api-reference/orderbook-management#list-my-limit-orders) | 100 requests / 60,000 ms | 100 rpm | list your orders |
| [`/fx/orderbook`](/api-reference/orderbook-management#get-orderbook) | 100 requests / 60,000 ms | 100 rpm | get public orderbook |
| [`/fx/orderbook/stats`](/api-reference/orderbook-management#get-orderbook-stats) | 60 requests / 60,000 ms | 60 rpm | orderbook statistics |
| [`/fx/orderbook-token`](/api-reference/orderbook-management#get-orderbook-token) | 30 requests / 60,000 ms | 30 rpm | generate Firebase token |
| [`/fx/quote`](/api-reference/trading-management#get-quote) | 60 requests / 60,000 ms | 60 rpm | generate trade quote |
| [`/fx/trade`](/api-reference/trading-management#execute-trade) | 30 requests / 60,000 ms | 30 rpm | execute trade |
| [`/fx/market-order`](/api-reference/trading-management#create-market-order) | 30 requests / 60,000 ms | 30 rpm | execute market order |
| [`/fx/trades/:tradeId`](/api-reference/trading-management#get-trade-status) | 60 requests / 60,000 ms | 60 rpm | get trade status |
| [`/v1/fx/trades`](/api-reference/trading-management#list-trades) | 100 requests / 60,000 ms | 100 rpm | list your trades |
| [`/swaptrigger`](/api-reference/transactions#swap-trigger) | 30 requests / 60,000 ms | 30 rpm | trigger async token swap |
| [`/swap`](/api-reference/transactions#user-swap) | 30 requests / 60,000 ms | 30 rpm | swap on user's smart wallet |
| [`/swapstatus`](/api-reference/transactions#swap-status) | 60 requests / 60,000 ms | 60 rpm | get swap request status |

#### Configuration & Management

| Endpoint | Configured limit | Equivalent (rpm) |
|---|---:|---:|
| [`/fx/mpc/register`](/api-reference/fx-setting#register-mpc-vault) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/fx/settings`](/api-reference/fx-setting#get-fx-settings) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/fx/auto-signing/enable`](/api-reference/fx-setting#enable-auto-signing) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/fx/auto-signing/disable`](/api-reference/fx-setting#disable-auto-signing) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/fx/auto-signing/threshold`](/api-reference/fx-setting#set-auto-signing-threshold) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/fx/auto-signing/status`](/api-reference/fx-setting#get-auto-signing-status) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/fx/auto-signing/stats`](/api-reference/fx-setting#get-auto-signing-stats) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/manageipallowlist`](/api-reference/management-api#manage-ip-addresses-allowed-to-access-your-api) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/regenerateapikey`](/api-reference/management-api#generate-a-new-api-key) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/updateonrampasset`](/api-reference/management-api) | default -> 30 requests / 60,000 ms | 30 rpm |
| [`/autosigning/config`](/api-reference/management-api) | default -> 30 requests / 60,000 ms | 30 rpm |

#### Webhooks & Third-Party Integration

| Endpoint | Configured limit | Equivalent (rpm) |
|---|---:|---:|
| [`/setwebhook`](/api-reference/management-api#configure-webhook-urls) | 10 requests / 60,000 ms | 10 rpm |
| [`/togglewebhookstatus`](/api-reference/management-api#toggle-webhook-status) | 10 requests / 60,000 ms | 10 rpm |
| [`/getwebhook`](/api-reference/management-api#retrieve-webhook-configuration) | 10 requests / 60,000 ms | 10 rpm |




### Handling Rate Limits

When you exceed rate limits, the API returns a `429 Too Many Requests` response:

```json
{
  "status": "error",
  "response_code": "429",
  "message": "Too many requests. Please try again later.",
  "data": {
    "error": "Rate limit exceeded",
    "retry_after": 60
  }
}
```

## Error Handling

### Standard Error Response

```json
{
  "status": "error",
  "response_code": "400",
  "message": "Human-readable error message",
  "data": {
    "error": "Validation failed",
    "details": {
      "field": "Additional error details"
    }
  }
}
```

### Common Error Codes

The following error codes apply globally across all API endpoints:

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `AUTHENTICATION_FAILED` | 401 | Invalid or missing API key |
| `AUTHORIZATION_FAILED` | 403 | Insufficient permissions |
| `VALIDATION_ERROR` | 400 | Invalid request parameters |
| `NOT_FOUND` | 404 | Resource not found |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |

<Info>
**Complete Reference**: For a comprehensive list of all status codes, error codes, and state transitions, see the [API Status & Error Reference](/troubleshooting).

**API-Specific Errors**: Each API also has domain-specific error codes documented in their respective sections:
- [Orderbook Management](/api-reference/orderbook-management)
- [Trading Management - Common Errors](/api-reference/trading-management#common-errors)
</Info>

## Security Best Practices

### API Key Management

1. **Store Securely**: Never commit API keys to version control
2. **Use Environment Variables**: Store keys in environment variables
3. **Rotate Regularly**: Rotate API keys periodically
4. **Limit Scope**: Use minimal required permissions for who or what service should have access to your keys.

### Request Security

1. **Use HTTPS**: Always use secure connections
2. **Validate Responses**: Check response status and structure
3. **Implement Timeouts**: Set reasonable request timeouts
4. **Log Carefully**: Avoid logging sensitive data

## Public Key / Encryption (store-public-key)

You may opt into an encryption feature where sensitive payloads are encrypted before being sent to Strails. To enable this, Use `/storepublickey` endpoint. This allows Strails to store the fintech's public key material for encrypting responses or for verifying encrypted requests.

### Endpoint

```http
POST /storepublickey
```

### Request body (example)

```json
{
  "data": "<hex-encoded-ciphertext>"
}
```
