# Development Tools — Zalo Mini App

This document summarizes the official tooling commonly used to build, test, debug, and deploy Zalo Mini Apps.

## Zalo Mini App CLI

### Installation

```bash
npm install -g zmp-cli
```

Verify:

```bash
zmp --help
```

### Common commands

#### Login

```bash
zmp login
```

This typically opens a browser to authenticate with your Zalo account.

#### Initialize a project

```bash
zmp init my-app
```

Creates a new Mini App project using a template.

#### Start development server

```bash
zmp start
```

Starts the dev server with live reload.

#### Deploy

```bash
# Deploy to Development
zmp deploy

# Deploy to Testing
zmp deploy --testing
```

### Environment configuration

`.env` example:

```env
MINI_APP_ID=your_mini_app_id
```

## Zalo Mini App Extension (VS Code)

### Installation

1. Open VS Code
2. Search for **“Zalo Mini App”** in Extensions
3. Install the extension

### Key features

- Project management (multiple Mini Apps)
- Quick deploy from VS Code
- Device Mode testing on real phones
- Simulator preview
- Console log viewer

### Device Mode

Device Mode lets you run the Mini App on a real phone while syncing local code.

Typical flow:
1. Open the extension panel
2. Select the **Device** tab
3. Scan the QR code using your phone
4. Code changes sync in real time

#### Direct connection (Android)

Requirements:
- Android Debug Bridge (adb) installed
- Android device connected via USB

```bash
adb --version
adb devices
```

## Debugging

### Live debugging

Append `?zDebug=true` to the deep link:

`https://zalo.me/s/<mini_app_id>/?zDebug=true`

### Remote debugging (Android)

Use Chrome DevTools:
- open `chrome://inspect`
- locate the Mini App WebView
- click **Inspect**

## Operational notes

- Prefer **Device Mode** for native features (camera, location, payments).
- Ensure your backend has CORS for `https://h5.zdn.vn`.
- Respect bundle constraints (10MB total / 3MB per file).

