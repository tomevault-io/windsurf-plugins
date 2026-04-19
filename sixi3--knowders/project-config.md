---
trigger: always_on
description: You are helping develop "Knowder" - a lightweight JavaScript library for displaying engaging facts during loading states. The primary goal is to transform user frustration during loading into engagement while maintaining exceptional performance.
---

# Knowder Library Development Rules

## Project Context
You are helping develop "Knowder" - a lightweight JavaScript library for displaying engaging facts during loading states. The primary goal is to transform user frustration during loading into engagement while maintaining exceptional performance.

## Core Principles (NON-NEGOTIABLE)

### 1. Bundle Size Supremacy
- **HARD LIMIT**: <5KB gzipped final bundle
- Every byte matters - question every character
- Use bundlesize tool to verify on every build
- NO external runtime dependencies (zero dependencies rule)
- Vanilla JavaScript only - no frameworks

### 2. Performance First
- Initialization must be <50ms
- Memory usage must stay <1MB
- Maintain 60fps animations
- CPU impact <1% during idle
- Use native CSS transitions/animations only
- Optimize for browser rendering engine

### 3. Developer Experience Excellence
- API must be intuitive (max 3 main methods)
- Zero configuration should work out of box
- Framework agnostic design
- Comprehensive documentation with examples
- Clear error messages and graceful degradation

## Technical Constraints

### Code Style & Patterns
- Use ES6+ features (target modern browsers only)
- Prefer functional programming patterns where possible
- Use singleton pattern for main API
- Implement proper error handling and input validation
- Comment complex logic thoroughly
- Use JSDoc for all public methods

### DOM Manipulation Rules
- Use native DOM APIs only (no jQuery)
- Minimize DOM queries and mutations
- Use documentFragment for complex DOM operations
- Implement proper cleanup to prevent memory leaks
- Respect user accessibility preferences

### Animation Guidelines
- CSS transitions/animations only (no JavaScript animations)
- Use transform and opacity for GPU acceleration
- Respect prefers-reduced-motion media query
- Keep animations smooth and non-jarring
- Aim for 60fps performance

### File Organization
```
src/
├── core/
│   ├── factLoader.js    # Main API class
│   ├── factStorage.js   # Data management
│   └── display.js       # DOM/animation engine
├── data/
│   └── defaultFacts.js  # Default fact database
└── index.js             # Public API exports
```

## Development Workflow Rules

### Testing Requirements
- Maintain 90%+ test coverage
- Write tests before implementing features (TDD)
- Test in multiple browsers (Chrome, Firefox, Safari, Edge)
- Include accessibility testing
- Performance regression testing

### Code Quality Gates
1. All tests must pass
2. Bundle size must be <5KB gzipped
3. ESLint must pass with zero warnings
4. No console.log statements in production code
5. Proper JSDoc documentation for all public APIs

### Git Commit Guidelines
- Use conventional commit format
- Include bundle size impact in commit messages
- Reference TODO items being completed
- Small, focused commits

## API Design Rules

### Public API Surface
```javascript
// Main methods (keep to 3 maximum)
Knowder.init(options)     // Initialize with config
Knowder.start(category)   // Start displaying facts
Knowder.stop()           // Stop and cleanup

// Utility methods (minimal)
Knowder.addFacts(category, facts)  // Add custom facts
Knowder.getCategories()            // List categories
```

### Configuration Schema
- Use sensible defaults for everything
- Make all options optional
- Validate input and provide clear error messages
- Support both programmatic and declarative configuration

### Error Handling Strategy
- Never crash the host application
- Use console.warn for non-critical errors
- Provide fallback behavior for all scenarios
- Clear error messages with suggested fixes

## Content Guidelines

### Default Facts Criteria
- Must be genuinely interesting and surprising
- Keep under 140 characters for readability
- Verify accuracy before inclusion
- Ensure cultural sensitivity and inclusivity
- Categorize appropriately (general, science, tech, history)

### Fact Quality Standards
- Prefer "did you know" style revelations
- Include credible sources in comments
- Avoid controversial or sensitive topics
- Make facts universally understandable
- Test readability with diverse audiences

## Browser Support Strategy

### Supported Browsers
- Chrome 88+ (ES6 modules, modern CSS)
- Firefox 78+ (similar feature support)
- Safari 14+ (WebKit parity)
- Edge 88+ (Chromium-based)

### Feature Detection
- Use feature detection over browser detection
- Graceful degradation for unsupported features
- No polyfills (to maintain zero dependencies)
- Progressive enhancement approach

## Build & Distribution Rules

### Build Process
- Use Rollup.js for optimal tree shaking
- Generate both ESM and UMD builds
- Aggressive minification with Terser
- Source maps for debugging
- Automated bundle analysis

### Quality Assurance
- Run bundlesize check on every build
- Lighthouse performance audits
- Cross-browser automated testing
- Accessibility compliance verification
- Security vulnerability scanning

## Documentation Standards

### Code Documentation
- JSDoc for all public methods and classes
- Include usage examples in comments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sixi3) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
