---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# OpenCV-JS Package Development Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## About This Project

OpenCV-JS is a TypeScript NPM package that provides OpenCV.js (JavaScript/WebAssembly version of OpenCV) for both Node.js and browser environments. The package wraps a pre-built 11MB OpenCV.js WASM binary with TypeScript type definitions.

**Repository:** https://github.com/TechStark/opencv-js  
**NPM Package:** [@techstark/opencv-js](https://www.npmjs.com/package/@techstark/opencv-js)  
**Version:** 5.0.0-release.1

## Technology Stack

- **Language:** TypeScript
- **Runtime:** Node.js 20.x (also supports browser environments)
- **Build Tool:** TypeScript Compiler (tsc)
- **Testing:** Jest with ts-jest
- **Package Manager:** npm
- **OpenCV Version:** 5.0.0 (WebAssembly/JavaScript build)
- **Key Dependencies:** Jimp (for image loading in tests)

## Table of Contents

- [Working Effectively](#working-effectively)
- [Package Usage Patterns](#package-usage-patterns)
- [Testing and Validation](#testing-and-validation)
- [File Structure and Navigation](#file-structure-and-navigation)
- [CI/CD and Publishing](#cicd-and-publishing)
- [Common Development Tasks](#common-development-tasks)
- [Browser vs Node.js Differences](#browser-vs-nodejs-differences)
- [Performance and Timing Expectations](#performance-and-timing-expectations)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [External Documentation](#external-documentation)
- [Contributing Guidelines](#contributing-guidelines)

## Working Effectively

### Initial Setup and Build
- Install dependencies: `npm install` -- takes ~15 seconds
- Build TypeScript: `npm run build` -- takes ~2 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
- Run tests: `npm test` -- takes ~8 seconds. NEVER CANCEL. Set timeout to 300+ seconds.
- Format code: `npm run format` -- takes ~1 second

### Build Process Validation
- ALWAYS run the complete build process: `npm install && npm run build && npm test`
- Test package creation: `npm pack` -- creates .tgz file for distribution testing
- ALWAYS run `npm audit fix` to address security vulnerabilities before committing

### Manual Testing and Validation
- ALWAYS test OpenCV functionality after making changes using this pattern:
```javascript
const cv = await require('./dist/opencv.js');
global.cv = cv;
const mat = new cv.Mat(3, 3, cv.CV_8UC1);
console.log(`Mat: ${mat.rows}x${mat.cols}, channels: ${mat.channels()}`);
mat.delete(); // CRITICAL: Always call delete() for memory management
```

### Memory Management Requirements
- ALWAYS call `.delete()` on OpenCV objects (Mat, Size, etc.) to prevent memory leaks
- NEVER forget memory cleanup in tests and examples
- Use try/catch with proper cleanup in finally blocks when appropriate

## Package Usage Patterns

### v4.11+ API (Current)
```javascript
import cvReadyPromise from "@techstark/opencv-js";
const cv = await cvReadyPromise;
// Use cv here
```

### v4.10- API (Legacy)
```javascript
import cv from "@techstark/opencv-js";
cv.onRuntimeInitialized = () => {
  // Use cv here
};
```

### Browser Configuration
- ALWAYS include webpack polyfills for browser usage:
```javascript
module.exports = {
  resolve: {
    fallback: {
      fs: false,
      path: false,
      crypto: false
    }
  }
};
```

## Testing and Validation

### Running Tests
- Unit tests use Jest with TypeScript preset
- Tests validate: Mat operations, image processing, color conversions, tracking
- Test files include: `Mat.test.ts`, `Tracker.test.ts`, `rect.test.ts`, `cvKeys.test.ts`
- ALWAYS wait for async OpenCV initialization in tests using `setupOpenCv()` helper

### Key Validation Scenarios
After making changes, ALWAYS test these scenarios:
1. Basic Mat creation and property access
2. Color space conversion (RGBA2GRAY)
3. Image filtering operations (GaussianBlur, threshold)
4. Contour detection and processing
5. Memory cleanup with .delete() calls

### Testing with Real Images
- Use `test/Lenna.png` for image processing tests
- Use Jimp library for loading images in Node.js environment
- Pattern: `const jimpSrc = await Jimp.read(path); const img = cv.matFromImageData(jimpSrc.bitmap);`

## File Structure and Navigation

### Key Directories
- `src/` - TypeScript source (mainly type definitions)
  - `src/index.ts` - Main export (32 bytes, exports from types/opencv)
  - `src/types/` - Comprehensive OpenCV type definitions
- `test/` - Jest test files with OpenCV functionality validation
- `dist/` - Build output including the 11MB opencv.js binary
- `.github/workflows/` - CI/CD configuration

### Important Files
- `dist/opencv.js` - Pre-built OpenCV.js WASM binary (11MB, core functionality)
- `package.json` - NPM configuration with build/test scripts
- `tsconfig.json` - TypeScript compilation settings
- `jest.config.js` - Jest testing configuration
- `doc/cvKeys.json` - Runtime OpenCV methods and properties reference

### Type Definitions Structure
- Over 100 TypeScript definition files in `src/types/opencv/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TechStark/opencv-js](https://github.com/TechStark/opencv-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
