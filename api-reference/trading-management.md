---
title: "FX Trading Operations API"
sidebarTitle: "Trading Management"
description: "REST API endpoints for FX trading operations including quote generation, trade execution, and trade status tracking."
---

This section documents endpoints fintechs use to **generate quotes, execute trades, and track trade status** for FX trading.

<Info>
All endpoints require fintech API key authentication and IP allowlist verification.
</Info>

---

## FX Trading Operations API — Endpoints Overview

| Section                                    | Endpoint              | Method | Description                                                       |
| ------------------------------------------ | --------------------- | ------ | ----------------------------------------------------------------- |
| [**Get Quote**](#get-quote)                | `/fx/quote`           | POST   | Get a price quote for trading with matched orders from orderbook  |
| [**Execute Trade**](#execute-trade)        | `/fx/trade`           | POST   | Execute a trade using quote ID, direct params, or targeted order ID |
| [**Create Market Order**](#create-market-order) | `/fx/market-order` | POST   | Execute an immediate market order at current best price           |
| [**Get Trade Status**](#get-trade-status)  | `/fx/trades/status` | GET    | Get the current status of a trade                                 |
| [**List Trades**](#list-trades)            | `/fx/trades`          | GET    | List your fintech's trades with optional filters                  |

---

### Get Quote

Get a price quote for trading. Returns matched orders from the orderbook with pricing and liquidity information. Quote is valid for 5 minutes.

<Tabs>

<Tab title="Request (Get Quote)">

```http
POST {{BASE_URL}}/fx/quote
```

</Tab>

<Tab title="Body (Get Quote)">

```json
{
  "pair": "CNGN-USDC",
  "side": "sell",
  "cngnAmount": "50000"
}
```

### Parameters

| Parameter    | Type   | Required | Description                                                                     |
|--------------|--------|----------|---------------------------------------------------------------------------------|
| `pair`       | string | Yes   | Trading pair in format `CNGN-TOKEN` (`CNGN-USDT` or `CNGN-USDC`)                |
| `side`       | string | Yes   | Order side: `buy` to buy stablecoins with cNGN, `sell` to sell for cNGN        |
| `cngnAmount` | string | Yes   | Amount in cNGN (human-readable with decimals). Minimum: 1000 cNGN              |

</Tab>

<Tab title="Response (Get Quote)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Quote generated successfully",
  "data": {
    "quote": {
      "quoteId": "quote_7f3a9d2c-8e41-4b29-9a1c-5d8e7f2b3a9c",
      "pair": "CNGN-USDC",
      "side": "sell",
      "cngnAmount": "50000",
      "usdcAmount": "31.25",
      "price": "1600.00",
      "spread": 0.5,
      "matchedOrders": [
        {
          "orderId": "order_abc123",
          "price": "1600.00",
          "cngnAmount": "50000",
          "usdcAmount": "31.25",
          "spread": 0.5
        }
      ],
      "expiresAt": "2026-04-04T10:30:00.000Z"
    },
    "version": "1.0.0"
  }
}
```

</Tab>

<Tab title="Error (Get Quote)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "No liquidity available for USDC",
  "error": "No active orders in orderbook for the requested pair"
}
```

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Invalid request data",
  "error": "cngnAmount must be at least 1000"
}
```

</Tab>

</Tabs>

---

### Execute Trade

Execute a trade using a quote ID or direct trade parameters. Initiates the FX trade flow: requested → locked → signed → settling → completed.

<Info>
There are three execution modes:
- **Mode 1 (Quote):** Use `quoteId` from `/fx/quote` for price consistency
- **Mode 2 (Direct):** Provide `pair`, `side`, `cngnAmount` for current market price
- **Mode 3 (Targeted):** Use `orderId` to execute against a specific LP order at exact price

Ensure the cNGN to trade is in your Fintech Smart Wallet [`/getfintechwallet`](/api-reference/wallet#retrieve-the-fintech-smart-wallet-and-all-external-wallets)
</Info>

<Tabs>

<Tab title="Request (Execute Trade)">

```http
POST {{BASE_URL}}/fx/trade
```

</Tab>

<Tab title="Body (Execute Trade)">

**Mode 1: Execute with quote ID:**
```json
{
  "quoteId": "quote_7f3a9d2c-8e41-4b29-9a1c-5d8e7f2b3a9c",
  "idempotencyKey": "trade_20260404_001"
}
```

**Mode 2: Market order (best-price):**
```json
{
  "pair": "CNGN-USDC",
  "side": "sell",
  "cngnAmount": "50000",
  "destinationWalletAddress":"receiver-wallet"
}
```

**Mode 3: Targeted order (exact LP price):**
```json
{
  "orderId": "order_abc123",
  "cngnAmount": "5000"
}
```

Execute against a specific LP order. Use this when you want to trade at an exact LP's price without going through the quote flow.

### Parameters

| Parameter        | Type   | Required | Description                                                          |
|------------------|--------|----------|----------------------------------------------------------------------|
| `quoteId`        | string | No    | Quote ID from `/fx/quote` endpoint. If provided, other params retrieved from quote |
| `orderId`        | string | No    | Order ID to execute against directly (for targeted execution)        |
| `pair`           | string | No    | Trading pair (required if quoteId/orderId not provided)              |
| `side`           | string | No    | Order side (required if quoteId/orderId not provided)                |
| `cngnAmount`     | string | No    | cNGN amount (required if quoteId not provided)                       |
| `destinationWalletAddress` | string | No    | Wallet address to receive the traded tokens                          |
| `idempotencyKey` | string | No    | Optional key to prevent duplicate trades                             |

</Tab>

<Tab title="Response (Execute Trade)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Trade executed successfully",
  "data": {
    "trade": {
      "tradeId": "trade_abc123",
      "orderId": "order_abc123",  // Only present for targeted execution
      "pair": "CNGN-USDC",
      "side": "sell",
      "cngnAmount": "50000",
      "usdcAmount": "31.25",
      "price": "1600.00",
      "strailsFeePercentage": 20,
      "status": "pending",
      "lockId": "lock_xyz789",
      "expiresAt": "2026-04-04T10:35:00.000Z",
      "createdAt": "2026-04-04T10:30:00.000Z"
    },
    "version": "1.0.0"
  }
}
```

</Tab>

<Tab title="Error (Execute Trade)">

```json
{
  "status": "Error",
  "response_code": "01",
  "message": "Quote has expired or is invalid",
  "error": "Quote ID is expired (> 5 minutes old) or doesn't exist"
}
```

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "Insufficient liquidity",
  "error": "Orderbook doesn't have enough liquidity for the requested amount"
}
```

```json
{
  "status": "Error",
  "response_code": "05",
  "message": "Self trade not allowed",
  "error": "Cannot trade against your own orders"
}
```

</Tab>

</Tabs>

---

### Get Trade Status

Get the current status of a trade. Use this to track trade progress through the execution flow.

<Tabs>

<Tab title="Request (Get Trade Status)">

```http
GET {{BASE_URL}}/fx/trades/status?tradeId=trade_abc123
```

### Parameters

| Parameter | Type   | Required | Location | Description                          |
|-----------|--------|----------|----------|--------------------------------------|
| `tradeId` | string | Yes   | Params or Query | Trade ID from execute trade response |

</Tab>

<Tab title="Response (Get Trade Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Trade status retrieved successfully",
  "data": {
    "trade": {
      "tradeId": "trade_abc123",
      "pair": "CNGN-USDC",
      "side": "sell",
      "cngnAmount": "50000",
      "usdcAmount": "31.25",
      "price": "1600.00",
      "fintechNetAmount": "16.00",
      "status": "completed",
      "lockId": "lock_xyz789",
      "createdAt": "2026-04-04T10:30:00.000Z",
      "priceLockExpiresAt": "2026-04-04T10:31:20.000Z",
      "errorMessage": null,
      "failedAt": null
    },
    "version": "1.0.0"
  }
}
```

</Tab>

<Tab title="Error (Get Trade Status)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "Trade not found or access denied"
}
```

</Tab>

</Tabs>

---

### List Trades

List your fintech's trades with optional filters. Returns paginated list of trades.

<Tabs>

<Tab title="Request (List Trades)">

```http
GET {{BASE_URL}}/fx/trades?status=completed&limit=50
```

### Query Parameters

| Parameter    | Type   | Required | Description                                                          |
|--------------|--------|----------|----------------------------------------------------------------------|
| `pair`       | string | No    | Filter by trading pair (`CNGN-USDT` or `CNGN-USDC`)                  |
| `side`       | string | No    | Filter by order side (`buy` or `sell`, case-insensitive)             |
| `status`     | string | No    | Filter by trade status (case-insensitive)                            |
| `limit`      | number | No    | Maximum number of trades to return (default: 50, max: 100)           |
| `startAfter` | string | No    | Trade ID to start pagination after                                   |

</Tab>

<Tab title="Response (List Trades)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Trades retrieved successfully",
  "data": {
    "trades": [
      {
        "tradeId": "trade_abc123",
        "pair": "CNGN-USDC",
        "side": "sell",
        "cngnAmount": "50000",
        "usdcAmount": "31.25",
        "price": "1600.00",
        "fintechNetAmount": "16.00",
        "status": "completed",
        "createdAt": "2026-04-04T10:30:00.000Z"
      }
    ],
    "count": 10,
    "version": "1.0.0"
  }
}
```

</Tab>

</Tabs>

---

### Create Market Order

Execute an immediate market order at the current best available price. Market orders are matched against existing limit orders in the orderbook.

<Info>
**Market Order Execution:**
- Executes immediately at best available price from limit orders
- Automatically handles settlement based on your auto-signing configuration
- Trade is locked in escrow during settlement process
- Your wallet must have sufficient balance for the trade
</Info>

<Tabs>

<Tab title="Request (Create Market Order)">

```http
POST {{BASE_URL}}/fx/market-order
```

</Tab>

<Tab title="Body (Create Market Order)">

```json
{
  "pair": "CNGN-USDC",
  "side": "buy",
  "cngnAmount": "1000",
  "destinationWalletAddress": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
  "idempotencyKey": "unique-key-12345"
}
```

### Parameters

| Parameter                  | Type   | Required | Description                                                                 |
|----------------------------|--------|----------|-----------------------------------------------------------------------------|
| `pair`                     | string | Yes   | Trading pair (`CNGN-USDC`, `CNGN-USDT`, or `CNGN`)                        |
| `side`                     | string | Yes   | Order side: `buy` to buy tokens with cNGN, `sell` to sell tokens for cNGN |
| `cngnAmount`               | string | Yes   | Amount in cNGN to trade (human-readable format, e.g., `1000`)              |
| `destinationWalletAddress` | string | No    | Destination wallet address for the trade settlement                        |
| `idempotencyKey`           | string | No    | Unique key to prevent duplicate order execution                            |

</Tab>

<Tab title="Response (Create Market Order)">

```json
{
  "responseCode": "00",
  "responseMessage": "Market order executed successfully",
  "data": {
    "trade": {
      "tradeId": "trade_xyz789",
      "pair": "CNGN-USDC",
      "side": "buy",
      "cngnAmount": "1000.00",
      "usdcAmount": "0.714285",
      "averagePrice": "1400.00",
      "status": "pending",
      "expiresAt": "2026-04-24T12:30:00.000Z",
      "createdAt": "2026-04-24T12:28:40.000Z"
    },
    "version": "1.0.0",
    "note": "Market order executes at current best price. Settlement proceeds automatically based on your auto-signing configuration."
  }
}
```

</Tab>

</Tabs>

---

### Trade Status Lifecycle

Trades follow a specific status lifecycle:

| Status     | Description                                                  |
|------------|--------------------------------------------------------------|
| `pending`  | Trade created, validating parameters                         |
| `locked`   | Tokens locked in escrow, 5-minute countdown active           |
| `signing`  | Awaiting MPC signature for blockchain transaction            |
| `settling` | Transaction submitted to blockchain, awaiting confirmation   |
| `completed`| Trade successfully completed, tokens transferred             |
| `failed`   | Trade failed (check `errorMessage` field)                    |
| `expired`  | Lock expired after 5 minutes without completion              |

### Status Flow

```
pending → locked → signing → settling → completed
                          ↘ failed      ↘ failed
         locked → expired (after 5 minutes)
         signing → expired (only after MPC signing completes/fails)
```

---

### Common Errors

| Code | Error                  | Description                              |
|------|------------------------|------------------------------------------|
| `01` | VALIDATION_ERROR       | Invalid request data                     |
| `01` | INVALID_QUOTE          | Quote has expired or is invalid          |
| `02` | NO_LIQUIDITY           | Insufficient liquidity in orderbook      |
| `02` | NOT_FOUND              | Trade not found or access denied         |
| `03` | AUTHENTICATION_FAILED  | Invalid or missing API key               |
| `03` | IP_BLOCKED             | Request from unauthorized IP address     |
| `04` | RATE_LIMIT_EXCEEDED    | Too many requests                        |
| `05` | INTERNAL_ERROR         | Internal server error                    |

---

### Notes

<Info>
**Important information about FX trading:**
- All amounts use 6 decimal places for cNGN and stablecoins
- Token amount field names are dynamic based on the trading pair (`usdcAmount`, `usdtAmount`, etc.)
- Quotes expire after 5 minutes
- Trade locks expire after 5 minutes
- All timestamps are in ISO 8601 format (UTC)
- Status and side parameters are case-insensitive in query strings
</Info>
