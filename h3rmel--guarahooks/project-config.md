---
trigger: always_on
description: This document establishes standards for creating performant React hooks, avoiding unnecessary re-renders and memory leaks.
---

# Performance Guidelines

This document establishes standards for creating performant React hooks, avoiding unnecessary re-renders and memory leaks.

## 🚀 Performance Principles

### 1. Reference Stability

- Functions should maintain stable reference
- Objects should be memoized when necessary
- Avoid unnecessary creation of objects/arrays

### 2. Re-render Optimization

- Minimize `useEffect` dependencies
- Use `useCallback` and `useMemo` strategically
- Avoid unnecessary state updates

### 3. Resource Cleanup

- Clean up timers and intervals
- Cancel ongoing requests
- Remove event listeners

## 🎯 Mandatory Optimizations

### useCallback for Functions

```typescript
// ✅ Correct: Stable function
const toggle = useCallback(() => {
  setValue(prev => !prev);
}, []);

// ✅ Correct: Callback with minimal dependencies
const handleChange = useCallback((newValue: T) => {
  setValue(newValue);
  onToggle?.(newValue);
}, [onToggle]);

// ❌ Incorrect: New function on every render
const handleClick = () => {
  setValue(!value);
};
```

### useMemo for Computations

```typescript
// ✅ Correct: Memoization of serialize/deserialize
const serialize = useMemo(
  () => options?.serialize ?? JSON.stringify,
  [options?.serialize]
);

// ✅ Correct: Memoized expensive computation
const expensiveValue = useMemo(() => {
  return heavyComputation(data);
}, [data]);

// ❌ Incorrect: Expensive computation on every render
const result = heavyComputation(data);
```

### useRef for Stable Values

```typescript
// ✅ Correct: Ref for initial value
const initialRef = useRef(initialValue);

// ✅ Correct: Ref for timer
const timeoutRef = useRef<NodeJS.Timeout>();

// ✅ Correct: Ref for latest callback
const latestCallback = useRef(callback);
latestCallback.current = callback;
```

## 🧹 Cleanup Patterns

### Timers and Intervals

```typescript
// ✅ Correct: Timeout cleanup
useEffect(() => {
  const timeoutId = setTimeout(() => {
    setValue(newValue);
  }, delay);

  return () => clearTimeout(timeoutId);
}, [delay, newValue]);

// ✅ Correct: Interval cleanup
useEffect(() => {
  const intervalId = setInterval(() => {
    updateData();
  }, 1000);

  return () => clearInterval(intervalId);
}, []);
```

### AbortController for Requests

```typescript
// ✅ Correct: Request cancellation
useEffect(() => {
  const abortController = new AbortController();

  fetch(url, { signal: abortController.signal })
    .then(response => response.json())
    .then(setData)
    .catch(error => {
      if (!abortController.signal.aborted) {
        setError(error);
      }
    });

  return () => abortController.abort();
}, [url]);
```

### Event Listeners

```typescript
// ✅ Correct: Event listener cleanup
useEffect(() => {
  const handleResize = () => {
    setWindowSize({
      width: window.innerWidth,
      height: window.innerHeight,
    });
  };

  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

## 🔄 State and Updates

### Avoid Unnecessary Updates

```typescript
// ✅ Correct: Update only if value changed
const setValue = useCallback((newValue: T) => {
  if (newValue !== currentValue) {
    setCurrentValue(newValue);
    localStorage.setItem(key, JSON.stringify(newValue));
  }
}, [currentValue, key]);

// ✅ Correct: Deep comparison when necessary
const setValue = useCallback((newValue: T) => {
  if (!isEqual(newValue, currentValue)) {
    setCurrentValue(newValue);
  }
}, [currentValue]);
```

### Update Batching

```typescript
// ✅ Correct: Use functional updates for batching
const updateMultipleValues = useCallback(() => {
  setValues(prev => ({
    ...prev,
    count: prev.count + 1,
    timestamp: Date.now(),
  }));
}, []);

// ❌ Incorrect: Multiple separate setState calls
const updateValues = () => {
  setCount(count + 1);
  setTimestamp(Date.now());
};
```

## 📊 Debounce and Throttle

### Debounced State

```typescript
// ✅ Performant debounce implementation
export function useDebouncedState<T>(
  defaultValue: T,
  delay: number = 500
) {
  const [value, setValue] = useState<T>(defaultValue);
  const timeoutRef = useRef<NodeJS.Timeout>();

  const debouncedSetValue = useCallback((newValue: T) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    timeoutRef.current = setTimeout(() => {
      setValue(newValue);
    }, delay);
  }, [delay]);

  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  return [value, debouncedSetValue] as const;
}
```

### Throttled Callbacks

```typescript
// ✅ Throttle for frequent events
const useThrottledCallback = <T extends (...args: any[]) => any>(
  callback: T,
  delay: number
): T => {
  const throttledRef = useRef<boolean>(false);
  const timeoutRef = useRef<NodeJS.Timeout>();

  return useCallback((...args: Parameters<T>) => {
    if (!throttledRef.current) {
      callback(...args);
      throttledRef.current = true;
      
      timeoutRef.current = setTimeout(() => {
        throttledRef.current = false;
      }, delay);
    }
  }, [callback, delay]) as T;
};
```

## 🎭 SSR and Hydration

### Safe Initial Values

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [h3rmel/guarahooks](https://github.com/h3rmel/guarahooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
