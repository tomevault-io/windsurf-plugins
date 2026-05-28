---
trigger: always_on
description: Claude should help users with the following tasks related to this Home Assistant integration suite for AprilBrother BLE Gateways and BLE device discovery:
---

# AprilBrother BLE Gateway Suite for Home Assistant

## How Claude Should Assist

Claude should help users with the following tasks related to this Home Assistant integration suite for AprilBrother BLE Gateways and BLE device discovery:

### Home Assistant Integration Component
- Modifications to Python code in the custom component
- Configuration flow adjustments
- MQTT integration enhancements
- Scanner functionality improvements
- Service definition updates

### BLE Discovery Add-on
- Modifications to Python code in `ble_discovery.py`
- Adjustments to YAML configuration files
- Docker-related configuration and build issues
- Add-on UI customization

## Key Commands to Run

### Custom Component
- Format: `black custom_components/`
- Lint: `flake8 custom_components/`
- Import sorting: `isort custom_components/`
- Test: `pytest tests/`

### BLE Discovery Add-on
- Lint and check Python code: `flake8 addon/ble_discovery.py`
- Run the add-on locally: `./addon/run.sh`
- Build Docker image: `docker build -t ble-discovery-addon ./addon`

## Important Files

### Custom Component
- `custom_components/ab_ble_gateway/`: Main component directory
- `custom_components/ab_ble_gateway/__init__.py`: Component initialization
- `custom_components/ab_ble_gateway/config_flow.py`: Configuration UI flow
- `custom_components/ab_ble_gateway/scanner.py`: BLE gateway scanner
- `custom_components/ab_ble_gateway/const.py`: Component constants

### BLE Discovery Add-on
- `addon/ble_discovery.py`: Main Python code for BLE device discovery
- `addon/ble_input_text.yaml`, `addon/ble_scripts.yaml`, `addon/btle_dashboard.yaml`: YAML configuration files
- `addon/config.json`: Add-on configuration
- `addon/Dockerfile`: Container build configuration
- `addon/run.sh`: Script to run the add-on

### Dashboard Files
- `btle_combined_dashboard.yaml`: Combined dashboard for BLE device discovery and gateway management
- `btle_dashboard.yaml`: Dashboard for BLE device discovery
- `btle_gateway_management.yaml`: Dashboard for BLE gateway management

## Dashboard Installation

To manually install the dashboard:

1. Go to Home Assistant UI
2. Navigate to Settings > Dashboards
3. Click "Add Dashboard" then select "From YAML"
4. Copy the contents from one of the dashboard YAML files
5. Save the dashboard

### Dashboard Options

1. **Basic Dashboard** (`basic_dashboard.yaml`):
   - Simple status display with gateway attributes
   - MQTT reconnect button
   - Detected devices counter
   - **Recommended for troubleshooting**

2. **Minimal Dashboard** (`minimal_dashboard.yaml`):
   - Ultra-simple view with only raw data
   - MQTT reconnect button
   - Device addition form
   - **Best for initial testing**

3. **Combined Dashboard** (`btle_combined_dashboard.yaml`):
   - Full-featured dashboard with all functionality
   - Multi-view structure with tabs
   - Device discovery and management
   - Gateway status monitoring
   - **Use once basic functionality is confirmed**

### Troubleshooting Dashboard Issues

1. **Entity Not Available**:
   - Make sure integration is correctly installed
   - Verify gateway is sending data (check MQTT Explorer)
   - Try the MQTT reconnect button
   - Restart Home Assistant if needed

2. **Missing Input Helpers**:
   - Create them manually if not automatically created:
     - `input_text.new_ble_device_name`
     - `input_text.new_ble_device_mac`
     - `input_text.new_ble_device_category`

3. **Dashboard Not Visible**:
   - Ensure "Show in sidebar" is enabled
   - Check browser cache or try different browser

4. **Template Syntax Errors**:
   - If dashboard shows errors like `TemplateSyntaxError: expected token ')', got '['`, 
     ensure multi-line JavaScript selectors in templates are on single lines
   - This commonly occurs in query selectors with brackets in device selection code

5. **Null Data Display**:
   - Verify gateway configuration
   - Check MQTT connection
   - Inspect raw payload format
   
6. **Script Issues**:
   - If you see errors like `TypeError: unsupported operand type(s) for +: 'int' and 'str'` in scripts,
     ensure proper type conversion using filters like `|int(-100)` when working with RSSI values
   - If errors mention `Referenced entities button.bluetooth_scan are missing`, the component 
     includes fallback mechanisms to handle missing button entities

## Key Concepts
- AprilBrother BLE Gateway integration
- BLE device scanning and discovery
- MQTT integration with Home Assistant
- Signal strength (RSSI) threshold settings
- Dashboard UI components
- Device categorization and management

## Style Guidelines
- Follow Home Assistant custom component conventions
- Line length: 88 characters (black default)
- Imports: Use isort with sections (stdlib, third-party, first-party)
- Type hints: Required for function parameters and return values
- Error handling: Use try/except blocks with specific exceptions, log errors
- Naming: snake_case for variables/functions, PascalCase for classes
- String formatting: Use f-strings

## Current Improvements (2024-04-13 - v0.3.22)

### Fixed Issues:
1. **Home Assistant Restart on Gateway Reconnect**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [festion/hass-ab-ble-gateway-suite](https://github.com/festion/hass-ab-ble-gateway-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
