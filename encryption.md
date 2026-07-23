---
title: "Encryption"
description: "How Strails encrypts API request and response payloads"
---

Strails uses **Ed25519 payload encryption** to protect sensitive request and response data in transit. This ensures that only the intended recipient can read the payload contents, even if the transport is intercepted.

<Info>
Encryption is optional for some endpoints but **strongly recommended** for production integrations that handle PII, wallet addresses, or financial data.
</Info>

---

## Security model

Every request/response exchange uses two key pairs:

| Party | Key pair | Purpose |
|-------|----------|---------|
| **Fintech** | Ed25519 key pair | Encrypt data sent to Strails, decrypt responses from Strails |
| **Strails** | Ed25519 public key | Encrypt data sent to Strails |

Strails distributes its public key during onboarding. You generate your own Ed25519 key pair and provide Strails with your public key so responses can be encrypted to you.

---

## Request encryption flow

```
Your payload + Strails public key -> libsodium box seal -> encrypted base64 string -> sent in request body
```

1. Build your JSON payload as documented for the endpoint.
2. Use the Strails Ed25519 public key to seal the payload (libsodium `crypto_box_seal`).
3. Send the resulting base64-encoded string in the `payload` field of the request.

## Response decryption flow

```
Encrypted base64 response + your private key -> libsodium box seal open -> original JSON payload
```

1. Extract the `payload` field from the response body.
2. Base64-decode it.
3. Use your Ed25519 private key to unseal it (`crypto_box_seal_open`).

---

## Example: encrypting a request in Node.js

```javascript
const sodium = require('libsodium-wrappers');

await sodium.ready;

const strailsPublicKey = sodium.from_hex('STRAILS_PUBLIC_KEY_HEX');

const payload = JSON.stringify({
  bvn: '12345678901',
  userId: 'user_uniqueID',
  email: 'user@example.com',
  phoneNumber: '+2348012345678'
});

const encrypted = sodium.crypto_box_seal(
  sodium.from_string(payload),
  strailsPublicKey
);

const base64Payload = sodium.to_base64(encrypted);

// Send base64Payload in the request body
```

## Example: decrypting a response in Node.js

```javascript
const sodium = require('libsodium-wrappers');

await sodium.ready;

const yourPrivateKey = sodium.from_hex('YOUR_PRIVATE_KEY_HEX');
const yourPublicKey = sodium.from_hex('YOUR_PUBLIC_KEY_HEX');

const encrypted = sodium.from_base64(response.payload);
const decrypted = sodium.crypto_box_seal_open(
  encrypted,
  yourPublicKey,
  yourPrivateKey
);

const data = JSON.parse(sodium.to_string(decrypted));
```

---

## Generating your key pair

### Node.js

```javascript
const sodium = require('libsodium-wrappers');
await sodium.ready;

const keyPair = sodium.crypto_box_keypair();
console.log('Public key:', sodium.to_hex(keyPair.publicKey));
console.log('Private key:', sodium.to_hex(keyPair.privateKey));
```

### Python

```python
import nacl.public
import nacl.utils

keypair = nacl.public.PrivateKey.generate()
print("Public key:", keypair.public_key.encode().hex())
print("Private key:", keypair.encode().hex())
```

---

## Best practices

- **Never expose your private key** in client-side code, logs, or version control.
- Store the private key in a secure vault (e.g., AWS Secrets Manager, HashiCorp Vault).
- Rotate keys periodically and coordinate the new public key with Strails support.
- Use the Strails test environment to validate encryption/decryption before going live.

---

## Need help?

Contact [support@strails.io](mailto:support@strails.io) to receive the current Strails public key, get your public key registered, or troubleshoot encryption issues.
