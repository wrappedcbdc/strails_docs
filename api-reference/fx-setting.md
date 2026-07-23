---
title: "FX Settings API"
sidebarTitle: "FX Settings"
description: "FX Settings API endpoint for retrieving fintech FX configuration and MPC vault settings"
---

The FX Settings API allows fintechs to retrieve their FX configuration including MPC vault details, auto-signing status, token wallet addresses, and notification settings.

---

## FX Settings API — Endpoints Overview

| Section                           | Endpoint                    | Method | Description                                                    |
| --------------------------------- | --------------------------- | ------ | -------------------------------------------------------------- |
| [**Register MPC Vault**](#register-mpc-vault) | `/fx/mpc/register`          | POST   | Register MPC vault configuration for FX trading operations.    |
| [**Get FX Settings**](#get-fx-settings) | `/fx/settings`              | GET    | Retrieve complete FX settings including MPC vault configuration and token wallets. |
| [**Enable Auto-Signing**](#enable-auto-signing) | `/fx/auto-signing/enable`   | POST   | Enable automatic transaction signing for FX trades.           |
| [**Disable Auto-Signing**](#disable-auto-signing) | `/fx/auto-signing/disable`  | POST   | Disable automatic transaction signing.                        |
| [**Set Auto-Signing Threshold**](#set-auto-signing-threshold) | `/fx/auto-signing/threshold`| POST   | Configure the USD threshold for auto-signing transactions.    |
| [**Get Auto-Signing Status**](#get-auto-signing-status) | `/fx/auto-signing/status`   | GET    | Retrieve auto-signing configuration and daily statistics.     |
| [**Get Auto-Signing Stats**](#get-auto-signing-stats) | `/fx/auto-signing/stats`    | GET    | Retrieve auto-signing statistics over a period of days.       |

---

## Important Notes

<Warning>
**Security Consideration**: The `mpcVaultApiKey` and `mpcClientSignerPrivateKey` are sensitive credentials. They are partially masked in the response. Store these values securely and never expose them in client-side code or logs.
</Warning>

<Info>
**MPCVault**: A third-party MPC wallet infrastructure provider and is not affiliated with Strails. To enable wallet functionality on Strails, customers are required to independently create and maintain an account with MPCVault at https://mpcvault.com and shall be subject to MPCVault’s terms and conditions.

By using MPCVault in connection with Strails, the customer acknowledges and agrees that all interactions with MPCVault are undertaken at the customer’s sole risk. Strails does not control, operate, or assume any responsibility or liability for MPCVault’s services, systems, or security.

Strails shall not be liable for any losses, liabilities, damages, claims, costs, or expenses arising out of or in connection with the use of MPCVault or any activities conducted on or through the MPCVault platform

**Auto-Signing IP Allowlist**: If auto-signing is enabled, you must add the `natExternalIp` address to your MPC Vault allowlist and API Vault IP whitelist for automatic transaction signing to work.

**Fallback Manual Signing**: If for any reason auto-signing fails, manual approval is activated so that our liquidity providers and FX traders don't loose profitable trades.
</Info>

---

### Register MPC Vault

Registers your MPC vault configuration to enable FX trading operations. This is required before you can create orders or execute trades on the FX orderbook.

<Tabs>

<Tab title="Request (Register MPC)">

```http
POST {{BASE_URL}}/fx/mpc/register
```

</Tab>

<Tab title="Body (Register MPC)">

```json
{
  "mpcVaultApiKey": "your-mpc-vault-api-key",
  "callbackClientSignerPublicKey": "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... mpcvault-client-signer",
  "mpcClientSignerPrivateKey": "-----BEGIN OPENSSH PRIVATE KEY-----\n<base64-line-1>\n<base64-line-2>\n<base64-line-3>\n<base64-line-4>\n<base64-line-5>\n-----END OPENSSH PRIVATE KEY-----",
  "vaultId": "your-mpc-vault-id",
  "tokenWallets": {
    "USDT": "0x...",
    "USDC": "0x..."
  },
  "notificationEmails": [
    "admin@yourcompany.com"
  ]
}
```

<Warning>
**Private Key Format**: The `mpcClientSignerPrivateKey` must include newline characters as `\n` escape sequences. Each line of your OpenSSH private key should be separated by `\n`. The example above shows the required format — replace the `<base64-line-X>` placeholders with your actual key content.
</Warning>

### Parameters

| Field                          | Type   | Required | Description                                                    |
| ------------------------------ | ------ | -------- | -------------------------------------------------------------- |
| `mpcVaultApiKey`               | string | Yes      | Your MPC Vault API key                                         |
| `callbackClientSignerPublicKey`| string | Yes      | SSH public key for callback signature verification             |
| `mpcClientSignerPrivateKey`    | string | Yes      | OpenSSH private key (with `\n` newline separators)             |
| `vaultId`                      | string | Yes      | Your MPC Vault identifier                                      |
| `tokenWallets`                 | object | Yes      | Wallet addresses for USDT and USDC tokens                      |
| `tokenWallets.USDT`            | string | Yes      | Wallet address for USDT operations                             |
| `tokenWallets.USDC`            | string | Yes      | Wallet address for USDC operations                             |
| `notificationEmails`           | array  | Yes      | Email addresses for FX notifications                           |

</Tab>

<Tab title="Response (Register MPC)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "MPC vault registered successfully. Awaiting admin approval.",
  "data": {
    "vaultId": "74e8defc-5569-4f01-86c9-0XXXXXXX",
    "status": "active",
    "tokenWallets": {
      "USDT": "0x9e9fd8e75E07A1xxxxxxxxx",
      "USDC": "0x9e9fd8e75E07A1xxxxxxxxx",
      "cNGN": "0x3af904Ff6FA747xxxxxxxxx"
    },
    "createdAt": "2026-05-01T10:30:00.000Z",
    "note": "Your MPC configuration is pending admin approval. You will be notified once approved. cNGN wallet automatically set to your smart wallet."
  }
}
```

**Response Fields:**

| Field                 | Type   | Description                                                    |
| --------------------- | ------ | -------------------------------------------------------------- |
| `vaultId`             | string | Your registered MPC Vault identifier                           |
| `status`              | string | Configuration status (`active`, `pending`)                     |
| `tokenWallets`        | object | Registered wallet addresses including auto-assigned cNGN wallet|
| `tokenWallets.USDT`   | string | Registered USDT wallet address                                 |
| `tokenWallets.USDC`   | string | Registered USDC wallet address                                 |
| `tokenWallets.cNGN`   | string | Auto-assigned cNGN wallet (your smart wallet)                  |
| `createdAt`           | string | ISO 8601 timestamp of registration                             |
| `note`                | string | Status information and next steps                              |

</Tab>

</Tabs>

---

### Get FX Settings

Retrieves the fintech's FX configuration after StRails Admin approval:
- MPC Vault credentials and status
- Auto-signing VM configuration
- Token wallet addresses (USDT, USDC, cNGN)
- Notification email settings

<Tabs>

<Tab title="Request (Get Settings)">

```http
GET {{BASE_URL}}/fx/settings
```

</Tab>

<Tab title="Response (Get Settings)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "FX settings retrieved successfully",
  "data": {
    "vaultId": "vaultId",
    "mpcVaultApiKey": "4nzx**************...",
    "mpcClientSignerPrivateKey": "tGyH****************...",
    "callbackClientSignerPublicKey": "ssh-ed25519 publicKey mpcvault-client-signer",
    "mpcConfigStatus": "active",
    "autoSigning": {
      "available": true,
      "enabled": true,
      "threshold": "1",
      "vmStatus": "running",
      "vmInstanceName": "mpc-signer-c8fe1911-d292-4f84-a-xxxxxx",
      "natExternalIp": "35.192.15.0",
      "note": "Add this IP to MPC Vault allowlist"
    },
    "tokenWallets": {
      "USDT": "0x9e9fd8e75E07A1xxxxxxxxx",
      "USDC": "0x9e9fd8e75E07A1xxxxxxxxx",
      "cNGN": "0x3af904Ff6FA747xxxxxxxxx"
    },
    "notificationEmails": [
      "admin@yourcompany.com"
    ]
  }
}
```
---

## Response Fields

### Root Fields

| Field           | Type   | Description                              |
| --------------- | ------ | ---------------------------------------- |
| `status`        | string | Request status (`Success` or `Error`)    |
| `response_code` | string | Response code (`00` = success)           |
| `message`       | string | Human-readable response message          |
| `data`          | object | FX settings data object                  |

### Data Object

| Field                          | Type   | Description                                                    |
| ------------------------------ | ------ | -------------------------------------------------------------- |
| `fintechId`                    | string | Unique identifier for the fintech                              |
| `vaultId`                      | string | MPC Vault identifier                                           |
| `mpcVaultApiKey`               | string | MPC Vault API key (partially masked for security)              |
| `mpcClientSignerPrivateKey`    | string | Client signer private key (partially masked for security)      |
| `callbackClientSignerPublicKey`| string | SSH public key for callback signature verification             |
| `mpcConfigStatus`              | string | MPC configuration status (`active`, `pending`, `inactive`)     |
| `autoSigning`                  | object | Auto-signing VM configuration details                          |
| `tokenWallets`                 | object | Wallet addresses for each supported token                      |
| `notificationEmails`           | array  | List of email addresses for FX notifications                   |

### Auto-Signing Object

| Field            | Type    | Description                                                          |
| ---------------- | ------- | -------------------------------------------------------------------- |
| `available`      | boolean | Whether auto-signing feature is available for this fintech          |
| `enabled`        | boolean | Whether auto-signing is currently enabled                           |
| `threshold`      | string  | Amount threshold for transaction to be auto-signed                   |
| `vmStatus`       | string  | VM instance status (`running`, `stopped`, `terminated`)             |
| `vmInstanceName` | string  | Google Cloud VM instance name                                       |
| `natExternalIp`  | string  | External IP address of the auto-signer VM                           |
| `note`           | string  | Important configuration note (e.g., IP allowlist reminder)          |

### Token Wallets Object

| Field   | Type   | Description                                      |
| ------- | ------ | ------------------------------------------------ |
| `USDT`  | string | Wallet address for USDT token operations         |
| `USDC`  | string | Wallet address for USDC token operations         |
| `cNGN`  | string | Wallet address for cNGN token operations         |

---

</Tab>

</Tabs>

---

### Enable Auto-Signing

Enables automatic transaction signing for FX trades. When enabled, transactions below the configured USD threshold will be automatically signed without manual approval.

<Tabs>

<Tab title="Request (Enable)">

```http
POST {{BASE_URL}}/fx/auto-signing/enable
```

</Tab>

<Tab title="Body (Enable)">

```json
{
  "enabled": true
}
```

### Parameters

| Field     | Type    | Required | Description                              |
| --------- | ------- | -------- | ---------------------------------------- |
| `enabled` | boolean | Yes      | Set to `true` to enable auto-signing    |

</Tab>

<Tab title="Response (Enable)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Auto-signing enabled successfully",
  "data": {
    "autoSigning": {
      "enabled": true,
      "usdThreshold": "1",
      "vmHealthy": true,
      "vmStatus": "running"
    }
  }
}
```

**Response Fields:**

| Field                       | Type    | Description                                           |
| --------------------------- | ------- | ----------------------------------------------------- |
| `autoSigning.enabled`       | boolean | Auto-signing status (now `true`)                      |
| `autoSigning.usdThreshold`  | string  | Current USD threshold for auto-signing                |
| `autoSigning.vmHealthy`     | boolean | Whether the auto-signer VM is healthy                 |
| `autoSigning.vmStatus`      | string  | VM status details (null if healthy)                   |

</Tab>

</Tabs>

---

### Disable Auto-Signing

Disables automatic transaction signing. All FX transactions will require manual approval after disabling.

<Tabs>

<Tab title="Request (Disable)">

```http
POST {{BASE_URL}}/fx/auto-signing/disable
```

</Tab>

<Tab title="Body (Disable)">

```json
{
  "enabled": false
}
```

### Parameters

| Field     | Type    | Required | Description                              |
| --------- | ------- | -------- | ---------------------------------------- |
| `enabled` | boolean | Yes      | Set to `false` to disable auto-signing  |

</Tab>

<Tab title="Response (Disable)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Auto-signing disabled successfully",
  "data": {
    "autoSigning": {
      "enabled": false
    }
  }
}
```

**Response Fields:**

| Field                 | Type    | Description                              |
| --------------------- | ------- | ---------------------------------------- |
| `autoSigning.enabled` | boolean | Auto-signing status (now `false`)        |

</Tab>

</Tabs>

---

### Set Auto-Signing Threshold

Configures the USD threshold for automatic transaction signing. Transactions with a USD value at or below this threshold will be automatically signed when auto-signing is enabled.

<Tabs>

<Tab title="Request (Threshold)">

```http
POST {{BASE_URL}}/fx/auto-signing/threshold
```

</Tab>

<Tab title="Body (Threshold)">

```json
{
  "usdThreshold": "1"
}
```

### Parameters

| Field          | Type   | Required | Description                                      |
| -------------- | ------ | -------- | ------------------------------------------------ |
| `usdThreshold` | string | Yes      | USD value threshold for auto-signing transactions|

</Tab>

<Tab title="Response (Threshold)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Auto-signing threshold updated successfully",
  "data": {
    "autoSigning": {
      "enabled": true,
      "usdThreshold": "1"
    }
  }
}
```

**Response Fields:**

| Field                      | Type    | Description                              |
| -------------------------- | ------- | ---------------------------------------- |
| `autoSigning.enabled`      | boolean | Current auto-signing status              |
| `autoSigning.usdThreshold` | string  | Updated USD threshold value              |

</Tab>

</Tabs>

---

### Get Auto-Signing Status

Retrieves the current auto-signing configuration and daily statistics including total transactions auto-signed, success rate, and execution times.

<Tabs>

<Tab title="Request (Status)">

```http
GET {{BASE_URL}}/fx/auto-signing/status
```

No request body required.

</Tab>

<Tab title="Response (Status)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Auto-signing configuration retrieved",
  "data": {
    "autoSigning": {
      "configured": true,
      "enabled": false,
      "usdThreshold": "1",
      "vmStatus": "running",
      "vmHealthy": true,
      "todayStats": {
        "totalAutoSigned": 0,
        "totalUsdAutoSigned": "0",
        "fallbackCount": 0,
        "successRate": 0,
        "avgExecutionTimeMs": 0
      }
    }
  }
}
```

**Response Fields:**

| Field                                | Type    | Description                                           |
| ------------------------------------ | ------- | ----------------------------------------------------- |
| `autoSigning.configured`             | boolean | Whether auto-signing has been configured              |
| `autoSigning.enabled`                | boolean | Whether auto-signing is currently enabled             |
| `autoSigning.usdThreshold`           | string  | Current USD threshold for auto-signing                |
| `autoSigning.vmStatus`               | string  | Auto-signer VM status                                 |
| `autoSigning.vmHealthy`              | boolean | Whether the auto-signer VM is healthy                 |
| `autoSigning.todayStats`             | object  | Daily statistics for auto-signed transactions         |

**Today Stats Object:**

| Field                | Type   | Description                                           |
| -------------------- | ------ | ----------------------------------------------------- |
| `totalAutoSigned`    | number | Total transactions auto-signed today                  |
| `totalUsdAutoSigned` | string | Total USD value of auto-signed transactions today     |
| `fallbackCount`      | number | Number of transactions that fell back to manual       |
| `successRate`        | number | Success rate percentage (0-100)                       |
| `avgExecutionTimeMs` | number | Average execution time in milliseconds                |

</Tab>

</Tabs>

---

### Get Auto-Signing Stats

Retrieves auto-signing statistics over a configurable period of days, including daily breakdowns and aggregate totals.

<Tabs>

<Tab title="Request (Stats)">

```http
GET {{BASE_URL}}/fx/auto-signing/stats
```

No request body required.

</Tab>

<Tab title="Response (Stats)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Auto-signing statistics retrieved",
  "data": {
    "stats": {
      "days": 7,
      "daily": [
        {
          "date": "2026-05-01",
          "totalAutoSigned": 0,
          "totalAmountAutoSigned": "0",
          "fallbackCount": 0,
          "successRate": 0,
          "avgExecutionTimeMs": 0
        }
      ],
      "totals": {
        "totalAutoSigned": 0,
        "totalAmountAutoSigned": "0",
        "totalFallbacks": 0,
        "avgSuccessRate": 0
      }
    }
  }
}
```

**Response Fields:**

| Field         | Type   | Description                                           |
| ------------- | ------ | ----------------------------------------------------- |
| `stats.days`  | number | Number of days included in the statistics             |
| `stats.daily` | array  | Array of daily statistics objects                     |
| `stats.totals`| object | Aggregate totals across all days                      |

**Daily Stats Object:**

| Field                   | Type   | Description                                           |
| ----------------------- | ------ | ----------------------------------------------------- |
| `date`                  | string | Date in YYYY-MM-DD format                             |
| `totalAutoSigned`       | number | Total transactions auto-signed on this day            |
| `totalAmountAutoSigned` | string | Total USD value of auto-signed transactions           |
| `fallbackCount`         | number | Number of transactions that fell back to manual       |
| `successRate`           | number | Success rate percentage (0-100)                       |
| `avgExecutionTimeMs`    | number | Average execution time in milliseconds                |

**Totals Object:**

| Field                   | Type   | Description                                           |
| ----------------------- | ------ | ----------------------------------------------------- |
| `totalAutoSigned`       | number | Total transactions auto-signed across all days        |
| `totalAmountAutoSigned` | string | Total USD value of auto-signed transactions           |
| `totalFallbacks`        | number | Total fallback count across all days                  |
| `avgSuccessRate`        | number | Average success rate across all days                  |

</Tab>

</Tabs>

