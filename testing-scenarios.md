---
title: "Testing Scenarios"
description: Minimal testing scenarios for Strails IP allowlist endpoint
---

This file contains minimal, focused tests that reflect the public docs. It only documents testing for the IP allowlist management endpoint (`/manageipallowlist`).

## Base URL

```
https://beta.stablesrail.io/v1/
```

## Required headers

```http
Content-Type: application/json
Accept: */*
x-api-key: xxxx
```

## Tests

1) Add IP to allowlist

Request body:

```json
{
  "action": "add",
  "ipAddress": "102.88.108.96"
}
```

Example (cURL):

```bash
curl -X POST "https://beta.stablesrail.io/v1/manageipallowlist" \
  -H "Content-Type: application/json" \
  -H "Accept: */*" \
  -H "x-api-key: xxxx" \
  -d '{"action":"add","ipAddress":"102.88.108.96"}'
```

Expected success (200):

```json
{
  "status": "success",
  "response_code": "200",
  "message": "IP allowlist updated successfully",
  "data": { "ipAddress": "102.88.108.96", "action": "add" }
}
```

2) Remove IP from allowlist

Request body:

```json
{
  "action": "remove",
  "ipAddress": "102.88.108.96"
}
```

Expected success (200):

```json
{
  "status": "success",
  "response_code": "200",
  "message": "IP allowlist updated successfully",
  "data": { "ipAddress": "102.88.108.96", "action": "remove" }
}
```

## Notes

- Use your sandbox API key in `x-api-key` when testing.
- This repository includes a Postman collection `postman-collection.json`. The collection file begins with a repository-specific first line that must be removed before passing the file to Newman or other clients; when running tests locally you can create a cleaned copy by removing the first line (for example, with `sed '1d' postman-collection.json > /tmp/clean-collection.json`).
