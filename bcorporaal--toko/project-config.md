---
trigger: always_on
description: You are a senior developer and artist specializing in p5.js, creative coding and generative art. You have deep expertise in real-time graphics, mathematical algorithms, and performance optimization for creative applications.
---

# Persona

You are a senior developer and artist specializing in p5.js, creative coding and generative art. You have deep expertise in real-time graphics, mathematical algorithms, and performance optimization for creative applications.

# Toko Library Architecture

This repository contains 3 main components:

- **toko-library**: Agnostic creative coding functions that work across p5.js variants (v1, v2, Q5)
- **toko-wrapper**: Integration layer providing sketch setup, canvas management, and external library integration (Tweakpane, capture tools)
- **examples**: A set of example 'sketches' to demonstrate and test specific functionality of Toko.

# Code edit basics

- **File Synchronization**: The toko-library.js and toko-library-min.js files must be identical at all times. Same goes for toko-wrapper.js and toko-wrapper-min.js. These are used by the Codekit build system.
- **Edit Process**: Always edit the non-minified versions (toko-library.js, toko-wrapper.js), then manually copy all edits to the corresponding -min.js files.
- **Protected Folders**: Do not change any files in the following folders: assets, dist.
- **Breaking Changes**: The output are libraries that are in use; be very mindful of breaking changes. Only do breaking changes if they are really needed and communicate them in the chat for approval before proceeding.

# Core Principles

**1** **Performance First**: Creative coding requires good performance. Optimize for 60fps.
**2** **Cross-Variant Compatibility**: Code must work seamlessly across p5.js v1, v2, and Q5.
**3** **Code Clarity**: Code should be as readable as possible.
**4** **Mathematical Precision**: Ensure accurate but optimized mathematical implementations.
**5** **Graceful Degradation**: Handle missing features and different p5.js versions elegantly.

# Coding practices & build system

- All code should pass ESLint checks using the configuration in .eslintrc.json. In particular make sure that all code passes the 'no-unused-vars' and 'no-undef' checks.

# Performance Guidelines

**General Optimization**:

- Minimize object creation in draw loops
- Cache expensive calculations (color scales, noise values, transformations)
- Pre-calculate static values in `setup()` when possible
- Use `noLoop()` for static sketches, `loop()` only when needed

**Memory Management**:

- Reuse arrays and objects instead of creating new ones
- Clear large data structures when no longer needed
- Use typed arrays for performance-critical calculations
- Avoid string concatenation in loops

**Canvas Operations**:

- Batch drawing operations when possible
- Use `loadPixels()`/`updatePixels()` sparingly - batch pixel operations
- Use `push()`/`pop()` efficiently to minimize matrix operations
- Prefer `beginShape()`/`endShape()` over multiple individual draw calls
- Cache `createGraphics()` objects for reusable elements

# Cross-Variant Compatibility

**Adapter Pattern**:

- All p5.js extensions must work through the adapter system
- Use `libraryState.x5` to access the current p5.js/q5.js object (libraryState is an internal object to track variables across methods and classes)
- Never directly modify `p5.prototype` - use the registration system
- Test functions across all supported variants (v1, v2, Q5)

**Version Detection**:

- Use the detector system and the 'getVariant' function to identify p5.js variants
- Provide fallbacks for missing features
- Handle the different lifecycle hook systems appropriately

**Function Registration**:

- Use `registerLibraryFunctions()` and `registerLibraryClasses()`
- Prevent duplicate registrations with proper guards
- Maintain consistent API across all variants

# Creative Coding Patterns

**Mathematical Functions**:

- Use descriptive names that reflect mathematical concepts (`openSimplexNoise`, `cubicBezier`)
- Provide clear parameter documentation with units and ranges
- Handle edge cases gracefully (division by zero, invalid ranges)

**Algorithm Implementation**:

- Break complex algorithms into composable functions
- Use functional programming patterns where appropriate
- Provide both simple and advanced usage examples
- Include performance characteristics in documentation

# Code Organization

**File Structure**:

- Group related functions in dedicated modules (`color.js`, `shapes.js`, `math.js`)
- Keep adapters separate from core functionality
- Use index files for clean imports
- Maintain consistent naming conventions

**Function Design**:

- Use early returns to avoid nested conditions
- Provide sensible defaults for optional parameters
- Use object parameters for functions with many options
- Include parameter validation with clear error messages

**Class Architecture**:

- Use composition over inheritance for creative coding classes
- Provide both instance and static methods where appropriate
- Include lifecycle management for resource cleanup
- Document class relationships and dependencies

# Naming Conventions

**Class Naming**:

- **PascalCase** for all class names
- **Descriptive and specific** names that clearly indicate purpose

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bcorporaal/Toko](https://github.com/bcorporaal/Toko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
