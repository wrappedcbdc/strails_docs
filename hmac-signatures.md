---
title: "HMAC Signatures"
description: "Verify webhook and callback integrity using HMAC signatures"
---

Strails signs outgoing webhooks and callbacks with an HMAC-SHA256 signature so you can verify that events originated from Strails and were not tampered with in transit.

---

## Headers sent with each webhook

| Header | Description |
|--------|-------------|
| `X-Strails-Signature` | HMAC-SHA256 hex digest of the timestamp and payload |
| `X-Strails-Timestamp` | ISO 8601 timestamp of when the event was sent |
| `X-Webhook-ID` | Unique UUID for this webhook delivery attempt |
| `X-Strails-Event` | Event type, e.g. `wallet.funding.completed` |

---

## Signature format

The signature is computed over the string:

```
{timestamp}.{payload}
```

Where:

- `timestamp` is the value of `X-Strails-Timestamp`.
- `payload` is the raw JSON request body as a string (do not parse/re-serialize it before verification).

```
signature = HMAC-SHA256(webhook_secret, timestamp + "." + payload)
```

---

## Verifying a webhook in Node.js

```javascript
const crypto = require('crypto');

function verifyStrailsWebhook(req, webhookSecret) {
  const signature = req.headers['x-strails-signature'];
  const timestamp = req.headers['x-strails-timestamp'];
  const payload = JSON.stringify(req.body);

  const expected = crypto
    .createHmac('sha256', webhookSecret)
    .update(`${timestamp}.${payload}`)
    .digest('hex');

  // Constant-time comparison to prevent timing attacks
  return crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expected, 'hex')
  );
}
```

## Verifying a webhook in Python

```python
import hmac
import hashlib

def verify_strails_webhook(body: bytes, timestamp: str, signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode(),
        f"{timestamp}.{body.decode()}".encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

---

## Webhook handler template

Keep your webhook handler fast. Acknowledge receipt immediately and process the event asynchronously.

```javascript
const express = require('express');
const crypto = require('crypto');

app.post('/webhooks/strails', express.json(), (req, res) => {
  const signature = req.headers['x-strails-signature'];
  const timestamp = req.headers['x-strails-timestamp'];

  const expected = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(`${timestamp}.${JSON.stringify(req.body)}`)
    .digest('hex');

  if (!crypto.timingSafeEqual(
    Buffer.from(signature, 'hex'),
    Buffer.from(expected, 'hex')
  )) {
    return res.status(401).send('Invalid signature');
  }

  // Enqueue event for background processing
  const { eventType, payload } = req.body;
  await queue.add({ eventType, payload });

  res.sendStatus(200);
});
```

---

## Webhook secret

Set your webhook secret when configuring your webhook URL via [`/setwebhook`](/api-reference/management-api#configure-webhook-urls):

```json
{
  "webhookUrl": "https://your-domain.com/webhooks/strails",
  "secret": "your_webhook_secret_key",
  "enabled": true
}
```

## Best practices

- Use a long, randomly generated secret (at least 32 bytes).
- Store the secret in an environment variable or secret manager, never in source code.
- Verify the signature before acting on the payload.
- Reject webhooks with timestamps older than 5 minutes to mitigate replay attacks.
- Return `200 OK` quickly; do heavy work asynchronously.

---

See [Webhook Events](/api-reference/webhook-events) for the complete list of event payloads.
