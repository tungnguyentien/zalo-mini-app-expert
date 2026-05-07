# Zalo Mini App Overview

## What is a Zalo Mini App?

Zalo Mini App is a lightweight web application that runs inside Zalo’s in-app WebView and can access native capabilities through the ZMP SDK.

### Key characteristics

- **No installation required**: runs directly inside Zalo.
- **Optimized runtime**: designed for Zalo’s WebView environment.
- **Zalo-native features**: share, OA, payments, etc.
- **Trusted distribution**: reviewed and distributed within the Zalo ecosystem.
- **Large reach**: multiple entry points (QR, deep link, OA, store).
- **Small bundle constraints**: max 10MB total, 3MB per file.

### Technology

Zalo Mini Apps are built with:
- **HTML/CSS/JavaScript** (standard web technologies)
- **React/Vue** (supported frameworks)
- **Vite** (default build tool)
- **ZJSBridge** (JavaScript bridge to Zalo native features)

## Development lifecycle

### 1) Create a Mini App

```
1. Register/use a Zalo App at `https://developers.zalo.me/`
2. Go to `https://mini.zalo.me/developers/`
3. Select your Zalo App → create a Mini App
4. Fill in required information and obtain a **Mini App ID**
```

### 2) Verify your Mini App

- **OA verification**: link an Official Account (OA)
- **Document verification**: provide business/legal documents (if required)

### 3) Build your Mini App

```bash
# Install the CLI
npm install -g zmp-cli

# Initialize a new project
zmp init my-app

# Start the development server
zmp start

# Build and deploy
zmp deploy
```

### 4) Release your Mini App

1. Open the Mini App management page
2. Submit for review
3. Wait for Zalo’s review decision
4. Publish after approval

## Typical project structure

```
my-mini-app/
├── app-config.json       # Mini App configuration
├── package.json
├── vite.config.js        # Vite configuration
├── index.html            # Entry point
├── src/
│   ├── app.jsx           # Root component
│   ├── pages/            # Pages
│   ├── components/       # Components
│   └── css/              # Styles
└── public/               # Static assets
```

## Install ZMP SDK

```bash
npm install zmp-sdk
```

### Import and usage

```javascript
import { getAccessToken, getUserInfo } from 'zmp-sdk';

// Get an access token
const token = await getAccessToken();

// Get user information
const userInfo = await getUserInfo();
```

## Entry points

A Mini App can be opened from multiple places:
- **QR code**
- **Deep link**: `https://zalo.me/s/<mini_app_id>`
- **Mini App Store**
- **Share entry** (from chat sharing)
- **Official Account** menu entry
- **Shortcut** on the device home screen

## Hosting and caching

- **Hosting**: your Mini App is packaged and hosted on Zalo’s CDN.
- **Caching**: resources are cached by Zalo to improve load performance.
- **Auto-update**: updates are fetched automatically when a new version is available.

## Security

- Apps may require verification/review prior to distribution.
- Sensitive capabilities require declared and approved permissions.
- **Secure context**: HTTPS only.

## Resources

- **Documentation**: `https://mini.zalo.me/documents/`
- **Community**: `https://mini.zalo.me/community/`
- **Design guidelines**: `https://mini.zalo.me/documents/intro/zalo-mini-app-design-guidelines/`
- **ZaUI components**: `https://mini.zalo.me/documents/zaui/`
