---
trigger: always_on
description: **Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions
---

# ioBroker Adapter Development with GitHub Copilot

**Version:** 0.4.2
**Template Source:** https://github.com/DrozmotiX/ioBroker-Copilot-Instructions

This file contains instructions and best practices for GitHub Copilot when working on ioBroker adapter development.

## Project Context

You are working on an ioBroker adapter. ioBroker is an integration platform for the Internet of Things, focused on building smart home and industrial IoT solutions. Adapters are plugins that connect ioBroker to external systems, devices, or services.

### Adapter-Specific Context: Sonoff/Tasmota MQTT Integration

This is an ioBroker adapter for Sonoff/Tasmota devices that enables MQTT communication between ioBroker and ESP8266/ESP32-based Sonoff devices running Tasmota firmware.

**Key Features:**
- MQTT server/client implementation for Tasmota devices
- Automatic device discovery and state mapping
- Support for sensors, switches, relays, LED controllers, and more
- Dynamic datapoint creation from MQTT telegrams

**Repository Structure:**
- `admin/` - Admin UI configuration files and translations
- `lib/` - Core library files
  - `datapoints.ts` - Defines all supported Tasmota datapoints and their properties
  - `server.ts` - Main MQTT server implementation
- `main.ts` - Adapter entry point
- `test/` - Test files
  - `integration.js` - Integration tests for MQTT message processing
  - `testPackageFiles.js` - Validates package.json and io-package.json

**Key Concepts:**

#### Datapoints
Datapoints in `lib/datapoints.ts` define how Tasmota device parameters are mapped to ioBroker states. Each datapoint has:
- `type` - Data type (string, number, boolean)
- `role` - ioBroker role (state, level, switch, etc.)
- `read/write` - Access permissions
- Optional `min/max` - Value ranges

#### MQTT Message Processing
The adapter processes MQTT messages from Tasmota devices:
- `tele/*/STATE` - Telemetry status messages
- `tele/*/SENSOR` - Sensor data messages
- `stat/*/RESULT` - Command responses
- `cmnd/*` - Commands to devices

**Development Guidelines:**
1. **Adding New Datapoints**: Update `lib/datapoints.ts` with proper type definitions
2. **Testing**: Add corresponding test cases in `test/integration.js` with real MQTT message examples
3. **Translations**: When adding new UI elements to admin configuration:
   - Add all English text entries to `admin/i18n/en/translations.json`
   - Run `npm run translate` to automatically generate translations for all supported languages
   - Verify translations were created in all language directories under `admin/i18n/`
4. **Changelog**: Add entries to README.md under `### **WORK IN PROGRESS**` section for user-facing changes. DO NOT modify `io-package.json` news section as it's automatically updated during releases
5. **Linting**: Always run `npm run lint` to verify code passes ESLint validation
6. **Compatibility**: Maintain backward compatibility with existing Tasmota configurations

**Code Style:**
- Use strict mode and JSHint directives
- Follow existing patterns for consistency
- Use meaningful variable names
- Add comments for complex logic
- Prefer ES6 features where appropriate

**Testing Strategy:**
- Integration tests validate end-to-end MQTT processing using `@iobroker/testing` framework
- Use realistic Tasmota message examples
- Test both positive and negative scenarios
- Ensure all new datapoints have corresponding tests

## Testing

### Unit Testing
- Use Jest as the primary testing framework for ioBroker adapters
- Create tests for all adapter main functions and helper methods
- Test error handling scenarios and edge cases
- Mock external API calls and hardware dependencies
- For adapters connecting to APIs/devices not reachable by internet, provide example data files to allow testing of functionality without live connections
- Example test structure:
  ```javascript
  describe('AdapterName', () => {
    let adapter;
    
    beforeEach(() => {
      // Setup test adapter instance
    });
    
    test('should initialize correctly', () => {
      // Test adapter initialization
    });
  });
  ```

### Integration Testing

**IMPORTANT**: Use the official `@iobroker/testing` framework for all integration tests. This is the ONLY correct way to test ioBroker adapters.

**Official Documentation**: https://github.com/ioBroker/testing

#### Framework Structure
Integration tests MUST follow this exact pattern:

```javascript
const path = require('path');
const { tests } = require('@iobroker/testing');

// Define test coordinates or configuration
const TEST_COORDINATES = '52.520008,13.404954'; // Berlin
const wait = ms => new Promise(resolve => setTimeout(resolve, ms));

// Use tests.integration() with defineAdditionalTests
tests.integration(path.join(__dirname, '..'), {
    defineAdditionalTests({ suite }) {
        suite('Test adapter with specific configuration', (getHarness) => {
            let harness;

            before(() => {
                harness = getHarness();
            });

            it('should configure and start adapter', function () {
                return new Promise(async (resolve, reject) => {
                    try {
                        harness = getHarness();
                        
                        // Configure adapter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ioBroker/ioBroker.sonoff](https://github.com/ioBroker/ioBroker.sonoff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
