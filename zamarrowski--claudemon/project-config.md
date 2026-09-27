---
trigger: always_on
description: This document provides instructions for AI agents working on the claudemon codebase.
---

# AI Agent Guidelines

This document provides instructions for AI agents working on the claudemon codebase.

claudemon is a terminal Pokémon game driven by Claude Code activity: plain ESM Node,
no build step, no runtime dependencies, rendered as ANSI lines into a terminal.

These rules are the house style. Where existing code contradicts them, the existing
code is wrong and gets brought in line as it is touched — do NOT copy a violation
because the file next to you has one, and do NOT cite existing code as precedent
against a rule here.

## Project shape

```
bin/claudemon        entry point, argument parsing, boot
src/                 the engine (battle, capture, exp, state, queue, sound, update)
src/ui/              rendering primitives (screen, ansi, sprite, grass, widgets)
src/ui/views/        one file per screen, each exporting draw() and onKey()
scripts/             Claude Code hook handlers and the status line
tools/               dev-time scripts (fetch data, fetch sprites, preview, capture, install)
test/                test suites
data/                generated dataset, checked in — never hand-edited
```

Everything is ESM, `.mjs`, Node >= 20.19. The runner is Vitest. Prettier owns
formatting (no semicolons, single quotes, 80 columns) — write the code and run
`npm run format`, never hand-align anything.

## General Guidelines

### General Rules

**NO COMMENTS**: NEVER add comments to the code. Code should be self-documenting and
clear without them. If a block needs explaining, extract it into a named function
instead. The only comments allowed anywhere are tooling directives
(`// prettier-ignore`, `// eslint-disable-next-line`).

**NAMING CONVENTIONS**: Always use camelCase for naming variables, functions and
files. Module-level constants use SCREAMING_SNAKE_CASE.

**ARROW FUNCTIONS**: Declare functions as arrows assigned to a const, and export the
const. NEVER use `function` declarations.

```js
// ✅ GOOD
export const partyIsWipedOut = (save) => {
  return save.party.length > 0 && save.party.every(isFainted)
}

// ❌ BAD
export function partyIsWipedOut(save) {
  return save.party.length > 0 && save.party.every(isFainted)
}
```

**ARROW FUNCTION BODIES**: If the arrow function body fits on the same line, you can
use the implicit return (no braces, no `return`). If it doesn't fit on the same line
and needs to wrap, you MUST use `{}` with an explicit return. NEVER use an implicit
return that wraps to the next line.

```js
// ✅ GOOD — fits on the same line, implicit return
const levelOf = (mon) => mon.level
const isFainted = (mon) => mon.hp <= 0

// ✅ GOOD — doesn't fit on the same line, braces + explicit return
const getUsableMoves = (mon, includeStatus) => {
  return mon.moves.filter(
    (slot) => slot.pp > 0 && (includeStatus || slot.power),
  )
}

// ❌ BAD — implicit return wrapping to the next line
const getUsableMoves = (mon, includeStatus) =>
  mon.moves.filter((slot) => slot.pp > 0 && (includeStatus || slot.power))
```

**CONSTANTS**: If a module has constants (with no logic), they MUST be extracted to a
`constants.mjs` file. Lookup tables, message strings, tunable rates and stock lists
all belong there — never inline inside a branch.

There is **one `constants.mjs` per directory**, and a module imports from the one that
sits next to it: `src/constants.mjs`, `src/ui/constants.mjs`,
`src/ui/views/constants.mjs`, `scripts/constants.mjs`, `tools/constants.mjs`. This is
what the flat **Project shape** above requires — do NOT promote a module to its own
folder just to give it a private constants file.

A constant only moves out if it is pure data. A value that calls a function, derives
from another value, or reads `process.env` is not a constant in this sense and stays in
the module that owns it — that is why every path in `src/paths.mjs` (all built with
`join()`) stays where it is.

```js
// ✅ GOOD — pure data, moves to src/constants.mjs
export const POISON_FRACTIONS = { poison: 8, burn: 16 }

// ✅ GOOD — derived at load time, stays in src/paths.mjs
export const SAVE_FILE = join(HOME, 'save.json')
```

**ONE CONCERN PER FILE**: Each file MUST cover one thing. NEVER put two unrelated
screens, two unrelated engines, or a screen and the logic it drives in the same file.

**MODULE TESTING**: Every time a new module is created, its corresponding tests MUST
be implemented.

**NO CONDITIONAL SPREAD**: NEVER use spread operators to assemble the arguments or
options a function receives. Always pass each field explicitly by name.

```js
// ✅ GOOD
createApp({
  screen: stubScreen(),
  save,
  config,
  playSound: noop,
})

// ❌ BAD
createApp({
  screen: stubScreen(),
  ...(save && { save }),
  ...options,
})
```

**NO INLINE CALLBACKS AS HANDLERS**: NEVER define an arrow function directly in the
call that registers it. Extract it into a named handler declared in the body above
and pass the reference. The only exception is inside iterators (`.map`, `.filter`,
`.some`), where the inline arrow is an argument, not a handler.

```js
// ✅ GOOD
const handleResize = () => paint(ctx)

screen.onResize(handleResize)

// ✅ GOOD — exception: inline inside an iterator
const usable = mon.moves.filter((slot) => slot.pp > 0)

// ❌ BAD
screen.onResize(() => paint(ctx))
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zamarrowski/claudemon](https://github.com/zamarrowski/claudemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
