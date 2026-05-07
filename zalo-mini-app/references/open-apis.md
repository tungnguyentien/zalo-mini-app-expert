# Open APIs — Server-side Integration Guide

This document describes **server-side** integrations for Zalo Mini App:
- verifying Zalo access tokens and fetching user profile data
- decoding sensitive tokens (phone number, location)
- sending notifications
- receiving and verifying webhooks

> **Security requirement**: never call these APIs directly from the Mini App client. Use your backend to protect secrets and credentials.

## 1) Credentials and setup

### Where to get credentials

In [Mini App Center](https://mini.zalo.me/developers/):
- **Mini App ID**
- **Zalo App ID**
- **Zalo App Secret Key**
- **API Key** (Open APIs section)

### `appsecret_proof` (mandatory)

All calls to `graph.zalo.me` require `appsecret_proof` in headers. It is an HMAC-SHA256 of the access token using your **Zalo App Secret Key**.

```js
// Node.js
const crypto = require("crypto");

function calculateAppsecretProof(accessToken, secretKey) {
  return crypto.createHmac("sha256", secretKey).update(accessToken).digest("hex");
}
```

```python
# Python
import hmac
import hashlib

def calculate_appsecret_proof(access_token: str, secret_key: str) -> str:
    return hmac.new(
        secret_key.encode("utf-8"),
        access_token.encode("utf-8"),
        hashlib.sha256,
    ).hexdigest()
```

## 2) User profile (server-side)

### Verify token and fetch profile

**Endpoint**: `GET https://graph.zalo.me/v2.0/me`

**Headers**
| Header | Value | Required |
|---|---|---|
| `access_token` | token from `getAccessToken()` | ✅ |
| `appsecret_proof` | HMAC-SHA256(access_token, app_secret) | ✅ |

**Query**
| Param | Description |
|---|---|
| `fields` | e.g. `id,name,birthday,picture` |

```js
// server/services/zaloAuth.js
const crypto = require("crypto");

const ZALO_APP_SECRET_KEY = process.env.ZALO_APP_SECRET_KEY;

function calculateAppsecretProof(accessToken) {
  return crypto
    .createHmac("sha256", ZALO_APP_SECRET_KEY)
    .update(accessToken)
    .digest("hex");
}

async function getZaloProfile(accessToken) {
  const appsecretProof = calculateAppsecretProof(accessToken);

  const resp = await fetch(
    "https://graph.zalo.me/v2.0/me?fields=id,name,birthday,picture",
    {
      method: "GET",
      headers: {
        access_token: accessToken,
        appsecret_proof: appsecretProof,
      },
    },
  );

  const data = await resp.json();
  if (data.error !== 0) throw new Error(data.message || "Failed to get profile");
  return data;
}

module.exports = { getZaloProfile, calculateAppsecretProof };
```

## 3) Decode sensitive tokens (phone / location)

### Decode phone number token

**Endpoint**: `GET https://graph.zalo.me/v2.0/me/info`

**Headers**
| Header | Value | Required |
|---|---|---|
| `access_token` | token from `getAccessToken()` | ✅ |
| `appsecret_proof` | HMAC-SHA256(access_token, app_secret) | ✅ |
| `code` | token from `getPhoneNumber()` | ✅ |
| `secret_key` | Zalo App Secret Key | ✅ |

```js
async function decodePhoneToken(phoneToken, accessToken) {
  const appsecretProof = calculateAppsecretProof(accessToken);

  const resp = await fetch("https://graph.zalo.me/v2.0/me/info", {
    method: "GET",
    headers: {
      access_token: accessToken,
      appsecret_proof: appsecretProof,
      code: phoneToken,
      secret_key: process.env.ZALO_APP_SECRET_KEY,
    },
  });

  const data = await resp.json();
  if (data.error !== 0) throw new Error(data.message || "Failed to decode phone");

  return { phoneNumber: data.data?.number };
}
```

### Decode location token

The decode mechanism is the same endpoint (`/me/info`) with a different `code` (from `getLocation()`).

## 4) Notifications (server-side)

### Send template notification to Mini App users

**Endpoint**: `POST https://openapi.mini.zalo.me/notification/template`

**Headers**
| Header | Value | Required |
|---|---|---|
| `X-Api-Key` | `Bearer <api-key>` | ✅ |
| `X-User-Id` | Zalo User ID | ✅ |
| `X-MiniApp-Id` | Mini App ID | ✅ |
| `Content-Type` | `application/json` | ✅ |

**Constraints (typical)**:
- user must have interacted recently (e.g., within 7 days)
- user must have opted in via `requestSendNotification`
- daily limits apply per app and per user

## 5) Webhooks

### Setup

In Mini App Center → **Settings** → **Webhook**:
- set webhook URL (your backend)
- select events

### Signature verification

Webhooks include a signature header (e.g., `x-zevent-signature`). Always verify before processing.

```js
const crypto = require("crypto");

function generateSignature(body, apiKey) {
  const keys = Object.keys(body).sort();
  let content = "";
  for (const key of keys) {
    const value = typeof body[key] === "object" ? JSON.stringify(body[key]) : body[key];
    content += value;
  }
  return crypto.createHash("sha256").update(`${content}${apiKey}`).digest("hex");
}

function verifyWebhook(req, apiKey) {
  const signature = req.headers["x-zevent-signature"];
  const expected = generateSignature(req.body, apiKey);
  return signature === expected;
}
```

## Error handling guidance

- Treat all external calls as fallible (timeouts, invalid tokens, used codes).
- Do not log secrets; if you must log, redact values (e.g., keep first 6 chars).
- Implement rate limiting and caching on your backend to reduce dependence on upstream calls.

