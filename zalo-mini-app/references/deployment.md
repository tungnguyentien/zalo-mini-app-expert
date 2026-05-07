# Deployment & Release — Zalo Mini App

This guide describes environments, deployment flows, review submission, and publishing for Zalo Mini Apps.

## Release pipeline (typical)

```
Development → Testing → Review → Live
```

## Environments

| Environment | Purpose | Access |
|---|---|---|
| Development | daily development | Developer/Admin |
| Testing | pre-release QA | Developer/Admin |
| Live | production | all users |

## Deploy to Development / Testing

### CLI

```bash
# Development
zmp deploy

# Testing
zmp deploy --testing
```

### VS Code Extension

1. Open Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`)
2. Select **“Zalo Mini App: Deploy”**
3. Choose target environment (Development/Testing)

### Build before deploy

```bash
npm run build
zmp deploy
```

## Submit for review

Only **Testing** versions can be submitted for review.

In [Mini App Center](https://mini.zalo.me/developers/):
1. Open your Mini App
2. Go to **Version management** → **Version list**
3. Select the Testing version
4. Fill in review information:
   - release notes (what changed)
   - reviewer notes (test steps, test accounts)
   - permissions requested/used
5. Submit and wait for the result

## Publishing to Live

After a version is approved:
1. Open the approved version
2. Click **Publish**
3. Confirm the operation

## Version states (common)

| State | Meaning |
|---|---|
| Development | in development |
| Testing | ready for internal testing |
| In review | pending review decision |
| Approved | passed review |
| Rejected | did not pass review |
| Live | available to all users |

## Rollback

To roll back to a previous Live version:
1. Open the version list
2. Find a prior Live version
3. Publish it again

## Entry points

### Deep link

`https://zalo.me/s/<mini_app_id>`

With params:

`https://zalo.me/s/<mini_app_id>/?page=/product&id=123`

### QR code

Generate a QR code from the deep link.

### Official Account (OA)

You can configure OA menu items to open the Mini App.

### Shortcut

Users can create a home screen shortcut (if supported) via the SDK (see `zmp-sdk-api.md`).

## Checklist (recommended)

Before deploying Testing:
- remove debug logs
- verify APIs have error handling
- confirm bundle size constraints
- test on Device Mode for native features

Before review submission:
- ensure permissions are requested and justified
- provide clear reviewer notes and test instructions
- confirm policy compliance (especially around authentication and external navigation)

