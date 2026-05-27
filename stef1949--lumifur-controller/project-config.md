---
trigger: always_on
description: LumiFur Controller is an embedded C++ project for ESP32 microcontrollers that controls LED matrix displays for Protogen masks. The system features facial expression rendering, smooth animations, and Bluetooth Low Energy (BLE) connectivity for remote control.
---

# GitHub Copilot Instructions for LumiFur Controller

## Project Overview
LumiFur Controller is an embedded C++ project for ESP32 microcontrollers that controls LED matrix displays for Protogen masks. The system features facial expression rendering, smooth animations, and Bluetooth Low Energy (BLE) connectivity for remote control.

### Key Technologies
- **Platform**: ESP32 (Espressif32) with Arduino framework
- **Build System**: PlatformIO with multiple environments
- **Display**: HUB75 64x32 LED matrix panels (2x units)
- **Graphics**: Adafruit GFX library for rendering
- **Communication**: NimBLE for Bluetooth LE connectivity
- **Testing**: Unity and GoogleTest frameworks

## Architecture & Code Structure

### Main Components
- `src/app/main.cpp` (~1500 lines): Core application logic, view management, animations
- `src/app/main.h`: Function declarations, configuration constants, helper functions
- `src/assets/bitmaps.h`: Bitmap data for facial expressions and graphics
- `src/ble/ble.h`: Bluetooth Low Energy communication handling
- `src/hardware/deviceConfig.h`: Hardware pin definitions and device-specific settings
- `src/config/userPreferences.h`: User settings and preferences management

### Key Concepts
- **Views**: Different facial expressions (idle, happy, angry, playful, etc.)
- **Maws**: Mouth/jaw variations for expressions
- **Frames**: Animation frame management for smooth transitions
- **Brightness Control**: Adaptive brightness using APDS9960 sensor
- **BLE Commands**: Remote control via Bluetooth characteristics

## Coding Guidelines & Patterns

### Embedded C++ Best Practices
- **Memory Management**: Be mindful of ESP32's limited RAM (~320KB)
- **Stack Usage**: Prefer stack allocation over dynamic allocation
- **Const Correctness**: Use `const` for read-only data, especially large bitmaps
- **PROGMEM**: Store large static data (bitmaps, fonts) in flash memory using `PROGMEM`
- **Volatile Variables**: Use `volatile` for variables modified in interrupts

### Naming Conventions
- **Variables**: camelCase (e.g., `currentView`, `userBrightness`)
- **Functions**: camelCase (e.g., `debounceButton()`, `setupMatrix()`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `PANEL_WIDTH`, `MAX_BRIGHTNESS`)
- **Classes**: PascalCase (rarely used in this embedded context)

### Code Organization Patterns
```cpp
// Hardware interaction functions
void setupMatrix();
void setupBLE();

// Animation and display functions  
void renderView(uint8_t viewNumber);
void updateAnimation();

// Utility functions
bool debounceButton(int pin);
float easeInOutQuad(float t);

// BLE callback handlers
void onBLECommand(uint8_t command);
```

### Error Handling
- Use return codes for functions that can fail
- Implement proper initialization checks for hardware
- Add debug output using `Serial.print()` when `DEBUG_MODE` is enabled
- Handle BLE connection states gracefully

## Hardware-Specific Considerations

### LED Matrix Management
- **DMA Usage**: The HUB75 library uses DMA for smooth refresh rates
- **Color Depth**: Work with 16-bit RGB565 color format
- **Brightness**: Linear brightness scaling can appear non-linear to human eye
- **Refresh Rate**: Maintain consistent frame rates for smooth animations

### Memory Constraints
- **Bitmap Storage**: Large bitmaps should use `PROGMEM` to save RAM
- **Buffer Management**: Be careful with large frame buffers
- **String Literals**: Use `F()` macro for string literals to save RAM

### Power Management
- **Sleep Modes**: Consider deep sleep for power conservation
- **Pin Management**: Properly configure unused pins
- **Current Draw**: LED matrices can draw significant current

## PlatformIO Environment Guidelines

### Available Environments
- `adafruit_matrixportal_esp32s3`: Primary target (default)
- `esp32dev`: Generic ESP32 development
- `native`: For unit testing
- `codeql`: Static analysis and testing

### Build Commands Context
```bash
# Build for default environment
pio run

# Build for specific environment
pio run -e adafruit_matrixportal_esp32s3

# Upload to device
pio run -t upload

# Run tests
pio test -e native

# Monitor serial output
pio device monitor
```

### Library Dependencies
All required libraries are defined in `platformio.ini`:
- Adafruit GFX Library (graphics primitives)
- ESP32 HUB75 LED MATRIX PANEL DMA Display (matrix control)
- NimBLE-Arduino (Bluetooth Low Energy)
- FastLED (color utilities)
- Adafruit sensor libraries (LIS3DH, APDS9960)

## Testing Guidelines

### Test Structure
- **Native Tests**: Use Unity framework for core logic testing
- **GoogleTest**: Available for more complex test scenarios  
- **Hardware Tests**: Test on actual hardware when possible
- **Coverage**: Code coverage analysis available via test environments

### Testing Patterns
```cpp
// Unity test example
void test_easing_function() {
    TEST_ASSERT_EQUAL_FLOAT(0.0f, easeInQuad(0.0f));
    TEST_ASSERT_EQUAL_FLOAT(1.0f, easeInQuad(1.0f));
}

// Mock hardware interactions for testing
void test_view_switching() {
    uint8_t initialView = currentView;
    // Test view switching logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stef1949/LumiFur_Controller](https://github.com/stef1949/LumiFur_Controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
