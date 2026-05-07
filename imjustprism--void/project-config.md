---
trigger: always_on
description: title: Void Development Rules
---

title: Void Development Rules

rules:
  critical:
    - new files use 2026 license header, NEVER edit existing headers
    - push back when user idea is wrong, user NOT always right
    - NEVER assume values/APIs/module shapes you dont know, ask or inspect via void-mcp
    - smallest possible change, no refactoring unless asked
    - ALWAYS explore @utils/, @components/, @api/ BEFORE writing anything — reuse existing, NEVER reimplement
    - if a helper/component/pattern exists elsewhere, import it, NEVER copy-paste or recreate
    - use void-mcp to inspect live code, NEVER guess module structure
    - run `bun run lint:fix` + `bun run lint:styles:fix` + `bun run tsc` after changes, NEVER `bun run build` unless asked
    - NEVER commit/push without explicit user instruction
    - NEVER commit design specs/plans/docs to the repo
  code:
    - DELETE dead code, never comment out
    - NEVER add comments unless asked, only for non-obvious gotchas
    - capital first letter, period at end for descriptions
    - Logger from @utils/Logger, NEVER console.log/warn/error
    - less code = smarter code, same functionality fewer lines always wins
    - if it can be simpler, make it simpler — no clever, no cute, just clear
    - NEVER overengineer, NEVER overcomplicate, NEVER speculative abstractions
    - NEVER wrapper functions that add nothing, NEVER helpers for one-time operations
    - NEVER backwards-compat shims, feature flags, or dead-code fallbacks — just change the code
    - NEVER add error handling/validation for scenarios that cant happen
    - inline single-use vars, hoist statics out of components/loops
    - no magic numbers, unused imports/exports, dead functions, unused types
    - DRY — if a pattern repeats 2+ times, extract to @utils/ or @components/
    - deduplicate ruthlessly, prefer importing over reimplementing
    - KISS, flat over nested, early returns, guard clauses at top, max 3 nesting levels
    - every code path returns a value, every subscribe() needs unsubscribe()
    - re-read state after await gaps, never trust snapshots across async
    - never Date.now() alone for IDs, add random suffix
    - never index arrays by enum ordinal, use Record maps
    - imports > Logger > finders > types > constants > functions > definePlugin
  perf:
    - Promise.all for parallel async, .find/.some over .filter()[0]
    - Map/Set over objects for dynamic keys, WeakMap for DOM metadata
    - no spread in hot loops, cache computed values, hoist RegExp/frozen objects to module scope
    - batch DOM reads before writes, never interleave (layout thrashing)
    - rAF for visual mutations, never force sync layout in event handlers
    - { passive: true } for scroll/wheel/touch, { once: true } for fire-once listeners
    - { signal } on addEventListener for bulk cleanup via AbortController
    - textContent over innerHTML for plain text
    - structuredClone over JSON.parse(JSON.stringify())
    - avoid async on functions that never await
    - queueMicrotask over Promise.resolve().then()
    - requestIdleCallback for non-critical init
  typescript:
    - prefer ?. ?? const arrow satisfies
    - destructuring, template literals, object shorthand, array methods, for...of, Object.entries
    - early returns, trust inference for locals, annotate exports/params, never nest ternaries
    - any over unknown when cant type properly, grok-types/ for Grok objects
  react:
    - ErrorBoundary.wrap(Component, fallback?)
    - return null not undefined for conditional rendering
    - always cleanup useEffect, prefer AbortController signal
    - never mutate params/settings/store objects, spread-copy first
    - useMemo/useCallback deps must be stable refs, memoize arrays/objects used as deps
    - useCallback over fast-changing state: use refs not closures
    - disabled={loading} over busyRef for double-click prevention
    - static objects/arrays/functions outside component body, no hook needed
    - never create new object/array/function literals in JSX props passed to memoized children
    - useRef for mutable values that dont affect UI, not useState
  security:
    - textContent/createTextNode for user text, never innerHTML with unsanitized data
    - validate external URLs against allowlist before fetch
    - wrap JSON.parse in try/catch at every call site
    - never eval(), setTimeout(string), setInterval(string)
    - wrap patch replacements in try/catch so failed patch never crashes host app
    - no nested regex quantifiers (ReDoS), .{0,N} bounded gaps only
    - reset lastIndex before .test()/.exec() on reused RegExp
  forbidden:
    - bare HTML elements (button, input, select, etc.) when a @components equivalent exists
    - empty catch (except turbopack/module iteration)
    - style={{}} (unless dynamic computed)
    - {arr.length && <Foo/>} (renders 0)
    - .map(x => cond ? <X/> : null) (use .filter().map())
    - barrel imports from @turbopack/common
    - reimplementing utils that exist in @utils/ or @components/
    - wrapper functions/components that add zero value
    - abstractions for a single call site
    - deep nesting beyond 3 levels, nested ternaries
    - direct mutation of Settings objects
    - delete obj.key in hot paths (deoptimizes hidden classes)
    - any type when unknown/generic works

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imjustprism/Void](https://github.com/imjustprism/Void) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
