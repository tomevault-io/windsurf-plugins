---
trigger: always_on
description: **Rule: All AI-generated documentation MUST be saved to `cursor_docs/` directory.**
---

# Zeronode - Cursor AI Rules

## Documentation Rule (PRIORITY)

**Rule: All AI-generated documentation MUST be saved to `cursor_docs/` directory.**

- ✅ **Always save to**: `cursor_docs/FILENAME.md`
- ❌ **Never save to**: Root directory (except README.md)
- 📝 **File naming**: Use SCREAMING_SNAKE_CASE (e.g., `HANDLER_SIGNATURE_MIGRATION.md`)
- 📏 **Keep docs concise**: Aim for under 400 lines per document
- 🎯 **One topic per doc**: Split large topics into focused documents

**Examples:**
- Architecture analysis → `cursor_docs/ARCHITECTURE_ANALYSIS.md`
- Performance benchmarks → `cursor_docs/BENCHMARK_RESULTS.md`
- Test coverage → `cursor_docs/COVERAGE_ANALYSIS.md`
- Bug fixes → `cursor_docs/BUG_FIX_SUMMARY.md`

---

## Project Overview

**Zeronode** is a minimal, high-performance Node.js microservices framework built on ZeroMQ.

**Tech Stack:**
- Language: JavaScript (ES6+) with Babel
- Transport: ZeroMQ 6.x
- Testing: Mocha + Chai
- Coverage: nyc
- Serialization: MessagePack

**Key Features:**
- Binary protocol with lazy envelope parsing
- Request/response and fire-and-forget patterns
- Peer discovery and heartbeat
- Pattern-based event routing (using @sfast/pattern-emitter-ts)

---

## Code Style & Standards

### Language & Patterns

- **ES6+ JavaScript** with Babel transpilation
- **Functional + OOP hybrid**: Classes for components, functional for utilities
- **WeakMap for private state**: `let _private = new WeakMap()`
- **Standard.js style**: Use `standard` linter
- **No semicolons** (Standard.js)
- **2-space indentation**
- **Single quotes** for strings

### Architecture

**Layered Architecture:**
```
Application Layer    → Node (orchestrator), Client, Server
Protocol Layer       → Protocol (request/response semantics)
Transport Layer      → DealerSocket, RouterSocket (ZeroMQ wrappers)
Envelope Layer       → Envelope (binary serialization)
```

**Key Principles:**
- **Protocol-first design**: Protocol layer is transport-agnostic
- **Lazy evaluation**: Envelope fields parsed on-demand (zero-copy)
- **No blocking operations**: All I/O is async
- **WeakMap for encapsulation**: Private state pattern

---

## Documentation Guidelines

### Location

**ALL documentation must go in `cursor_docs/` directory.**

```
✅ CORRECT:   cursor_docs/FEATURE_ANALYSIS.md
❌ WRONG:     FEATURE_ANALYSIS.md (root)
❌ WRONG:     docs/FEATURE_ANALYSIS.md (user docs only)
```

### Document Length

**Keep documents concise and focused:**
- ✅ Maximum **400 lines** per document
- ✅ If longer, split into multiple docs with clear naming:
  - `PROTOCOL_DESIGN.md`
  - `PROTOCOL_IMPLEMENTATION.md`
  - `PROTOCOL_TESTING.md`

### Document Structure

**Use this template:**

```markdown
# Feature Name

## 🎯 Goal

One paragraph explaining what this achieves.

## 📊 Context (7 lines max)

```javascript
// Show relevant code context (7 lines before/after)
// This helps verify suggestions
```

## 🏗️ Implementation

Step-by-step with code blocks.

## ✅ Verification

How to test/verify the changes.

## 📝 Summary

Key takeaways (3-5 bullet points).
```

### Naming Convention

Use **SCREAMING_SNAKE_CASE** for all documents:
- `ARCHITECTURE_LAYERS.md`
- `PROTOCOL_INTERNAL_API.md`
- `ENVELOPE_OPTIMIZATION.md`

---

## Code Guidelines

### Context Verification (Rule of 7)

**ALWAYS show context (7 lines) when suggesting changes:**

```javascript
// ❌ BAD: No context
function tick() {
  // suggested change
}

// ✅ GOOD: With context
class Protocol extends EventEmitter {
  // ... (context line 1)
  // ... (context line 2)
  tick ({ to, event, data } = {}) {
    let { socket } = _private.get(this)
    
    // ✅ SUGGESTED CHANGE HERE
    if (event.startsWith('_system:')) {
      throw new ProtocolError(...)
    }
    
    validateEventName(event, false)
    // ... (context line 6)
    // ... (context line 7)
  }
}
```

**Why 7 lines?**
- Enough to understand the change
- Not too much to overwhelm
- Easy to verify correctness

### File Organization

```
src/
├── envelope.js        # Binary protocol (low-level)
├── protocol.js        # Request/response semantics (mid-level)
├── client.js          # Client application layer
├── server.js          # Server application layer
├── node.js            # Orchestrator (N clients + 1 server)
├── sockets/           # ZeroMQ transport wrappers
│   ├── dealer.js
│   ├── router.js
│   └── socket.js (base)
├── *-errors.js        # Custom error classes
└── utils.js           # Utilities
```

### Naming Conventions

**Classes:** PascalCase
```javascript
class Protocol extends EventEmitter { }
class DealerSocket extends Socket { }
```

**Private methods:** Underscore prefix
```javascript
_handleTick (buffer) { }
_sendSystemTick ({ event }) { }
```

**Protected methods (for subclasses):** Underscore prefix + JSDoc
```javascript
/**
 * @protected
 */
_getSocket () { }
```

**Public API:** camelCase
```javascript
tick ({ event, data }) { }
request ({ event, data, timeout }) { }
```

**Constants:** SCREAMING_SNAKE_CASE
```javascript
export const ProtocolErrorCode = {
  NOT_READY: 'PROTOCOL_NOT_READY',
  INVALID_EVENT: 'INVALID_EVENT'
}
```

### Error Handling

**Use layer-specific error classes:**

```javascript
// Protocol layer
throw new ProtocolError({
  code: ProtocolErrorCode.NOT_READY,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sfast) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
