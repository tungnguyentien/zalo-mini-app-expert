# AI Agent Instructions — Zalo Mini App Development

This document defines guidance for an AI agent assisting with **Zalo Mini App** development. Treat it as the top-level ruleset that governs recommendations, code examples, and troubleshooting.

## Platform context (high-level)

Zalo Mini App is a web application running inside **Zalo’s in-app WebView**. It can access Zalo-native capabilities via **ZMP SDK** through a JS bridge (ZJSBridge).

Key differences vs a normal web app:
- **Runtime**: Zalo WebView (not a standard browser environment)
- **Identity**: Zalo user context is available; traditional login forms are generally discouraged
- **Native bridge**: camera, storage, sharing, device features
- **Distribution**: QR / deep link / OA entry points / Mini App store

## Non-negotiable priorities (always in this order)

1. **Security**: never leak secrets; verify/compute sensitive values server-side.
2. **Compatibility**: check minimum SDK/Zalo versions and permission requirements before proposing an API.
3. **Performance**: respect bundle constraints; prefer small dependencies, code-splitting, caching.
4. **User experience**: mobile-first patterns; request permissions contextually.

## Mandatory rules

### 1) Never call Zalo server-side APIs from the client

Do not recommend calling endpoints that require **secret keys**, **API keys**, or **OA access tokens** from the Mini App frontend.

```js
// ❌ Never do this in the client (secrets will leak)
await fetch("https://graph.zalo.me/v2.0/me/info", {
  headers: {
    access_token: accessToken,
    appsecret_proof: appsecretProof, // derived using secret key
    secret_key: SECRET_KEY,          // secret!
  },
});

// ✅ Correct pattern: client -> your backend -> Zalo API
await fetch("https://your-server.com/api/user/phone", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ accessToken, phoneToken }),
});
```

### 2) Always validate compatibility before proposing an API

Before recommending an API, check:
- Minimum **ZMP SDK** version
- Minimum **Zalo app** version (iOS/Android)
- Whether the API requires **platform approval** and/or **user consent**

Reference: `compatibility.md`.

### 3) Always handle error codes and fallbacks

```js
// ❌ Bad: no error handling
const token = await getAccessToken();

// ✅ Good: structured error handling
try {
  const token = await getAccessToken();
  if (!token) throw new Error("Empty token");
  // continue...
} catch (error) {
  if (error?.code === -1401) {
    // User authentication required
    showAuthorizationUI();
  } else if (error?.code === -1404) {
    // API not supported in this Zalo version
    showUpdateZaloPrompt();
  } else {
    showGenericError(error?.message ?? "Unknown error");
  }
}
```

### 4) Treat development vs production as different products

| Scenario | Development/Testing | Live (production) |
|---|---|---|
| Who can access | Developer/Admin | All users |
| Permissions | Often available for testing | May require platform approval |
| Debugging | Full access | typically requires `?zDebug=true` for in-app debug tools |

## Decision trees (quick routing)

### Storage selection

```
Need to store data?
│
├─ Temporary/session data?
│   └─ Use Web Storage (setStorage/getStorage)
│      - no permission approval
│      - cleared when the app is closed
│
├─ Persistent data?
│   └─ Use Native Storage (setItem/getItem)
│      - may require permission approval
│      - persists across app restarts
│
└─ Sensitive data (credentials, secrets)?
    └─ Store server-side only
       - keep only short-lived session tokens on device
       - validate on the backend
```

### Phone number strategy

```
Need user identification?
│
├─ Unique ID is sufficient?
│   └─ Use Zalo User ID (getUserID)
│      - no user prompt
│      - per-app unique identifier
│
├─ Must link to a legacy system by phone number?
│   └─ Use getPhoneNumber (requires approval + user consent)
│      - request contextually with explanation UI
│      - decode token server-side
│
└─ Need to contact users?
    └─ Prefer OA follow + OA messaging (no phone number required)
```

### When to use simulator vs device mode

```
What are you testing?
│
├─ Layout/basic UI?
│   └─ Simulator is usually sufficient
│
├─ Native features (camera, location, payment)?
│   └─ Use Device Mode (real phone required)
│
└─ Live-only issues?
    └─ Use Remote Debug / `?zDebug=true`
```

## Common mistakes (signals to warn about)

- Permission prompts on app mount (e.g., `getPhoneNumber()` in a `useEffect([])`).
- HTTP or IP-based API endpoints (Mini Apps require HTTPS).
- CORS not allowing `https://h5.zdn.vn`.
- Calling `graph.zalo.me` / `openapi.zalo.me` directly from client code.
- Ignoring safe-area insets (notch / home indicator issues).
- Large dependencies that bloat bundles (e.g., importing full `lodash` / `moment`).

## Reference map (topic -> file)

| Topic | File |
|---|---|
| Platform overview | `general.md` |
| ZMP SDK (client-side) | `zmp-sdk-api.md` |
| Open APIs (server-side) | `open-apis.md` |
| ZaUI React | `zaui-react.md` |
| Authentication | `authentication.md` |
| Permissions | `permissions.md` |
| Payments | `payment.md` |
| App config | `app-config.md` |
| Devtools | `devtools.md` |
| Deployment | `deployment.md` |
| Compatibility | `compatibility.md` |
| Troubleshooting | `troubleshooting.md` |
| Best practices | `best-practices.md` |
| End-to-end examples | `use-cases.md` |

