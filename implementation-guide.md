---
title: "Implementation Guide"
description: Step-by-step guide to integrating Strails API into your application
---

This guide walks you through integrating the Strails API from initial setup to processing your first transaction.

---

## Prerequisites

Before you begin, ensure you have:

1. **API Key** - Received via secure onboarding email from Strails
2. **Development Environment** - Staging endpoint for testing
3. **Webhook Endpoint** - (Optional but recommended) HTTPS endpoint to receive notifications
4. **Test Data** - Valid Nigerian BVN for testing user onboarding

<Info>
**New to Strails?** Contact [support@strails.io](mailto:support@strails.io) to start the onboarding process and receive your API credentials.
</Info>

---

## Step 1: Test Your API Connection

Start by verifying your API key works correctly.

### Get Your Fintech Wallet

```bash
curl -X GET https://beta.stablesrail.io/v1/getfintechwallet \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

**Expected Response:**
```json
{
  "status": "success",
  "response_code": "00",
  "message": "Fintech wallet retrieved successfully",
  "data": {
    "smartWallet": "0x1234...abcd",
    "mpcVault": "0x5678...efgh",
    "managedWallet": "0xabcd...1234"
  }
}
```

<Check>
**Success!** If you see your wallet addresses, your API key is working correctly.
</Check>

<Warning>
**Authentication Failed?** Double-check your API key and ensure you're using the correct header: `x-api-key` (not `X-API-Key` or `api-key`).
</Warning>

---

## Step 2: Configure Webhook (Recommended)

Set up your webhook endpoint to receive real-time notifications about transaction status.

### Set Webhook URL

```bash
curl -X POST https://beta.stablesrail.io/v1/setwebhook \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "webhookUrl": "https://yourapp.com/webhooks/strails",
    "events": ["all"]
  }'
```

### Webhook Handler Example (Node.js)

```javascript
const express = require('express');
const crypto = require('crypto');

app.post('/webhooks/strails', (req, res) => {
  const signature = req.headers['x-strails-signature'];
  const timestamp = req.headers['x-strails-timestamp'];
  
  // Verify signature
  const payload = JSON.stringify(req.body);
  const expectedSignature = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(`${timestamp}.${payload}`)
    .digest('hex');
  
  if (signature !== expectedSignature) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process webhook event
  const { event, data } = req.body;
  console.log(`Received event: ${event}`, data);
  
  // Store event for processing
  // Update user status in your database
  // Send notifications to users
  
  res.status(200).send('OK');
});
```

**See also:** [Webhook Events Reference](/api-reference/webhook-events)

---

## Step 3: Onboard Your First User

### 3.1 Onboard User with BVN Verification

```bash
curl -X POST https://beta.stablesrail.io/v1/onboarduser \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "bvn": "12345678901"
  }'
```

**Response:**
```json
{
  "status": "success",
  "response_code": "00",
  "message": "User onboarding initiated",
  "data": {
    "userId": "user_uniqueID",
    "status": "pending_verification",
    "requestId": "req_xyz789"
  }
}
```

### 3.2 Check Onboarding Status

```bash
curl -X GET "https://beta.stablesrail.io/v1/onboardstatus?userId=user_uniqueID" \
  -H "x-api-key: YOUR_API_KEY"
```

**Successful Verification:**
```json
{
  "status": "success",
  "response_code": "00",
  "data": {
    "userId": "user_uniqueID",
    "status": "verified",
    "bvnVerified": true,
    "createdAt": "2026-04-10T10:00:00Z"
  }
}
```

<Info>
**BVN Verification Time:** Typically completes in 2-5 seconds. If status is still `pending_verification`, wait a few seconds and retry.
</Info>

---

## Step 4: Fund a User Wallet

### 4.1 Initiate Onramp (Funding Request)

The `/cngnonramp` endpoint creates a wallet, generates a virtual bank account, and initiates the funding process.

```bash
curl -X POST https://beta.stablesrail.io/v1/cngnonramp \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "user_uniqueID",
    "amount": 5000,
    "assetSwap": "USDC",
    "autoSwap": true,
    "sweepToOfframp": false
  }'
```

**Response:**
```json
{
  "status": "Success",
  "response_code": "00",
  "message": "cNGN request submitted successfully",
  "data": {
    "requestId": "f16c0e3c-0c0f-473f-91f7-ab2b85bc9117",
    "walletAddress": "0xF94384E5792F44fDF231A56cea4f4C14d998e58c",
    "status": "requested",
    "message": "Ensure you pay 5075.00 naira to the bank account generated. Use requestId to get the bank account details for payment.",
    "autoSwapEnabled": true,
    "targetAsset": "USDC",
    "feeBreakdown": {
      "baseAmount": 5000,
      "totalFee": 75,
      "totalAmount": 5075
    }
  }
}
```

<Info>
**What happens next:** A virtual bank account is generated (active for 30 minutes). Use the `requestId` to retrieve account details via `/getvirtualaccount`.
</Info>

### 4.2 Get Virtual Account Details (Optional)

Retrieve the bank account details to display to your user:

```bash
curl -X POST https://beta.stablesrail.io/v1/getvirtualaccount \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "requestId": "f16c0e3c-0c0f-473f-91f7-ab2b85bc9117"
  }'
```

**Response:**
```json
{
  "status": "Success",
  "response_code": "00",
  "data": {
    "accountNumber": "9800001234",
    "bankName": "FAME Bank",
    "accountName": "Test User Strails",
    "amount": 5000,
    "totalAmountWithFee": 5075,
    "expiresAt": "2026-04-10T11:30:00Z"
  }
}
```

<Warning>
**Account Expiry:** Virtual accounts expire **30 minutes** after creation. Users must complete payment before expiry.
</Warning>

**Payment Flow:**
1. User pays ₦5,075 to the generated virtual account
2. System receives deposit → mints cNGN to user's wallet
3. Auto-swap cNGN → USDC/T via Aerodrome DEX (if `autoSwap: true`)
4. Display cNGN → USDC/T quote on onramp status
5. Execute quote returned
6. Webhook notification sent on completion
7. USDC arrives in user's wallet

### 4.3 Check Transaction Status

```bash
curl -X POST https://beta.stablesrail.io/v1/cngnonrampstatus \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "requestId": "f16c0e3c-0c0f-473f-91f7-ab2b85bc9117"
  }'
```

**Response (Completed):**
```json
{
  "status": "success",
  "response_code": "00",
  "data": {
    "requestId": "f16c0e3c-0c0f-473f-91f7-ab2b85bc9117",
    "status": "completed",
    "walletAddress": "0xF94384E5792F44fDF231A56cea4f4C14d998e58c",
    "amount": "5000000000",
    "swapCompleted": true,
    "finalAsset": "USDC",
    "txHash": "0xabcd1234...",
    "completedAt": "2026-04-10T10:35:00Z"
  }
}
```

---

## Step 5: Process User Withdrawal (Offramp)

### 5.1 Get Supported Banks

```bash
curl -X GET https://beta.stablesrail.io/v1/getbankscode \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**
```json
{
  "status": "success",
  "response_code": "00",
  "data": [
    { "bankCode": "058", "bankName": "GTBank" },
    { "bankCode": "044", "bankName": "Access Bank" },
    { "bankCode": "033", "bankName": "UBA" }
  ]
}
```

### 5.2 Initiate Offramp to User's Bank Account

```bash
curl -X POST https://beta.stablesrail.io/v1/cngnofframp \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "user_uniqueID",
    "amount": 3000,
    "accountNumber": "0123456789",
    "bankCode": "058",
    "ticker": "CNGN"
  }'
```

**Response:**
```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Vault return token transfer completed, payout processing initiated",
  "data": {
    "requestId": "req_offramp_xyz",
    "status": "payout_pending",
    "stage": "payout"
  }
}
```

### 5.3 Check Offramp Status

```bash
curl -X POST https://beta.stablesrail.io/v1/cngnofframpstatus \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "requestId": "req_offramp_xyz"
  }'
```

---

## Step 6: FX Trading (Fintech-to-Fintech)

### 6.1 Create Orderbook Entry

```bash
curl -X POST https://beta.stablesrail.io/v1/fx/orders \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "orderType": "sell",
    "baseCurrency": "cNGN",
    "quoteCurrency": "USDC",
    "price": "1350000",
    "minAmount": "500000000",
    "maxAmount": "2000000000"
  }'
```

**Response:**
```json
{
  "status": "success",
  "response_code": "00",
  "data": {
    "orderId": "order_abc123",
    "status": "active",
    "price": "1350000",
    "availableLiquidity": "2000000000"
  }
}
```

### 6.2 Get FX Quote

```bash
curl -X GET "https://beta.stablesrail.io/v1/fx/quote?baseCurrency=cNGN&quoteCurrency=USDC&amount=1000000000" \
  -H "x-api-key: YOUR_API_KEY"
```

### 6.3 Execute Trade

```bash
curl -X POST https://beta.stablesrail.io/v1/fx/trade \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "quoteId": "quote_xyz789",
    "quoteCurrency": "USDC"
  }'
```

**See also:** [FX Orderbook Guide](/api-reference/orderbook-management) | [Trading Guide](/api-reference/trading-management)

---

## Common Integration Patterns

### Pattern 1: Simple Wallet Funding
```
User onboarding → Send virtual account → User pays → cNGN minted → Done
```

### Pattern 2: Wallet Funding with Auto-Swap
```
User onboarding → Onramp with autoSwap: true → cNGN minted → Auto-swap to USDC → Done
```

### Pattern 3: Fintech Liquidity Management
```
Create FX order → Wait for match → Trade executes → Receive USDC → Escrow settles counter-party
```

---

## Testing Tips

### Use the Postman Collection

1. Download the [Postman Collection](postman-collection.json)
2. Import into Postman
3. Set environment variables:
   - `BASE_URL`: `https://beta.stablesrail.io/v1`
   - `API_KEY`: Your API key
4. Run the collection to test all endpoints

### Test with Small Amounts

Start with small amounts (₦100-500) to validate your integration before processing larger transactions.

### Monitor Webhook Logs

Use tools like [webhook.site](https://webhook.site) or [ngrok](https://ngrok.com) to test webhooks locally during development.

---

## Troubleshooting

### Authentication Errors

**Error:** `401 Unauthorized` or `403 Forbidden`

**Solutions:**
- Verify API key is correct
- Check header name is `x-api-key` (lowercase)
- Ensure your IP is in the allowlist (if enabled)

### Rate Limit Errors

**Error:** `429 Too Many Requests`

**Solutions:**
- Implement exponential backoff
- Review [rate limits](/authentication#rate-limits)
- Contact support for higher limits

### BVN Verification Fails

**Error:** `BVN verification failed`

**Solutions:**
- Verify BVN is 11 digits
- Check user details match BVN exactly

### Transaction Stuck in Pending

**Problem:** Transaction status remains `pending` for >10 minutes

**Solutions:**
1. Check transaction status via `/cngnonrampstatus`
2. Trigger manual recovery: `GET /v1/manualstatusrecovery?requestId=YOUR_REQUEST_ID`
3. Contact support if issue persists

### Webhook Not Receiving Events

**Solutions:**
- Verify webhook URL is accessible (HTTPS required)
- Check webhook signature validation logic
- Test webhook with [webhook.site](https://webhook.site)
- Review [webhook documentation](/api-reference/webhook-events)

---

## Security Best Practices

1. **Never expose API keys** - Use environment variables, never commit to git
2. **Validate webhook signatures** - Always verify HMAC signatures
3. **Use HTTPS** - All API calls and webhooks must use HTTPS
4. **Implement IP allowlisting** - Restrict API access to known IPs
5. **Store requestIds** - Log all `requestId` values for transaction tracking
6. **Handle idempotency** - Check for duplicate webhook events using `requestId`

---

## Next Steps

Now that you've completed the basic integration:

1. **Configure Fee Management** - [Set custom fees](/api-reference/fee-management-api) for your users
2. **Enable Advanced Features** - Set up [auto-swap](/api-reference/transactions#user-onramp-user-wallet-funding) and [sweep-to-offramp](/api-reference/transactions)
3. **Production Launch Checklist:**
   - [ ] Configure production webhook URL
   - [ ] Set up IP allowlist
   - [ ] Test with small real transactions
   - [ ] Monitor webhook delivery
   - [ ] Set up error alerting

---

## Support

Need help? Reach out to:

- **Email:** [support@strails.io](mailto:support@strails.io)
- **Documentation:** This site
- **API Status:** Check [authentication.md](/authentication) for service updates

---

**Last updated:** April 2026
