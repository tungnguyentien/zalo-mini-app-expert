# Permissions — Zalo Mini App

This document explains permission concepts in Zalo Mini Apps: which permissions exist, how to obtain platform approval, how to request user consent, and how to design a permission UX that passes review and protects conversion.

## Permission categories (conceptual)

1. **User device permissions**
   - Location
   - Camera / microphone
   - Storage

2. **User information permissions**
   - Phone number (`scope.userPhonenumber`)
   - User profile information (`scope.userInfo`)

   **Note**: some user information permissions can be time-bound and may be revoked after expiration.

3. **Zalo feature permissions**
   - QR scanning
   - OA QR display
   - Share / Follow OA

4. **Mini App capability permissions**
   - navigation bar configuration
   - open/close Mini App
   - persistent storage APIs

## Permission matrix (selected)

| API | Category | Requires platform approval | Requires user consent |
|---|---|---:|---:|
| `getPhoneNumber` | User Information | ✅ | ✅ |
| `getLocation` | User Device | ✅ | ✅ |
| `openMediaPicker` | User Device | ✅ | - |
| `requestCameraPermission` | User Device | ✅ | ✅ |
| `keepScreen` | Mini App | ✅ | - |
| Native Storage APIs | Mini App | ✅ | - |
| `getUserInfo` | User Information | - | ✅ |

## Platform approval (developer action)

### Step 1: Open Mini App Center

Go to [Mini App Center](https://mini.zalo.me/developers/) → select your Mini App → **Permissions management**.

### Step 2: Select required permissions

Select only what your app truly needs.

### Step 3: Provide justification and evidence

Prepare:
- **Reason**: why the permission is necessary (business + UX rationale)
- **Screenshots**: UI screenshots showing the permission usage context
- **User-facing description** (optional): text shown to users

### Step 4: Submit and wait for review

Approvals are typically evaluated during version review.

## Requesting user consent (client-side)

### Request scopes via `authorize()`

```js
import { authorize } from "zmp-sdk";

export async function requestUserScopes() {
  const result = await authorize({
    scopes: ["scope.userInfo", "scope.userPhonenumber"],
  });

  if (result["scope.userInfo"]) {
    // User granted profile access
  }

  if (result["scope.userPhonenumber"]) {
    // User granted phone access
  }
}
```

### Location permission

```js
import { getLocation } from "zmp-sdk";

export async function getCurrentLocation() {
  try {
    return await getLocation();
  } catch (error) {
    // Use error.code to decide UX fallback
    throw error;
  }
}
```

### Camera permission

```js
import { checkZaloCameraPermission, requestCameraPermission } from "zmp-sdk";

export async function ensureCameraPermission() {
  const hasPermission = await checkZaloCameraPermission();
  if (!hasPermission) {
    await requestCameraPermission();
  }
}
```

## UX best practices

### 1) Request permissions at the point of need

✅ Do:
- ask for the permission only when the user initiates a feature that requires it
- show a short explanation screen/modal before triggering the permission prompt

❌ Don’t:
- request on app mount
- retry prompts aggressively after denial

### 2) Handle denial paths explicitly

Use `getSetting()` to detect previously denied scopes and guide users to settings when necessary.

```js
import { getSetting, openPermissionSetting } from "zmp-sdk";

export async function ensureLocationPermission() {
  const settings = await getSetting();
  if (settings["scope.userLocation"] === false) {
    openPermissionSetting();
    return;
  }
  // Otherwise request normally (when user performs the action)
}
```

## Operational notes

- Developer/Admin accounts may appear to have broader access during testing; always validate behavior on real users in Live.
- Make sure your review submission clearly documents permission usage and includes the required screenshots.
