# Use Cases — End-to-End Reference Implementations

This document provides **copy-pasteable** reference implementations for common Zalo Mini App flows. The patterns here prioritize:
- security (backend verification, no secrets in client)
- compatibility (permission + version considerations)
- UX (permission prompts at the point of need)

> These examples are illustrative. Adapt to your project architecture (monolith, BFF, serverless) and data model.

---

## Use case 1: Authentication (silent sign-in)

### Client-side (Mini App): Auth context

```jsx
// src/contexts/AuthContext.jsx
import { createContext, useContext, useEffect, useRef, useState } from "react";
import { getAccessToken, getUserInfo } from "zmp-sdk";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const sessionTokenRef = useRef(null);

  useEffect(() => {
    initAuth();
  }, []);

  const initAuth = async () => {
    try {
      setLoading(true);
      setError(null);

      // 1) Get access token from Zalo
      const accessToken = await getAccessToken();

      // 2) Get basic user info (name, avatar)
      const { userInfo } = await getUserInfo({ autoRequestPermission: true });

      // 3) Send token to backend for verification + user linkage
      const response = await fetch("https://your-server.com/api/auth/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ accessToken, zaloUserInfo: userInfo }),
      });

      if (!response.ok) throw new Error("Login failed");
      const { user: serverUser, sessionToken } = await response.json();

      // 4) Keep session token in memory only (cleared on reload)
      sessionTokenRef.current = sessionToken;

      setUser({ ...serverUser, zaloInfo: userInfo });
    } catch (err) {
      console.error("Auth error:", err);
      setError(err?.message ?? "Authentication failed");
    } finally {
      setLoading(false);
    }
  };

  const logout = () => {
    sessionTokenRef.current = null;
    setUser(null);
  };

  const getSessionToken = () => sessionTokenRef.current;

  return (
    <AuthContext.Provider value={{ user, loading, error, logout, refresh: initAuth, getSessionToken }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);
```

### Server-side (Node.js): verify token, issue session token

```js
// server/routes/auth.js
const express = require("express");
const crypto = require("crypto");
const jwt = require("jsonwebtoken");

const router = express.Router();

const ZALO_APP_SECRET_KEY = process.env.ZALO_APP_SECRET_KEY;
const JWT_SECRET = process.env.JWT_SECRET;

function appsecretProof(accessToken) {
  return crypto.createHmac("sha256", ZALO_APP_SECRET_KEY).update(accessToken).digest("hex");
}

async function verifyZaloToken(accessToken) {
  const resp = await fetch("https://graph.zalo.me/v2.0/me?fields=id,name,picture", {
    method: "GET",
    headers: {
      access_token: accessToken,
      appsecret_proof: appsecretProof(accessToken),
    },
  });

  const data = await resp.json();
  if (data.error !== 0) throw new Error(data.message || "Invalid token");
  return data;
}

router.post("/login", async (req, res) => {
  try {
    const { accessToken } = req.body;

    // 1) Verify token with Zalo Graph API
    const zaloProfile = await verifyZaloToken(accessToken);

    // 2) Find or create user in your DB
    const user = await upsertUserFromZaloProfile(zaloProfile);

    // 3) Issue your app session token
    const sessionToken = jwt.sign({ userId: user.id, zaloId: user.zaloId }, JWT_SECRET, {
      expiresIn: "7d",
    });

    res.json({ user, sessionToken });
  } catch (error) {
    res.status(401).json({ error: error.message });
  }
});

module.exports = router;
```

---

## Use case 2: Link phone number (contextual prompt + server decode)

### Client-side: request phone token when needed

```js
import { getAccessToken, getPhoneNumber } from "zmp-sdk";
import { useAuth } from "../contexts/AuthContext";

export function usePhoneNumber() {
  const { getSessionToken } = useAuth();

  return async function requestPhoneNumber() {
    // Show explanation UI before calling this in a real app
    const { token: phoneToken } = await getPhoneNumber();
    const accessToken = await getAccessToken();

    const resp = await fetch("https://your-server.com/api/user/phone", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${getSessionToken()}`,
      },
      body: JSON.stringify({ phoneToken, accessToken }),
    });

    if (!resp.ok) throw new Error("Unable to decode phone number");
    return await resp.json();
  };
}
```

### Server-side: decode phone token

Use the patterns in `open-apis.md` and always validate/handle used/expired tokens.

---

## Use case 3: Checkout payment (server MAC + createOrder)

See `payment.md` for the secure integration pattern:
- backend generates MAC
- client calls `createOrder()`
- backend verifies webhook callbacks

---

## Use case 4: Location-based experience (permission + fallback)

Use `getLocation()` only after user intent (e.g., “Find nearby stores”). If denied, provide a manual fallback (search input / city selector).

---

## Recommended project structure (example)

```
my-mini-app/
├── app-config.json
├── src/
│   ├── app.jsx
│   ├── contexts/
│   ├── pages/
│   ├── components/
│   └── css/
└── public/
```

