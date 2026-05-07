# Troubleshooting — Zalo Mini App

This guide covers common issues in Zalo Mini App development and provides actionable fixes. Use it as the first stop before escalating to platform support.

## 1) Network errors

### Common causes

- **CORS** (most common)
- API endpoint is not **HTTPS**
- Expired/invalid TLS certificate
- Disallowed custom headers
- General connectivity issues

### 1.1) CORS errors

**Symptoms**: The API works in Postman/localhost but fails in the Mini App.

**Fix (server-side)**: allow Zalo Mini App origins (at minimum `https://h5.zdn.vn`) and handle preflight.

```js
// Express.js example
const allowedOrigins = ["https://h5.zdn.vn", "http://localhost:3000"];

app.use((req, res, next) => {
  const origin = req.headers.origin;

  if (allowedOrigins.includes(origin)) {
    res.header("Access-Control-Allow-Origin", origin);
  }

  res.header("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS");
  res.header("Access-Control-Allow-Headers", "Content-Type, Authorization");

  if (req.method === "OPTIONS") {
    return res.sendStatus(200);
  }
  next();
});
```

**Pitfall**:

```
❌ Access-Control-Allow-Origin: https://h5.zdn.vn,http://localhost:3000
✅ Access-Control-Allow-Origin: https://h5.zdn.vn
```

### 1.2) Non-HTTPS endpoints

Mini Apps must use HTTPS.

```js
// ❌ Invalid
fetch("http://api.example.com/data");
fetch("https://192.168.1.100:3000/api"); // IP-based endpoints often fail

// ✅ Valid
fetch("https://api.example.com/data");
```

## 2) Minified React errors

**Symptom**: `Minified React error #...` in production builds.

**Cause**: Production builds compress React error messages.

**Fix**: open the linked decoder page from the error message; review common React issues:
- hooks used conditionally
- setting state during render
- invalid element rendering

## 3) Images not rendering after deploy

```jsx
// ❌ Not reliable after build
<img src="/coffee.jpg" />
<img src="../public/coffee.jpg" />

// ✅ Import the asset (bundled)
import coffee from "./coffee.jpg";
<img src={coffee} />

// ✅ Or host via CDN
<img src="https://cdn.example.com/coffee.jpg" />
```

## 4) Calling server-to-server APIs from the Mini App (security violation)

Do **not** call these from client code:
- decoding phone/location tokens
- sending notifications
- any Open API requiring API keys/secrets

**Fix**: move the logic to your backend and expose a minimal client-facing endpoint.

## 5) API succeeds but returns no data (development environment)

**Cause**: Some ZMP SDK flows (e.g., `getAccessToken`, `createOrder`) require real device context and may not behave correctly in a browser-like simulator.

**Fix**: use **Device Mode** on a real phone.

## 6) Live issues with no visibility

### Option A: Enable in-app debugging

Append `?zDebug=true` to your deep link:

`https://zalo.me/s/<mini_app_id>/?zDebug=true`

Use the debug UI to inspect Console / Network / Elements.

### Option B: Remote debugging (Android)

1. Enable USB debugging
2. Connect the phone via USB
3. Open `chrome://inspect`
4. Locate the Mini App WebView and click **Inspect**

## 7) Issues only affect non-developer users

**Cause**: missing platform approval for sensitive permissions (e.g., `getPhoneNumber`, `getLocation`, `openMediaPicker`, `requestCameraPermission`, `keepScreen`, native storage).

**Fix**: request and obtain approval in Mini App Center → **Permissions management**, then deploy the approved version to Testing/Live.

## 8) “This page is not found or invalid”

**Cause**: opening Development/Testing versions with an account that is not in the Developer/Admin allowlist.

**Fix**: use an authorized account or publish a Live version.

## 9) “This application is under development”

**Cause**: the Mini App has not published any Live version yet.

**Fix**: publish at least one version to Live.

## 10) Mini App not discoverable in the store

Common causes:
- the app is blocked from search due to policy violations
- the app uses a traditional username/password login flow

Fix:
- align authentication with Zalo Mini App standards
- submit an updated version and follow platform instructions for re-enabling search

## 11) Extension copy/paste issues (macOS)

Workaround: select text → menu **Edit** → **Copy/Paste**.

## 12) `Cannot find module '@vitejs/plugin-react-refresh'`

Update Vite config to use the modern React plugin.

```js
// ❌ Old
import reactRefresh from "@vitejs/plugin-react-refresh";
plugins: [reactRefresh()]

// ✅ New
import react from "@vitejs/plugin-react";
plugins: [react()]
```

## 13) Deployment quota exceeded

You may hit:

`You have reached your 30-day deployment limit`

Typical quotas:
- Development: 300/month
- Testing: 60/month

## 14) File size / bundle size limits

Limits:
- total bundle: 10MB
- per file: 3MB

Mitigations:
- host large assets on CDN
- enable code splitting

## 15) CI/CD “Permission denied. Please login again.”

Validate:
- `ZALO_APP_SECRET` and `ZALO_REFRESH_TOKEN` match `ZALO_APP_ID`
- do not confuse `MINI_APP_ID` with `ZALO_APP_ID`
- no environment variable overrides for auth tokens

## 16) ES2015 transform error

```
Transforming async generator functions to "es2015" is not supported
```

Options:
- raise build target (e.g., `esnext`)
- switch to compatible dependencies

## 17) PDF viewing differences (iOS vs Android)

**Issue**: different default behavior across platforms.

**Fix**: render PDFs inside the app (e.g., `react-pdf@5.x`) if you need consistent UX.

## Debug decision tree

```
Where does the issue happen?
│
├─ Live only?
│   ├─ Only non-dev users?
│   │   └─ Check permission approval in Mini App Center
│   └─ No visibility?
│       └─ Use ?zDebug=true / remote debug
│
├─ Device Mode only?
│   ├─ Native SDK feature?
│   │   └─ Expected: test on real device; verify permissions
│   └─ Connectivity?
│       └─ Same Wi‑Fi, disable VPN, retry QR pairing
│
└─ Network errors?
    ├─ CORS?
    │   └─ Allow https://h5.zdn.vn
    └─ HTTPS?
        └─ Use valid HTTPS endpoints only
```

