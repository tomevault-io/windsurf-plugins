---
trigger: always_on
description: Apply when writing React components. Hook discipline, state placement, performance, async cleanup, and list keys.
---


# Sub-Skill: React Best Practices
<!-- target: ~2600 tokens (real tiktoken count) | 17 rules with severity classification -->

**Purpose:** Prevents the React-specific mistakes LLMs make repeatedly — wrong state placement, stale closures, unnecessary re-renders, and broken async patterns. Concrete rules with code examples.

## Rule classification

- **MUST** — load-bearing. Violating causes infinite loops, stale data, leaked subscriptions, or invisible UI bugs. Never break.
- **SHOULD** — default behavior. Deviation needs a documented reason in the code or PR.
- **AVOID** — usually wrong; documented exception inline where needed.

**Where these rules don't strictly apply:** test fixtures, Storybook stories, design-system primitives in isolation, and small in-tutorial demo components may legitimately differ. The rules below apply to **production application code**.

---

## Component Design

1. **SHOULD: Co-locate state with the component that owns it.** Lift state only when two siblings genuinely share it. Lifting to a grandparent "just in case" causes unnecessary re-renders across the tree.

   ```tsx
   // Avoid: form state lifted to page-level parent
   function Page() {
     const [email, setEmail] = useState('');
     return <Form email={email} setEmail={setEmail} />;
   }

   // Prefer: state lives in the component that uses it
   function Form() {
     const [email, setEmail] = useState('');
     return <input value={email} onChange={e => setEmail(e.target.value)} />;
   }
   ```

2. **SHOULD: Split components at ~100 lines or when a section has its own data concern.** One component = one responsibility. Extract `<UserAvatar>`, `<OrderSummary>` rather than one `<ProfilePage>` that does everything. *Exception: pages that are mostly markup with little logic may exceed 100 lines.*

3. **SHOULD: Replace prop drilling beyond two levels with composition or context.** Passing `userId` through four components to reach a button is a design smell.

   ```tsx
   // Avoid: drilling through intermediaries
   <Layout userId={userId}><Sidebar userId={userId}><Nav userId={userId} /></Sidebar></Layout>

   // Prefer: context or render-prop composition
   <UserContext.Provider value={userId}><Layout /></UserContext.Provider>
   ```

4. **MUST: Never create component definitions inside render.** Inner components are recreated on every render, destroying their state and forcing full remounts.

   ```tsx
   // Wrong
   function Parent() {
     const Child = () => <div>hello</div>; // new reference every render
     return <Child />;
   }

   // Correct: define outside
   const Child = () => <div>hello</div>;
   function Parent() { return <Child />; }
   ```

---

## State Management

5. **MUST: Never mutate state directly.** React compares references. Mutating in place skips re-renders silently.

   ```tsx
   // Wrong
   const [items, setItems] = useState([]);
   items.push(newItem); // mutation — React does not re-render
   setItems(items);

   // Correct
   setItems(prev => [...prev, newItem]);
   ```

6. **SHOULD: Compute derived values in render, not in useEffect.** If a value can be calculated from existing state/props, calculate it inline. useEffect for derived state creates a one-render lag and extra state variables.

   ```tsx
   // Avoid
   const [fullName, setFullName] = useState('');
   useEffect(() => { setFullName(`${first} ${last}`); }, [first, last]);

   // Prefer
   const fullName = `${first} ${last}`;
   ```

7. **MUST: Use useRef for values that must not trigger re-renders** (timers, DOM nodes, previous values). Use useState for anything the UI depends on. Mixing them causes invisible bugs.

   ```tsx
   // Wrong: ref for displayed value
   const count = useRef(0);
   count.current++; // UI never updates

   // Wrong: state for a timer ID
   const [timerId, setTimerId] = useState(null); // triggers re-render on set

   // Correct
   const timerId = useRef(null);
   ```

---

## Hooks

8. **MUST: Specify complete dependency arrays in useEffect.** Omitting a dependency creates a stale closure. The ESLint rule `exhaustive-deps` must be enabled and respected.

   ```tsx
   // Wrong: stale closure over userId
   useEffect(() => { fetchUser(userId); }, []); // runs once, userId never updates

   // Correct
   useEffect(() => { fetchUser(userId); }, [userId]);
   ```

9. **MUST: Cancel async operations in useEffect cleanup.** Fetch without an AbortController causes state updates on unmounted components and race conditions.

   ```tsx
   useEffect(() => {
     const controller = new AbortController();
     fetch(`/api/user/${id}`, { signal: controller.signal })
       .then(r => r.json())
       .then(setUser)
       .catch(err => { if (err.name !== 'AbortError') setError(err); });
     return () => controller.abort();
   }, [id]);
   ```

10. **MUST: Never call hooks conditionally or inside loops.** Hook call order must be identical on every render. Wrap conditional logic inside the hook body, not around the hook call.

    ```tsx
    // Wrong
    if (isLoggedIn) { const user = useUser(); }

    // Correct
    const user = useUser(); // hook always called; handle null inside
    ```

---

## Performance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sordi-ai/skill-everything](https://github.com/sordi-ai/skill-everything) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
