---
trigger: always_on
description: This file documents the development process, discoveries, and technical insights from creating the `slewenv~` external for Max/MSP.
---

# slewenv~ Development Notes

This file documents the development process, discoveries, and technical insights from creating the `slewenv~` external for Max/MSP.

## Project Overview

**Objective**: Create a Make Noise Maths-style function generator external that provides integrator-based envelope generation with configurable curve shaping and real-time parameter control.

**Status**: ✅ **COMPLETED** - Production ready, universal binary tested

**Key Achievement**: Discovery and documentation of critical MSP signal/float inlet handling bug that affects all multi-inlet MSP externals.

---

## Development Timeline

### Phase 1: Initial Design (Traditional Envelope Approach)
- **Approach**: Complex phase-based envelope generator (0.0-1.0 phase)
- **Algorithm**: Traditional ADSR-style with exponential time scaling (1ms to 20 minutes)
- **Problem**: Overly complex, timing issues, didn't match Make Noise Maths behavior
- **Result**: Abandoned for simpler integrator approach

### Phase 2: Integrator Redesign (Breakthrough)
- **Realization**: Make Noise Maths is an integrator, not a phase-based envelope
- **New Approach**: Simple per-sample integration like capacitor charging/discharging
- **Algorithm**: `output(n) = output(n-1) + rate_per_sample`
- **Result**: Much simpler, more accurate to hardware behavior

### Phase 3: Parameter Control Crisis (Critical Bug Discovery)
- **Problem**: Parameters were set correctly but external behavior never changed
- **Symptoms**: Float messages received, struct values updated, but always processed default values
- **Investigation**: Extensive debugging revealed MSP signal/float inlet conflict
- **Discovery**: Max provides zero-signals to all `dsp_setup()` inlets, overriding float values
- **Solution**: Force float values or use `count` array for proper signal detection

### Phase 4: Curve Shaping and Polish
- **Feature**: Exponential/linear/logarithmic curve shaping
- **Bug**: Negative linearity values caused envelope to stick at peak
- **Fix**: Improved rate modification to prevent zero rates
- **Result**: Smooth curves across full linearity range

---

## Critical Technical Discoveries

### 🚨 MSP Signal/Float Inlet Conflict (MAJOR BUG)

**The Problem**
When using `dsp_setup((t_pxobject*)x, N)` to create multiple inlets, Max automatically provides zero-valued signals to ALL inlets, even when no cables are connected. This causes float messages to parameter inlets to be overridden.

**Symptoms**
```
// Console output shows parameters are being set:
slewenv~: Rise time set to 1.000 (stored: rise_float=1.000, rise_time=1.000)
slewenv~: Fall time set to 1.000 (stored: fall_float=1.000, fall_time=1.000)

// But debug shows integrator receives default values:
PARAMS IN: rise_time=0.001, fall_time=0.001 -> rates: rise=0.00226757, fall=0.00226757
```

**Root Cause**
```c
// BROKEN: This logic fails because Max provides zero-signals
double rise_time = rise_in ? rise_in[i] : x->rise_float;  // BUG!
// rise_in is NOT NULL (Max provides buffer), but contains zeros
```

**Solution**
```c
// WORKING: Force float values for parameters
double rise_time = x->rise_float;     // Always use stored float
double fall_time = x->fall_float;     // Always use stored float
double linearity = x->linearity_float; // Always use stored float
```

**Impact**: This bug affects ALL multi-inlet MSP externals and cost hours of debugging. Now documented in main CLAUDE.md for future reference.

---

## Technical Architecture

### Integrator State Machine
```
State 0: Idle (stopped)
State 1: Rising (integrating upward)
State 2: Falling (integrating downward)

Transitions:
- Trigger → State 1 (start rising)
- Reach 1.0 → State 2 (start falling)  
- Reach 0.0 → State 0 (stop) or State 1 (loop)
```

### Rate Calculation
```c
// Convert normalized time to seconds
double rise_seconds = rise_time * 10.0;  // 0.1 → 1 sec, 1.0 → 10 sec

// Calculate per-sample rate
double rise_rate = 1.0 / (rise_seconds * sample_rate);
```

**NOTE**: Current timing mapping is approximated based on documentation. Should be verified against real hardware for accurate emulation.

### Curve Shaping Implementation
```c
// Exponential (linearity < 0): Fast start, slow end
if (linearity < -0.001) {
    double progress = x->current_output;
    double curve_factor = 1.0 + (-linearity * 2.0);
    increment *= (1.0 - progress * 0.8) * curve_factor;
}

// Logarithmic (linearity > 0): Slow start, fast end  
else if (linearity > 0.001) {
    double progress = x->current_output;
    increment *= (0.2 + progress * 0.8) * (1.0 + linearity * 1.5);
}
```

---

## Code Organization

### File Structure
```
slewenv~/
├── slewenv~.c      # Main implementation
├── CMakeLists.txt         # Build configuration
├── README.md              # User documentation
├── CLAUDE.md              # Development notes (this file)
└── build/                 # CMake build directory
```

### Function Hierarchy
```
ext_main()                 # Max registration
├── maths_envelope_new()   # Constructor
├── maths_envelope_float() # Message handlers
├── maths_envelope_dsp64() # DSP setup
└── maths_envelope_perform64() # Audio processing
    ├── trigger_integrator()   # Start integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [little-scale/slewenv-max-external](https://github.com/little-scale/slewenv-max-external) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
