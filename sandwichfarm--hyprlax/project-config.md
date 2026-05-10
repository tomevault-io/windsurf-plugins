---
trigger: always_on
description: Instructions for AI coding agents working on hyprlax.
---

# AGENTS.md

Instructions for AI coding agents working on hyprlax.

## Project Overview

Hyprlax is a smooth parallax wallpaper animation system for Hyprland (Wayland compositor). It creates depth effects by moving multiple image layers at different speeds when switching workspaces.

### Core Technologies
- **Language**: C (C99 standard)
- **Graphics**: OpenGL ES 2.0 with EGL
- **Windowing**: Wayland (layer-shell protocol)
- **Build System**: GNU Make
- **Image Loading**: stb_image (header-only library)

### Architecture
- Modular architecture with separated concerns:
  - **Core**: Animation engine, configuration, layer management
  - **Platform**: Wayland abstraction layer
  - **Compositor**: Adapter system for different compositors
  - **Renderer**: OpenGL ES 2.0 rendering with custom shaders
- Runtime IPC system for dynamic control
- Multi-compositor support (Hyprland, Sway, Wayfire, Niri, River, etc.)
- Integrated control interface via `hyprlax ctl` subcommand
- Comprehensive test suite with memory leak detection

## Development Environment

### Required Tools
```bash
# Install dependencies (Arch Linux)
sudo pacman -S base-devel wayland wayland-protocols mesa

# Clone and build
git clone https://github.com/sandwichfarm/hyprlax.git
cd hyprlax
make
```

### Build Commands
```bash
make            # Standard optimized build
make debug      # Debug build with symbols
make clean      # Clean build artifacts
make install    # Install to /usr/local/bin
make test       # Run comprehensive test suite
make memcheck   # Memory leak testing with Valgrind
make coverage   # Test coverage analysis
```

### Testing
```bash
# Test single layer
./hyprlax test.jpg

# Test multi-layer with debug output
./hyprlax --debug --layer bg.jpg:0.3:1.0:expo:0:1.0:3.0 \
                  --layer fg.png:1.0:0.8

# Runtime control (integrated control interface)
./hyprlax ctl status                    # Check daemon status
./hyprlax ctl add image.jpg 1.5 0.8     # Add layer dynamically
./hyprlax ctl set fps 144               # Change frame rate
./hyprlax ctl set duration 2.0          # Modify animation duration
./hyprlax ctl get fps                   # Query current settings
./hyprlax ctl list                      # List active layers

# Run test suite
make test

# Check for memory leaks
make memcheck
valgrind --leak-check=full ./hyprlax test.jpg
```

## Code Style Guidelines

### C Code Conventions
- **Indentation**: 4 spaces (no tabs)
- **Line Length**: Max 100 characters
- **Braces**: K&R style
- **Naming**:
  - Functions: `snake_case`
  - Variables: `snake_case`
  - Constants: `UPPER_SNAKE_CASE`
  - Structs: `snake_case`
  - Enums: `snake_case_t`

### Code Structure
```c
// Good function example
int load_layer(struct layer *layer, const char *path, 
               float shift_multiplier, float opacity) {
    // Input validation first
    if (!layer || !path) {
        return -1;
    }
    
    // Core logic
    // ...
    
    // Resource cleanup
    // ...
    
    return 0;
}
```

### Error Handling
- Return -1 for errors, 0 for success
- Always check malloc/calloc returns
- Free resources in reverse order of allocation
- Use early returns for error conditions

### Comments
- Use `//` for single-line comments
- Document complex algorithms
- Add TODO comments for future work
- No commented-out code in commits

## Contribution Guidelines

### DO's
- ✅ Maintain backward compatibility
- ✅ Add debug output for new features (behind `config.debug`)
- ✅ Update documentation when adding features
- ✅ Test with both single and multi-layer modes
- ✅ Check for memory leaks with valgrind
- ✅ Follow existing code patterns
- ✅ Add examples for new features
- ✅ Keep performance in mind (144 FPS target)

### DON'Ts
- ❌ Break existing command-line interface
- ❌ Add dependencies without discussion
- ❌ Use C++ features (keep it pure C)
- ❌ Ignore compiler warnings
- ❌ Add blocking operations in render loop
- ❌ Use global variables unnecessarily
- ❌ Mix tabs and spaces
- ❌ Leave debug prints in production code

## File Organization

```
hyprlax/
├── src/
│   ├── core/               # Core functionality
│   │   ├── animation.c     # Animation engine
│   │   ├── config.c        # Configuration management
│   │   ├── easing.c        # Easing functions
│   │   └── layer.c         # Layer management
│   ├── platform/           # Platform abstraction
│   │   ├── platform.c      # Platform detection/initialization
│   │   ├── wayland.c       # Wayland implementation
│   ├── compositor/         # Compositor adapters
│   │   ├── compositor.c    # Compositor detection/management
│   │   ├── hyprland.c      # Hyprland IPC integration
│   │   ├── sway.c          # Sway/i3 IPC integration
│   │   ├── wayfire.c       # Wayfire (2D workspaces)
│   │   ├── niri.c          # Niri (scrollable workspaces)
│   │   ├── river.c         # River (tag-based workspaces)
│   │   ├── generic_wayland.c # Generic Wayland fallback
│   ├── renderer/           # Rendering system
│   │   ├── renderer.c      # Renderer interface
│   │   ├── shader.c        # Shader management
│   │   └── gles2.c         # OpenGL ES 2.0 implementation
│   ├── include/            # Header files
│   │   ├── core.h          # Core module definitions
│   │   ├── platform.h      # Platform abstractions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandwichfarm/hyprlax](https://github.com/sandwichfarm/hyprlax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
