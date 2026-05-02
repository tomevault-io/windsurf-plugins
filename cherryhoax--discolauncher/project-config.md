---
trigger: always_on
description: Disco Launcher is a metro-styled Android launcher application with a web-based UI built using Node.js, webpack, and SASS. The project includes both a hybrid web interface and an Android application that packages the web assets for mobile deployment.
---

# Disco Launcher

Disco Launcher is a metro-styled Android launcher application with a web-based UI built using Node.js, webpack, and SASS. The project includes both a hybrid web interface and an Android application that packages the web assets for mobile deployment.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

Bootstrap, build, and test the repository:
- `npm install` -- takes 60 seconds. NEVER CANCEL. Set timeout to 90+ seconds.
- `npm run build` -- takes 30 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
- `npm run build:production` -- takes 45-55 seconds with minification. NEVER CANCEL. Set timeout to 120+ seconds.

Run the web development server:
- `npm run debug:web` -- starts live development server with auto-rebuild on port 8080. NEVER CANCEL.
- Open `http://localhost:8080/www/` in browser for testing.

Individual build commands:
- `npm run build:webpack` -- JavaScript bundling, takes ~20 seconds
- `npm run build:scss` -- SASS compilation, takes ~5 seconds  
- `npm run build:internal-apps` -- Internal apps compilation, takes ~10 seconds
- `npm run build:android-assets` -- Android asset preparation, takes ~5 seconds

Production builds (for Android release):
- `npm run build:production` -- Full production build with minification, takes 45-55 seconds. NEVER CANCEL. Set timeout to 120+ seconds.

## Android Build Requirements

**IMPORTANT**: Android builds require Android SDK and are NOT available in most development environments.
- Android Studio with SDK 34+
- Java 17+ (OpenJDK Temurin recommended)
- The gradle build will FAIL without proper Android SDK setup
- Use web testing instead: `npm run debug:web`

Android commands (only work with proper SDK setup):
- `npm run build:android` -- Build APK (requires Android SDK)
- `npm run debug:android` -- Build and install debug APK on connected device

## Validation

**CRITICAL**: Always test your changes by running the web application and exercising complete user scenarios.

### Essential Validation Steps
1. **Bootstrap and Build**: Run `npm install && npm run build` to ensure clean build
2. **Start Web Server**: Run `npm run debug:web` and verify server starts on port 8080
3. **Test Application Launch**: Open `http://localhost:8080/www/` and verify application loads
4. **Complete Setup Wizard**: Click through the entire welcome/setup process:
   - Welcome screen with "next" button
   - Ease of access settings (scaling, contrast, etc.)
   - Color selection (accent color picker)
   - Permissions screen
   - What's new screen  
   - Final "Get ready to Disco" screen
5. **Test Main Interface**: Verify the metro-style launcher loads with:
   - Resizable app tiles (Phone, Messages, Chrome, etc.)
   - Live tiles with dynamic content (Contacts with photos, YouTube Music)
   - Navigation between home screen and app list
   - Tile drag and drop functionality
6. **Test Internal Apps**: Access Disco Settings, Disco Tweaks, and Disco Store from app list
   - **Direct Testing**: You can also test internal apps directly at:
     - Disco Settings: `http://localhost:8080/www/apps/disco.internal.settings/`
     - Disco Tweaks: `http://localhost:8080/www/apps/disco.internal.tweaks/`
     - Disco Store: `http://localhost:8080/www/apps/disco.internal.store/`
   - **Expected**: Some internal apps may appear mostly black or show limited functionality in web mode - this is normal

### Validation Scenarios for Code Changes
- **UI/Style Changes**: Always test by running `npm run debug:web`, navigate to affected screens, take screenshots
- **JavaScript Logic Changes**: Test complete user flows from setup through main interface usage
- **Build System Changes**: Test both development (`npm run debug:web`) and production (`npm run build:production`) builds
- **Internal Apps Changes**: Test the specific internal app (Settings/Tweaks/Store) affected

## Project Structure and Navigation

### Key Directories
- `src/` -- Source code for web application
  - `src/script.js` -- Main application entry point
  - `src/scripts/` -- Core application logic and utilities
  - `src/styles.scss` -- Main SASS stylesheet
  - `src/apps/` -- Internal applications (Settings, Tweaks, Store)
- `www/` -- Compiled web assets (generated, do not edit directly)
  - `www/index.html` -- Main launcher interface
  - `www/welcome.html` -- Setup wizard
  - `www/dist/` -- Compiled JavaScript and CSS
  - `www/apps/` -- Compiled internal applications
- `android/` -- Android application project
- `scripts/` -- Build scripts for compilation
- `themes/` -- Community theme gallery

### Important Files for Common Tasks
- **Main UI Logic**: `src/scripts/DiscoBoard.js` -- Core launcher functionality
- **Styling**: `src/styles.scss` and files in `src/styles/`
- **Internal Apps**: Individual folders in `src/apps/` (each has style.scss, script.js, index.html)
- **Mock/Testing**: `src/scripts/discoMock.js` -- Web testing utilities
- **Build Config**: `package.json`, `webpack.config.js`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cherryhoax/DiscoLauncher](https://github.com/cherryhoax/DiscoLauncher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
