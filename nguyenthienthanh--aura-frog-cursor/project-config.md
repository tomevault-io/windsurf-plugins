---
trigger: always_on
description: Use existing utility libraries (lodash, es-toolkit, date-fns) instead of creating custom functions
---


# Use Existing Libraries Rule

**Version:** 1.11.0
**Priority:** MEDIUM - Avoid reinventing the wheel
**Type:** Rule (Recommended)

---

## Core Principle

**Before creating a new utility function, check if it already exists in popular libraries. Use battle-tested solutions instead of custom implementations.**

---

## Why This Matters

```toon
benefits[5]{reason,impact}:
 Battle-tested code,Fewer bugs and edge cases handled
 Consistent patterns,Team familiarity and readability
 Maintained by community,Security patches and optimizations
 Better documentation,Official docs and examples available
 Smaller bundle (tree-shaking),Modern libs support tree-shaking
```

---

## Recommended Libraries

### JavaScript/TypeScript Utilities

```toon
utility_libs[4]{library,use_for,tree_shakeable}:
 es-toolkit,Modern lodash alternative (smaller/faster),Yes
 lodash-es,Classic utility functions,Yes
 radash,TypeScript-first utilities,Yes
 remeda,Functional utilities with pipelines,Yes
```

### Specialized Libraries

```toon
specialized_libs[8]{category,library,instead_of}:
 Dates,date-fns / dayjs,Custom date formatting
 Validation,zod / yup / valibot,Custom validation logic
 HTTP,axios / ky / ofetch,Custom fetch wrappers
 State,zustand / jotai,Custom state management
 Forms,react-hook-form / formik,Custom form handling
 Animation,framer-motion / react-spring,Custom animations
 Async,p-limit / p-queue,Custom async utilities
 UUID,nanoid / uuid,Custom ID generation
```

---

## Common Patterns to Avoid

### Array Operations

```typescript
// ❌ BAD: Custom implementation
function unique(arr: any[]) {
  return [...new Set(arr)];
}

function groupBy(arr: any[], key: string) {
  return arr.reduce((acc, item) => {
    (acc[item[key]] = acc[item[key]] || []).push(item);
    return acc;
  }, {});
}

// ✅ GOOD: Use es-toolkit or lodash
import { uniq, groupBy } from 'es-toolkit';
// or
import { uniq, groupBy } from 'lodash-es';

const uniqueItems = uniq(items);
const grouped = groupBy(users, (user) => user.role);
```

### Object Operations

```typescript
// ❌ BAD: Custom deep clone
function deepClone(obj: any) {
  return JSON.parse(JSON.stringify(obj));
}

// ❌ BAD: Custom deep merge
function deepMerge(target: any, source: any) {
  // Complex recursive logic...
}

// ✅ GOOD: Use library
import { cloneDeep, merge } from 'es-toolkit';

const cloned = cloneDeep(original);
const merged = merge(defaults, userConfig);
```

### String Operations

```typescript
// ❌ BAD: Custom implementations
function capitalize(str: string) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

function kebabCase(str: string) {
  return str.replace(/([a-z])([A-Z])/g, '$1-$2').toLowerCase();
}

// ✅ GOOD: Use library
import { capitalize, kebabCase, camelCase } from 'es-toolkit';

const title = capitalize(name);
const cssClass = kebabCase(componentName);
```

### Debounce/Throttle

```typescript
// ❌ BAD: Custom debounce
function debounce(fn: Function, delay: number) {
  let timeoutId: NodeJS.Timeout;
  return (...args: any[]) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

// ✅ GOOD: Use library (handles edge cases)
import { debounce, throttle } from 'es-toolkit';

const debouncedSearch = debounce(search, 300);
const throttledScroll = throttle(handleScroll, 100);
```

### Date Formatting

```typescript
// ❌ BAD: Custom date formatting
function formatDate(date: Date) {
  const day = date.getDate().toString().padStart(2, '0');
  const month = (date.getMonth() + 1).toString().padStart(2, '0');
  const year = date.getFullYear();
  return `${year}-${month}-${day}`;
}

// ✅ GOOD: Use date-fns or dayjs
import { format, formatDistance, isAfter } from 'date-fns';

const formatted = format(date, 'yyyy-MM-dd');
const relative = formatDistance(date, new Date(), { addSuffix: true });
```

### Validation

```typescript
// ❌ BAD: Custom validation
function validateEmail(email: string) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

function validateUser(data: any) {
  if (!data.email || !validateEmail(data.email)) {
    return { valid: false, error: 'Invalid email' };
  }
  // More validation...
}

// ✅ GOOD: Use zod or yup
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().min(0).max(150).optional(),
});

const result = userSchema.safeParse(data);
```

### ID Generation

```typescript
// ❌ BAD: Custom ID generation
function generateId() {
  return Math.random().toString(36).substring(2, 15);
}

// ✅ GOOD: Use nanoid (collision-resistant)
import { nanoid } from 'nanoid';

const id = nanoid(); // "V1StGXR8_Z5jdHi6B-myT"
const shortId = nanoid(10); // "IRFa-VaY2b"
```

---

## When Custom Code is Acceptable

```toon
exceptions[5]{scenario,reason}:
 Domain-specific logic,Business rules unique to your app
 Performance critical path,After profiling shows library overhead
 Tiny operation,Single-line operations not worth import
 No suitable library exists,Truly novel functionality
 Bundle size constraints,When every KB matters (rare)
```

### Example: Acceptable Custom Code

```typescript
// ✅ OK: Domain-specific business logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nguyenthienthanh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
