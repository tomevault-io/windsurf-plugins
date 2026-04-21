---
trigger: always_on
description: **Always use `bun` for package management, not `npm`, `yarn`, or `pnpm`.**
---

# Coding Style Guidelines

## Package Manager

**Always use `bun` for package management, not `npm`, `yarn`, or `pnpm`.**

Examples:
- `bun install` (not `npm install`)
- `bun add <package>` (not `npm install <package>`)
- `bun run dev` (not `npm run dev`)

## React UI Components

### shadcn/ui Components

**Always prefer using shadcn/ui components when designing React UI.**

shadcn/ui provides high-quality, accessible, and customizable components that follow best practices. Use them instead of building custom components from scratch.

**Installing new shadcn components:**
```bash
bunx shadcn@latest add <component-name>
```

Examples:
- `bunx shadcn@latest add button`
- `bunx shadcn@latest add dialog`
- `bunx shadcn@latest add card`

**Benefits:**
- Accessibility built-in (ARIA, keyboard navigation)
- Consistent design system
- Tailwind CSS integration
- Fully customizable (copied to your project)
- TypeScript support
- No additional bundle size overhead

**When to use shadcn:**
- Forms (Input, Label, Select, Checkbox, etc.)
- Dialogs and Modals
- Buttons and interactive elements
- Cards and containers
- Tables and data display
- Navigation components
- Feedback components (Alert, Toast, etc.)

**Available components:** Check [ui.shadcn.com](https://ui.shadcn.com) for the full list of components.

## Functional Programming Principles

### 1. Prefer Functional Style Over Class-Based

Always use functional programming patterns with closures instead of classes:

**✅ Good - Functional with closures:**
```typescript
export function createVoiceHandler() {
    const sessions = new Map()
    const connections = new Map()
    
    function handleConnection(socket, userId) {
        // Logic here
        return cleanup
    }
    
    return { handleConnection }
}
```

**❌ Avoid - Class-based:**
```typescript
export class VoiceHandler {
    private sessions = new Map()
    private connections = new Map()
    
    handleConnection(socket, userId) {
        // Logic here
    }
}
```

**Benefits:**
- Encapsulation via closures (truly private state)
- No `this` binding issues
- Simpler mental model
- Easier to compose and test
- Better tree-shaking

### 2. Prefer Inline Anonymous Functions

Use inline arrow functions instead of creating discrete named functions, especially for callbacks and event handlers:

**✅ Good - Inline anonymous functions:**
```typescript
socket.on("message", (data) => handleMessage(data))
socket.on("error", (err) => console.error("Socket error:", err))
socket.on("close", () => cleanup())

const handlers = {
    onData: (data) => processData(data),
    onError: (err) => logError(err),
    onComplete: () => finalize()
}
```

**❌ Avoid - Named function declarations:**
```typescript
const handleMessage = (data) => {
    // Logic
}

const handleError = (err) => {
    console.error("Socket error:", err)
}

const handleClose = () => {
    cleanup()
}

socket.on("message", handleMessage)
socket.on("error", handleError)
socket.on("close", handleClose)
```

**Exception:** Create named functions only when:
- The function is complex and needs clear documentation
- The function is used in multiple places
- Debugging requires a clear stack trace name

### 3. Resource Cleanup Pattern

**All functions that allocate resources MUST return cleanup functions:**

**✅ Good - Returns cleanup function:**
```typescript
function handleConnection(socket, userId) {
    // Setup resources
    const handlers = {
        message: (data) => process(data),
        close: () => cleanup()
    }
    
    socket.on("message", handlers.message)
    socket.on("close", handlers.close)
    
    // Return cleanup function
    return () => {
        socket.off("message", handlers.message)
        socket.off("close", handlers.close)
    }
}

// Usage
const cleanup = handleConnection(socket, userId)
// Later: cleanup()
```

**✅ Good - Nested resource cleanup:**
```typescript
function createSession(config) {
    let cleanupDB = null
    let cleanupCache = null
    
    const db = connectDB(config.dbUrl)
    cleanupDB = () => db.disconnect()
    
    const cache = connectCache(config.cacheUrl)
    cleanupCache = () => cache.close()
    
    // Combined cleanup
    const cleanup = async () => {
        if (cleanupCache) cleanupCache()
        if (cleanupDB) await cleanupDB()
    }
    
    return { db, cache, cleanup }
}
```

**❌ Avoid - Manual cleanup without returned function:**
```typescript
function handleConnection(socket, userId) {
    socket.on("message", handleMessage)
    socket.on("close", handleClose)
    // No cleanup function returned!
}

// Cleanup must be done manually elsewhere - error prone!
```

### 4. Self-Contained Functions

Functions should be as self-contained as possible:

**✅ Good - Self-contained with all dependencies:**
```typescript
function processData(input, config, logger) {
    const result = transform(input, config)
    logger.log("Processed:", result)
    return result
}
```

**❌ Avoid - Implicit external dependencies:**
```typescript
// Depends on external state
let globalConfig
let globalLogger

function processData(input) {
    const result = transform(input, globalConfig)
    globalLogger.log("Processed:", result)
    return result
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lingosandi/mono-sandbox](https://github.com/lingosandi/mono-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
