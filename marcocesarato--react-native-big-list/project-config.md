---
trigger: always_on
description: <project-type>React Native Library</project-type>
---

# React Native Big List - GitHub Copilot Instructions

<meta>
  <project-type>React Native Library</project-type>
  <platforms>iOS, Android, Web, Expo</platforms>
  <main-language>JavaScript (ES6+)</main-language>
  <framework>React Native</framework>
  <test-framework>Jest</test-framework>
  <docs-framework>Docusaurus</docs-framework>
  <license>MIT</license>
</meta>

## 🎯 Project Goal

<goal>
Make small, safe, well-tested improvements to this high-performance React Native list library. Focus on:
- **Bug fixes** with proper test coverage
- **Documentation** improvements and corrections
- **Small feature additions** that maintain backward compatibility
- **Performance optimizations** that are opt-in or transparent
- **Testing** enhancements and edge case coverage

⚠️ **Always prefer minimal, reversible changes with comprehensive validation**
</goal>

## 🗺️ Repository Architecture

### Core Components
<core-files>
| File | Purpose | Key Responsibilities |
|------|---------|---------------------|
| `lib/BigList.jsx` | Main component | Item rendering, scroll/layout event handling, public API |
| `lib/BigListProcessor.js` | Layout engine | Computes visible items, spacers, scroll positions |
| `lib/BigListItemRecycler.js` | Memory optimization | Recycles view objects, prevents memory leaks |
| `lib/BigListItem.jsx` | Item wrapper | Individual item rendering and lifecycle |
| `lib/BigListSection.jsx` | Section support | Section headers/footers, grouped data |
| `lib/BigListPlaceholder.jsx` | Loading states | Placeholder component for unrendered items |
</core-files>

### Public API & Types
<api-files>
| File | Purpose | Update Triggers |
|------|---------|----------------|
| `lib/index.d.ts` | TypeScript definitions | Any prop/method signature changes |
| `index.js` | Main export | New components or utility exports |
| `lib/utils.js` | Shared utilities | Helper functions used across components |
</api-files>

### Development & Testing
<dev-files>
| Directory | Purpose | Usage |
|-----------|---------|-------|
| `__tests__/` | Jest test suites | Validate behaviors, catch regressions |
| `example/` | Expo demo app | Smoke testing, performance validation |
| `docs/` | Docusaurus documentation | API docs, guides, examples |
| `scripts/` | Build tooling | Distribution, publishing automation |
</dev-files>

## 🧩 Architecture Patterns

### Core Design Principles
<patterns>
<pattern name="Pure JavaScript Implementation">
- **Why**: Cross-platform compatibility (iOS, Android, Web, Expo)
- **Rule**: Never add native modules or platform-specific code
- **Validation**: Test on multiple platforms including web
</pattern>

<pattern name="Height-First Rendering">
- **Concept**: All items must have deterministic heights
- **Implementation**: Heights specified as numbers or functions
- **Key Methods**: `BigList.getItemHeight()`, `BigListProcessor.getItemHeight()`
- **Validation**: Check height calculations in processor tests
</pattern>

<pattern name="Processor-Centric Layout">
- **Engine**: `BigListProcessor.process()` computes all layout
- **Outputs**: Visible items array, spacer components, total height
- **Invariants**: Spacer math must be consistent with recycling logic
- **Testing**: Layout changes require processor unit tests
</pattern>

<pattern name="View Recycling">
- **Manager**: `BigListItemRecycler` preserves React elements
- **Benefit**: Prevents expensive re-renders, maintains scroll performance
- **Caution**: Item key/shape changes need recycler mapping updates
- **Debug**: Visual glitches often indicate recycler issues
</pattern>

<pattern name="Dual Data Modes">
- **Flat Mode**: `data` prop with simple array
- **Section Mode**: `sections` prop with grouped data
- **Branching**: `this.hasSections()` determines code path
- **Requirement**: Update both modes for data-related changes
</pattern>
</patterns>

## 🛠️ Development Workflows

### Testing & Validation
<commands>
```bash
# Run complete test suite
yarn test
npm test

# Run tests in watch mode
yarn test --watch

# Run specific test file
yarn test BigList.basic.test.js

# Test with coverage
yarn test --coverage
```
</commands>

### Example App Development
<commands>
```bash
# Navigate to example directory
cd example/

# Install dependencies
npm install

# Start Expo development server
expo start

# Run on specific platform
expo start --ios
expo start --android
expo start --web
```
</commands>

### Build & Distribution
<commands>
```bash
# Build package for distribution
npm run prepare

# This runs:
# 1. bob build (creates dist files)
# 2. node scripts/dist.js (finalizes distribution)

# Lint and format code
npm run prettify  # Prettier formatting
npm run lint      # ESLint checking

# Pre-commit hooks run lint-staged automatically
```
</commands>

## 🔧 Change Guidelines

### Bug Fixes & Runtime Issues
<change-type category="bugfix">
**Focus Areas:**
- Scrolling math errors (off-by-one, position calculations)
- Spacer height inconsistencies
- Memory leaks in recycler
- Event handler edge cases

**Validation Process:**
1. Create focused Jest test reproducing the issue
2. Implement minimal fix
3. Verify test passes
4. Run full test suite
5. Test in example app with edge cases
</change-type>

### API Changes & Extensions
<change-type category="api">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcocesarato/react-native-big-list](https://github.com/marcocesarato/react-native-big-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
