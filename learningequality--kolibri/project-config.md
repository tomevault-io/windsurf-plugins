---
trigger: always_on
description: Instructions for Claude agents working autonomously with the Android emulator. Follow these sections in order when starting from scratch, or jump to the relevant section for ongoing work.
---

# Android Emulator Agent Guide

Instructions for Claude agents working autonomously with the Android emulator. Follow these sections in order when starting from scratch, or jump to the relevant section for ongoing work.

## 1. Emulator Setup and Launch

### Check if the emulator is already running
```bash
adb devices
```
If you see `emulator-5554 device`, the emulator is ready — skip to section 2.
If the list is empty or shows `offline`, you need to start the emulator.

### First-time setup (only needed once)
If no AVD has been created yet:
```bash
make setup
```
This downloads the Android SDK, system image, and creates the `kolibri-test` AVD.

### Start the emulator
```bash
make emulator
```
This launches the emulator in the background. Wait for it to finish booting:
```bash
adb wait-for-device
adb shell getprop sys.boot_completed
```
Poll `sys.boot_completed` until it returns `1`. The boot can take 30-60 seconds.

**If the emulator segfaults or crashes**, it's likely a GPU issue. Start with software rendering instead:
```bash
"${ANDROID_SDK_ROOT:-$HOME/Android/Sdk}/emulator/emulator" -avd kolibri-test -gpu guest -no-snapshot &
```

## 2. Build and Install the App

This project uses a Makefile as the primary build interface. Run `make help` to see all available targets.

### Build and install in one step
```bash
make install
```
This builds the debug APK via Gradle and installs it on the connected emulator. The first build takes several minutes; subsequent builds are faster.

Watch for:
- **BUILD SUCCESSFUL**: Proceed to install
- **Compilation errors**: Fix before continuing
- **Python errors**: Check Chaquopy output for syntax issues

Or just build without installing:
```bash
make kolibri.apk.unsigned
```

### Verify the app is installed
```bash
adb shell pm list packages | grep kolibri
```
Should output: `package:org.learningequality.Kolibri`

### Launch the app
```bash
adb shell am start -n org.learningequality.Kolibri/org.learningequality.Kolibri.WebViewActivity
```

### Force stop the app
```bash
adb shell am force-stop org.learningequality.Kolibri
```

### Clear app data (resets to fresh state)
```bash
adb shell pm clear org.learningequality.Kolibri
```
This is needed after Python code changes since Chaquopy caches bytecode.

### Uninstall and reinstall (for signing key mismatches)
```bash
make uninstall && make install
```

### Makefile reference

| Target | Description |
|--------|-------------|
| `make setup` | Complete SDK + emulator setup (first time) |
| `make emulator` | Start the emulator |
| `make kolibri.apk.unsigned` | Build debug APK to dist/ |
| `make install` | Build and install debug APK |
| `make uninstall` | Uninstall app from device |
| `make logcat` | View filtered Kolibri logs |
| `make clean` | Clean build artifacts |
| `make test` | Run unit tests |
| `make lint` | Run Android linter |

### Quick commands

Build + Install + Launch:
```bash
make install && adb shell am start -n org.learningequality.Kolibri/org.learningequality.Kolibri.WebViewActivity
```

Clear logs and monitor:
```bash
adb logcat -c && make logcat
```

## 3. Visual Inspect-Act Loop

This is the core workflow for autonomous UI interaction. Use `/project:screenshot` to run the full loop with instructions, or follow these steps:

### Capture the screen
```bash
mkdir -p /tmp/claude
adb exec-out screencap -p > /tmp/claude/screenshot.png
```
Read the screenshot image at `/tmp/claude/screenshot.png` to see the screen visually.

### Inspect: CDP vs uiautomator

Kolibri is a WebView app. **WebView content and native Android UI require different tools:**

| What you see | Tool | Why |
|---|---|---|
| Kolibri UI (buttons, forms, nav, text) | `python3 scripts/cdp_helper.py dump` | WebView DOM is invisible to uiautomator |
| Native Android dialogs (permissions, system prompts) | `adb shell uiautomator dump /sdcard/window_dump.xml && adb shell cat /sdcard/window_dump.xml` | System dialogs are invisible to CDP |

**Rule of thumb:** If a system dialog with rounded corners is overlaying the app, use uiautomator. For everything else, use CDP.

### Interact: CDP vs adb input

**WebView elements** — click by text via CDP (no coordinate math needed):
```bash
python3 scripts/cdp_helper.py click "CONTINUE"
python3 scripts/cdp_helper.py click "EXPLORE"
```

**Native elements** — tap by coordinates from uiautomator bounds:
```bash
# bounds="[137,1177][943,1331]" → center at (540, 1254)
adb shell input tap 540 1254
```

**Other interactions:**
```bash
adb shell input text "<text>"                    # Type text (encode spaces as %s)
adb shell input swipe 540 1500 540 500 300       # Scroll down
adb shell input swipe 540 500 540 1500 300       # Scroll up
adb shell input keyevent 4                       # Press BACK
adb shell input keyevent 66                      # Press ENTER
adb shell input keyevent 3                       # Press HOME
```

### Verify
Take another screenshot after every interaction. Confirm the UI changed as expected before proceeding.

### CDP helper reference


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [learningequality/kolibri](https://github.com/learningequality/kolibri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
