---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ROS2 package for high-precision GNSS positioning using u-blox receivers: Generation 9 (ZED-F9P/F9R) and the ZED-X20P (all-band). The driver supports DGNSS rover configurations, moving base station setups, and RTCM correction data integration for centimeter-level positioning accuracy. Device family is selected via the `DEVICE_FAMILY` parameter (F9P/F9R/X20P).

## Build System & Common Commands

### Building
```bash
# Build entire workspace
colcon build

# ROS2 distro: prefer the latest installed released distro (Lyrical Luth, May 2026 LTS)
# over jazzy. Source ONE distro consistently - do not mix:
#   bash -c 'source /opt/ros/lyrical/setup.bash 2>/dev/null || source /opt/ros/jazzy/setup.bash; colcon build'
# Stale build cache: if CMake errors reference a DIFFERENT distro (e.g. /opt/ros/rolling),
# that package's build/ dir is stale. Fix: rm -rf build/<pkg> install/<pkg>, then rebuild.

# Build specific package
colcon build --packages-select ublox_dgnss_node

# Build with specific package override
colcon build --packages-select ublox_dgnss_node --allow-overriding ublox_dgnss_node
```

### Code Formatting
```bash
# Format code using uncrustify (config in uncrustify.cfg)
ament_uncrustify --reformat ublox_dgnss_node/src/

# Check formatting without modifying (no --check flag, just omit --reformat)
ament_uncrustify ublox_dgnss_node/src/ ublox_dgnss_node/include/
```

### Testing
```bash
# Full lint suite (cpplint, uncrustify, copyright, lint_cmake, xmllint). cppcheck shows
# as "skipped" when the tool isn't installed - that is not a failure.
colcon test --packages-select ublox_dgnss_node ublox_ubx_msgs
colcon test-result --all --verbose
```

### Running
```bash
# Basic launch with parameter override
ros2 run ublox_dgnss_node ublox_dgnss_node --ros-args -p CFG_USBOUTPROT_NMEA:=False

# Launch configurations
ros2 launch ublox_dgnss ublox_rover_hpposllh.launch.py
ros2 launch ublox_dgnss ublox_rover_hpposecef.launch.py
ros2 launch ublox_dgnss ublox_rover_hpposllh_satellite.launch.py

# NTRIP client for RTCM corrections
ros2 launch ublox_dgnss ntrip_client.launch.py host:=ntrip.data.gnss.ga.gov.au port:=443 mountpoint:=MBCH00AUS0
```

## Architecture Overview

### Component Structure
- **ublox_dgnss_node**: Core driver with USB communication, parameter management, and UBX protocol handling
- **ublox_nav_sat_fix_hp_node**: Converts UBX messages to standard ROS NavSatFix messages
- **ntrip_client_node**: Fetches RTCM correction data from NTRIP casters
- **ublox_ubx_msgs**: ROS2 message definitions for UBX protocol
- **ublox_ubx_interfaces**: ROS2 service definitions for device control

### Key Architectural Patterns

#### Parameter Management System (ENHANCED)
The system uses a sophisticated parameter state machine with thread-safe operations and unified ParameterManager architecture:

**Parameter States:**
- `PARAM_INITIAL`: Default/unknown values
- `PARAM_USER`: User-set values (launch file, runtime)  
- `PARAM_LOADED`: Retrieved from GPS device
- `PARAM_VALSET`: Sent to GPS device
- `PARAM_VALGET`: Reading from GPS device

**Parameter Value Sources:**
- `UNKNOWN`: Parameter exists but value unknown (std::nullopt)
- `DEVICE_ACTUAL`: Real value from u-blox device via CFG-VALGET
- `START_ARG`: From launch file/yaml/command args during initialization
- `RUNTIME_USER`: From ros2 param set during operation

**State Transitions:**
```
PARAM_INITIAL → PARAM_VALGET → PARAM_LOADED        (device fetch)
PARAM_LOADED → PARAM_VALGET → PARAM_LOADED         (hot plug re-fetch)
PARAM_USER → PARAM_VALSET → PARAM_LOADED           (user override)
PARAM_LOADED → PARAM_VALSET → PARAM_LOADED         (modify device value)
```

**Key Classes:**
- `ParameterManager` (`parameters.hpp/cpp`): Unified parameter handling with ParamState class and Builder pattern
- `ParamState` class: Modern parameter state structure with validation and optional values
- Uses `set_parameter_cache()` for initialization and `update_parameter_cache()` for state transitions
- **Critical Fix**: Parameter initialization timing issue resolved - constructor now properly retrieves parameter values

**3-Phase Parameter Initialization:**
1. **Phase 1**: `ublox_send_user_params_async()` - Send ALL user parameters to device FIRST (highest priority)
2. **Phase 2**: `ublox_declare_missing_params()` - Declare missing parameters as PARAM_INITIAL  
3. **Phase 3**: `ublox_fetch_device_params_async()` - Fetch device parameter values

#### USB Hot-Plug Architecture (ENHANCED)
**USB Driver States:** `DISCONNECTED` → `CONNECTING` → `CONNECTED` → `ERROR` (4 states total)

**Device Readiness States:** `UNREADY` → `READY` (parameter-aware state management)

**Enhanced Integration:** 
- ParameterManager coordinates with DeviceReadinessState for parameter synchronization
- Device readiness transitions to `READY` only after parameters are synchronized  
- Reconnection optimization: Only user parameters restored (not full re-initialization)
- Automatic parameter restoration via `restore_user_parameters_to_device()` on hotplug

#### ROS2 Component Architecture
Uses composition-based design with:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aussierobots/ublox_dgnss](https://github.com/aussierobots/ublox_dgnss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
