---
title: "Management & Configuration API"
sidebarTitle: "Management & Config"
description: "Management and configuration API endpoints for webhooks, security, and asset management"
---

This section documents endpoints fintechs use to **manage webhooks, IP allowlists, asset preferences, and API keys**.

---

## Management & Configuration API — Endpoints Overview

| Section                                                      | Endpoint              | Method | Description                                                          |
| ------------------------------------------------------------ | --------------------- | ------ | -------------------------------------------------------------------- |
| [**Generate a new API key**](#generate-a-new-api-key)        | `/regenerateapikey`   | GET    | Generate a new API key for your fintech account.                     |
| [**Configure Webhook URLs**](#configure-webhook-urls)        | `/setwebhook`         | POST   | Configure or update webhook URLs and settings for notifications.     |
| [**Toggle Webhook Status**](#toggle-webhook-status) | `/togglewebhookstatus` | POST | Enable or disable webhook delivery without changing URL or secret. |
| [**Retrieve Webhook Configuration**](#retrieve-webhook-configuration) | `/getwebhook` | GET    | Retrieve the current webhook configuration and settings.             |
| [**Manage IP Allowlist**](#manage-ip-addresses-allowed-to-access-your-api) | `/manageipallowlist` | POST   | Manage IP addresses allowed to access your API (list, add, remove, check). |
| [**Update User Asset Preferences**](#update-user-asset-preferences) | `/updateonrampasset` | POST   | Update user onramp asset preferences, such as token to buy and token to swap to before request is finalized. |
| [**Auto-Signing of cNGN Onramp**](#auto-signing-of-cngn-onramp) | `/autosigning/config` | GET | Get auto approval threshold assigned to fintech when buying cNGN.    |

---

### Generate a new API key

<Tabs>
<Tab title="Request (Regenerate Key)">

```http
GET {{BASE_URL}}/regenerateapikey
```

</Tab>

<Tab title="Response (Regenerate Key)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "API key regenerated successfully",
  "data": {
    "apiKey": "key-xxxxxxxxxxxxxxxxx",
    "rotatedAt": "2025-08-26T10:20:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

### Configure webhook URLs

<Tabs>
<Tab title="Request (Set Webhook)">

```http
POST {{BASE_URL}}/setwebhook
```

</Tab>

<Tab title="Body (Set Webhook)">

```json
{
  "webhookUrl": "https://your-domain.com/webhook",
  "secret": "your_webhook_secret_key",
  "enabled": true
}
```

### Parameters

| Parameter      | Type    | Required | Description |
|----------------|---------|----------|-------------|
| `webhookUrl`   | string  | Yes   | URL where webhook events will be sent |
| `secret`       | string  | No    | Secret key used to verify webhook signatures |
| `enabled`      | boolean | Yes   | Enable or disable the webhook |

</Tab> 

<Tab title="Response (Set Webhook)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Webhook configured successfully",
  "data": {
    "webhookUrl": "https://your-domain.com/webhook",
    "enabled": true,
    "hasSecret": true,
    "urlValidated": true,
    "updatedAt": "2025-08-26T11:00:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

### Toggle webhook status

<Tabs>
<Tab title="Request (Toggle Status)">

```http
POST {{BASE_URL}}/togglewebhookstatus
```

</Tab>

<Tab title="Body (Toggle Status)">

```json
{
  "enabled": true
}
```

### Parameters

| Parameter | Type    | Required | Description |
|-----------|---------|----------|-------------|
| `enabled` | boolean | Yes   | Set to `true` to enable webhook delivery or `false` to disable it |

</Tab>

<Tab title="Response (Toggle Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Webhook activated successfully",
  "data": {
    "webhookUrl": "htt***com/webhook", 
    "enabled": true,
    "updatedAt": "2026-06-23T12:00:00.000Z"
  }
}
```

</Tab>
</Tabs>

---

### Retrieve webhook configuration

<Tabs>
<Tab title="Request (Get Webhook)">

```http
GET {{BASE_URL}}/getwebhook
```

</Tab>

<Tab title="Response (Get Webhook)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Webhook configuration retrieved successfully",
  "data": {
    "webhookUrl": "https://sgh***.ms/webhook",
    "enabled": true,
    "hasSecret": true,
    "updatedAt": "2026-01-29T05:11:06.862Z"
  }
}
```

</Tab>
</Tabs>

---

### Manage IP addresses allowed to access your API

Perform operations on your API IP allowlist: **list**, **add**, **remove**, or **check** IP addresses.

<Tabs>
<Tab title="List IPs">

### Request

```http
POST {{BASE_URL}}/manageipallowlist
```

### Body

```json
{
  "action": "list"
}
```

### Parameters

| Parameter | Type   | Required | Description |
|-----------|--------|----------|-------------|
| `action`  | string | Yes   | Operation: `list` |

### Response

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "IP allowlist operation completed",
  "data": {
    "action": "list",
    "ipAddresses": [
      {
        "index": 0,
        "ip": "203.0.***.***",
        "description": "Office network"
      },
      {
        "index": 1,
        "ip": "192.168.***.***",
        "description": "VPN endpoint"
      }
    ],
    "totalIps": 2
  }
}
```

</Tab>

<Tab title="Add IP">

### Request

```http
POST {{BASE_URL}}/manageipallowlist
```

### Body

```json
{
  "action": "add",
  "ipAddress": "203.0.113.45",
  "description": "Office network IP"
}
```

### Parameters

| Parameter     | Type   | Required | Description |
|---------------|--------|----------|-------------|
| `action`      | string | Yes   | Operation: `add` |
| `ipAddress`   | string | Yes   | IP address to add to allowlist |
| `description` | string | No    | Optional note describing the IP |

### Response

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "IP allowlist operation completed",
  "data": {
    "action": "add",
    "ipAddress": "203.0.***.***",
    "totalIps": 4,
    "message": "IP address added successfully. Changes will take effect within 30 seconds across all servers.",
    "propagationTime": "30 seconds"
  }
}
```

</Tab>

<Tab title="Remove IP (by index)">

### Request

```http
POST {{BASE_URL}}/manageipallowlist
```

### Body

```json
{
  "action": "remove",
  "index": 1
}
```

### Parameters

| Parameter | Type   | Required | Description |
|-----------|--------|----------|-------------|
| `action`  | string | Yes   | Operation: `remove` |
| `index`   | number | Yes   | Index of IP to remove (from list response) |

### Response

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "IP allowlist operation completed",
  "data": {
    "action": "remove",
    "removed": "IP at index 1",
    "totalIps": 2,
    "message": "IP address removed successfully. Changes will take effect within 30 seconds across all servers.",
    "propagationTime": "30 seconds"
  }
}
```

</Tab>

<Tab title="Remove IP (by address)">

### Request

```http
POST {{BASE_URL}}/manageipallowlist
```

### Body

```json
{
  "action": "remove",
  "ipAddress": "10.0.0.5"
}
```

### Parameters

| Parameter   | Type   | Required | Description |
|-------------|--------|----------|-------------|
| `action`    | string | Yes   | Operation: `remove` |
| `ipAddress` | string | Yes   | IP address to remove from allowlist |

### Response

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "IP allowlist operation completed",
  "data": {
    "action": "remove",
    "removed": "203.0.***.***",
    "totalIps": 2,
    "message": "IP address removed successfully. Changes will take effect within 30 seconds across all servers.",
    "propagationTime": "30 seconds"
  }
}
```

</Tab>

<Tab title="Check IP">

### Request

```http
POST {{BASE_URL}}/manageipallowlist
```

### Body

```json
{
  "action": "check"
}
```

### Parameters

| Parameter | Type   | Required | Description |
|-----------|--------|----------|-------------|
| `action`  | string | Yes   | Operation: `check` |

### Response

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "IP allowlist operation completed",
  "data": {
    "action": "check",
    "yourIp": "192.168.***.***",
    "isAllowed": true,
    "matchedRule": "192.168.***.***",
    "totalConfiguredIps": 3,
    "note": "Your IP is allowed. Cache refreshed on this server instance.",
    "cacheInfo": {
      "ttlSeconds": 30,
      "propagationTime": "Changes propagate to other servers within 30 seconds",
      "cacheRefreshed": true
    }
  }
}
```

</Tab>
</Tabs>

---

### Update user asset preferences

<Tabs>
<Tab title="Request (Assets)">

```http
POST {{BASE_URL}}/updateonrampasset
```

</Tab>

<Tab title="Body (Assets)">

```json
{
  "userId": "user_hash",
  "preferences": {
    "defaultAsset": "USDC",
    "autoSwap": true,
    "slippageTolerance": "0.5"
  }
}
```

### Parameters

| Parameter       | Type   | Required | Description |
|-----------------|--------|----------|-------------|
| `userId`        | string | Yes   | Unique identifier (hash/UUID) of the user |
| `preferences`   | object | No    | User asset preferences |

#### `preferences` Object

| Field               | Type    | Required | Description |
|--------------------|---------|----------|-------------|
| `defaultAsset`      | string  | No    | Asset symbol set as default for swaps or funding (e.g., `USDC`) |
| `autoSwap`          | boolean | No    | Automatically swap deposited assets into `defaultAsset` |
| `slippageTolerance` | string  | No    | Maximum allowable slippage for swaps (e.g., `"0.5"` for 0.5%) |

</Tab>
<Tab title="Response (Assets)">

```json
{
  "status": "00",
  "message": "Asset settings updated successfully",
  "data": {
    "requestId": "op_uuid",
    "updatedAt": "2025-08-26T09:58:30.000Z",
    "autoSwap": true
  }
}
```

</Tab>
</Tabs>

---

### Auto-Signing of cNGN OnRamp

<Tabs>
<Tab title="Request (Auto-Signing)">

```http
GET {{BASE_URL}}/autosigning/config
```

### Query Parameters

| Parameter   | Type   | Required | Description                                   |
|-------------|--------|----------|-----------------------------------------------|
| `fintechId` | string | Yes   | Unique identifier (hash/UUID) of the fintech |

</Tab>
<Tab title="Response (Assets)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Configuration retrieved successfully",
  "data": {
    "enabled": true,
    "thresholdAmount": "***",
    "currency": "CNGN",
    "maxDailyAmount": "100000000",
    "restrictToFintech": true
  }
}
```

</Tab>
</Tabs>