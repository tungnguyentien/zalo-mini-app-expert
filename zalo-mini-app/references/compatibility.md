# Compatibility & Platform Differences (iOS / Android)

This document provides a practical compatibility reference for Zalo Mini App development: minimum SDK / Zalo versions, platform differences, and safe feature-detection patterns.

> Always validate with the latest official documentation before shipping.

## 1) Compatibility matrix (selected)

| API / Feature | Min SDK | Min Zalo | Notes |
|---|---:|---|---|
| `getAccessToken` | 2.0.0 | - | core |
| `getUserInfo` | 2.0.0 | - | may require user consent |
| `getUserID` | 2.0.0 | - | per-app unique identifier |
| `getSystemInfo` | 2.0.0 | - | device + runtime info |
| `configAppView` | 2.25.0 | 23.02.01.r2 | runtime view configuration |
| `closeLoading` | 2.17.0 | - | used with `selfControlLoading` |
| `setItem/getItem` (native storage) | 2.0.0 | - | may require platform approval |
| `setStorage/getStorage` (web storage) | 2.0.0 | - | session-scoped |
| `getLocation` | 2.0.0 | - | requires approval + user consent |
| `getPhoneNumber` | 2.0.0 | - | requires approval + user consent; decode server-side |
| Camera context APIs | 2.20.0 | 22.05.02 | requires camera permission |
| `openBioAuthentication` | 2.20.0 | 22.05.02 | biometric support varies by device |
| `createOrder` | 2.0.0 | - | Checkout SDK |

### App config properties (selected)

| Property | Min SDK | Min Zalo | Notes |
|---|---:|---|---|
| `statusBar: "transparent"` / `"hidden"` | 2.25.0 | 23.02.01.r2 | full-screen experiences |
| `actionBarHidden` | 2.25.0 | 23.02.01.r2 | hide default navigation bar |
| `hideAndroidBottomNavigationBar` | 2.25.0 | 23.02.01.r2 | Android-only UX control |
| `hideIOSSafeAreaBottom` | 2.25.0 | 23.02.01.r2 | iOS-only UX control |
| `selfControlLoading` | 2.17.0 | - | dismiss splash via `closeLoading()` |

## 2) iOS vs Android differences (practical)

### Safe area handling

| Platform | Typical behavior | Recommended handling |
|---|---|---|
| iOS | notch + home indicator | use `env(safe-area-inset-*)` |
| Android | notch + gesture/navigation bar varies | use `getSystemInfo().safeArea` where needed |

```css
.container {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}
```

```js
import { getSystemInfo } from "zmp-sdk";

const { safeArea } = await getSystemInfo();
// safeArea = { top, bottom, left, right, width, height }
```

### PDF viewing

| Platform | Typical behavior |
|---|---|
| iOS | may open in Safari/Preview |
| Android | may require an external PDF reader |

If you require a consistent in-app PDF UX, render PDFs inside the Mini App (library choice depends on your stack and constraints).

## 3) Feature detection patterns

### Compare versions

```js
function compareVersion(v1, v2) {
  const a = v1.split(".").map(Number);
  const b = v2.split(".").map(Number);
  for (let i = 0; i < Math.max(a.length, b.length); i++) {
    const x = a[i] || 0;
    const y = b[i] || 0;
    if (x > y) return 1;
    if (x < y) return -1;
  }
  return 0;
}
```

### Progressive enhancement

```js
async function tryFeature(fn, fallback) {
  try {
    return await fn();
  } catch (err) {
    if (err?.code === -1404) return fallback();
    throw err;
  }
}
```

## 4) Environment detection (heuristic)

```js
import { getSystemInfo } from "zmp-sdk";

export async function detectEnvironment() {
  const systemInfo = await getSystemInfo();

  return {
    isSimulator: window.location.hostname === "localhost",
    isProduction: window.location.hostname === "h5.zdn.vn",
    platform: systemInfo.platform,
    zaloVersion: systemInfo.zaloVersion,
    sdkVersion: systemInfo.version,
    language: systemInfo.language,
  };
}
```

