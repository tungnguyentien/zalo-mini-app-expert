# Best Practices — Zalo Mini App

This document summarizes recommended engineering practices for Zalo Mini App projects, with emphasis on security, compatibility, performance, and UX.

## 1) Authentication & user identity

### Use Zalo identity (no traditional login forms)

```js
// ✅ Recommended: use Zalo identity and verify server-side
import { getAccessToken } from "zmp-sdk";

export async function login() {
  const accessToken = await getAccessToken();
  // Send to backend for verification and session issuance
  return await api.login({ accessToken });
}
```

Avoid username/password forms unless explicitly required by policy and design constraints.

### Request phone number contextually

```js
// ❌ Do not request on app mount
useEffect(() => {
  getPhoneNumber();
}, []);

// ✅ Request only when needed (checkout, membership, etc.)
const handleCheckout = async () => {
  const { token: phoneToken } = await getPhoneNumber();
  // Continue checkout...
};
```

## 2) API calls & backend integration

### Always implement robust error handling

```js
export async function fetchJson(url, options) {
  const resp = await fetch(url, options);
  if (!resp.ok) throw new Error(`HTTP ${resp.status}`);
  return await resp.json();
}
```

### Never call Open APIs from the client

```js
// ❌ Never embed secrets/tokens in client code
await fetch("https://openapi.zalo.me/...", {
  headers: { access_token: "OA_ACCESS_TOKEN" },
});

// ✅ Use backend proxy endpoints
await fetch("https://your-server.com/api/oa/send-message", { method: "POST" });
```

### Use retries carefully

Use exponential backoff for transient failures. Avoid infinite retries.

## 3) Performance & bundle constraints

### Prefer smaller dependencies

```js
// ❌ Avoid large default imports
import _ from "lodash";
import moment from "moment";

// ✅ Prefer selective imports / lighter alternatives
import debounce from "lodash/debounce";
import dayjs from "dayjs";
```

### Code splitting / lazy loading

Lazy-load heavy routes/components where appropriate to improve first load.

## 4) UI/UX guidelines

- Provide clear loading, error, and empty states.
- Respect safe-area insets for notch/home indicators.
- Use ZaUI components for consistency with mobile design patterns.
- Ask for permissions with explanation UI and only at the point of need.

## 5) Storage

- Use `setStorage/getStorage` for session-scoped data (no approval).
- Use native storage (`setItem/getItem`) only when persistence is required and permissions are approved.
- Never store secrets in client storage.

## 6) Security checklist

- Validate all client input server-side.
- Verify payment callbacks (MAC/signature) before updating order state.
- Configure CORS for `https://h5.zdn.vn`.
- Do not log secrets (redact tokens and MAC values).

## 7) Testing checklist

- Test on both iOS and Android.
- Use Device Mode for native features (camera, location, payments).
- Reproduce Live-only issues using `?zDebug=true` and/or remote debugging.

