---
title: "Wallet Management API"
sidebarTitle: "Wallet Management"
description: "Wallet Management API endpoints for viewing and managing fintech system and external wallets"
---

The Wallet Management API allows fintechs to manage their wallets:

- **[Smart Wallet](/smart-wallet)** — Contract-based wallet for cNGN, USDC, USDT. Can be fintech or Strails-controlled.
- **MPC Vault** — Multi-party computation wallet for secure custody of USDC/USDT.
- **External Wallets** — User-registered wallets for withdrawals and storage.

---

## Wallet Management API — Endpoints Overview

| Section                    | Endpoint                         | Method | Description                                                                                          |
| -------------------------- | -------------------------------- | ------ | ---------------------------------------------------------------------------------------------------- |
| [**Get Fintech Wallet**](#retrieve-the-fintech-smart-wallet-and-all-external-wallets)     | `/getfintechwallet`           | GET    | Retrieve fintech wallet information including the **system (MPC) wallet** and **external wallets**. |
| [**Add External Wallet**](#add-an-external-wallet)    | `/addexternalwallet`          | POST   | Add an external wallet to your fintech account. Not usable for escrow.                               |
| [**Update External Wallet**](#update-external-wallet-status) | `/updateexternalwalletstatus` | PUT    | Activate or deactivate an external wallet.                                                           |
| [**Remove External Wallet**](#remove-an-external-wallet) | `/removeexternalwallet`       | DELETE | Remove an external wallet from the fintech account.                                                  |
| [**Get User Wallets**](#retrieve-user-wallets)       | `/listuserwallets`            | POST   | Retrieve all wallets associated with a specific user, including balances.                            |
| [**Migrate User Wallets**](#migrate-user-wallets)   | `/migrateuserwallets`         | POST   | Create default wallets for existing users created before wallet provisioning was automated.          |
| [**Fintech System Wallet Balance**](#fintech-system-wallet-balance) | `/balance/multi` | GET | Returns the authenticated fintech's own system/smart wallet balance across cNGN EVM networks. |
| [**Balance for a Specific Address**](#balance-for-a-specific-address) | `/balance/address` | POST | Returns the token balance for any specific EVM address across networks. |
| [**End-User Balance**](#end-user-balance) | `/balance/user/:userId` | GET | Returns a fintech end-user's wallet balances. |

---

### Retrieve the fintech smart wallet and all external wallets

<Tabs>

<Tab title="Request (Fintech Wallet)">

```http
GET {{BASE_URL}}/getfintechwallet
```

</Tab>


<Tab title="Response (Fintech Wallet)">

```json
{
  "status": "success",
  "response_code": "00",
  "message": "Fintech wallet retrieved successfully",
  "data": {
    "fintechId": "fintech_abc123",
    "fintechName": "Example Fintech Ltd",
    "smartWallet": {
        "address": "0x69E4eA33BF7E2f6bAE7F1DEF247534C0aD3515C3",
        "owner": "0x684f5f118ed7d0e00b1c0a27dcdce6e37cf652e1",
        "deployed": true,
        "createdAt": "2026-04-02T08:45:02.007Z",
        "saltVersion": "v2",
        "previousAddress": "0x69E4eA33BF7E2f6bAE7F1DEF247534C0aD3515C3",
        "deploymentTxHash": "0x3cd779012c51172781202e9fff8eff749fda01f8f9e530bd0fb3a0dbab68ca46"
        },
    "externalWallets": [
      {
        "address": "0xabcdefabcdefabcdefabcdefabcdefabcdefabcd",
        "label": "Cold Storage Wallet",
        "blockchain": "base",
        "type": "cold",
        "addedAt": "2026-02-09T11:44:38.960Z",
        "addedBy": "4e769009-7a30-42df-89eb-7234a29e53c2",
        "status": "active",
        "isDefault": false,
        "purpose": "Long-term storage"
      }
    ],
    "totalUsers": 150,
    "createdAt": "2026-01-15T15:03:03.977Z",
    "updatedAt": "2026-04-29T13:20:36.574Z"
  }
}
```

</Tab>

</Tabs>

---

### Add an external wallet

<Tabs>

<Tab title="Request (Add Wallet)">

```http
POST {{BASE_URL}}/addexternalwallet
```

</Tab>

<Tab title="Body (Add Wallet)">

```json
{
  "address": "0x-WalletAddress",
  "label": "Hot Wallet - Operations",
  "blockchain": "base",
  "type": "hot",
  "purpose": "Daily operations and withdrawals",
  "isDefault": true,
  "metadata": {
    "custodian": "Internal",
    "location": "AWS KMS",
    "backupExists": true
  }
}
```

### Parameters

| Parameter     | Type    | Required | Description |
|--------------|---------|----------|-------------|
| `address`     | string  | Yes   | Wallet address to be added to the platform |
| `label`       | string  | No    | Human-readable name for easy identification |
| `blockchain`  | string  | Yes   | Blockchain network the wallet belongs to (e.g. `base`) |
| `type`        | enum  | Yes   | Wallet type (`hot`, `cold`, `custodial` or `other`) |
| `purpose`     | string  | No    | Intended use of the wallet (e.g. operations, withdrawals) |
| `isDefault`   | boolean | No    | true or false (defaults to false) |
| `metadata`    | object  | No    | Additional wallet information and custody details |

#### `metadata` Object

| Field           | Type    | Required | Description |
|----------------|---------|----------|-------------|
| `custodian`     | string  | No    | Entity responsible for wallet custody |
| `location`      | string  | No    | Key or wallet storage location (e.g. KMS, HSM) |
| `backupExists`  | boolean | No    | Indicates whether a secure backup exists |

</Tab>

<Tab title="Response (Add Wallet)">

```json
{
  "status": "success",
  "response_code": "00",
  "message": "External wallet added successfully",
  "data": {
    "wallet": {
      "address": "wallet-address",
      "status": "active",
      "type": "hot",
      "blockchain": "base"
    }
  }
}
```

</Tab>

<Tab title="Error (Add Wallet)">

```json
{
  "status": "error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Invalid wallet address format"
}
```

```json
{
  "status": "error",
  "response_code": "01",
  "message": "Wallet already exists",
  "error": "This wallet address is already registered"
}
```

</Tab>

</Tabs>

---

### Update external wallet status

<Tabs>

<Tab title="Request (Update)">

```http
PUT {{BASE_URL}}/updateexternalwalletstatus
```

</Tab>


<Tab title="Body (Update)">

```json
{
  "address": "0xabcdefabcdefabcdefabcdefabcdefabcdefabcd",
  "status": "inactive"
}
```

### Parameters

| Parameter | Type   | Required | Description |
|----------|--------|----------|-------------|
| `address` | string | Yes   | Wallet address to be updated |
| `status`  | string | Yes   | New wallet status (`active` or `inactive`) |

</Tab>

<Tab title="Response (Update)">

```json
{
  "status": "success",
  "response_code": "00",
  "message": "External wallet status updated successfully",
  "data": {
    "address": "0xabcdefabcdefabcdefabcdefabcdefabcdefabcd",
    "status": "inactive"
  }
}
```

</Tab>

<Tab title="Error (Update)">

```json
{
  "status": "error",
  "response_code": "02",
  "message": "Wallet not found",
  "error": "No external wallet found with the provided address"
}
```

```json
{
  "status": "error",
  "response_code": "01",
  "message": "Validation error",
  "error": "Status must be 'active' or 'inactive'"
}
```

</Tab>

</Tabs>

---

### Remove an external wallet

<Tabs>

<Tab title="Request (Remove)">

```http
DELETE {{BASE_URL}}/removeexternalwallet
```

</Tab>


<Tab title="Body (Remove)">

```json
{
  "address": "0xabcdefabcdefabcdefabcdefabcdefabcdefabcd"
}
```

</Tab>

<Tab title="Response (Remove)">

```json
{
  "status": "success",
  "response_code": "00",
  "message": "External wallet removed successfully",
  "data": {
    "address": "0xabcdefabcdefabcdefabcdefabcdefabcdefabcd"
  }
}
```

</Tab>

<Tab title="Error (Remove)">

```json
{
  "status": "error",
  "response_code": "02",
  "message": "Wallet not found",
  "error": "No external wallet found with the provided address"
}
```

</Tab>

</Tabs>

---

### Retrieve user wallets

<Tabs>

<Tab title="Request (User Wallets)">

```http
POST {{BASE_URL}}/listuserwallets
```

</Tab>


<Tab title="Body (User Wallets)">

```json
{
  "userId": "user_uniqueID"
}
```

</Tab>

<Tab title="Response (User Wallets)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "User wallets retrieved successfully",
  "data": {
    "wallets": [
       {
                "walletAddress": "sample_0x23341e77675b4513bEbA945B05afb2",
                "status": "smart_wallet",
                "createdAt": "2026-06-03T10:46:37.494Z",
                "owner": "strails_internal_0x684f5f118ed7d0e00b1c0",
                "tokenBuy": "CNGN",
                "blockchain": "base",
                "balances": {
                    "CNGN": {
                        "balance": "0.00"
                    },
                    "USDC": {
                        "balance": "0.00"
                    },
                    "USDT": {
                        "balance": "0.00"
                    }
                }
            },
            {
                "walletAddress": "sample_GAFIWE3JABCQ5LTOUO4BPFQ3QRBEHTVOKETYLB",
                "status": "multi_chain",
                "createdAt": "2026-06-03T10:46:37.695Z",
                "owner": "system_controlled",
                "tokenBuy": "CNGN",
                "blockchain": "bantu",
                "balances": {
                    "cngn": null,
                    "tokenBuy": null
                }
            },
            {
                "walletAddress": "sample_4ZpwXDXZUPiyi2NNTtMFZyMojFZrYFs",
                "status": "multi_chain",
                "createdAt": "2026-06-03T10:46:38.324Z",
                "owner": "system_controlled",
                "tokenBuy": "CNGN",
                "blockchain": "solana",
                "balances": {
                     "cngn": null,
                    "tokenBuy": null
                }
            }
    ],
    "count": 5,
    "userId": "user_uniqueID",
    "version": "1.0.0"
  }
}
```

</Tab>

<Tab title="Error (User Wallets)">

```json
{
  "status": "Error",
  "response_code": "02",
  "message": "User not found",
  "error": "No verified user found with the provided userId"
}
```

</Tab>

</Tabs>

---

### Migrate user wallets

<Warning>
Ensure funds are transferred out of the existing wallet before calling this endpoint.
</Warning>

<Tabs>

<Tab title="Request (Migrate)">

```http
POST {{BASE_URL}}/migrateuserwallets
```

</Tab>

<Tab title="Body (Migrate)">

```json
{
  "userId": ["userId", "userId1"],
  "force": true
}
```

### Parameters

| Parameter   | Type    | Required | Description |
|------------|---------|----------|-------------|
| `userId` | array   | Yes   | List of user identifiers (hash/UUID) to be migrated |
| `force`    | boolean | No    | Force migration even if the user already exists or has partial data |

</Tab>

<Tab title="Response (Migrate)">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Wallet migration completed",
  "data": {
    "requestId": "migrate_fintech123_1706457600000",
    "totalProcessed": 3,
    "created": 2,
    "migrated": 0,
    "skipped": 1,
    "failed": 0,
    "results": [
      { "userId": "abc123", "status": "created", "walletAddress": "0x...", "deployed": true },
      { "userId": "def456", "status": "created", "walletAddress": "0x...", "deployed": true },
      { "userId": "ghi789", "status": "skipped", "walletAddress": "0x...", "reason": "Wallet already exists" }
    ]
  }
}
```

</Tab>

</Tabs>

---

### Fintech System Wallet Balance

**Endpoint:** `GET {{BASE_URL}}/balance/multi`

**Purpose:** Returns the authenticated fintech's own system/smart wallet balance across cNGN EVM networks.

<Tabs>

<Tab title="Request">

```http
GET {{BASE_URL}}/balance/multi?token=CNGN&networks=base,eth,bsc
```

</Tab>

<Tab title="Response">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Multi-network balance retrieved",
  "data": {
    "token": "CNGN",
    "walletAddress": "0x25AC115350Af0a6B53475d2e667898D7267C78B1",
    "totalBalance": "1299.974550",
    "hasBaseNetworkFunds": true,
    "hasCrossNetworkFunds": false,
    "baseNetworkBalance": "1299.974550",
    "crossNetworkBalance": "0.000000",
    "bridgingRequired": false,
    "networkBalances": [
      {
        "network": "base",
        "balance": "1299.974550"
      },
      {
        "network": "eth",
        "balance": "0",
        "error": "All RPC endpoints failed for eth"
      }
    ],
    "walletsWithFunds": [
      {
        "network": "base",
        "balance": "1299.974550"
      }
    ],
    "checkedAt": "2026-07-22T09:12:49.339Z"
  }
}
```

</Tab>

</Tabs>

#### Query parameters

| Query param | Required | Default | Description |
|---|---|---|---|
| `token` | No | `CNGN` | Token ticker |
| `networks` | No | all EVM networks | Comma-separated or repeated list of networks |

---

### Balance for a Specific Address

**Endpoint:** `POST {{BASE_URL}}/balance/address`

**Purpose:** Returns the token balance for any specific EVM address across networks.

<Tabs>

<Tab title="Request">

```http
POST {{BASE_URL}}/balance/address
Content-Type: application/json
```

```json
{
  "address": "0xF223A5962c3901159465A0511Db4a74FFaCF3f8A",
  "token": "CNGN",
  "networks": ["base", "eth"]
}
```

</Tab>

<Tab title="Response">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "Address balance retrieved",
  "data": {
    "token": "CNGN",
    "address": "0xF223A5962c3901159465A0511Db4a74FFaCF3f8A",
    "totalBalance": "326.695000",
    "networkBalances": [
      {
        "network": "base",
        "balance": "326.695000"
      },
      {
        "network": "eth",
        "balance": "0",
        "error": "All RPC endpoints failed for eth"
      }
    ],
    "checkedAt": "2026-07-22T09:01:29.058Z"
  }
}
```

</Tab>

</Tabs>

#### Body parameters

| Body field | Required | Default | Description |
|---|---|---|---|
| `address` | Yes | — | EVM address |
| `token` | No | `CNGN` | Token ticker |
| `networks` | No | all EVM networks | Array of network keys |

---

### End-User Balance

**Endpoint:** `GET {{BASE_URL}}/balance/user/:userId`

**Purpose:** Returns a fintech end-user's wallet balances. Wallets are resolved from `verifiedUsers` (smart wallet + `networkAddresses`) and `fintechWallets`.

<Tabs>

<Tab title="Request">

```http
GET {{BASE_URL}}/balance/user/abc123?token=CNGN
```

</Tab>

<Tab title="Response">

```json
{
  "status": "Success",
  "response_code": "00",
  "message": "User balance retrieved",
  "data": {
    "userId": "abc123",
    "token": "CNGN",
    "totalBalance": "326.695000",
    "networkBalances": [
      {
        "network": "base",
        "walletAddress": "0x...",
        "balance": "326.695000"
      },
      {
        "network": "eth",
        "walletAddress": "0x...",
        "balance": "0",
        "error": "All RPC endpoints failed for eth"
      }
    ],
    "checkedAt": "2026-07-22T09:01:29.058Z"
  }
}
```

</Tab>

</Tabs>

#### Query parameters

| Query param | Required | Default | Description |
|---|---|---|---|
| `token` | No | `CNGN` | Token ticker |
