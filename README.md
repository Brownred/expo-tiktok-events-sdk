# expo-tiktok-events-sdk

[![npm version](https://img.shields.io/npm/v/expo-tiktok-events-sdk.svg)](https://www.npmjs.com/package/expo-tiktok-events-sdk)
[![Build Status](https://img.shields.io/github/actions/workflow/status/Brownred/expo-tiktok-events-sdk/ci.yml)](https://github.com/Brownred/expo-tiktok-events-sdk/actions)
[![npm downloads](https://img.shields.io/npm/dw/expo-tiktok-events-sdk.svg)](https://www.npmjs.com/package/expo-tiktok-events-sdk)
[![License](https://img.shields.io/npm/l/expo-tiktok-events-sdk.svg)](https://github.com/Brownred/expo-tiktok-events-sdk/blob/main/LICENSE)

A React Native bridge for the TikTok Business SDK, with first-class Expo support.

This library provides a modern, promise-based interface for the TikTok Business SDK, enabling JavaScript apps to initialize the SDK, identify users, and track various events (standard, content, and custom events) with full TypeScript support and comprehensive error handling.

> This is a fork of [react-native-tiktok-business-sdk](https://github.com/mtebele/react-native-tiktok-business-sdk) by [mtebele](https://github.com/mtebele), with added Expo setup documentation and iOS CocoaPods fix included.

---

## ☕ Support

If this saved you time, consider buying me a coffee — it helps keep this maintained!

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-support-yellow?style=flat&logo=buy-me-a-coffee)](https://buymeacoffee.com/YOUR_USERNAME)

---

## ✨ Features

- 📱 **Cross-platform** — iOS and Android support
- ⚡ **Expo support** — Works with Expo managed workflow and EAS Build
- 🎯 **Promise-based API** — Modern async/await support
- 🔒 **TypeScript** — Full type safety and IntelliSense
- 🧪 **Comprehensive testing** — 100% test coverage with 100+ tests
- 📊 **Event tracking** — Standard, content, custom events, and ad revenue tracking
- 🛡️ **Error handling** — Robust error handling with specific error codes
- 🎨 **Developer friendly** — Simple API with detailed documentation

---

## Installation

```sh
npm install expo-tiktok-events-sdk
# or
yarn add expo-tiktok-events-sdk
```

---

## Setup

Choose the setup guide for your project type:

- [Expo (Managed Workflow / EAS Build)](#expo-setup)
- [Bare React Native](#bare-react-native-setup)

---

## Expo Setup

> Tested with Expo SDK 52+. This is the recommended approach if you are using Expo managed workflow or EAS Build.

### 1. Install expo-build-properties

```sh
npx expo install expo-build-properties
```

### 2. Update app.json

Add both the `expo-build-properties` plugin (to fix the iOS CocoaPods modular headers issue) and the `NSUserTrackingUsageDescription` key required by the TikTok SDK on iOS 14+:

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSUserTrackingUsageDescription": "This identifier will be used to deliver personalized ads to you."
      }
    },
    "plugins": [
      [
        "expo-build-properties",
        {
          "ios": {
            "extraPods": [
              {
                "name": "TikTokBusinessSDK",
                "modular_headers": true
              }
            ]
          }
        }
      ]
    ]
  }
}
```

> The `extraPods` entry fixes the following EAS Build error you would otherwise hit on iOS:
> `[!] The following Swift pods cannot yet be integrated as static libraries: The Swift pod 'TikTokBusiness' depends upon 'TikTokBusinessSDK', which does not define modules.`

### 3. Build with EAS

You do not need to run `pod install` manually. EAS Build handles it:

```sh
eas build --platform ios
eas build --platform android
```

That's it for Expo. No Xcode or Mac required for configuration.

---

## Bare React Native Setup

### iOS Setup

1. Run pod install in your `ios` directory:

```sh
cd ios && pod install
```

2. The SDK will automatically link the TikTok Business SDK dependency.

3. **iOS 14+ — App Tracking Transparency**: Add the following key to your `ios/<YourApp>/Info.plist`:

```xml
<key>NSUserTrackingUsageDescription</key>
<string>This identifier will be used to deliver personalized ads to you.</string>
```

Customize the string to match your app's use case. Without this entry, the install event and other automatic tracking may not function correctly on iOS 14+.

### Android Setup

1. The SDK dependency is automatically included via Gradle.
2. Add ProGuard rules to prevent code obfuscation (see [ProGuard section](#proguard-android) below).

---

## Usage

### Importing the Library

```js
import { TikTokBusiness } from 'expo-tiktok-events-sdk';
```

### Initialize the SDK

Before using any event tracking methods, you must initialize the SDK. You can call `initializeSdk` with your appId, tiktokAppId, accessToken, and optionally set debug mode.

#### Single TikTok App ID

```js
async function initializeTikTokSDK() {
  try {
    await TikTokBusiness.initializeSdk(
      'YOUR_APP_ID',         // Your app ID (e.g., Android package name or iOS bundle ID)
      'YOUR_TIKTOK_APP_ID',  // Your TikTok App ID from TikTok Events Manager
      'YOUR_ACCESS_TOKEN',   // Your access token from TikTok Events Manager (REQUIRED)
      true                   // Enable debug mode (optional, defaults to false)
    );
    console.log('TikTok SDK initialized successfully!');
  } catch (error) {
    console.error('Error initializing TikTok SDK:', error);
  }
}

initializeTikTokSDK();
```

#### Multiple TikTok App IDs

Starting from SDK v1.3.1, you can track events with multiple TikTok App IDs by passing an array:

```js
async function initializeTikTokSDKWithMultipleAppIds() {
  try {
    await TikTokBusiness.initializeSdk(
      'YOUR_APP_ID',
      ['TIKTOK_APP_ID_1', 'TIKTOK_APP_ID_2', 'TIKTOK_APP_ID_3'],
      'YOUR_ACCESS_TOKEN',
      true
    );
    console.log('TikTok SDK initialized with multiple App IDs!');
  } catch (error) {
    console.error('Error initializing TikTok SDK:', error);
  }
}
```

Alternatively, you can use comma-separated string format:

```js
await TikTokBusiness.initializeSdk(
  'YOUR_APP_ID',
  '11,22,33',  // Comma-separated App IDs (no spaces)
  'YOUR_ACCESS_TOKEN',
  true
);
```

**Important Validation Rules:**
- App IDs must be numeric strings
- No spaces allowed in comma-separated format
- No trailing or leading commas
- Array format is recommended for better readability and type safety
- All TikTok App IDs must match your App ID (SDK requirement)

#### Disabling Automatic Tracking Events

By default, the TikTok SDK automatically reports install, launch, 2D-retention, and in-app purchase events. You can disable any of these at initialization time:

```js
import { TikTokBusiness } from 'expo-tiktok-events-sdk';
import type { TikTokSdkConfig } from 'expo-tiktok-events-sdk';

// Disable all automatic events
await TikTokBusiness.initializeSdk(
  'YOUR_APP_ID',
  'YOUR_TIKTOK_APP_ID',
  'YOUR_ACCESS_TOKEN',
  false,
  { disableAutoTracking: true }
);

// Or disable specific events
const options: TikTokSdkConfig = {
  disableInstallTracking: true,
  disableLaunchTracking: true,
  disableRetentionTracking: true,
  disablePaymentTracking: true,
};

await TikTokBusiness.initializeSdk(
  'YOUR_APP_ID',
  'YOUR_TIKTOK_APP_ID',
  'YOUR_ACCESS_TOKEN',
  false,
  options
);
```

### Identify a User

```js
async function identifyUser() {
  try {
    await TikTokBusiness.identify(
      'user_12345',
      'john_doe',
      '+1234567890',
      'john@example.com'
    );
    console.log('User identified successfully!');
  } catch (error) {
    console.error('Error identifying user:', error);
  }
}
```

### Logout

```js
async function logoutUser() {
  try {
    await TikTokBusiness.logout();
    console.log('User logged out successfully!');
  } catch (error) {
    console.error('Error logging out:', error);
  }
}
```

### Track a Standard Event

```js
import { TikTokBusiness, TikTokEventName } from 'expo-tiktok-events-sdk';

async function trackStandardEvent() {
  try {
    await TikTokBusiness.trackEvent(TikTokEventName.REGISTRATION);

    await TikTokBusiness.trackEvent(
      TikTokEventName.SEARCH,
      'search_001',
      { query: 'coffee' }
    );

    console.log('Standard event tracked successfully!');
  } catch (error) {
    console.error('Error tracking standard event:', error);
  }
}
```

### Track a Content Event

```js
import { 
  TikTokBusiness, 
  TikTokContentEventName, 
  TikTokContentEventParameter, 
  TikTokContentEventContentsParameter 
} from 'expo-tiktok-events-sdk';

async function trackContentEvent() {
  try {
    await TikTokBusiness.trackContentEvent(TikTokContentEventName.PURCHASE, {
      [TikTokContentEventParameter.CURRENCY]: 'USD',
      [TikTokContentEventParameter.VALUE]: '29.99',
      [TikTokContentEventParameter.CONTENT_TYPE]: 'product',
      [TikTokContentEventParameter.DESCRIPTION]: 'Premium coffee purchase',
      [TikTokContentEventParameter.CONTENTS]: [
        {
          [TikTokContentEventContentsParameter.CONTENT_ID]: 'coffee_001',
          [TikTokContentEventContentsParameter.CONTENT_NAME]: 'Premium Coffee',
          [TikTokContentEventContentsParameter.BRAND]: 'Coffee Brand',
          [TikTokContentEventContentsParameter.PRICE]: 29.99,
          [TikTokContentEventContentsParameter.QUANTITY]: 1,
        },
      ],
    });

    console.log('Content event tracked successfully!');
  } catch (error) {
    console.error('Error tracking content event:', error);
  }
}
```

### Track a Custom Event

```js
async function trackCustomEvent() {
  try {
    await TikTokBusiness.trackCustomEvent('user_achievement', {
      description: 'Level Completed!',
    });
    console.log('Custom event tracked successfully!');
  } catch (error) {
    console.error('Error tracking custom event:', error);
  }
}
```

### Track Ad Revenue Events

```js
import { TikTokBusiness, trackAdRevenueEvent } from 'expo-tiktok-events-sdk';
import type { AdRevenueData } from 'expo-tiktok-events-sdk';

async function trackAdRevenue() {
  try {
    const adRevenueData: AdRevenueData = {
      revenue: 0.05,
      currency: 'USD',
      adNetwork: 'AdMob',
      adUnit: 'banner_main',
      adFormat: 'banner',
      placement: 'home_screen',
      country: 'US',
      precision: 'exact',
    };

    await trackAdRevenueEvent(adRevenueData, 'ad-revenue-001');
    console.log('Ad revenue tracked successfully!');
  } catch (error) {
    console.error('Error tracking ad revenue:', error);
  }
}
```

### Flush Events

```js
async function flushEvents() {
  try {
    await TikTokBusiness.flush();
    console.log('Events flushed successfully!');
  } catch (error) {
    console.error('Error flushing events:', error);
  }
}
```

> **Note:** Events are normally sent automatically at regular intervals. Only use flush when you need immediate synchronization, such as before the app terminates.

---

## API Reference

### Available Methods

```typescript
initializeSdk(appId: string, ttAppId: string | string[], accessToken: string, debug?: boolean): Promise<string>

identify(externalId: string, externalUserName: string, phoneNumber: string, email: string): Promise<string>
logout(): Promise<string>

trackEvent(eventName: TikTokEventName, eventId?: string, properties?: object): Promise<string>
trackContentEvent(eventName: TikTokContentEventName, properties?: object): Promise<string>
trackCustomEvent(eventName: string, properties?: object): Promise<string>
trackAdRevenueEvent(adRevenueData: AdRevenueData, eventId?: string): Promise<string>

flush(): Promise<string>
```

### AdRevenueData Interface

```typescript
interface AdRevenueData {
  revenue?: number;
  currency?: string;
  adNetwork?: string;
  adUnit?: string;
  adFormat?: string;
  placement?: string;
  country?: string;
  precision?: string;
  [key: string]: string | number | boolean | undefined;
}
```

### Enums

- **TikTokEventName** – Standard event names (LOGIN, REGISTRATION, PURCHASE, etc.)
- **TikTokContentEventName** – Content event names (ADD_TO_CART, VIEW_CONTENT, etc.)
- **TikTokContentEventParameter** – Parameters for content events (CURRENCY, VALUE, etc.)
- **TikTokContentEventContentsParameter** – Parameters for content items (CONTENT_ID, PRICE, etc.)

### Complete Import Example

```js
import {
  TikTokBusiness,
  trackAdRevenueEvent,
  TikTokEventName,
  TikTokContentEventName,
  TikTokContentEventParameter,
  TikTokContentEventContentsParameter,
} from 'expo-tiktok-events-sdk';
import type { AdRevenueData } from 'expo-tiktok-events-sdk';
```

---

## Error Handling

```js
try {
  await TikTokBusiness.initializeSdk(appId, ttAppId, accessToken, debug);
} catch (error) {
  switch (error.code) {
    case 'INIT_ERROR':
      console.error('Failed to initialize SDK:', error.message);
      break;
    case 'IDENTIFY_ERROR':
      console.error('Failed to identify user:', error.message);
      break;
    case 'TRACK_EVENT_ERROR':
      console.error('Failed to track event:', error.message);
      break;
    default:
      console.error('Unknown error:', error.message);
  }
}
```

---

## Requirements

- React Native 0.60+
- iOS 11.0+
- Android API level 16+
- Expo SDK 52+ (if using Expo)

---

## ProGuard (Android)

If you're using ProGuard, add these rules to your `proguard-rules.pro`:

```
-keep class com.tiktok.** { *; }
-keep class com.android.billingclient.api.** { *; }
-keep class androidx.lifecycle.** { *; }
-keep class com.android.installreferrer.** { *; }
```

---

## Troubleshooting

### Expo: pod install fails with Swift static library error

If you see:

```
[!] The following Swift pods cannot yet be integrated as static libraries:
The Swift pod 'TikTokBusiness' depends upon 'TikTokBusinessSDK', which does not define modules.
```

Make sure you have added the `expo-build-properties` plugin with the `extraPods` config as shown in the [Expo Setup](#expo-setup) section above.

### "TikTokBusinessModule.initializeSdk got X arguments, expected Y"

- Ensure you're passing all required parameters including `accessToken`
- Check that you're using the latest version of the library

### "The package doesn't seem to be linked"

- Run `npx react-native clean` and rebuild your project
- For iOS: `cd ios && pod install && cd ..`
- For Android: `cd android && ./gradlew clean && cd ..`

### Events not appearing in TikTok Events Manager

- Verify your `accessToken` is correct and has proper permissions
- Check that your App ID and TikTok App ID are correctly configured
- Enable debug mode (`true` as the 4th argument) to see detailed logs
- Call `TikTokBusiness.flush()` during testing to force immediate event delivery

---

## Contributing

See the [contributing guide](CONTRIBUTING.md) to learn how to contribute to the repository and the development workflow.

---

## License

MIT

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for detailed release notes and version history.

---

## Credits

Forked from [react-native-tiktok-business-sdk](https://github.com/mtebele/react-native-tiktok-business-sdk) by [mtebele](https://github.com/mtebele). Original work and core implementation by mtebele. This fork adds Expo setup documentation and the iOS CocoaPods fix.
