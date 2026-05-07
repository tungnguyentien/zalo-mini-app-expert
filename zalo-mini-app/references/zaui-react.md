# ZaUI React Components (v1.11.0)

ZaUI is Zalo’s UI component library for Zalo Mini Apps. It provides a mobile-first design system, common interaction patterns, and routing utilities designed for the Zalo WebView environment.

## Installation

```bash
npm install zmp-ui
```

## Basic setup (recommended)

```jsx
import React from "react";
import { App, ZMPRouter, AnimationRoutes } from "zmp-ui";
import { Route } from "react-router-dom";

// Import ZaUI styles once at app entry
import "zmp-ui/zaui.css";

export default function MyApp() {
  return (
    <App>
      <ZMPRouter>
        <AnimationRoutes>
          <Route path="/" element={<HomePage />} />
        </AnimationRoutes>
      </ZMPRouter>
    </App>
  );
}
```

## What to use (by category)

This bundle references the official versioned docs located at:
`zaui-react_versioned_docs/version-1.11.0/`

### Layout & navigation

- `App`, `Page`
- `Header`, `Tabs`, `BottomNavigation`
- `ZMPRouter`, `AnimationRoutes`

### Containers & layout helpers

- `Box`, `Stack`, `Grid`, `Center`, `Cluster`, `ZBox`

### Form & input

- `Button`, `Input`, `TextArea`
- `Checkbox`, `Radio`, `Switch`, `Slider`
- `Select`, `Picker`, `DatePicker`
- `OTP`, `Password`, `Search`

### Display

- `Text`, `Icon`, `Avatar`
- `List`, `Progress`, `Spinner`
- `Swiper`, `Calendar`, `ImageViewer`

### Overlay & feedback

- `Modal`, `Sheet`, `ActionSheet`
- `SnackbarProvider`

### Foundation (design system)

- `Colors`, `Typography`, `Spacing`, `Shadow`, `CornerRadius`
- design tokens and guidelines

## Operational guidance

- Prefer ZaUI components for consistency with Zalo mobile UX patterns.
- Respect safe-area constraints (see `compatibility.md`).
- Keep the bundle small; avoid adding UI libraries that overlap with ZaUI.

## Official references

- ZaUI documentation: `https://mini.zalo.me/documents/zaui/`
- Design system (Figma): `https://www.figma.com/file/ME5AP7pt79huTBTMdnT7yX/%5BPUBLIC%5D-Zalo-Mini-App-Framework-2.0`

