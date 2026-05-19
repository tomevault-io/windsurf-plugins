---
trigger: always_on
description: Detailed Do/Don't examples and fix guidance for React anti-patterns R1-R21 (hooks, effects, state, performance, SRE reliability). Load when a PR review flags a pattern, when implementing features that touch hooks/effects/state, or for the /attack SRE audit command.
---


# React Anti-Patterns and Code Quality Rules

These rules cover the most critical code-level issues in React applications. During `/review` or when implementing features, the agent MUST check for these patterns.

---

## R1 — Missing useEffect Cleanup
Subscriptions, event listeners, and timers created in `useEffect` must be cleaned up to prevent memory leaks.

**Don't**
```tsx
useEffect(() => {
  const subscription = dataSource.subscribe(handleData);
  window.addEventListener('resize', handleResize);
  const timer = setInterval(pollData, 5000);
  // No cleanup - resources persist after unmount
}, []);
```

**Do**
```tsx
useEffect(() => {
  const subscription = dataSource.subscribe(handleData);
  window.addEventListener('resize', handleResize);
  const timer = setInterval(pollData, 5000);

  // REACT: cleanup subscriptions, listeners, timers (R1)
  return () => {
    subscription.unsubscribe();
    window.removeEventListener('resize', handleResize);
    clearInterval(timer);
  };
}, []);
```

**Agent behavior**
- Flag any `useEffect` with `addEventListener`, `subscribe`, `setInterval`, or `setTimeout` that lacks a cleanup return.
- Ensure WebSocket connections, MutationObservers, and ResizeObservers are disconnected on cleanup.
- When reviewing, check that every resource acquisition has a corresponding release.

---

## R2 — Stale Closure in Callbacks
Closures in `useEffect` or callbacks capture variable values at creation time. Without proper handling, they reference stale data.

**Don't**
```tsx
const [count, setCount] = useState(0);

useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1); // Always reads initial count (0)
  }, 1000);
  return () => clearInterval(timer);
}, []); // count not in deps - stale closure
```

**Do**
```tsx
const [count, setCount] = useState(0);

useEffect(() => {
  const timer = setInterval(() => {
    // REACT: functional update avoids stale closure (R2)
    setCount(prev => prev + 1);
  }, 1000);
  return () => clearInterval(timer);
}, []);

// Alternative: useRef for latest value
const countRef = useRef(count);
useEffect(() => {
  countRef.current = count;
}, [count]);
```

**Agent behavior**
- When state is used inside `setInterval`, `setTimeout`, or event callbacks, prefer functional updates (`setState(prev => ...)`) over direct state references.
- If a callback needs the latest value but can't use functional updates, use a ref to track current state.
- Flag any `useEffect` where state variables are used but not in the dependency array without functional updates.

---

## R3 — Object/Array Dependencies Causing Infinite Loops
Objects and arrays are compared by reference. Creating new objects in render causes `useEffect` to see them as "changed" every time.

**Don't**
```tsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  // New object every render
  const options = { userId, includeDetails: true };
  
  useEffect(() => {
    fetchUser(options).then(setUser);
  }, [options]); // Infinite loop - options is always "new"
}
```

**Do**
```tsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  // REACT: memoize object dependencies (R3)
  const options = useMemo(
    () => ({ userId, includeDetails: true }),
    [userId]
  );
  
  useEffect(() => {
    fetchUser(options).then(setUser);
  }, [options]);
}

// Better: use primitive dependencies directly
useEffect(() => {
  fetchUser({ userId, includeDetails: true }).then(setUser);
}, [userId]); // Primitives compare by value
```

**Agent behavior**
- Flag any object or array literal in a `useEffect` dependency array.
- Recommend `useMemo` for complex objects or extracting primitive values as dependencies.
- When reviewing, trace dependency array items to their definitions—if defined inline during render, they will cause re-runs.

---

## R4 — State Update on Unmounted Component
Async operations may complete after component unmounts. Attempting to `setState` on unmounted components causes memory leaks.

**Don't**
```tsx
useEffect(() => {
  fetch(`/api/users/${userId}`)
    .then(res => res.json())
    .then(setData); // May run after unmount
}, [userId]);
```

**Do**
```tsx
useEffect(() => {
  const controller = new AbortController();
  
  fetch(`/api/users/${userId}`, { signal: controller.signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => {
      // REACT: ignore abort errors (R4)
      if (err.name !== 'AbortError') throw err;
    });
  
  return () => controller.abort();
}, [userId]);

// Alternative: mounted flag
useEffect(() => {
  let isMounted = true;
  
  fetchData().then(result => {
    if (isMounted) setData(result);
  });
  
  return () => { isMounted = false; };
}, []);
```

**Agent behavior**
- For any `fetch` or async operation in `useEffect`, require either `AbortController` or a mounted flag.
- Prefer `AbortController` as it actually cancels the network request.
- Flag async callbacks that call `setState` without cancellation handling.

---

## R5 — Direct State Mutation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-pathfinder-app](https://github.com/grafana/grafana-pathfinder-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
