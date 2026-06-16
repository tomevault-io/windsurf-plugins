---
trigger: always_on
description: This file documents the development process, discoveries, and technical insights from creating the `tide~` external for Max/MSP.
---

# tide~ Development Notes

This file documents the development process, discoveries, and technical insights from creating the `tide~` external for Max/MSP.

## Project Overview

**Objective**: Create a Mutable Instruments Tides-inspired LFO/envelope generator external that captures the core waveshaping algorithms while being self-contained and buildable without external dependencies.

**Status**: ✅ **COMPLETED** - Production ready, universal binary tested

**Key Achievement**: Successful demonstration of C++/C integration patterns for incorporating complex DSP algorithms into Max externals via clean wrapper interfaces.

---

## Development Timeline

### Phase 1: Requirements Analysis (SESSIONPROMPT.md Study)
- **Goal**: Implement authentic Tides 2 PolySlopeGenerator algorithm
- **Challenge**: Original Tides code had complex stmlib dependencies
- **Decision**: Create simplified recreation rather than direct port
- **Result**: Self-contained implementation with authentic behavior

### Phase 2: Architecture Design (C++ Integration Strategy)
- **Approach**: Separate C++ DSP core from Max C interface
- **Pattern**: `extern "C"` wrapper functions for clean language boundary
- **Structure**: Opaque pointer pattern for C++ object management
- **Result**: Clean separation of concerns, easy to maintain

### Phase 3: Algorithm Implementation (Simplified Tides)
- **Core**: Asymmetric ramp generator with variable slope control
- **Shaping**: Exponential/linear/logarithmic curve morphing
- **Smoothing**: Dual-mode processing (filtering + wavefolding)
- **Modes**: AD (envelope), Loop (LFO), AR (gate-controlled)
- **Result**: Captures essential Tides character without complexity

### Phase 4: Max Integration (Apply Previous Lessons)
- **Inlet Design**: 5 signal/float dual inlets (frequency, trigger, shape, slope, smooth)
- **Parameter Safety**: Applied `slewenv~` lessons about MSP signal/float conflicts
- **Build System**: CMake with C++ standard and universal binary support
- **Result**: Robust external with proper parameter handling

### Phase 5: Testing and Documentation
- **Build Verification**: Universal binary (x86_64 + ARM64)
- **Help File**: Interactive Max patch with all modes demonstrated
- **Documentation**: Complete README and development notes
- **Result**: Production-ready external with comprehensive documentation

---

## Technical Architecture

### File Structure
```
tide~/
├── tide~.c              # Main Max external (C interface)
├── tides_wrapper.cpp    # C++ DSP core and wrapper functions
├── CMakeLists.txt       # Build configuration
├── README.md            # User documentation
├── CLAUDE.md            # Development notes (this file)
└── build/               # CMake build directory
```

### C++ Integration Pattern

**Core Design Principle**: Clean separation between Max C interface and C++ DSP algorithms

```c
// tide~.c - Max external interface
typedef struct _tide {
    t_pxobject ob;
    void* poly_slope_generator;  // Opaque pointer to C++ object
    // ... Max-specific state
} t_tide;

// Forward declarations for C++ wrapper
extern "C" {
    void* tides_create(void);
    void tides_destroy(void* obj);
    void tides_render(void* obj, /* parameters */);
}
```

```cpp
// tides_wrapper.cpp - C++ implementation
namespace tides {
    class PolySlopeGenerator {
        // Pure C++ DSP implementation
    };
}

// C wrapper functions
extern "C" {
    void* tides_create(void) {
        return new tides::PolySlopeGenerator();
    }
    
    void tides_destroy(void* obj) {
        delete static_cast<tides::PolySlopeGenerator*>(obj);
    }
}
```

### Algorithm Design

**Simplified Tides Recreation Strategy**:

1. **Ramp Generation**: Core asymmetric ramp with variable slope
2. **Shape Processing**: Mathematical curve transformations
3. **Smoothness Effects**: Filtering (< 0.5) and folding (> 0.5)
4. **Mode Handling**: State machine for AD/Loop/AR behaviors

```cpp
// Core algorithm structure
float GenerateRamp(RampMode mode, float frequency) {
    // Asymmetric ramp with slope-controlled attack/decay
}

float ApplyShaping(float input, float shape, float slope) {
    // Exponential/linear/logarithmic curve morphing
}

float ApplySmoothing(float input, float smoothness) {
    // Dual-mode: filtering or wavefolding
}
```

---

## Critical Technical Discoveries

### 🎯 C++ Integration Best Practices

**The Pattern**: Clean language boundary with opaque pointers
```c
// GOOD: Opaque pointer with wrapper functions
void* cpp_object;
cpp_object = cpp_create();
cpp_process(cpp_object, params);
cpp_destroy(cpp_object);

// AVOID: Direct C++ in Max external
// Mixing C++ objects directly in Max structs leads to complications
```

**Benefits**:
- Clean compilation (separate C and C++ files)
- Easy debugging (clear language boundaries)
- Maintainable code (encapsulated C++ logic)
- Portable pattern (works for any C++ library)

### 🎯 Algorithm Recreation Strategy

**The Approach**: Simplified recreation vs. direct port

**Why Recreation Works Better**:
- No external dependencies (stmlib, etc.)
- Easier to understand and modify
- Self-contained build process
- Captures essential character without complexity

**Implementation Focus**:
```cpp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [little-scale/tide-max-external](https://github.com/little-scale/tide-max-external) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
