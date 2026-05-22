---
trigger: always_on
description: JavaScript best practices, ES2022+ features, and modern patterns for scalable applications
---

# JavaScript Best Practices

Modern JavaScript development guide focusing on ES2022+ features, performance, and clean code patterns.

## 1. Modern JavaScript Fundamentals

### 1.1 Variable Declarations and Scope

```javascript
// Use const for immutable bindings
const API_KEY = process.env.API_KEY;
const config = Object.freeze({
  timeout: 5000,
  retries: 3
});

// Use let only when reassignment is needed
let retryCount = 0;
while (retryCount < MAX_RETRIES) {
  retryCount++;
}

// Block scope with const/let
{
  const temp = calculateTemp();
  // temp is only available in this block
}
```

### 1.2 Destructuring Patterns

```javascript
// Advanced destructuring with defaults and renaming
const { 
  data: users = [], 
  meta: { total = 0, page = 1 } = {},
  ...rest 
} = response;

// Array destructuring with rest
const [first, second, ...remaining] = items;

// Destructuring in function parameters
function createUser({ 
  name, 
  email, 
  role = 'user',
  metadata: { source = 'web' } = {}
} = {}) {
  return { id: generateId(), name, email, role, source };
}

// Dynamic property destructuring
const key = 'name';
const { [key]: value } = object;
```

### 1.3 Object and Array Operations

```javascript
// Object property shorthand and computed properties
const name = 'John';
const age = 30;
const user = {
  name,
  age,
  [`is${age}YearsOld`]: true,
  // Method shorthand
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

// Array methods for immutability
const doubled = numbers.map(n => n * 2);
const filtered = users.filter(u => u.active);
const sum = numbers.reduce((acc, n) => acc + n, 0);
const found = users.find(u => u.id === targetId);
const hasAdmin = users.some(u => u.role === 'admin');
const allActive = users.every(u => u.active);

// Object transformation
const transformed = Object.entries(data)
  .filter(([key, value]) => value != null)
  .reduce((acc, [key, value]) => ({ ...acc, [key]: transform(value) }), {});
```

## 2. Advanced Functions and Closures

### 2.1 Function Patterns

```javascript
// Default parameters with destructuring
function fetchData(url, { 
  method = 'GET', 
  headers = {}, 
  timeout = 5000 
} = {}) {
  return fetch(url, { method, headers, signal: AbortSignal.timeout(timeout) });
}

// Rest parameters and spread
function combine(separator, ...parts) {
  return parts.filter(Boolean).join(separator);
}

// Immediately Invoked Function Expression (IIFE)
const module = (() => {
  let privateVar = 0;
  
  return {
    increment() { privateVar++; },
    getCount() { return privateVar; }
  };
})();

// Function composition
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

// Currying
const curry = (fn) => {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...nextArgs) => curried(...args, ...nextArgs);
  };
};
```

### 2.2 Closures and Private State

```javascript
// Module pattern with private state
function createCounter(initial = 0) {
  let count = initial;
  
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    reset() { count = initial; return count; },
    valueOf() { return count; }
  };
}

// Memoization with closure
function memoize(fn, keyFn = JSON.stringify) {
  const cache = new Map();
  
  return function memoized(...args) {
    const key = keyFn(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

## 3. Asynchronous JavaScript

### 3.1 Promises and Async Patterns

```javascript
// Promise creation and chaining
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Promise combinators
async function fetchAllData(urls) {
  // Parallel execution
  const results = await Promise.all(urls.map(url => fetch(url)));
  
  // Handle partial failures
  const settledResults = await Promise.allSettled(urls.map(url => fetch(url)));
  
  // Race condition
  const fastest = await Promise.race([
    fetch('/api/primary'),
    fetch('/api/backup')
  ]);
  
  // First successful result
  const firstSuccess = await Promise.any([
    fetch('/api/server1'),
    fetch('/api/server2'),
    fetch('/api/server3')
  ]);
}

// Async iteration
async function* fetchPages(baseUrl) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const response = await fetch(`${baseUrl}?page=${page}`);
    const data = await response.json();
    yield data.items;
    hasMore = data.hasNextPage;
    page++;
  }
}

// Using async generators
for await (const items of fetchPages('/api/users')) {
  processItems(items);
}
```

### 3.2 Advanced Error Handling

```javascript
// Retry mechanism with exponential backoff
async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = baseDelay * Math.pow(2, i);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Circuit breaker pattern
class CircuitBreaker {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DVC2/cursor_prompts](https://github.com/DVC2/cursor_prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
