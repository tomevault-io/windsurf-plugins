---
trigger: always_on
description: Plan, design, and ship production-grade software end-to-end. Project planning - discovery, PRD, ADRs, work breakdown, roadmap, definition-of-done. SDLC playbooks - Agile, Lean/MVP, DevOps/CI-CD, TDD/property-test-first, Waterfall for regulated medical and finance, DDD/hexagonal. Principles - SOLID, DRY, YAGNI, KISS, parse-don't-validate, boring-tech-first. Data structures - balanced trees (AVL/RB/treap), B/B+/LSM, segment/Fenwick/persistent, heaps, hash tables (Robin Hood, cuckoo), graphs, tries
---


# dsa-pro: Plan, design, and ship the right structures and algorithms

A senior-engineer reference for taking a software project from fuzzy intent to production-grade code, then keeping it production-grade as it evolves. Combines a complete project-planning + SDLC playbook with the original dsa-pro data-structure and algorithm catalogue. Optimized for "translate a fuzzy ask into a deliverable plan and a correct, fast implementation the first time."

## When to invoke

Invoke when *any* of the following are true:

**Planning / architecture / process:**
- "Help me plan / scope / architect / design [a system]" — anything where the user needs to go from intent to plan.
- "What's the right SDLC for [domain]?" — picking between Agile, Lean, DevOps, TDD-first, Waterfall, DDD-style. Regulated domains (medical, finance), greenfield startup, enterprise migration, brownfield refactor — each has a default playbook.
- "Write me a PRD / ADR / spec / RFC / work breakdown / roadmap" — producing planning artifacts.
- "I have a vague idea and I don't know where to start" — discovery + decomposition.
- "How do I split this into modules / services / bounded contexts?" — DDD-style decomposition.
- "How do we test / verify / harden this?" — verification strategy across unit, property, integration, fuzz, contract, chaos.
- "Review my plan / architecture / design" — critique with the same principled lens used to make picks.

**Data-structure / algorithm choice (the original dsa-pro scope):**
- Storing, indexing, searching, sorting, traversing, batching, ranking, joining, deduplicating, or streaming data — and the answer is not a one-liner.
- Choosing between `Map` / `Set` / `PriorityQueue` and the right *shape* (insertion-order? sorted? concurrent? on-disk? approximate?) isn't obvious from the call site.
- Designing an index, cache, queue, batcher, scheduler, or hot data path.
- "Make it faster" lands without a named culprit and the suspect is a container or traversal pattern.
- The problem name maps to a known algorithm (shortest path, k-th element, top-K, substring search, range sum, count distinct, etc.).
- Reviewing code that holds a quadratic loop, naive scan, growing memory, or O(n) operation on a hot path.

If the task is `arr.push(x)` or `Object.keys(o)`, this skill is overkill.

## Core principles (the floor)

Read [`references/principles.md`](references/principles.md) for the full treatment. The shortlist Claude leans on by default:

1. **Specify before naming.** State operations + access pattern + N + constraints *before* naming a structure, a framework, or a methodology. Most wrong answers happen because the question was named wrong.
2. **Stdlib first, boring tech first.** Beating the stdlib, replacing the database, building a custom framework — these are projects, not functions. Reach for them only when you can *name* what the standard option lacks.
3. **Constants and the memory hierarchy dominate Big-O at the sizes you actually run.** L1 ≈ 4 cycles, DRAM ≈ 100–300, NVMe ≈ 10⁵, network ≈ 10⁶⁺. Optimize for the hottest layer the data lives in.
4. **Encode invariants as property tests, not comments.** Every non-trivial structure, module, or workflow has invariants. If you can't write the invariant down as code, you don't fully understand it yet. Property tests against a reference catch refactor regressions that example-based tests miss.
5. **Make invalid states unrepresentable.** Push correctness into types and parsers; avoid validation-as-comment. (See `principles.md` → Parse, don't validate.)
6. **Measure in the target environment.** Big-O sets the lower bound; constants determine whether you ship. Same structure can run 10× faster or slower with different key distributions, cardinalities, working-set sizes, allocators, or JITs.
7. **Match process to risk.** Regulated medical / finance ≠ early-stage MVP ≠ infra refactor. An honest SDLC pick saves more time than any framework choice. See [`references/sdlc.md`](references/sdlc.md).

## Workflow

There are two entry points. Most non-trivial requests start at **Planning** and pass through **Implementation** for each non-trivial module. Trivial drop-ins (one structure, one function, one bug fix) can start at Implementation directly.

### A. Planning workflow (use when the user is going from idea → buildable plan)

Produce artifacts via guided conversation — *not* a 20-page document up front. Ask only the questions whose answers actually change the plan; default to producing a v0 draft and iterating.

1. **Discovery.** What problem, for whom, with what success measure. Existing constraints (compliance, latency, team, language). Surface unknowns explicitly; flag risks. Use `references/templates.md` → *Discovery brief* as the prompt skeleton.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ananyakaushik48/dsa-pro](https://github.com/ananyakaushik48/dsa-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
