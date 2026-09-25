---
trigger: always_on
description: Handles button state and timing.
---

# Universe 2025 badge app development archive

This guidance applies only to the archived Universe 2025 sources under
`badge25/`. New development belongs in `badge/` and must follow
[`badge/AGENTS.md`](../badge/AGENTS.md). Paths written as `/badge/apps/` in
older examples correspond to `/badge25/apps/` in this repository.

This document provides context for maintaining applications for the GitHub
Universe 2025 Tufty badge. Its Badgeware imports, physical buttons, display
behavior, and lifecycle are not compatible with the Universe 2026 runtime.

## Hardware Overview

The GitHub Universe 2025 badge is a custom Pimoroni Tufty 2350 device with the following specifications:

- **Processor**: RP2350 Dual-core ARM Cortex-M33 @ 200MHz
- **Memory**: 512kB SRAM, 16MB QSPI XiP flash
- **Display**: 320x240 full colour IPS (pixel-doubled to 160x120 logical pixels for performance)
- **Screen Dimensions**: WIDTH=160, HEIGHT=120 (all app coordinates use these logical pixels)
- **Runtime**: MicroPython v1.14-5485 with custom badgeware library
- **Connectivity**: 2.4GHz WiFi and Bluetooth 5
- **Battery**: 1000mAh rechargeable lithium polymer (up to 8 hours runtime)
- **Buttons**: 
  - **Front**: UP, DOWN, A, B, C
  - **Back**: HOME (returns to launcher menu)
  - **Hardware**: RESET, BOOTSEL
- **IR**: Receiver (pin 21) and transmitter for beacon hunting and remote control
- **LEDs**: 4-zone backlight (TOP_LEFT, TOP_RIGHT, BOTTOM_LEFT, BOTTOM_RIGHT)
- **Ports**: USB-C (charging/programming), Qw/ST connector, SWD debug
- **GPIO**: 4 additional GPIO pins + power available through solder pads

## App Structure and Lifecycle

### Directory Layout
```
/system/apps/<app_name>/
    __init__.py         # Required - contains init(), update(), on_exit()
    icon.png            # Required - 24x24 PNG icon for launcher
    assets/             # Optional - images, fonts, data files
        *.png           # Images (PNG format, true color or paletted)
        *.ppf           # Pixel fonts (if not using system fonts)
        *.json          # Data files
```

### Required Functions

#### `update()` - Main Loop (REQUIRED)
Called every frame by the main loop. This is where all app logic, input handling, and rendering happens.

```python
def update():
    # 1. Handle input
    if io.BUTTON_A in io.pressed:
        # Button A was just pressed
        pass
    
    if io.BUTTON_B in io.held:
        # Button B is being held down
        pass
    
    # 2. Update game state/logic
    # Use io.ticks for milliseconds since boot
    # Use io.ticks_delta for frame delta time
    
    # 3. Clear screen
    screen.brush = brushes.color(0, 0, 0)
    screen.clear()
    
    # 4. Draw everything
    screen.brush = brushes.color(255, 255, 255)
    screen.text("Hello", 10, 10)
    
    # No explicit display update needed - handled automatically
```

#### `init()` - Initialization (OPTIONAL)
Called once when the app starts. Use for loading resources, setting up state, etc.

```python
def init():
    global game_state, sprite_sheet, font
    
    # Load resources
    sprite_sheet = SpriteSheet("/system/apps/myapp/assets/sprites.png", 4, 2)
    font = PixelFont.load("/system/assets/fonts/nope.ppf")
    
    # Initialize state
    game_state = {
        "score": 0,
        "level": 1,
        "player_x": 80,
        "player_y": 60
    }
    
    # Set up screen
    screen.font = font
    screen.antialias = Image.X2
```

#### `on_exit()` - Cleanup (OPTIONAL)
Called when the user presses HOME or the app terminates. Use for saving state, cleanup, etc.

```python
def on_exit():
    # Save state to file
    try:
        with open("/myapp_save.json", "w") as f:
            json.dump(game_state, f)
    except:
        pass  # Handle gracefully
```

### Starting the App
At the bottom of `__init__.py`, call `run()` with your update function:

```python
run(update)
```

## badgeware API Reference

### Core Modules

#### `screen` - Main Display (160x120 Image object)
The primary drawing surface. All rendering happens on this object.

**Properties:**
- `screen.width` - Always 160
- `screen.height` - Always 120
- `screen.brush` - Current brush (color) for drawing
- `screen.font` - Current font for text rendering
- `screen.antialias` - Antialiasing mode (Image.OFF, Image.X2, Image.X4)
- `screen.alpha` - Global alpha transparency (0-255)

**Methods:**
```python
# Clear screen with current brush color (fastest)
screen.clear()

# Draw shapes (requires screen.brush to be set)
screen.draw(shape)  # shape from shapes module

# Draw text at position
screen.text("Hello", x, y)

# Measure text size
width = screen.measure_text("Hello")

# Blit (copy) image at position
screen.blit(image, x, y)

# Scale blit (resize while blitting, negative dims flip)
screen.scale_blit(image, x, y, width, height)
```

#### `io` - Input and Timing
Handles button state and timing.

**Button Constants:**
- `io.BUTTON_A`, `io.BUTTON_B`, `io.BUTTON_C`
- `io.BUTTON_UP`, `io.BUTTON_DOWN`
- `io.BUTTON_HOME`

**Button State Sets:**
```python
# Buttons just pressed this frame (single fire)
if io.BUTTON_A in io.pressed:
    # Triggered once per press
    
# Buttons currently held down
if io.BUTTON_B in io.held:
    # Triggered every frame while held
    
# Buttons just released this frame
if io.BUTTON_C in io.released:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [badger/home](https://github.com/badger/home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
