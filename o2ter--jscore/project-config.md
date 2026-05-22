---
trigger: always_on
description: Both SwiftJS and KotlinJS follow similar architectural patterns while adapting to their respective platform ecosystems.
---

# SwiftJS & KotlinJS - Cross-Platform JavaScript Runtimes

## 1. Project Overview

### Architecture Overview

Both SwiftJS and KotlinJS follow similar architectural patterns while adapting to their respective platform ecosystems.

#### SwiftJS Architecture
SwiftJS is a JavaScript runtime built on Apple's JavaScriptCore, providing a bridge between Swift and JavaScript with Node.js-like APIs:

- **Core Layer** (`swift/Sources/SwiftJS/core/`): JavaScript execution engine and value marshaling
- **Library Layer** (`swift/Sources/SwiftJS/lib/`): Native Swift implementations of JS APIs
- **Polyfill Layer** (`resources/polyfill.js`): Shared JavaScript polyfills for missing APIs

#### KotlinJS Architecture
KotlinJS is a JavaScript runtime built on Javet (Java + V8), providing a bridge between Kotlin and JavaScript with Node.js-like APIs:

- **Core Layer** (`java/jscore/`): JavaScript execution engine and value marshaling using Javet V8
- **Library Layer** (`java/jscore/`): Native Kotlin implementations of JS APIs
- **Platform Layer** (`java/jscore-android/`, `java/jscore-jvm/`): Platform-specific context implementations
- **Polyfill Layer** (`resources/polyfill.js`): Shared JavaScript polyfills for missing APIs

### Unified Project Structure

```
JSCore/
├── swift/                     # SwiftJS implementation
│   ├── Sources/SwiftJS/       # Swift source code
│   ├── Sources/SwiftJSRunner/ # Swift CLI runner
│   └── Tests/                 # Swift test suite
├── java/                      # KotlinJS implementation
│   ├── jscore/                # Core Kotlin module
│   ├── jscore-android/        # Android platform context
│   ├── jscore-jvm/            # JVM platform context
│   │   └──src/test/           # Kotlin test suite
│   └── jscore-runner/         # Kotlin CLI runner
└── resources/                 # Shared JavaScript resources
    └── polyfill.js            # Common polyfills for both platforms
```

**CRITICAL**: Never create circular dependencies between modules. Core abstractions in `java/jscore/` must remain platform-independent, and shared resources in `resources/` should not depend on platform-specific implementations.

### Native Bridge Consistency

#### **CRITICAL:** Native Bridge Consistency Requirements

**Both engines MUST expose identical native bridge modules** to ensure the shared `polyfill.js` file works correctly across platforms:

1. **Synchronized Module Set**: All native bridge modules must be implemented in both SwiftJS and KotlinJS
2. **Identical API Signatures**: Each bridge module must expose the same methods and properties with identical behavior
3. **No Engine-Specific Modules**: Do not add native bridge modules to only one engine - either add to both or don't add at all
4. **Shared Polyfill Dependency**: The `resources/polyfill.js` file depends on these exact module names and APIs

#### Common Native APIs (Both Engines)
- `compression`: Compression and decompression (compress, decompress methods for gzip, deflate, deflate-raw)
- `crypto`: Cryptographic functions (randomUUID, randomBytes, hashing)
- `performance`: High-resolution timing API (now, mark, measure, getEntries, getEntriesByType, getEntriesByName, clearMarks, clearMeasures)
- `processInfo`: Process information (PID, arguments, environment)
- `processControl`: Process control operations (exit, etc.)
- `deviceInfo`: Device identification
- `bundleInfo`: Application metadata
- `FileSystem`: File operations
- `URLSession`: HTTP requests (accepts plain JavaScript objects, not native bridge constructors)
- `WebSocket`: WebSocket connections for bidirectional real-time communication

**Important:** `__NATIVE_BRIDGE__` is passed as a private parameter to the polyfill system and is not exposed as a global object to user JavaScript code.

**Design Principle: Prefer Plain JavaScript Objects Over Native Bridge Constructors**

For short-term data transfer to native APIs, use plain JavaScript objects instead of creating native bridge constructors:

- ✅ **Good**: Pass `{ url: "...", method: "GET", headers: {...} }` to `URLSession.httpRequestWithRequest()`
- ❌ **Bad**: Create `new __NATIVE_BRIDGE__.URLRequest(url)` that requires lifecycle management

**Benefits of plain objects:**
- Simpler architecture with no object lifecycle management
- Reduced memory overhead (no bridge object allocation)
- Better performance (direct property access vs method calls)
- Cleaner code in both JavaScript and native layers

**Example: URLSession API**
The URLSession API was refactored from using a native `URLRequest` constructor to accepting plain JavaScript objects:

```javascript
// Modern approach - plain JavaScript object
const request = {
  url: 'https://example.com',
  httpMethod: 'POST',
  headers: { 'Content-Type': 'application/json' },
  httpBody: JSON.stringify({ data: 'test' }),
  timeoutInterval: 30.0
};
session.httpRequestWithRequest(request, bodyStream, progressHandler);
```

This eliminates the need for:
- Native `URLRequest` class in Swift (`JSURLRequest`)
- Native `URLRequest` class in Kotlin
- Object registry and lifecycle tracking
- Complex property getters/setters through the bridge

**When adding new native bridge modules:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [o2ter/JSCore](https://github.com/o2ter/JSCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
