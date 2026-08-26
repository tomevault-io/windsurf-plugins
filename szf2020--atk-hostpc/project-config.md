---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 语言设置 / Language Settings

**重要提示：请始终使用中文与用户交流**
- 本项目的用户偏好使用中文进行沟通
- 所有回复、解释和文档都应使用中文
- 代码注释和变量名可以保持英文，但所有说明文字请使用中文

**Important: Always communicate in Chinese with users**
- Users of this project prefer Chinese communication
- All responses, explanations, and documentation should be in Chinese
- Code comments and variable names can remain in English, but all descriptive text should be in Chinese

## Project Overview

This is an industrial glue dispensing control software (工业点胶设备上位机控制软件) v2.0.0 built with Qt6 C++17. It's a professional HMI (Human-Machine Interface) application for precision glue dispensing, automated assembly, and electronic manufacturing applications.

## Build Commands

### Quick Build
```bash
# Standard build
./build.sh Release

# Debug build with full features
./build.sh Debug

# Build with tests
./build.sh Release test

# Build with installation package
./build.sh Release test package

# Quick incremental build (faster for development)
./quick_build.sh Debug
```

### Manual CMake Build
```bash
# Create build directory
mkdir build && cd build

# Configure with different build targets
cmake -DCMAKE_BUILD_TYPE=Release ..                           # Full application
cmake -DCMAKE_BUILD_TYPE=Debug -DBUILD_DEBUG=ON ..           # Debug version
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SIMPLE=ON ..        # Simple demo

# Build
cmake --build . --config Release -j$(nproc)

# Run tests
ctest --output-on-failure

# Create package
cpack
```

### Lint and Quality Checks
```bash
# Build with strict compiler warnings (for lint checking)
mkdir build_lint && cd build_lint
cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-Wall -Wextra -Wpedantic -Werror" ..
make -j4
```

### Running Tests
```bash
# Run all tests from tests/ directory
cd build && ctest --output-on-failure

# Or run the test executable directly
cd build && ./bin/GlueDispensePC_tests

# Build and run tests in one command
cd tests && cmake --build ../build --target GlueDispensePC_tests && ../build/GlueDispensePC_tests
```

## Architecture Overview

### Layered Architecture
The application follows a layered MVC/MVVM architecture:

1. **UI Layer** (`src/ui/`, `src/mainwindow.*`)
   - Device control widgets, parameter configuration, process monitoring
   - Data recording widgets, security/permission management
   - Qt-based widgets with real-time data visualization

2. **Core Business Logic** (`src/core/`)
   - `BusinessLogicManager`: Main business operations coordinator
   - `SystemManager`: System-level management (config, logging, security, monitoring)
   - `UIManager`: UI component management and menu/toolbar setup
   - `EventCoordinator`: Event handling and inter-component communication

3. **Communication Layer** (`src/communication/`)
   - Multi-protocol support: Serial (RS232/RS485), TCP/IP, CAN bus, Modbus RTU/TCP
   - Worker-based threading pattern for device communication
   - Protocol parsers and data processing workers

### Key Components

#### Data Models (`src/data/`)
- **BaseDataModel**: Abstract interface for all data models with serialization/validation
- **ProductionRecord**: Manufacturing batch data and quality tracking
- **BatchRecord**: Batch management with completion/quality tracking
- **AlarmRecord**: Alarm management with levels (Info/Warning/Error/Critical/Emergency)
- **SensorData**: Real-time sensor readings with status monitoring
- **User**: Multi-level permission management (Admin/Operator/Technician)
- **SystemConfig**: System configuration with encrypted storage support
- **DeviceConfig**: Device connection parameters and settings

#### Communication Workers
- **SerialWorker/TCPWorker/CANWorker**: Protocol-specific communication handlers
- **ModbusWorker**: Modbus RTU/TCP protocol implementation
- **DataProcessWorker**: Real-time data processing and filtering
- **ProtocolParser**: Message parsing and validation

#### Configuration Management (`src/config/`)
- **ConfigManager**: Centralized configuration with auto-save
- **AdaptiveConfigManager**: Dynamic configuration adaptation based on performance metrics
- Supports user settings, device parameters, and system preferences
- JSON-based configuration files with validation

#### Performance Management (`src/core/`)
- **PerformanceMonitor**: Real-time system performance monitoring
- **PerformanceAnalyzer**: NEW - Advanced performance analysis with trend detection and smart alerts
- **PerformanceConfigManager**: Performance-aware configuration management
- **MemoryOptimizer**: Memory usage optimization and garbage collection
- **UIUpdateOptimizer**: UI refresh rate optimization for smooth rendering
- **ContinuousOptimizer**: Continuous system optimization based on usage patterns
- **MLPerformancePredictor**: Machine learning-based performance prediction
- **IntelligentAnalyzer**: Intelligent system analysis and recommendations
- **LoadBalancer**: Load balancing for communication and processing tasks
- Performance configuration in `config/performance_config.json`

#### Logging System (`src/logger/`)
- **LogManager**: Multi-level logging (Debug/Info/Warning/Error/Critical)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [szf2020/ATK-HostPC](https://github.com/szf2020/ATK-HostPC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
