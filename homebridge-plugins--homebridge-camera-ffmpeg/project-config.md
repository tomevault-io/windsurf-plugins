---
trigger: always_on
description: Homebridge Camera FFmpeg is a TypeScript-based plugin that provides FFmpeg-based camera support for the Homebridge ecosystem. This plugin supports both Homebridge and HOOBS platforms, includes a custom configuration UI, and provides MQTT/HTTP automation features.
---

# Homebridge Camera FFmpeg Development Guide

Homebridge Camera FFmpeg is a TypeScript-based plugin that provides FFmpeg-based camera support for the Homebridge ecosystem. This plugin supports both Homebridge and HOOBS platforms, includes a custom configuration UI, and provides MQTT/HTTP automation features.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Initial Setup
Run these commands in sequence to set up the development environment:

```bash
# Install FFmpeg (required for camera functionality)
sudo apt update && sudo apt install -y ffmpeg

# Install dependencies (IMPORTANT: Use npm ci to respect package-lock.json)
npm ci  # Takes ~35 seconds. NEVER CANCEL. Use npm ci (not npm install) to maintain version consistency.

# Build the project
npm run build  # Takes ~5 seconds. NEVER CANCEL. Set timeout to 30+ seconds.

# Run tests to verify setup
npm run test  # Takes ~1.3 seconds. NEVER CANCEL. Set timeout to 30+ seconds.

# Verify linting
npm run lint  # Takes ~2.4 seconds. NEVER CANCEL. Set timeout to 30+ seconds.
```

**CRITICAL**: Always use `npm ci` (not `npm install`) for dependency installation to maintain exact version compatibility defined in package-lock.json. Using `npm install` may upgrade dependencies and cause ESLint configuration conflicts.

### Development Workflow
- Build: `npm run build` -- Compiles TypeScript and copies UI files. Takes ~5 seconds.
- Test: `npm run test` -- Runs vitest test suite. Takes ~1.3 seconds.
- Test Coverage: `npm run test-coverage` -- Runs tests with coverage report. Takes ~2 seconds.
- Test Watch: `npm run test:watch` -- Runs tests in watch mode for development.
- Lint: `npm run lint` -- Runs eslint. Takes ~2.4 seconds.
- Lint Fix: `npm run lint:fix` -- Auto-fixes linting issues. Takes ~2-3 seconds.
- Clean: `npm run clean` -- Removes dist/ directory. Takes <1 second.
- Check: `npm run check` -- Checks for outdated dependencies. Takes ~10 seconds.
- Watch: `npm run watch` -- Development mode with nodemon and homebridge.

### Plugin Testing and Validation
To test the plugin locally with Homebridge:

```bash
# Install Homebridge globally
npm install -g homebridge

# Link the plugin locally
npm link

# Create a test configuration in /tmp/homebridge-test-config.json
# Then run Homebridge with the plugin
HOMEBRIDGE_CONFIG_DIR=/tmp homebridge -C /tmp/homebridge-test-config.json -D
```

The plugin should load successfully and display "Loaded plugin: @homebridge-plugins/homebridge-camera-ffmpeg@4.0.1".

## Validation

### CRITICAL Build and Test Requirements
- **NEVER CANCEL** any build, test, or lint commands. They complete quickly but require appropriate timeouts.
- **ALWAYS** run `npm run lint:fix` before committing changes or the CI (.github/workflows/build.yml) will fail.
- **ALWAYS** run the complete workflow after making changes:
  1. `npm run build` (5 seconds)
  2. `npm run test` (1.3 seconds) 
  3. `npm run lint` (2.4 seconds)
- **Test Coverage**: Current coverage is ~5% (low due to limited functional tests). This is normal for camera plugins.
- **Outdated Dependencies**: Running `npm run check` will show outdated packages. This is normal and expected.

### Manual Functional Testing Scenarios
Always test these scenarios after making significant changes:

1. **Plugin Loading Test**: Verify the plugin loads in Homebridge without errors
   ```bash
   HOMEBRIDGE_CONFIG_DIR=/tmp homebridge -C /tmp/test-config.json -D
   ```

2. **Configuration Validation**: Test that the config schema validates properly
   ```bash
   node -e "const config = require('./dist/index.js'); console.log('✓ Plugin exports:', typeof config.default);"
   ```

3. **Camera Configuration**: Create a minimal camera configuration and verify it's accepted
   ```json
   {
     "platform": "Camera-ffmpeg",
     "cameras": [{
       "name": "Test Camera",
       "videoConfig": {
         "source": "-f lavfi -i testsrc2=size=320x240:rate=1",
         "maxStreams": 1,
         "audio": false
       }
     }]
   }
   ```

4. **UI Component**: If changing UI components, verify the custom UI in src/homebridge-ui/ works

### Testing Camera Functionality
While full camera testing requires actual RTSP streams, you can validate basic functionality:

```bash
# Test FFmpeg is available and working
ffmpeg -version

# Test FFmpeg with synthetic test source (validates camera pipeline)
ffmpeg -f lavfi -i testsrc2=size=320x240:rate=1 -t 2 -y /tmp/test_output.mp4

# Verify plugin can be imported and configured
node -e "console.log(require('./dist/index.js'))"

# Test complete development workflow
npm run clean && npm run build && npm run test && npm run lint
```

**Expected Results:**
- FFmpeg should be version 6.1.1+ with H.264 support
- Synthetic test source should create a valid MP4 file
- Plugin should export a function 
- All commands should complete in under 10 seconds total

## Common Tasks

### Repository Structure
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [homebridge-plugins/homebridge-camera-ffmpeg](https://github.com/homebridge-plugins/homebridge-camera-ffmpeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
