# User Authentication in Zalo Mini App

## Overview

Zalo Mini App provides user authentication via the user’s Zalo account. This lets you identify the user and link them to your own backend system without a traditional username/password login form.

## Login flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Mini App   │     │ Your Server  │     │  Zalo API    │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │
       │ 1. getAccessToken()│                    │
       │────────────────────>                    │
       │                    │                    │
       │ 2. Send token      │                    │
       │────────────────────>                    │
       │                    │ 3. Verify token    │
       │                    │────────────────────>
       │                    │                    │
       │                    │ 4. Return profile  │
       │                    │<────────────────────
       │                    │                    │
       │ 5. Return user data│                    │
       │<────────────────────                    │
```

## Step 1: Get an access token from the Mini App

```javascript
import { getAccessToken } from 'zmp-sdk';

async function login() {
  try {
    const accessToken = await getAccessToken();

    // Send the token to your backend for verification
    const response = await fetch('https://your-server.com/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ accessToken })
    });
    
    const userData = await response.json();
    return userData;
  } catch (error) {
    console.error('Login failed:', error);
  }
}
```

## Step 2: Verify the token on your server

### Security requirement (mandatory since 2024-01-01)

When calling Zalo Graph APIs, your server must include `appsecret_proof` in the request headers.

```javascript
// Node.js
const crypto = require('crypto');

const ZALO_APP_SECRET_KEY = process.env.ZALO_APP_SECRET_KEY;

function calculateHMacSHA256(data, secretKey) {
  const hmac = crypto.createHmac('sha256', secretKey);
  hmac.update(data);
  return hmac.digest('hex');
}

async function getZaloProfile(accessToken) {
  const appsecretProof = calculateHMacSHA256(accessToken, ZALO_APP_SECRET_KEY);
  
  const response = await fetch(
    'https://graph.zalo.me/v2.0/me?fields=id,name,birthday,picture',
    {
      method: 'GET',
      headers: {
        'access_token': accessToken,
        'appsecret_proof': appsecretProof
      }
    }
  );
  
  return response.json();
}
```

### Example response

```json
{
  "is_sensitive": false,
  "name": "Ted Mosby",
  "id": "1234567890",
  "error": 0,
  "message": "Success",
  "picture": {
    "data": {
      "url": "https://s120-ava-talk.zadn.vn/..."
    }
  }
}
```

**Note**: `id` is unique **per Zalo App ID** (the same person may have different IDs across different Zalo apps).

## Step 3: Create/link a user in your system

```javascript
// Server-side
async function handleLogin(zaloProfile) {
  // Find an existing user in your system
  let user = await User.findOne({ zaloId: zaloProfile.id });
  
  if (!user) {
    // Create a new user
    user = await User.create({
      zaloId: zaloProfile.id,
      name: zaloProfile.name,
      avatar: zaloProfile.picture?.data?.url
    });
  }
  
  // Issue your own session token (e.g., JWT) for your app
  const token = jwt.sign({ userId: user.id }, JWT_SECRET);
  
  return { user, token };
}
```

## Linking by phone number

Use this when your existing system relies on phone numbers as the primary identifier.

### Step 1: Get a phone token from the Mini App

```javascript
import { getPhoneNumber } from 'zmp-sdk';

async function getPhone() {
  try {
    const result = await getPhoneNumber();
    // result = { token: 'encrypted_token' }

    // Send the token to your backend to decode
    const response = await fetch('https://your-server.com/api/decode-phone', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ token: result.token })
    });
    
    return response.json();
  } catch (error) {
    console.error('Get phone failed:', error);
  }
}
```

### Step 2: Decode the phone token on your server

```javascript
// Server-side - call Zalo Graph API to decode the token
async function decodePhoneToken(phoneToken, accessToken) {
  const appsecretProof = calculateHMacSHA256(accessToken, ZALO_APP_SECRET_KEY);
  
  const response = await fetch(
    'https://graph.zalo.me/v2.0/me/info',
    {
      method: 'GET',
      headers: {
        'access_token': accessToken,
        'appsecret_proof': appsecretProof,
        'code': phoneToken,
        'secret_key': ZALO_APP_SECRET_KEY
      }
    }
  );
  
  return response.json();
  // Response: { number: '84901234567', ... }
}
```

## Best Practices

### When to request the phone number

**✅ Do:**
- Request the phone number only when the user triggers a flow that truly requires it (checkout, membership enrollment, etc.).
- Show a clear in-app explanation of why you need it.

**❌ Don’t:**
- Request it immediately on app launch.
- Request it without explaining the purpose.

### Recommended flow

```
┌─────────────────────────────────────────┐
│  User opens the Mini App                │
│  → Silent sign-in via Zalo              │
│  → Can use basic features              │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│  User starts a phone-required action    │
│  (e.g., checkout)                       │
│  → Show explanation UI                  │
│  → Request phone number                 │
└─────────────────────────────────────────┘
```

### Onboarding for apps that require a phone number

If your app cannot function without a phone number, implement an onboarding screen/modal to set user expectations.

```jsx
function OnboardingScreen() {
  return (
    <div className="onboarding">
      <h2>Welcome</h2>
      <p>To unlock all features, we need your phone number to:</p>
      <ul>
        <li>Look up membership information</li>
        <li>Notify you about order status</li>
        <li>Contact you when necessary</li>
      </ul>
      <button onClick={handleRequestPhone}>
        Allow
      </button>
    </div>
  );
}
```

## Important notes

1. **Never call Zalo server-side APIs from the Mini App client**. Use your backend to protect secrets.

2. **Access tokens expire**. Handle expiration and re-authentication gracefully.

3. **User IDs are per-app**. The same Zalo user can have different IDs across different Zalo apps.

4. **`getPhoneNumber` requires approval**. Request the capability in Mini App Center under **Permissions management**.
