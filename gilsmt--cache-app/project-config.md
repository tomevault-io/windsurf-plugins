---
trigger: always_on
description: [Cache](https://www.cachd.app) is a modern well-crafted purpose-built personal bookmark knowledge web application tool that unifies user bookmarks across all mainstream platforms into a single, searchable, actionable library. Read [README.md](README.md) for more.
---

# AGENTS.md

## Project overview

[Cache](https://www.cachd.app) is a modern well-crafted purpose-built personal bookmark knowledge web application tool that unifies user bookmarks across all mainstream platforms into a single, searchable, actionable library. Read [README.md](README.md) for more.

## Development workflow

Cache has a zero technical debt policy. Do it right the first time: the design that lands in the codebase should be the correct one, with no intentional debt in that surface. A problem solved in design costs less than one solved in implementation, which costs less than one solved in production. "Right the first time" describes the landed output, not the exploration that produced it — see simplicity below. When rules conflict, prefer in order: correctness and safety of the change surface, then scope discipline (task-only files and isolation), then local coherence in files you already touch, then YAGNI, then style.

Leave the codebase better than you found it.

Suggest solutions or alternatives I didn’t think about and anticipate my needs.

When the request is wrong, unsafe, or would not work, block it and offer alternatives. When it is merely suboptimal, challenge once with a concrete alternative, then execute the user's choice unless a hard constraint still fails. Reframe from first principles when that reaches a better answer.

Learn from existing code: Study and plan before implementing. Identify recurring patterns and design influences in the code. Keep rules or constraints of the task in mind.

Trace how parts connect, such as data flow between functions, stage dependencies, or what module owns what.

If a tradeoff is required, choose correctness and robustness over short-term convenience or shortcuts.

Define success criteria. Loop until verified.

It is not about formatting or syntax. Linters handle that. It is about how to think, how to make decisions, and what to value when building software.

Read the full implementation of what you change and its direct callers/callees, not just the signature and not the whole repo.

Simple and elegant systems are easier to design correctly, more efficient in execution, and more reliable. That simplicity requires hard work and discipline.

Simplicity is not the first attempt. It is the hardest revision. It takes thought, multiple passes, and the willingness to throw work away. The goal is to find the idea that solves multiple problems at once.

Follow the rationale that each function should have a single, named responsibility. Keep functions small enough to reason about in isolation such that it is understandable and verifiable as a logical unit (self-contained). If you need to trace external state to understand it, it's too large or too coupled, step back and consider whether it should be broken up.

Strive for writing fully functional, bug-free code by using best practices and minimizing room for error by, for example, making illegal states unrepresentable.

Prohibit over-encapsulation and over-abstraction of code.

Avoid unnecessary code indirection. Extract when the same reason to change applies in two or more modules and the name is obvious; similar code with different futures may stay duplicated. Extracting a className string into a constant just because it is used twice is not justified.

Follow YAGNI. Prefer the smallest clear unit, not the fewest lines — one-liners only for pure expressions with no branching, I/O, or error paths.

Composition over inheritance. Prefer dependency injection.

Handle errors at the appropriate scopes. Never silently swallow exceptions. If you think an error cannot happen, assert that assumption explicitly.

Never compromise type safety: avoid `any`, `!` (non-null assertion), and `as Type` casting as they usually indicate wrong assumptions or bad implementation. A cast is allowed only at a trust boundary (SDK, ORM, framework) when the invariant is runtime-checked or guaranteed by a typed wrapper one layer in. Prefer narrowing (`zod`, predicates, exhaustiveness). If you need a cast deeper than the boundary, fix the model.

Declare variables at the smallest possible scope. Minimize the number of variables in play at any point. This reduces the probability of using the wrong variable and makes code easier to reason about. Calculate or check variables close to where they are used. Do not introduce variables before they are needed or leave them around when they are not.

Plugin architectures allow for extensibility and isolation; most functionality should live in plugins, not the core, enabling parallel development and future-proofing. Apply a plugin boundary when pluggability is itself a current requirement (sync adapters, export formats, AI providers). YAGNI governs speculative features — do not extract a plugin boundary for a single implementation.

Minimize risk by anticipating what’s most likely to fail (platforms, language changes, hardware, people...) and insulating your system from those points of failure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gilsmt/cache-app](https://github.com/gilsmt/cache-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
