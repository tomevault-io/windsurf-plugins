---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run tests (uses bun test)
bun test                                    # All tests
bun test test/titan-engine.test.ts          # Single test file
bun test --watch                            # Watch mode

# Run examples
bun run dev                                 # Hello counter (default)
bun run examples/showcase/01-hello-counter.ts  # Or run directly

# Type checking
bun run typecheck                           # Root package

```

## Architecture

### Package Structure
- **Root (`@rlabs-inc/tui`)**: Core framework - primitives, state, layout engine
- **`@rlabs-inc/signals`**: Fine-grained reactivity (separate repo)

### Core Pipeline
```
User Signals → Slot Parallel Arrays → layoutDerived → frameBufferDerived → render effect
```

The framework uses **parallel arrays** (ECS-style) instead of component objects:
- Each ARRAY stores one property type: `width[]`, `height[]`, `color[]`
- Each INDEX represents one component

### Critical Rules
3. **One render effect**: Pipeline is all derived, only final render is an effect

### Key Files
| File | Purpose |
|------|---------|
| `src/pipeline/layout/titan-engine.ts` | TITAN flexbox layout engine |
| `src/primitives/box.ts`, `text.ts`, `input.ts` | UI primitives |
| `src/primitives/each.ts`, `show.ts`, `when.ts` | Template primitives (reactive control flow) |
| `src/engine/arrays/` | Parallel arrays (core, dimensions, spacing, layout, visual, text, interaction) |
| `src/state/keyboard.ts` | Keyboard handling with escape sequence parsing |
| `src/state/focus.ts` | Focus management and tab navigation |
| `src/state/context.ts` | Reactive context system (createContext/provide/useContext) |
| `src/engine/lifecycle.ts` | Component lifecycle hooks (onMount/onDestroy) |

### Keyboard API
```typescript
// Subscribe to specific key
keyboard.onKey('Enter', (event) => { ... })
keyboard.onKey('ArrowUp', handler)

// Subscribe to all keys
keyboard.on((event) => { ... })

// Focus-aware handlers (only fires when component has focus)
keyboard.onFocused(componentIndex, handler)
```

### Box Primitive (Focusable)

Boxes can be made focusable for keyboard interaction:

```typescript
box({
  focusable: true,
  onKey: (e) => {
    // Only fires when this box has focus
    if (e.key === 'Enter') handleAction()
    return true  // consume event
  },
  onFocus: () => console.log('Focused'),
  onBlur: () => console.log('Blurred'),
  children: () => text({ content: 'Press Enter' })
})
```

**Features:**
- `focusable: true` - Enables Tab navigation
- `onKey` - Keyboard handler (fires only when focused)
- `onFocus`/`onBlur` - Focus state callbacks
- Self-contained - no external keyboard handler registration needed

This makes custom focusable components fully self-contained.

### Mouse Props on Primitives

All primitives (box, text, input) support mouse event props:

```typescript
const isHovered = signal(false)

box({
  bg: () => isHovered.value ? t.surface : null,
  onClick: (event) => console.log(`Clicked at ${event.x}, ${event.y}`),
  onMouseDown: (event) => console.log('Mouse down'),
  onMouseUp: (event) => console.log('Mouse up'),
  onMouseEnter: () => { isHovered.value = true },
  onMouseLeave: () => { isHovered.value = false },
  onScroll: (event) => console.log(`Scrolled ${event.scroll?.direction}`),
  children: () => text({ content: 'Hover and click me' })
})
```

**Features:**
- `onClick`, `onMouseDown`, `onMouseUp` - Click events (return `true` to consume)
- `onMouseEnter`, `onMouseLeave` - Hover tracking
- `onScroll` - Scroll wheel events
- **Click-to-Focus**: Focusable elements auto-focus when clicked

This is the recommended approach for component mouse handling. Use global `mouse.*` handlers only for app-wide events.

### TITAN Layout Engine
Complete flexbox: direction, wrap, grow, shrink, basis, justify-content, align-items, align-self, gap, min/max constraints, percentage dimensions. Skips `visible=false` components (takes no space).

### Template Primitives

Reactive control flow primitives for dynamic UIs:

```typescript
// each() - Reactive lists (key is stable, use for selection!)
each(() => items.value, (getItem, key) => {
  text({ content: () => getItem().name, id: `item-${key}` })
}, { key: item => item.id })

// show() - Conditional rendering
show(() => isVisible.value,
  () => text({ content: 'Visible!' }),
  () => text({ content: 'Hidden!' })  // optional else
)

// when() - Async/Suspense
when(() => fetchData(), {
  pending: () => text({ content: 'Loading...' }),
  then: (data) => text({ content: data }),
  catch: (err) => text({ content: err.message })
})
```

**Pattern**: All template primitives capture parent context, render synchronously, then use an internal effect for reactive updates. Components inside use normal props (signals/getters work!).

### Input Primitive

Single-line text input with full reactivity and focus management:

```typescript
import { signal, input, mount, focusManager } from '@rlabs-inc/tui'

const username = signal('')
const password = signal('')

// Basic input with placeholder
input({
  value: username,
  placeholder: 'Enter username...',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RLabs-Inc/tui](https://github.com/RLabs-Inc/tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
