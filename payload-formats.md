---
title: "Request & Response Formats"
sidebarTitle: "Payload Formats"
description: "Request and response formats for Strails API endpoints"
---

The Strails API uses consistent JSON structures for all requests and responses. This page details the request and response payloads for each endpoint group.

<Info>
For the standard response envelope, response codes, and amount format conventions, see [Core Concepts](/core-concepts#response-format).
</Info>

## Virtual Accounts API Formats

### Create Virtual Account Request

<Tabs>
<Tab title="Request Body">
```json
{
  "customer": {
    "name": "Adebayo Ogundimu",
    "email": "adebayo@example.com",
    "phone": "+2348012345678",
    "bvn": "12345678901"
  },
  "account": {
    "currency": "NGN",
    "account_name": "Adebayo Ogundimu - CNGN Wallet",
    "preferred_bank": "GTBank"
  }
}
```
</Tab>
</Tabs>

### Virtual Account Response

<Tabs>
<Tab title="Response Body">
```json
{
  "status": "success",
  "response_code": "201",
  "message": "Virtual account created successfully",
  "data": {
    "virtual_account": {
      "id": "va_1702890600",
      "account_number": "9876543210",
      "account_name": "Adebayo Ogundimu - CNGN Wallet",
      "bank_name": "GTBank",
      "bank_code": "058",
      "currency": "NGN",
      "status": "active",
      "balance": {
        "available": 50000,
        "pending": 0,
        "currency": "NGN"
      },
      "created_at": "2023-12-07T10:30:00Z",
      "updated_at": "2023-12-07T10:30:00Z"
    },
    "customer": {
      "id": "cust_1702890600",
      "name": "Adebayo Ogundimu",
      "email": "adebayo@example.com",
      "phone": "+2348012345678"
    }
  }
}
```
</Tab>
</Tabs>

## Transactions API Formats

### Wallet Funding Request

<Tabs>
<Tab title="Request Body">
```json
{
  "virtual_account_id": "va_1702890600",
  "amount": 5000,
  "currency": "NGN",
  "reference": "FUND_REF_001",
  "customer": {
    "id": "cust_1702890600",
    "wallet_id": "wallet_abc123"
  },
  "metadata": {
    "source": "bank_transfer",
    "description": "Wallet funding from GTBank"
  }
}
```
</Tab>
</Tabs>

### Payout Request

<Tabs>
<Tab title="Request Body">
```json
{
  "customer": {
    "id": "cust_1702890600",
    "wallet_id": "wallet_abc123"
  },
  "amount": 10000,
  "currency": "NGN",
  "reference": "PAYOUT_REF_001",
  "destination": {
    "bank_code": "058",
    "account_number": "0123456789",
    "account_name": "Adebayo Ogundimu"
  },
  "narration": "Withdrawal request - Strails",
  "metadata": {
    "withdrawal_reason": "user_request",
    "ip_address": "192.168.1.100"
  }
}
```
</Tab>
</Tabs>

### Transaction Response

<Tabs>
<Tab title="Response Body">
```json
{
  "status": "success",
  "response_code": "200",
  "message": "Transaction processed successfully",
  "data": {
    "transaction": {
      "id": "txn_1702890600_001",
      "reference": "FUND_REF_001",
      "type": "wallet_funding",
      "status": "completed",
      "amount": 5000,
      "currency": "NGN",
      "virtual_account": {
        "id": "va_1702890600",
        "account_number": "9876543210",
        "bank_name": "GTBank"
      },
      "customer": {
        "id": "cust_1702890600",
        "name": "Adebayo Ogundimu",
        "wallet_id": "wallet_abc123"
      },
      "fees": {
        "platform_fee": 50,
        "processing_fee": 25,
        "total_fees": 75
      },
      "net_amount": 4925,
      "status_history": [
        {
          "status": "pending",
          "timestamp": "2023-12-07T10:30:00Z"
        },
        {
          "status": "processing",
          "timestamp": "2023-12-07T10:30:05Z"
        },
        {
          "status": "completed",
          "timestamp": "2023-12-07T10:30:15Z"
        }
      ],
      "created_at": "2023-12-07T10:30:00Z",
      "updated_at": "2023-12-07T10:30:15Z"
    }
  }
}
```
</Tab>
</Tabs>

## Webhook Event Formats

We send webhook events as POST requests to your application's webhook endpoint (configured via the `/setwebhook` API). Two payload formats are supported:

### Legacy Format Events

The legacy format uses the `event` and `data` structure:

<Tabs>
<Tab title="Wallet Funding Event">
```json
{
  "event": "wallet_funding",
  "data": {
    "status": "successful",
    "transaction_id": "TXN_1702890600_001",
  "reference": "FUND_ABC123",
    "amount": 5000,
    "currency": "NGN",
    "virtual_account_id": "VA_1702890600",
    "sender_name": "Adebayo Ogundimu",
    "sender_account_number": "0123456789",
    "sender_bank": "GTBank",
  "narration": "Payment for services - Strails funding",
    "timestamp": "2023-12-07T10:30:15Z"
  }
}
```
</Tab>

<Tab title="Payout Event">
```json
{
  "event": "payout",
  "data": {
    "status": "successful",
    "transaction_id": "PAYOUT_1702890600_001",
  "reference": "PAYOUT_DEF456",
    "amount": 10000,
    "currency": "NGN",
    "recipient_name": "Fatima Abdullahi",
    "recipient_account_number": "2233445566",
    "recipient_bank": "Zenith Bank",
    "narration": "Disbursement - Withdrawal request",
    "timestamp": "2023-12-07T11:15:00Z"
  }
}
```
</Tab>

<Tab title="Bank Transfer Event">
```json
{
  "event": "banktransfer",
  "data": {
    "status": "successful",
    "transaction_id": "TRANSFER_1702890600_001",
  "reference": "TRANSFER_GHI789",
    "amount": 2500,
    "currency": "NGN",
    "sender_name": "Kemi Adebayo",
    "sender_account_number": "3344556677",
    "sender_bank": "UBA",
  "recipient_name": "Strails Platform",
    "recipient_account_number": "4455667788",
    "recipient_bank": "Wema Bank",
    "narration": "Inter-bank transfer - Platform funding",
    "timestamp": "2023-12-07T12:00:00Z"
  }
}
```
</Tab>
</Tabs>

### Virtual Account Format Events

The virtual account format uses the `notify`, `notifyType`, and `Data` structure:

<Tabs>
<Tab title="Successful Funding">
```json
{
  "notify": "wallet_funding",
  "notifyType": "successful",
  "Data": {
    "Amount": 5000,
    "Status": "successful",
    "TransactionReference": "VA_FUND_XYZ789",
    "Currency": "NGN",
    "Id": "VA_TXN_1702890600_001",
    "VirtualAccountId": "VA_1702890600",
    "SenderName": "Adebayo Ogundimu",
    "SenderAccountNumber": "0123456789",
    "SenderBank": "GTBank",
  "Narration": "Payment for services - Strails funding",
    "Timestamp": "2023-12-07T10:30:00Z",
    "Domain": "cngn-dex",
    "Channel": "virtual_account",
    "ChargedAmount": 5000,
    "TransactionFee": 0,
    "TransactionType": "credit",
    "GatewayResponseCode": "00",
    "GatewayResponseMessage": "Transaction successful"
  }
}
```
</Tab>

<Tab title="Failed Transaction">
```json
{
  "notify": "wallet_funding",
  "notifyType": "failed",
  "Data": {
    "Amount": 1000,
    "Status": "failed",
    "TransactionReference": "VA_FAILED_ABC123",
    "Currency": "NGN",
    "Id": "VA_TXN_FAILED_1702890600",
    "VirtualAccountId": "VA_1702890600",
    "SenderName": "Test User",
    "SenderAccountNumber": "1111111111",
    "SenderBank": "Test Bank",
    "Narration": "Failed transaction - Should not trigger funding",
    "Timestamp": "2023-12-07T10:35:00Z",
    "Domain": "cngn-dex",
    "FailureReason": "Insufficient funds"
  }
}
```
</Tab>
</Tabs>

### Webhook Headers



### Legacy vs Virtual Account Format Differences

| Field | Legacy Format | Virtual Account Format |
|-------|---------------|------------------------|
| **Event Type** | `event` | `notify` |
| **Event Status** | `data.status` | `notifyType` + `Data.Status` |
| **Transaction ID** | `data.transaction_id` | `Data.Id` |
| **Amount** | `data.amount` | `Data.Amount` |
| **Reference** | `data.reference` | `Data.TransactionReference` |
| **Timestamp** | `data.timestamp` | `Data.Timestamp` |
| **Sender Name** | `data.sender_name` | `Data.SenderName` |
| **Virtual Account** | `data.virtual_account_id` | `Data.VirtualAccountId` |

## Data Types and Validation

### String Fields

| Field Type | Format | Example |
|------------|--------|---------|
| **ID Fields** | Alphanumeric with prefix | `va_1702890600`, `txn_1702890600_001` |
| **Email** | Valid email format | `user@example.com` |
| **Phone** | International format | `+2348012345678` |
| **BVN** | 11-digit numeric | `12345678901` |
| **Bank Code** | 3-digit numeric | `058` |
| **Currency** | ISO 4217 code | `NGN` |

### Numeric Fields

| Field Type | Format | Description |
|------------|--------|-------------|
| **Amount** | Integer (kobo) | Amount in smallest currency unit |
| **Timestamp** | Unix timestamp | Seconds since epoch |
| **Percentage** | Decimal (0-1) | `0.1` for 10% |

### Date/Time Fields

All timestamps use ISO 8601 format in UTC:

<Tabs>
<Tab title="Timestamp Format">
```json
{
  "created_at": "2023-12-07T10:30:00Z",
  "updated_at": "2023-12-07T10:30:15Z"
}
```
</Tab>
</Tabs>

## Pagination Format

List endpoints return paginated results:

<Tabs>
<Tab title="Pagination Response">
```json
{
  "status": "success",
  "response_code": "200",
  "message": "Data retrieved successfully",
  "data": {
    "items": [
      // Array of objects
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 5,
      "total_count": 243,
      "per_page": 50,
      "has_next": true,
      "has_previous": false
    }
  }
}
```
</Tab>
</Tabs>

### Pagination Query Parameters

<Tabs>
<Tab title="Query Parameters">
```http
GET /getvirtualaccount?userId=77aefb50d96d0a63bfb913a00a07e361dc63ed1e43696e2d7d9c17b1d6630933
```
</Tab>
</Tabs>

## Error Response Details

### Validation Errors

<Tabs>
<Tab title="Error Response">
```json
{
  "status": "error",
  "response_code": "400",
  "message": "Request validation failed",
  "data": {
    "error": "Validation failed",
    "details": {
      "customer.email": "Invalid email format",
      "customer.phone": "Phone number must start with +234",
      "amount": "Amount must be greater than 0"
    }
  }
}
```
</Tab>
</Tabs>

### Authentication Errors

<Tabs>
<Tab title="Error Response">
```json
{
  "status": "error",
  "response_code": "401",
  "message": "Invalid API key provided",
  "data": {
    "error": "Authentication failed",
    "details": {
      "api_key": "API key not found or inactive"
    }
  }
}
```
</Tab>
</Tabs>

### Rate Limit Errors

<Tabs>
<Tab title="Error Response">
```json
{
  "status": "error",
  "response_code": "429",
  "message": "Too many requests. Please try again later.",
  "data": {
    "error": "Rate limit exceeded",
    "details": {
      "limit": 300,
      "remaining": 0,
      "reset_time": 1702890660,
      "retry_after": 60
    }
  }
}
```
</Tab>
</Tabs>

## Request Validation Rules

### Customer Data

<Tabs>
<Tab title="Validation Rules">
```json
{
  "customer": {
    "name": {
      "required": true,
      "type": "string",
      "min_length": 2,
      "max_length": 100,
      "pattern": "^[a-zA-Z\\s]+$"
    },
    "email": {
      "required": true,
      "type": "email",
      "max_length": 255
    },
    "phone": {
      "required": true,
      "type": "string",
      "pattern": "^\\+234[789][01][0-9]{8}$"
    },
    "bvn": {
      "required": false,
      "type": "string",
      "pattern": "^[0-9]{11}$"
    }
  }
}
```
</Tab>
</Tabs>

### Transaction Data

<Tabs>
<Tab title="Validation Rules">
```json
{
  "transaction": {
    "amount": {
      "required": true,
      "type": "integer",
      "minimum": 100,
      "maximum": 10000000
    },
    "currency": {
      "required": true,
      "type": "string",
      "enum": ["NGN"]
    },
    "reference": {
      "required": true,
      "type": "string",
      "min_length": 1,
      "max_length": 50,
      "pattern": "^[a-zA-Z0-9_-]+$"
    }
  }
}
```
</Tab>
</Tabs>

## Content Types

### Request Content Type

All requests must include the correct content type:

<Tabs>
<Tab title="Request Headers">
```http
Content-Type: application/json
```
</Tab>
</Tabs>

### Response Content Type

All responses are returned as JSON:

<Tabs>
<Tab title="Response Headers">
```http
Content-Type: application/json; charset=utf-8
```
</Tab>
</Tabs>

## HTTP Status Codes

| Status Code | Description | Usage |
|-------------|-------------|-------|
| `200` | OK | Successful GET, PATCH requests |
| `201` | Created | Successful POST requests |
| `400` | Bad Request | Invalid request parameters |
| `401` | Unauthorized | Invalid or missing authentication |
| `403` | Forbidden | Insufficient permissions |
| `404` | Not Found | Resource not found |
| `409` | Conflict | Resource already exists |
| `422` | Unprocessable Entity | Valid JSON but invalid data |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server error |

## Amount Formatting

### Currency Representation

All amounts are represented in the smallest currency unit (kobo for NGN):

<Tabs>
<Tab title="Amount Example">
```json
{
  "amount": 5000,     // Represents ₦50.00
  "currency": "NGN"
}
```
</Tab>
</Tabs>

### Conversion Examples

| Display Amount | API Amount | Calculation |
|----------------|------------|-------------|
| ₦1.00 | 100 | 1.00 × 100 |
| ₦50.00 | 5000 | 50.00 × 100 |
| ₦1,000.00 | 100000 | 1000.00 × 100 |

### Helper Functions

<Tabs>
<Tab title="JavaScript">
```javascript
// Convert NGN to kobo
function toKobo(naira) {
  return Math.round(naira * 100);
}

// Convert kobo to NGN
function toNaira(kobo) {
  return kobo / 100;
}

// Format amount for display
function formatAmount(kobo) {
  return `₦${(kobo / 100).toLocaleString('en-NG', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  })}`;
}
```
</Tab>

<Tab title="Python">
```python
def to_kobo(naira: float) -> int:
    """Convert NGN to kobo"""
    return round(naira * 100)

def to_naira(kobo: int) -> float:
    """Convert kobo to NGN"""
    return kobo / 100

def format_amount(kobo: int) -> str:
    """Format amount for display"""
    naira = kobo / 100
    return f"₦{naira:,.2f}"
```
</Tab>

<Tab title="PHP">
```php
function toKobo($naira) {
    return round($naira * 100);
}

function toNaira($kobo) {
    return $kobo / 100;
}

function formatAmount($kobo) {
    $naira = $kobo / 100;
    return '₦' . number_format($naira, 2);
}
```
</Tab>
</Tabs>
| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Transaction status: "successful", "failed", "pending" |
| `transaction_id` | string | Unique transaction identifier |
| `reference` | string | Transaction reference for tracking |
| `amount` | number | Transaction amount in kobo/minor currency units |
| `currency` | string | Currency code (usually "NGN") |
| `timestamp` | string | ISO 8601 timestamp |

## Virtual Account Format Structure

### Basic Structure
```json
{
  "notify": "wallet_funding",
  "notifyType": "successful", 
  "Data": {
    "Amount": 5000,
    "Status": "successful",
    "TransactionReference": "VA_FUND_REF_001",
    "Currency": "NGN",
    "Id": "VA_TXN_1234567890_001",
    "VirtualAccountId": "VA_1234567890",
    "SenderName": "Adebayo Ogundimu",
    "SenderAccountNumber": "0123456789",
    "SenderBank": "GTBank", 
    "Narration": "Payment for services",
    "Timestamp": "2023-12-07T10:30:00Z",
    "Domain": "cngn-dex",
    "Channel": "virtual_account",
    "ChargedAmount": 5000,
    "TransactionFee": 0,
    "TransactionType": "credit",
    "GatewayResponseCode": "00",
    "GatewayResponseMessage": "Transaction successful"
  }
}
```

### Enhanced Fields
| Field | Type | Description |
|-------|------|-------------|
| `notify` | string | Event type identifier |
| `notifyType` | string | Event status: "successful", "failed" |
| `Domain` | string | Service domain identifier |
| `Channel` | string | Transaction channel |
| `ChargedAmount` | number | Final charged amount |
| `TransactionFee` | number | Associated transaction fee |
| `TransactionType` | string | Type: "credit", "debit" |
| `GatewayResponseCode` | string | Gateway response code |
| `GatewayResponseMessage` | string | Gateway response message |

## Format Detection and Normalization

### Detection Logic

#### JavaScript/Node.js

```javascript
function detectPayloadFormat(payload) {
    if (payload.notify && payload.Data) {
        return 'virtual_account';
    } else if (payload.event && payload.data) {
        return 'legacy';
    }
    return 'unknown';
}
```

#### Python

```python
def detect_payload_format(payload):
    """Detect webhook payload format"""
    if 'notify' in payload and 'Data' in payload:
        return 'virtual_account'
    elif 'event' in payload and 'data' in payload:
        return 'legacy'
    return 'unknown'
```

#### TypeScript

```typescript
type PayloadFormat = 'virtual_account' | 'legacy' | 'unknown';

interface LegacyPayload {
    event: string;
    data: Record<string, any>;
}

interface VirtualAccountPayload {
    notify: string;
    notifyType?: string;
    Data: Record<string, any>;
}

function detectPayloadFormat(payload: any): PayloadFormat {
    if (payload.notify && payload.Data) {
        return 'virtual_account';
    } else if (payload.event && payload.data) {
        return 'legacy';
    }
    return 'unknown';
}
```

### Normalization Example

#### JavaScript/Node.js

```javascript
function normalizePayload(payload) {
    const format = detectPayloadFormat(payload);
    
    if (format === 'virtual_account') {
        // Convert virtual account format to standard format
        return {
            event: payload.notify,
            data: {
                status: payload.Data.Status?.toLowerCase() || payload.notifyType,
                transaction_id: payload.Data.Id,
                reference: payload.Data.TransactionReference,
                amount: payload.Data.Amount,
                currency: payload.Data.Currency,
                virtual_account_id: payload.Data.VirtualAccountId,
                sender_name: payload.Data.SenderName,
                sender_account_number: payload.Data.SenderAccountNumber,
                sender_bank: payload.Data.SenderBank,
                narration: payload.Data.Narration,
                timestamp: payload.Data.Timestamp,
                // Additional virtual account fields
                domain: payload.Data.Domain,
                channel: payload.Data.Channel,
                charged_amount: payload.Data.ChargedAmount,
                transaction_fee: payload.Data.TransactionFee,
                transaction_type: payload.Data.TransactionType,
                gateway_response_code: payload.Data.GatewayResponseCode,
                gateway_response_message: payload.Data.GatewayResponseMessage
            }
        };
    }
    
    // Return legacy format as-is
    return payload;
}
```

#### Python

```python
def normalize_payload(payload):
    """Normalize payload to standard format"""
    format_type = detect_payload_format(payload)
    
    if format_type == 'virtual_account':
        # Convert virtual account format to standard format
        data = payload.get('Data', {})
        return {
            'event': payload.get('notify'),
            'data': {
                'status': data.get('Status', '').lower() or payload.get('notifyType'),
                'transaction_id': data.get('Id'),
                'reference': data.get('TransactionReference'),
                'amount': data.get('Amount'),
                'currency': data.get('Currency'),
                'virtual_account_id': data.get('VirtualAccountId'),
                'sender_name': data.get('SenderName'),
                'sender_account_number': data.get('SenderAccountNumber'),
                'sender_bank': data.get('SenderBank'),
                'narration': data.get('Narration'),
                'timestamp': data.get('Timestamp'),
                # Additional virtual account fields
                'domain': data.get('Domain'),
                'channel': data.get('Channel'),
                'charged_amount': data.get('ChargedAmount'),
                'transaction_fee': data.get('TransactionFee'),
                'transaction_type': data.get('TransactionType'),
                'gateway_response_code': data.get('GatewayResponseCode'),
                'gateway_response_message': data.get('GatewayResponseMessage')
            }
        }
    
    # Return legacy format as-is
    return payload
```

#### TypeScript

```typescript
interface NormalizedPayload {
    event: string;
    data: {
        status: string;
        transaction_id: string;
        reference: string;
        amount: number;
        currency: string;
        virtual_account_id: string;
        sender_name: string;
        sender_account_number: string;
        sender_bank: string;
        narration: string;
        timestamp: string;
        domain?: string;
        channel?: string;
        charged_amount?: number;
        transaction_fee?: number;
        transaction_type?: string;
        gateway_response_code?: string;
        gateway_response_message?: string;
    };
}

function normalizePayload(payload: LegacyPayload | VirtualAccountPayload): NormalizedPayload {
    const format = detectPayloadFormat(payload);
    
    if (format === 'virtual_account') {
        const virtualPayload = payload as VirtualAccountPayload;
        // Convert virtual account format to standard format
        return {
            event: virtualPayload.notify,
            data: {
                status: virtualPayload.Data.Status?.toLowerCase() || virtualPayload.notifyType || '',
                transaction_id: virtualPayload.Data.Id,
                reference: virtualPayload.Data.TransactionReference,
                amount: virtualPayload.Data.Amount,
                currency: virtualPayload.Data.Currency,
                virtual_account_id: virtualPayload.Data.VirtualAccountId,
                sender_name: virtualPayload.Data.SenderName,
                sender_account_number: virtualPayload.Data.SenderAccountNumber,
                sender_bank: virtualPayload.Data.SenderBank,
                narration: virtualPayload.Data.Narration,
                timestamp: virtualPayload.Data.Timestamp,
                // Additional virtual account fields
                domain: virtualPayload.Data.Domain,
                channel: virtualPayload.Data.Channel,
                charged_amount: virtualPayload.Data.ChargedAmount,
                transaction_fee: virtualPayload.Data.TransactionFee,
                transaction_type: virtualPayload.Data.TransactionType,
                gateway_response_code: virtualPayload.Data.GatewayResponseCode,
                gateway_response_message: virtualPayload.Data.GatewayResponseMessage
            }
        };
    }
    
    // Return legacy format as-is
    return payload as NormalizedPayload;
}
```

## Event Type Mapping

### Wallet Funding Events

**Legacy Format:**
```json
{
  "event": "wallet_funding",
  "data": {
    "status": "successful",
    "amount": 5000,
    "virtual_account_id": "VA_123"
  }
}
```

**Virtual Account Format:**
```json
{
  "notify": "wallet_funding", 
  "notifyType": "successful",
  "Data": {
    "Amount": 5000,
    "Status": "successful", 
    "VirtualAccountId": "VA_123"
  }
}
```

### Payout Events

**Legacy Format:**
```json
{
  "event": "payout",
  "data": {
    "status": "successful",
    "amount": 10000,
    "recipient_name": "John Doe"
  }
}
```

**Virtual Account Format:**
```json
{
  "notify": "payout",
  "notifyType": "successful", 
  "Data": {
    "Amount": 10000,
    "Status": "successful",
    "RecipientName": "John Doe"
  }
}
```

## Best Practices

### 1. Format-Agnostic Processing

#### JavaScript/Node.js (Express)

```javascript
app.post('/webhook', (req, res) => {
    const normalizedPayload = normalizePayload(req.body);
    const format = detectPayloadFormat(req.body);
    
    // Process using normalized format
    processTransaction(normalizedPayload, format);
    
    res.json({ 
        status: '00', 
        message: 'Webhook processed successfully',
        payloadFormat: format
    });
});
```

#### Python (FastAPI)

```python
from fastapi import FastAPI

@app.post('/webhook')
async def webhook(payload: dict):
    normalized_payload = normalize_payload(payload)
    format_type = detect_payload_format(payload)
    
    # Process using normalized format
    await process_transaction(normalized_payload, format_type)
    
    return {
        'status': '00',
        'message': 'Webhook processed successfully',
        'payloadFormat': format_type
    }
```

#### TypeScript (Express)

```typescript
app.post('/webhook', (req: Request, res: Response) => {
    const payload = req.body;
    const normalizedPayload = normalizePayload(payload);
    const format = detectPayloadFormat(payload);
    
    // Process using normalized format
    processTransaction(normalizedPayload, format);
    
    res.json({ 
        status: '00', 
        message: 'Webhook processed successfully',
        payloadFormat: format
    });
});
```

### 2. Validation

#### JavaScript/Node.js

```javascript
function validatePayload(payload, format) {
    const required = ['status', 'amount', 'reference'];
    const data = format === 'virtual_account' ? payload.Data : payload.data;
    
    for (const field of required) {
        const value = format === 'virtual_account' 
            ? data[capitalize(field)] 
            : data[field];
            
        if (!value) {
            throw new Error(`Missing required field: ${field}`);
        }
    }
}
```

#### Python

```python
def validate_payload(payload, format_type):
    """Validate required fields in payload"""
    required_fields = ['status', 'amount', 'reference']
    data = payload.get('Data', {}) if format_type == 'virtual_account' else payload.get('data', {})
    
    for field in required_fields:
        if format_type == 'virtual_account':
            field_name = field.title().replace('_', '')  # Convert to PascalCase
            value = data.get(field_name)
        else:
            value = data.get(field)
            
        if not value:
            raise ValueError(f"Missing required field: {field}")
```

#### TypeScript

```typescript
function validatePayload(payload: any, format: PayloadFormat): void {
    const required = ['status', 'amount', 'reference'];
    const data = format === 'virtual_account' ? payload.Data : payload.data;
    
    for (const field of required) {
        const value = format === 'virtual_account' 
            ? data[capitalize(field)] 
            : data[field];
            
        if (!value) {
            throw new Error(`Missing required field: ${field}`);
        }
    }
}

function capitalize(str: string): string {
    return str.charAt(0).toUpperCase() + str.slice(1);
}
```

### 3. Logging
```javascript
function logWebhook(payload, format) {
    console.log(`Received webhook: ${format} format`, {
        event: format === 'virtual_account' ? payload.notify : payload.event,
        status: format === 'virtual_account' ? payload.notifyType : payload.data.status,
        reference: format === 'virtual_account' ? payload.Data.TransactionReference : payload.data.reference
    });
}
```

## Migration Considerations

When migrating from legacy to virtual account format:

1. **Maintain Backward Compatibility**: Support both formats during transition
2. **Field Mapping**: Create comprehensive field mapping tables
3. **Testing**: Test extensively with both format types
4. **Monitoring**: Monitor format usage to track migration progress
5. **Documentation**: Update client documentation with format examples