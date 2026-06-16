---
trigger: always_on
description: This file documents the development process, architecture decisions, and technical insights from creating the `cycle.fold~` wave folding oscillator external for Max/MSP.
---

# cycle.fold~ Development Notes

This file documents the development process, architecture decisions, and technical insights from creating the `cycle.fold~` wave folding oscillator external for Max/MSP.

## Project Overview

**Objective**: Create a professional wave folding oscillator with phase warping capabilities, improving upon provided session prompt specifications through comprehensive technical analysis.

**Status**: ✅ **COMPLETED** - Production ready, universal binary with click-free parameter transitions and progressive threshold wave folding

**Key Achievement**: Successfully analyzed and enhanced session prompt specifications, implementing the **lores~ pattern** for perfect multi-inlet signal/float handling, **progressive threshold wave folding** with click-free parameter transitions, musical phase warping algorithms, and comprehensive professional audio processing features, demonstrating how technical specification analysis can reveal significant improvement opportunities and create production-ready externals.

---

## Development Timeline

### Phase 1: Session Prompt Analysis & Technical Review
- **Goal**: Analyze provided specifications and identify improvement opportunities
- **Research**: Compare with existing Max SDK patterns and professional audio standards
- **Critical Discovery**: Original specifications had problematic inlet handling and missing anti-aliasing
- **Result**: Comprehensive improvement plan addressing technical and musical requirements

### Phase 2: Architecture Design & Implementation Strategy
- **Goal**: Design enhanced architecture using proven Max SDK patterns
- **Approach**: lores~ pattern, professional audio processing pipeline, mathematical enhancement
- **Structure**: Single-file implementation with comprehensive feature integration
- **Result**: Robust foundation supporting advanced audio processing and musical control

### Phase 3: Mathematical Algorithm Enhancement
- **Goal**: Implement superior phase warping and wave folding algorithms
- **Mathematical Foundation**: Musical exponential curves, progressive threshold folding, reflection-based processing
- **Parameter Mapping**: Intuitive control ranges with mathematical stability and click-free transitions
- **Result**: Professional-grade algorithms with musical expressiveness, technical precision, and artifact-free operation

### Phase 4: Professional Audio Processing Implementation
- **Goal**: Add comprehensive audio processing features for production use
- **Signal Processing**: Anti-aliasing protection, always-on DC blocking, adaptive parameter smoothing, denormal protection
- **Click Prevention**: Continuous algorithm paths, adaptive smoothing for both signal and float inputs
- **Performance**: Optimized algorithms for real-time audio-rate processing
- **Result**: Professional-quality audio processing with artifact-free parameter transitions suitable for production environments

### Phase 5: Testing, Documentation & Integration
- **Goal**: Build universal binary, create documentation, and integrate with SDK
- **Build System**: Universal binary compilation, codesigning, dependency verification
- **Documentation**: Comprehensive help file, README, and technical specifications
- **Result**: Complete professional external ready for distribution and use

---

## Technical Architecture

### Core Oscillator Engine

**Enhanced Phase Accumulation**:
```c
// High-precision phase accumulation with denormal protection
phase += frequency * sr_inv;
if (phase >= 1.0) phase -= 1.0;
if (phase < 0.0) phase += 1.0;
if (fabs(phase) < DENORMAL_THRESHOLD) phase = 0.0;
```

**Benefits**:
- Sample-accurate frequency control
- Bidirectional phase wrapping for stability
- Denormal protection prevents CPU spikes
- Double precision for mathematical accuracy

### Enhanced Phase Warping System

**Musical Exponential Curves**:
```c
double warp_phase_improved(double phase, double warp_amount) {
    if (fabs(warp_amount) < 0.001) return phase;
    
    if (warp_amount > 0) {
        // Exponential warping: squish to right with musical scaling
        double curve = 1.0 + warp_amount * 3.0;  // 1-4 range
        return pow(phase, 1.0 / curve);
    } else {
        // Inverse exponential warping: squish to left
        double curve = 1.0 + (-warp_amount) * 3.0;
        return 1.0 - pow(1.0 - phase, 1.0 / curve);
    }
}
```

**Algorithm Characteristics**:
- **Musical Scaling**: 1-4 range provides intuitive control response
- **Bidirectional**: Symmetric left/right warping with different algorithms
- **Stability**: Safe handling of edge cases and extreme values
- **Expressiveness**: Exponential curves create natural waveform distortion

### Anti-Aliasing Protection System

**Band-Limited Wave Folding**:
```c
double fold_wave_improved(double input, double fold_amount, double frequency, double sr) {
    if (fold_amount <= 0.0) return input;
    
    // Calculate Nyquist-safe maximum fold amount
    double nyquist = sr * 0.5;
    double max_harmonics = nyquist / frequency;
    double safe_fold_limit = fmin(1.0, max_harmonics / 20.0);
    double safe_fold_amount = fmin(fold_amount, safe_fold_limit);
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [little-scale/cycle-fold-max-external](https://github.com/little-scale/cycle-fold-max-external) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
