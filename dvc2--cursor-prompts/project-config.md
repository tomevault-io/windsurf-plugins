---
trigger: always_on
description: description: Systematic debugging approaches with minimal tool calls
---

---
description: Systematic debugging approaches with minimal tool calls
globs: 
  - "**/*.ts"
  - "**/*.js"
  - "**/*.py"
  - "**/*.java"
  - "**/*.go"
  - "**/*.rb"
  - "**/*.php"
  - "**/*.cs"
  - "**/*.cpp"
alwaysApply: false
---

# Efficient Debugging Strategies

## 🎯 Debugging Philosophy

### The 5-Step Debug Protocol
```
1. REPRODUCE → Confirm the issue exists
2. ISOLATE → Find the smallest failing case
3. DIAGNOSE → Identify root cause
4. FIX → Apply minimal solution
5. VERIFY → Ensure fix works
```

## 🔍 Initial Assessment

### Quick Triage (1 Tool Call)
```bash
# Get complete context in one call
echo "=== Debug Context ===" && \
pwd && \
echo -e "\n--- Error State ---" && \
tail -50 error.log 2>/dev/null || echo "No error log" && \
echo -e "\n--- Recent Changes ---" && \
git diff --stat HEAD~1 2>/dev/null || echo "No git" && \
echo -e "\n--- Running Processes ---" && \
ps aux | grep -E "(node|python|java)" | grep -v grep | head -5 && \
echo -e "\n--- Test Status ---" && \
npm test -- --no-coverage 2>&1 | tail -20 || echo "No tests"
```

## 🐛 Common Bug Patterns

### Type 1: Null/Undefined Errors
```javascript
// SYMPTOM: "Cannot read property 'x' of undefined"

// ❌ BAD DEBUG: Random changes
console.log(user);
console.log(user.profile);
console.log(user.profile.name);

// ✅ GOOD DEBUG: Systematic check
console.log({
  hasUser: !!user,
  hasProfile: !!user?.profile,
  profileKeys: user?.profile ? Object.keys(user.profile) : 'none',
  nameValue: user?.profile?.name
});

// FIX PATTERN:
const name = user?.profile?.name ?? 'Default';
```

### Type 2: Async/Promise Issues
```javascript
// SYMPTOM: "Promise pending" or race conditions

// ✅ DIAGNOSTIC PATTERN:
console.log({
  stage: 'before-await',
  timestamp: Date.now()
});

const result = await operation();

console.log({
  stage: 'after-await',
  timestamp: Date.now(),
  hasResult: !!result,
  resultType: typeof result
});

// FIX PATTERNS:
// 1. Missing await
const data = await fetchData();

// 2. Race condition
const [result1, result2] = await Promise.all([op1(), op2()]);

// 3. Error handling
try {
  const data = await riskyOperation();
} catch (error) {
  console.error('Operation failed:', error.message);
}
```

### Type 3: State Management Issues
```javascript
// SYMPTOM: "State not updating" or "Stale values"

// ✅ STATE DEBUG HELPER:
function debugState(label, state) {
  console.log(`[${label}]`, {
    state: JSON.stringify(state, null, 2),
    type: typeof state,
    isArray: Array.isArray(state),
    timestamp: new Date().toISOString()
  });
}

// USE:
debugState('Before update', currentState);
updateState(newValue);
debugState('After update', currentState);
```

## 🔧 Debugging Tools

### Universal Debug Logger
```javascript
// Add to any file for instant debugging
const DEBUG = {
  log: (label, data) => {
    console.log(`\n🔍 [${label}]`, {
      data,
      stack: new Error().stack.split('\n')[2],
      time: new Date().toISOString()
    });
  },
  
  checkpoint: (name) => {
    console.log(`✓ Checkpoint: ${name} at ${new Date().toISOString()}`);
  },
  
  measure: async (label, fn) => {
    const start = performance.now();
    try {
      const result = await fn();
      console.log(`⏱️ ${label}: ${(performance.now() - start).toFixed(2)}ms`);
      return result;
    } catch (error) {
      console.log(`❌ ${label} failed after ${(performance.now() - start).toFixed(2)}ms`);
      throw error;
    }
  }
};

// Usage:
DEBUG.checkpoint('Starting process');
const data = await DEBUG.measure('Database query', () => db.query(sql));
DEBUG.log('Query result', data);
```

### Error Boundary Pattern
```javascript
// Wrap suspicious code
function safeTry(operation, fallback = null) {
  try {
    return operation();
  } catch (error) {
    console.error('Safe operation failed:', {
      error: error.message,
      stack: error.stack,
      operation: operation.toString()
    });
    return fallback;
  }
}

// Usage:
const config = safeTry(() => JSON.parse(configString), {});
```

## 📊 Systematic Debugging

### Binary Search Debug
```javascript
// For "worked before, broken now" issues
// 1. Find last working commit
git bisect start
git bisect bad HEAD
git bisect good abc123  // last known good

// 2. Test each commit
npm test || git bisect bad
npm test && git bisect good

// 3. Find exact breaking change
git bisect reset
git show <bad-commit>
```

### Divide & Conquer
```javascript
// For complex flows
function debugFlow() {
  console.log('Step 1: Input validation');
  // If fails here, input issue
  
  console.log('Step 2: Data transformation');
  // If fails here, transformation issue
  
  console.log('Step 3: Business logic');
  // If fails here, logic issue
  
  console.log('Step 4: Output formatting');
  // If fails here, formatting issue
}
```

## 🎪 Performance Debugging

### Quick Performance Check
```javascript
// One-liner performance wrapper
const perf = (fn, label = 'Operation') => {
  const start = Date.now();
  const result = fn();
  console.log(`${label}: ${Date.now() - start}ms`);
  return result;
};

// Usage:
const data = perf(() => processLargeDataset(), 'Dataset processing');
```

### Memory Leak Detection
```javascript
// Memory snapshot helper
const memorySnapshot = (label) => {
  if (global.gc) global.gc();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DVC2/cursor_prompts](https://github.com/DVC2/cursor_prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
