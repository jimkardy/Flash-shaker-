<p align="center">
  <h1 align="center">QuickTorch</h1>
  <p align="center">Shake your phone. Light comes on. That's it.</p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License">
  <img src="https://img.shields.io/badge/Android-7.0%2B-green.svg" alt="Android 7.0+">
  <img src="https://img.shields.io/badge/React_Native-0.86-61dafb.svg" alt="React Native 0.86">
  <img src="https://img.shields.io/badge/no%20ads-100%25-success.svg" alt="No Ads">
  <img src="https://img.shields.io/badge/no%20tracking-100%25-success.svg" alt="No Tracking">
  <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome">
</p>

---

### Free. Open source. No ads. No tracking. No nonsense.

QuickTorch is a flashlight app for Android that activates when you shake your phone. No unlocking, no fumbling in the dark, no digging through app drawers. Just a quick shake and the torch turns on. Shake again to turn it off. It works even when your screen is off and your phone is locked.

There are no ads. There is no analytics. There is no telemetry. There is no internet permission requested at all. The code is fully open source — you can build it yourself and verify every single line. This is the flashlight app that respects you.

---

## Features

- **Shake to toggle** — Detects accelerometer movement and turns the flashlight on or off instantly. Works like Motorola's "chop to flashlight" gesture, but on any Android device.
- **Works when locked** — Runs as a lightweight foreground service with a silent, low-priority notification. Shake detection stays active even with the screen off.
- **Zero ads, zero tracking** — No ad SDKs, no analytics, no data collection. The app does not request internet access, location, contacts, or any permission beyond what's strictly needed.
- **Minimal permissions** — Camera (to access the LED flash), foreground service (to keep shake detection alive in the background), and notifications (required by Android for foreground services). That's it.
- **32-bit and 64-bit** — Builds for both `armeabi-v7a` and `arm64-v8a` architectures, so it runs on everything from older budget phones to the latest flagships.
- **Lightweight** — No bloated frameworks or unnecessary dependencies. The native layer is pure Kotlin with zero third-party libraries.

---

## How It Works

QuickTorch uses a two-layer architecture: React Native handles the UI and permission flows, while Kotlin native code runs the performance-critical parts — shake detection and torch control.

**Shake detection** reads raw accelerometer data via Android's `SensorManager` and `TYPE_ACCELEROMETER`. The magnitude of acceleration is calculated as:

```
magnitude = sqrt(x^2 + y^2 + z^2) - 9.81
```

If the magnitude exceeds a threshold of 12.0 (with a 1500ms cooldown between triggers), it registers as a shake and toggles the flashlight. The sensor runs at `SENSOR_DELAY_GAME` rate for responsive detection without excessive battery drain.

**Torch control** uses Android's `Camera2 API` — specifically `CameraManager.setTorchMode()` — to directly control the rear camera's LED flash unit. No camera preview is opened, no image is captured.

The relevant native source files live in `android/app/src/main/java/com/quicktorch/torch/`:

| File | Purpose |
|------|---------|
| `ShakeDetector.kt` | Accelerometer listener with magnitude-based shake detection and cooldown logic |
| `TorchService.kt` | Foreground service that owns the shake detector and toggles the torch |
| `TorchModule.kt` | React Native bridge module exposing service control to JavaScript |
| `TorchPackage.kt` | TurboReactPackage registration for the new architecture |
| `NotificationHelper.kt` | Silent, low-priority ongoing notification for the foreground service |

---

## Installation

1. Download the latest APK from the [Releases](../../releases) page.
2. Open the APK file on your Android device.
3. If prompted, allow installation from unknown sources in your device settings.
4. Open the app, go through the quick onboarding, tap "Let's Go", then tap the power button to start the shake detection service.

Once the service is running, just shake your phone to toggle the flashlight. You can leave the app — it keeps listening in the background.

**Device requirements:** Android 7.0 (API 24) or higher, a rear camera with an LED flash, and an accelerometer (pretty much every phone has both).

---

## Building from Source

**Prerequisites:**
- Node.js >= 22.11.0
- Android SDK with API 36, Build Tools 36.0.0, NDK 27.1.12297006
- Kotlin 2.1.20 (managed by Gradle plugin)

```bash
git clone https://github.com/quicktorch-app/QuickTorch.git
cd QuickTorch
npm install
npx react-native run-android
```

For a release build:

```bash
cd android
./gradlew assembleRelease
```

The signed APK will be at `android/app/build/outputs/apk/release/app-release.apk`.

For development with live reload:

```bash
npx react-native start
# In a separate terminal:
npx react-native run-android
```

---

## Permissions Explained

Transparency matters. Here's every permission this app uses and exactly why:

| Permission | Why it's needed |
|---|---|
| `CAMERA` | Required by Android to access the LED flash hardware via Camera2 API. The app never opens the camera, never takes photos, never records video. |
| `FOREGROUND_SERVICE` | Keeps the shake detection service alive when the app is in the background or the screen is off. Without this, Android would kill the accelerometer listener. |
| `FOREGROUND_SERVICE_SPECIAL_USE` | Required on Android 14+ to declare that this foreground service falls under a special-use category. The manifest explicitly states: "QuickTorch needs to monitor accelerometer sensor data to detect shake gestures for torch control." |
| `POST_NOTIFICATIONS` | Required on Android 13+ to show the foreground service notification. The notification is silent, low-priority, non-intrusive, and cannot be swiped away while the service runs. |
| `WAKE_LOCK` | Keeps the CPU awake briefly when processing a shake event in the background. |

**What's NOT here and never will be:**
No `INTERNET`. No `ACCESS_NETWORK_STATE`. No `ACCESS_FINE_LOCATION`. No `READ_CONTACTS`. No `RECORD_AUDIO`. No `READ_PHONE_STATE`. No advertising IDs. No analytics SDKs. This app has no network capability whatsoever. It literally cannot phone home even if it wanted to.

---

## Tech Stack

- **React Native 0.86** — Cross-platform UI with the new architecture enabled (TurboModules, Fabric)
- **TypeScript** — Type-safe JavaScript layer
- **Kotlin 2.1.20** — All native Android code
- **Camera2 API** — Direct LED flash control via `CameraManager`
- **SensorManager** — Accelerometer-based shake detection
- **Hermes** — JavaScript engine (optimized for React Native)
- **Zero third-party native dependencies** — No native libraries beyond React Native itself

---

## Contributing

Pull requests are welcome. A few things to keep in mind:

- This project's whole identity is "no ads, no tracking, open source." If your PR adds a dependency that phones home, collects data, or shows ads, it's getting rejected.
- Feature ideas that fit the minimalist vibe: configurable shake sensitivity, strobe/SOS mode, brightness control, widget for quick toggling.
- Keep the native layer dependency-free. React Native libraries in JS are fine, but the Kotlin code should stay lean.

---

## License

MIT — do whatever you want with it. Shine it in your own eyes at your own risk.

---

<p align="center">
  Made with frustration at every other flashlight app on the Play Store having 47 tracking SDKs baked in.
</p>