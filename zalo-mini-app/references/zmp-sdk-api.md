# ZMP SDK API Reference

## Installation

```bash
npm install zmp-sdk
```

## Events API

### Subscribe to events

```javascript
import { events, EventName } from "zmp-sdk/apis";

// Add a listener for an event
events.on(EventName.AppPaused, () => {
  console.log("App moved to background");
});

// Listener runs once
events.once(EventName.AppResumed, () => {
  console.log("App returned to foreground");
});

// Remove a specific listener
events.off(EventName.AppPaused, myListener);

// Remove all listeners for an event
events.removeAllListeners(EventName.AppPaused);
```

### Event names

| Event | Description |
|-------|-------------|
| `AppPaused` | Mini App transitions from foreground to background |
| `AppResumed` | Mini App transitions from background to foreground |
| `NetworkChanged` | Network connectivity changes |
| `OnDataCallback` | Receive data from a previous Mini App |
| `OpenApp` | Mini App is reopened from background |

## User APIs

### Authorization

```javascript
import { authorize } from "zmp-sdk/apis";

// Request permission scopes for APIs
const result = await authorize({
  scopes: ['scope.userInfo', 'scope.userPhonenumber']
});
```

### User information

```javascript
import { getUserID, getUserInfo, getAccessToken, getPhoneNumber } from "zmp-sdk/apis";

// Get Zalo User ID (per app)
const userId = await getUserID({});

// Get user profile info
const userInfo = await getUserInfo();
// { userInfo: { id, name, avatar } }

// Get access token (send to your backend for verification)
const token = await getAccessToken();

// Get phone number token (requires permission)
const phoneResult = await getPhoneNumber();
// { token: 'encrypted_token' } - send this token to your backend to decode
```

### User settings

```javascript
import { getSetting } from "zmp-sdk/apis";

// Get current permission settings
const settings = await getSetting();
```

## Basic APIs

```javascript
import { getAppInfo, getSystemInfo, getDeviceIdAsync, getContextAsync } from "zmp-sdk/apis";

// Get Mini App info
const appInfo = await getAppInfo();

// Get system and device info
const systemInfo = await getSystemInfo();
// { platform, zaloVersion, deviceName, ... }

// Get device ID (unique per device)
const deviceId = await getDeviceIdAsync();

// Get entry context (source, params, etc.)
const context = await getContextAsync();
```

## Routing APIs

```javascript
import { 
  closeApp, 
  openMiniApp, 
  openWebview, 
  sendDataToPreviousMiniApp,
  getRouteParams 
} from "zmp-sdk/apis";

// Close the Mini App
closeApp();

// Open another Mini App
await openMiniApp({
  appId: 'OTHER_MINI_APP_ID',
  path: '/page',
  params: { key: 'value' }
});

// Open a webview
await openWebview({ url: 'https://example.com' });

// Send data back to the previous Mini App
sendDataToPreviousMiniApp({ result: 'success' });

// Get route params
const params = getRouteParams();
```

## Storage APIs

### Native storage (persistent — requires approval/permission)

```javascript
import { setItem, getItem, removeItem, clear, getNativeStorageInfo } from "zmp-sdk/apis";

// Save data
await setItem({ key: 'myKey', value: 'myValue' });

// Read data
const data = await getItem({ key: 'myKey' });

// Remove an item
await removeItem({ key: 'myKey' });

// Clear all
await clear();

// Get storage metadata
const info = await getNativeStorageInfo();
```

### Web storage (session-based)

```javascript
import { setStorage, getStorage, removeStorage, clearStorage, getStorageInfo } from "zmp-sdk/apis";

// Similar to native storage, but data is session-scoped
await setStorage({ key: 'myKey', data: { foo: 'bar' } });
const result = await getStorage({ keys: ['myKey'] });
```

## UI APIs

### Feedback

```javascript
import { showToast, closeLoading } from "zmp-sdk/apis";

// Show a toast
showToast({ message: 'Success!' });

// Dismiss splash loading (when selfControlLoading: true)
closeLoading();
```

### View configuration

```javascript
import { 
  configAppView, 
  setNavigationBarColor, 
  setNavigationBarLeftButton,
  setNavigationBarTitle 
} from "zmp-sdk/apis";

// Configure the full app view
await configAppView({
  headerColor: '#1843EF',
  headerTextColor: 'white',
  statusBarType: 'normal',
  actionBarHidden: false
});

// Set navigation bar color
await setNavigationBarColor({ color: '#ffffff' });

// Set the left button (back/none)
await setNavigationBarLeftButton({ type: 'back' });

// Set the title
await setNavigationBarTitle({ title: 'Home' });
```

### Keyboard

```javascript
import { hideKeyboard } from "zmp-sdk/apis";

hideKeyboard();
```

## Location API

```javascript
import { getLocation } from "zmp-sdk/apis";

// Get current location (requires permission)
const location = await getLocation();
// { latitude, longitude, accuracy, ... }
```

## Media APIs

### Camera

```javascript
import { 
  createCameraContext, 
  checkZaloCameraPermission,
  requestCameraPermission 
} from "zmp-sdk/apis";

// Check camera permission
const hasPermission = await checkZaloCameraPermission();

// Request camera permission
await requestCameraPermission();

// Create a camera context
const camera = createCameraContext({
  videoElement: document.getElementById('video'),
  options: { facingMode: 'user' }
});

await camera.start();
const photo = await camera.takePhoto();
camera.stop();
```

### File/Image

```javascript
import { 
  chooseImage, 
  openMediaPicker, 
  saveImageToGallery,
  saveVideoToGallery,
  downloadFile 
} from "zmp-sdk/apis";

// Pick images from album/camera
const result = await chooseImage({
  count: 3,
  sourceType: ['album', 'camera']
});

// Open media picker
const media = await openMediaPicker({
  type: 'photo', // 'photo' | 'video' | 'file'
  maxSelectItem: 5
});

// Save an image to gallery
await saveImageToGallery({ imageUrl: 'https://...' });

// Download a file
await downloadFile({ url: 'https://...', filename: 'file.pdf' });
```

## Device APIs

```javascript
import { 
  getNetworkType, 
  openPhone, 
  openSMS, 
  keepScreen, 
  vibrate 
} from "zmp-sdk/apis";

// Get network type
const network = await getNetworkType();
// { networkType: 'wifi' | '4g' | '3g' | 'none' | ... }

// Open dialer
openPhone({ phoneNumber: '0123456789' });

// Open SMS app
openSMS({ phoneNumber: '0123456789', body: 'Message content' });

// Keep screen on (requires permission)
keepScreen({ keepScreenOn: true });

// Vibrate device
vibrate();
```

### QR code

```javascript
import { scanQRCode } from "zmp-sdk/apis";

// Scan a QR code
const result = await scanQRCode();
// { content: '...' }
```

### Biometric authentication

```javascript
import { openBioAuthentication, checkStateBioAuthentication } from "zmp-sdk/apis";

// Check biometric availability/state
const state = await checkStateBioAuthentication();

// Open biometric authentication UI
const result = await openBioAuthentication({
  title: 'Authentication',
  description: 'Use fingerprint/Face ID to authenticate'
});
```

## Permission APIs

```javascript
import { requestSendNotification, openPermissionSetting } from "zmp-sdk/apis";

// Request notification permission
await requestSendNotification();

// Open permission settings
openPermissionSetting();
```

## Zalo APIs

```javascript
import {
  openProfile,
  openProfilePicker,
  openChat,
  followOA,
  unfollowOA,
  openShareSheet,
  openPostFeed,
  createShortcut,
  viewOAQr,
  requestUpdateZalo,
  minimizeApp,
  favoriteApp,
  addRating
} from "zmp-sdk/apis";

// Open user/OA profile
openProfile({ id: 'user_id', type: 'user' }); // type: 'user' | 'oa'

// Open friend picker
const selected = await openProfilePicker();

// Open chat with user/OA
openChat({ id: 'oa_id', type: 'oa' });

// Follow/Unfollow OA
await followOA({ id: 'oa_id' });
await unfollowOA({ id: 'oa_id' });

// Share
await openShareSheet({
  type: 'zmp', // 'zmp' | 'link' | 'image'
  data: {
    path: '/page',
    title: 'Title',
    description: 'Description',
    thumbnail: 'https://...'
  }
});

// Post to feed
await openPostFeed({
  type: 'link',
  data: { link: 'https://...', title: 'Title' }
});

// Create a home screen shortcut
await createShortcut();

// Show OA QR
viewOAQr({ id: 'oa_id' });

// Request Zalo update
requestUpdateZalo();

// Minimize Mini App
minimizeApp();

// Add to favorites
await favoriteApp();

// Open rating flow
await addRating();
```

## Advertising APIs

```javascript
import { setupAd, loadAd, displayAd, refreshAd } from 'zmp-sdk/apis';

// Setup advertising (call once when app starts)
setupAd();

// Load an ad slot (rendered into an element with the same ID)
// Example: <div id="ZMA_Middle"></div>
loadAd({
  ids: ["ZMA_Middle"],
  config: {
    display: false, // set true to auto-display after loading
  },
});

// Display a loaded ad slot
displayAd({ id: "ZMA_Middle" });

// Clear an ad slot in the current session
refreshAd({ id: "ZMA_Middle" });
```

## Widgets APIs

```javascript
import { showOAWidget, showFunctionButtonWidget } from 'zmp-sdk/apis';

// Show Follow OA widget
showOAWidget({
  id: "oaWidget",
  guidingText: "Get the latest promotions from our store",
  color: "#0068FF",
  onStatusChange: (status) => {
    console.log(status);
  },
});

// Show function button widget
showFunctionButtonWidget({
  id: "orderButton",
  type: "ORDER",
  text: "Order",
  color: "#0068FF",
  textColor: "#FFFFFF",
  borderRadius: "48px",
  onDataReceived: (messageToken) => {
    console.log(messageToken);
  },
  onError: (error) => {
    console.error("onError:", error);
  },
  buttons: [
    { icon: 'phone', onClick: () => {} },
    { icon: 'chat', onClick: () => {} }
  ]
});
```

## Error codes

APIs may return errors in the following format:

```javascript
{
  error: -1,
  message: 'Error message'
}
```

For the full list, see `https://mini.zalo.me/documents/api/errorCode/`.
