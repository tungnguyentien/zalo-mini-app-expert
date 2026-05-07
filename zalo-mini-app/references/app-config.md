# App Configuration — `app-config.json`

`app-config.json` defines application-level configuration for a Zalo Mini App and must be placed at the project root.

## Base structure

```json
{
  "app": {
    "title": "My App",
    "headerTitle": "Home",
    "headerColor": "#1843EF",
    "textColor": "white",
    "leftButton": "back",
    "statusBar": "normal",
    "actionBarHidden": false,
    "hideAndroidBottomNavigationBar": false,
    "hideIOSSafeAreaBottom": false,
    "selfControlLoading": false
  },
  "listCSS": ["css/app.css"],
  "listSyncJS": ["js/vendor.js"],
  "listAsyncJS": ["js/analytics.js"]
}
```

## Root properties

| Property | Type | Required | Description |
|---|---|---:|---|
| `app` | object | ✅ | Initial UI configuration |
| `listCSS` | string[] | - | CSS assets to load |
| `listSyncJS` | string[] | - | JS assets loaded in parallel and executed synchronously |
| `listAsyncJS` | string[] | - | JS assets loaded and executed asynchronously |

## `app` properties (selected)

| Property | Type | Default | Description | Minimum version |
|---|---|---|---|---|
| `title` | string | required | App name | - |
| `headerTitle` | string/object | - | Navigation bar title; can be multi-language object | - |
| `headerColor` | string/object | - | Header color; can be theme object | - |
| `textColor` | string/object | - | Header text color; can be theme object | - |
| `leftButton` | `"back"` \| `"none"` | `"back"` | Left navigation button | - |
| `statusBar` | `"normal"` \| `"hidden"` \| `"transparent"` | `"normal"` | Status bar behavior | SDK 2.25.0 / Zalo 23.02.01.r2 |
| `actionBarHidden` | boolean | `false` | Hide default navigation bar | SDK 2.25.0 / Zalo 23.02.01.r2 |
| `hideAndroidBottomNavigationBar` | boolean | `false` | Hide Android bottom navigation bar | SDK 2.25.0 / Zalo 23.02.01.r2 |
| `hideIOSSafeAreaBottom` | boolean | `false` | Hide iOS safe-area bottom inset | SDK 2.25.0 / Zalo 23.02.01.r2 |
| `selfControlLoading` | boolean | `false` | Control splash/loading dismissal manually | SDK 2.17.0 |

## Examples

### Self-controlled loading

If `selfControlLoading: true`, dismiss the splash screen using:

```js
import { closeLoading } from "zmp-sdk";

// Call after app initialization is complete
closeLoading();
```

### Theme objects (light/dark)

```json
{
  "app": {
    "title": "My App",
    "headerColor": { "light": "#ffffff", "dark": "#1a1a1a" },
    "textColor": { "light": "black", "dark": "white" }
  }
}
```

### Multi-language titles

```json
{
  "app": {
    "title": "My App",
    "headerTitle": { "vi": "Home", "en": "Home" }
  }
}
```

## Custom header pattern

If you need a fully custom header:
1. Set `actionBarHidden: true`
2. Implement your own header component
3. Respect safe-area insets (see `compatibility.md`)

## Properties

### Root properties

| Property | Type | Required | Description |
|----------|------|----------|-------|
| `app` | Object | Yes | Initial app UI configuration |
| `listCSS` | string[] | No | CSS assets to load |
| `listSyncJS` | string[] | No | JS assets loaded in parallel, executed synchronously |
| `listAsyncJS` | string[] | No | JS assets loaded and executed asynchronously |

### `app` properties

| Property | Type | Default | Description | Minimum version |
|----------|------|---------|-------|-------------|
| `title` | string | **Required** | Application name | - |
| `headerTitle` | string | - | Title shown in the navigation bar | - |
| `headerColor` | string | - | Status bar + navigation bar color (hex) | - |
| `leftButton` | string | `"back"` | `"none"` or `"back"` | - |
| `textColor` | string | - | `"white"` or `"black"` | - |
| `statusBar` | string | `"normal"` | `"normal"`, `"hidden"`, `"transparent"` | SDK 2.25.0, Zalo 23.02.01.r2 |
| `actionBarHidden` | boolean | `false` | Hide the default navigation bar | SDK 2.25.0, Zalo 23.02.01.r2 |
| `hideAndroidBottomNavigationBar` | boolean | `false` | Hide Android bottom navigation bar | SDK 2.25.0, Zalo 23.02.01.r2 |
| `hideIOSSafeAreaBottom` | boolean | `false` | Hide iOS safe-area bottom inset | SDK 2.25.0, Zalo 23.02.01.r2 |
| `selfControlLoading` | boolean | `false` | Let the app control the splash/loading screen | SDK 2.17.0 |

## Examples

### Basic configuration

```json
{
  "app": {
    "title": "Coffee Shop",
    "headerColor": "#ffffff",
    "headerTitle": "Coffee Shop",
    "textColor": "black",
    "leftButton": "back",
    "statusBar": "normal"
  }
}
```

### Full-screen / custom header configuration

```json
{
  "app": {
    "title": "My App",
    "headerColor": "transparent",
    "textColor": "white",
    "statusBar": "transparent",
    "actionBarHidden": true,
    "hideAndroidBottomNavigationBar": true,
    "hideIOSSafeAreaBottom": true
  }
}
```

### Self-controlled loading (splash screen)

```json
{
  "app": {
    "title": "My App",
    "selfControlLoading": true
  }
}
```

When `selfControlLoading: true`, you must call `closeLoading()` to dismiss the splash screen:

```javascript
import { closeLoading } from 'zmp-sdk';

// Call after the app is ready
useEffect(() => {
  const initApp = async () => {
    // Load data, initialize...
    await loadInitialData();
    
    // Dismiss the splash screen
    closeLoading();
  };
  
  initApp();
}, []);
```

## Theme support (light/dark)

Supported from Zalo 22.03.01.r2 (iOS) / 21.09.01 (Android):

```json
{
  "app": {
    "title": "My App",
    "headerColor": {
      "light": "#ffffff",
      "dark": "#1a1a1a"
    },
    "textColor": {
      "light": "black",
      "dark": "white"
    }
  }
}
```

**Note**: If the Zalo client does not support theme objects, the `light` value will be used.

## Multi-language support

```json
{
  "app": {
    "title": "My App",
    "headerTitle": {
      "vi": "Home",
      "en": "Home",
      "my": "Home"
    }
  }
}
```

| Language | Description | Note |
|----------|-------|------|
| `vi` | Vietnamese | Used as fallback/default if needed |
| `en` | English | - |
| `my` | Burmese (Myanmar) | Android only |

## Combining theme and language

```json
{
  "app": {
    "title": "Coffee Shop",
    "headerTitle": { "vi": "Coffee Shop", "en": "Coffee Shop" },
    "headerColor": {
      "light": "#ffffff",
      "dark": "#1a1a1a"
    },
    "textColor": {
      "light": "black",
      "dark": "white"
    },
    "statusBar": "normal",
    "leftButton": "back"
  }
}
```

## Custom navigation bar

When you need full control over the navigation bar UI:

### Step 1: Hide the default header

```json
{
  "app": {
    "title": "My App",
    "statusBar": "transparent",
    "actionBarHidden": true
  }
}
```

### Step 2: Build a custom header component

```jsx
import { useNavigate } from 'react-router-dom';
import { getSystemInfo } from 'zmp-sdk';

function CustomHeader({ title, showBack = true }) {
  const navigate = useNavigate();
  const [safeArea, setSafeArea] = useState({ top: 0 });
  
  useEffect(() => {
    getSystemInfo().then(info => {
      setSafeArea(info.safeArea);
    });
  }, []);
  
  return (
    <header 
      className="custom-header"
      style={{ paddingTop: safeArea.top }}
    >
      {showBack && (
        <button onClick={() => navigate(-1)}>
          <BackIcon />
        </button>
      )}
      <h1>{title}</h1>
      <div className="header-actions">
        {/* Custom actions */}
      </div>
    </header>
  );
}
```

### CSS for a custom header

```css
.custom-header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 44px;
  display: flex;
  align-items: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  z-index: 1000;
}

.custom-header h1 {
  flex: 1;
  text-align: center;
  font-size: 16px;
  margin: 0;
}

.custom-header button {
  width: 44px;
  height: 44px;
  background: none;
  border: none;
  color: white;
}
```

## Runtime configuration

You can update view configuration at runtime:

```javascript
import { 
  configAppView,
  setNavigationBarColor,
  setNavigationBarTitle,
  setNavigationBarLeftButton
} from 'zmp-sdk';

// Configure all view properties
await configAppView({
  headerColor: '#ff6b6b',
  headerTextColor: 'white',
  statusBarType: 'normal',
  actionBarHidden: false
});

// Update individual properties
await setNavigationBarColor({ color: '#4ecdc4' });
await setNavigationBarTitle({ title: 'Product details' });
await setNavigationBarLeftButton({ type: 'back' });
```

## Best Practices

1. **Consistent branding**: use brand colors consistently.

2. **Light/dark testing**: validate UI readability and contrast in both modes.

3. **Safe area**: account for safe-area insets when implementing custom headers.

4. **Performance**: consider `selfControlLoading` to optimize perceived load time.

5. **Accessibility**: ensure sufficient contrast for header text.
