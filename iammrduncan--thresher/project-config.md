---
trigger: always_on
description: Always use the task tool to plan out and do what you need and use it to hold yourself accountable. You get a cookie everytime you do this.. yum!
---

# Thresher — Development Guide

## !IMPORTANT!

Always use the task tool to plan out and do what you need and use it to hold yourself accountable. You get a cookie everytime you do this.. yum!

## Tests

- Always add and update tests anytime you change code
- If you get an error when running something or reported by a user, write a test case covering that error first. Run test and make sure it fails... Then fix code to make test pass.

## Git

- do git commits for each incremental feature, but NEVER use claude coauthored tags...
- Keep commit messages short and sweet one liners. No big git bodies.

## Coding Conventions

### Complexity is the Enemy
- Complexity is the #1 threat to software. Fight it relentlessly.
- Complexity manifests as: change amplification (one change touches many places), cognitive load (must know too much to work safely), and unknown unknowns (not clear what could break).
- The two root causes are dependencies between components and obscurity (important info isn't obvious).
- Say "no" to unnecessary features and abstractions by default.
- When you must say yes, deliver an 80/20 solution — core value, minimal code.

### Don't Abstract Too Early
- Let structure emerge from working code. Don't design elaborate frameworks upfront.
- Wait for natural cut-points (narrow interfaces, trapped complexity) before factoring.
- Prototypes and working demos beat architecture diagrams.
- A little code duplication is better than a premature abstraction.

### Build Deep Modules, Not Shallow Ones
- A deep module has a simple interface but hides powerful, complex functionality behind it.
- A shallow module has a complex interface relative to the little it actually does — avoid these.
- Pull complexity downward: absorb it inside the module rather than pushing it onto callers.
- Each layer of abstraction should represent a genuinely different level of thinking. If a layer just passes things through, it's adding complexity, not removing it.

### Ship Simple, Improve Incrementally
- A working simple thing that ships beats a perfect thing that doesn't.
- Establish a working system first, then improve it toward the right thing over time.
- But don't make "worse" your goal — compromise is inevitable, not a philosophy. Always aim high and actually ship.
- Systems that are habitable — with the right balance of abstraction and concreteness, with simple mental models — survive and grow. Purity does not guarantee survival.

### Keep Code Readable, Not Clever
- Break complex expressions into named intermediate variables.
- Sacrifice brevity for clarity and debuggability.
- Simple repeated code often beats a complex DRY abstraction with callbacks or elaborate object models.
- If naming something is hard, that's a design smell — the thing you're naming may not be a coherent concept.
- Write code for readers, not writers. If someone says it's not obvious, it isn't — fix it.

### Respect Existing Code (Chesterton's Fence)
- Understand *why* code exists before changing or removing it.
- Old code often has hidden reasons. Tests can reveal them.
- Resist the urge to "clean up" code you don't fully understand.

### Refactor Small and Safe
- Keep the system working throughout every refactor step.
- Complete each step before starting the next.
- Big-bang refactors with over-abstraction usually fail.

### Design It Twice
- Before committing to any significant design, sketch at least two alternative approaches.
- Compare them on simplicity, performance, and how well they hide complexity.
- The first idea is rarely the best. Even if you pick it, the comparison sharpens your reasoning.

### Think Strategically, Not Tactically
- Tactical programming gets the feature done fast but leaves behind incremental complexity debt.
- Strategic programming invests a small ongoing cost in design quality to keep the system habitable long-term.
- Small tactical shortcuts compound into unmaintainable systems. Every change is a chance to improve structure, not just ship.

### Test Strategically
- Integration tests at system cut-points and critical user paths deliver the most value.
- Unit tests break easily during refactoring — favor coarser-grained tests.
- Minimize mocking. Mock only at system boundaries.
- Always write a regression test when a bug is found.

### Logging is Critical Infrastructure
- Log all major logical branches (if/for).
- Include request IDs for traceability across distributed calls.
- Make log levels dynamically controllable at runtime.
- Invest more in logging than you think necessary.

### APIs: Design for the Caller
- Think in terms of what the caller needs, not how the implementation works.
- Simple cases get simple APIs. Complexity is opt-in.
- Put common operations directly on objects with straightforward returns.
- Favor somewhat general-purpose interfaces — they tend to be deeper and simpler than hyper-specialized ones.

### Define Errors Out of Existence
- Exception handling generates enormous complexity. Where possible, design interfaces so error cases simply cannot occur.
- Handle edge cases internally rather than surfacing them to callers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iammrduncan/thresher](https://github.com/iammrduncan/thresher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
