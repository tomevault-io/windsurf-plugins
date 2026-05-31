---
trigger: always_on
description: This is an **ESPHome custom component** for monitoring Tigo solar optimizers via RS485/UART. Two main C++ components work together:
---

# ESPHome Tigo Monitor AI Agent Instructions

## Architecture Overview

This is an **ESPHome custom component** for monitoring Tigo solar optimizers via RS485/UART. Two main C++ components work together:

- **`tigo_monitor/`**: UART frame parser, device tracking, sensor publishing, CCA sync
- **`tigo_server/`**: ESP-IDF HTTP web server with 5 pages + RESTful JSON API

**Critical: ESP-IDF framework required** (not Arduino). All async operations use FreeRTOS tasks/semaphores.

## Memory Architecture: PSRAM is King

**PSRAM usage is fundamental** - not optional for 15+ devices:

```cpp
// PSRAM-backed containers (see tigo_monitor.h lines 1-90)
psram_vector<DeviceData> devices_;           // Device list in PSRAM
psram_map<std::string, NodeInfo> node_table_; // Barcode mapping in PSRAM
PSRAMString json_buffer;                      // API responses in PSRAM
```

**Always use PSRAM containers** for large data structures. Internal RAM is <200KB; PSRAM is 8MB. Web server allocates all JSON/HTML in PSRAM via `PSRAMString` class.

## Configuration Validation Pattern

Sensors use **keyword-based type inference** (see `sensor.py` lines 290-330):

```python
# Sensor type detected from name keywords
has_energy_keywords = any(keyword in sensor_name for keyword in ["energy", "kwh"])
has_frame_keywords = any(keyword in sensor_name for keyword in ["frame", "missed"])
# Returns appropriate schema: ENERGY_SUM_CONFIG_SCHEMA, MISSED_FRAME_CONFIG_SCHEMA, etc.
```

**When adding new sensor types**, update both keyword detection and schema mapping.

## Code Organization Rules

### 1. Python Config (\_\_init\_\_.py, sensor.py, button.py)
- **Schema validation only** - no runtime logic
- Use `cv.float_range()` for bounded floats (e.g., `power_calibration: 0.5-2.0`)
- Keyword-based sensor type inference (see pattern above)

### 2. C++ Headers (tigo_monitor.h, tigo_web_server.h)
- **Setter methods** for config values: `set_power_calibration(float)`
- **Getter methods** for cached data: `get_total_power()`, `get_device_count()`
- Use forward declarations to avoid circular includes

### 3. C++ Implementation (tigo_monitor.cpp, tigo_web_server.cpp)
- **Frame processing**: `process_frame()` → `process_power_frame()` / `process_09_frame()` / `process_27_frame()`
- **Sensor publishing**: Apply `power_calibration_` multiplier to ALL power calculations
- **Web handlers**: Static methods with `httpd_req_t*` parameter

## Development Workflows

### Building & Flashing
```bash
# From workspace root
esphome run tigo-monitor.yaml

# Compile only (check for errors)
esphome compile tigo-monitor.yaml
```

**Common compile errors:**
- Missing `get_*()` method after renaming → Update web server API calls
- Undefined PSRAM types → Add `#ifdef USE_ESP_IDF` guards
- `'class X' has no member 'Y'` → Check for stale method names after refactoring

### Testing Changes
1. **Sensor changes**: Check Home Assistant entities update correctly
2. **Web UI changes**: Test both light/dark modes, mobile responsive
3. **CCA sync**: Verify barcode fuzzy matching works (see `match_barcode()`)
4. **Memory leaks**: Monitor heap before/after midnight reset (see CHANGELOG 1.2.0)

### UART Optimization Context
- **RX buffer size**: 8192 bytes for 30+ devices (see `tigo-monitor.yaml` line 15)
- **ISR in IRAM**: `CONFIG_UART_ISR_IN_IRAM: "y"` reduces frame loss (see `UART_OPTIMIZATION.md`)
- **Processing rate**: `MAX_BYTES_PER_LOOP = 4096` (doubled from 2048 to handle display overhead)
- **Typical miss rate**: 0.02-0.04% is excellent for multi-drop RS485

## Common Patterns

### Adding a Power Calculation
**Always apply `power_calibration_` multiplier**:

```cpp
// WRONG
float power = voltage_out * current_in;

// CORRECT (see tigo_monitor.cpp line 1151, 1223, 1273, etc.)
float power = voltage_out * current_in * power_calibration_;
```

Applied at: individual sensors, string aggregation, web API, power sum calculation.

### Adding Web UI Features
1. **HTML page** in `tigo_web_server.cpp` (see dashboard: lines 1800-2100)
2. **JSON API endpoint** for data (see `api_devices_handler()`)
3. **Register handler** in `setup()` with `httpd_register_uri_handler()`
4. **Use PSRAM**: `PSRAMString html; html.reserve(50000);`
5. **Add auth check**: `if (!check_web_auth(req)) return ...;`

### String Grouping Pattern
CCA provides MPPT/string hierarchy:

```cpp
// Node structure (tigo_monitor.h line 145)
struct NodeInfo {
  std::string inverter_name;  // e.g., "South Inverter"
  std::string mppt_label;     // e.g., "MPPT1"
  std::string panel_name;     // e.g., "A1" (from CCA)
  // ...
};
```

Web dashboard groups devices by `mppt_label`, shows aggregated metrics per string.

### Persistent Data Pattern
Use `esphome::ESPPreferences` for flash storage (see `save_peak_power_data()`):

```cpp
// Reuse string buffer to prevent memory leaks (CHANGELOG 1.2.0)
static std::string pref_key;
pref_key.reserve(32);
for (auto &device : devices_) {
  pref_key = "peak_"; pref_key += device.addr;  // Reuse, not allocate
  prefs.put_float(pref_key.c_str(), device.peak_power);
}
```

**Avoid**: `std::string pref_key = "peak_" + device.addr;` in loops (causes heap fragmentation).

## Web Server Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RAR/esphome-tigomonitor](https://github.com/RAR/esphome-tigomonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
