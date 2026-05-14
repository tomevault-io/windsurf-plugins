---
trigger: always_on
description: Provides  []query.Symbol    // Symbols this phase produces
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Test Commands

```bash
go test -count=1 ./...          # Full suite
go test -v ./package -run Test  # Single test
```

**Do NOT add `-timeout` to `go test` commands, or use `-timeout 0`.** Use the default timeout. No exceptions.

---

## Architectural Authority

**The user owns all architectural decisions. Claude implements them.**

Before making ANY of these decisions, ASK:
- Introducing new patterns (globals, managers, abstractions)
- Changing existing patterns (options → globals, Relations → Bindings)
- Adding new cross-cutting concerns (configuration, logging, caching)
- Deviating from established conventions for any reason

**If you're unsure whether something is an "architectural decision":**
- Would it affect multiple files/packages?
- Would it change how components interact?
- Would it require other code to change to accommodate it?
- Are you thinking "I'll ask forgiveness later"?

**Then ASK first.**

**Red flags that indicate you're overstepping:**
- "This is just temporary/experimental"
- "I'll refactor this later"
- "It's faster to do it this way"
- "It's simpler/easier this way" (when deviating from a plan or established pattern)
- Making a choice between multiple valid approaches without consulting

**Bugs do not authorize design changes:**
- Discovering a bug does not authorize you to change the agreed design. Report it and ask.
- If something we agreed on doesn't work, STOP and ask. Do not substitute alternatives.
- If you're about to do something different from what was discussed/agreed, ASK FIRST. No exceptions.

**The user's job**: Set direction, make architectural choices, review designs
**Your job**: Implement, follow patterns, propose options (not make choices)

## When Tests Fail

**Failing tests are information, not obstacles.**

When tests fail after you make a change, the correct response is:
1. Understand WHY the test is failing
2. Report the failure to the user with context
3. Ask how they want to proceed

**NEVER change architecture or add code just to make tests pass.** If a test fails, it's telling you something important about the change - maybe:
- The change has unintended consequences
- The approach needs to be different
- The test expectations need updating
- The feature isn't ready

All of these are decisions for the user, not you.

**Wrong**: "The cache tests are failing, so I'll add a new cache type to make them pass."
**Right**: "The cache tests are failing because ClauseBasedPlanner doesn't integrate with the cache the same way. How do you want to handle this?"

## When Asked to Revert

**Revert IMMEDIATELY. Do not defer.**

When the user says "revert", do it as your very next action. Do not:
- Explain why you made the change
- Read files to "understand context"
- Plan what you'll do after
- Make any other changes first

**Why this matters:** When you defer reverting and wander off making more changes, the original correct state gets buried in context. If context compaction happens, the original code may be lost from the summary entirely. Future Claude inherits a mess with no way to recover.

- **Immediate revert** = recoverable state preserved
- **Deferred revert** = potentially permanent damage

## Project Overview

This repository contains a Datomic-style Datalog engine implementation in Go, inspired by memories of previous single-node and distributed implementations.

The Go implementation takes a pragmatic middle ground: production-ready with features like aggregations, annotations, time functions, and persistent storage.

## Architecture Summary

For a complete architecture overview, see [ARCHITECTURE.md](ARCHITECTURE.md).

The Datalog engine consists of these core components:
1. **EDN Parser**: Parses Clojure-style EDN syntax for queries
2. **Query Parser**: Transforms EDN into internal query representation
3. **Query Engine**: Executes queries using relational algebra operations
4. **EAVT Storage**: Entity-Attribute-Value-Transaction storage with multiple indices
5. **Type System**: Direct Go types without complex wrappers

## Key Architectural Insights

### Relation-Based Query Execution
The engine uses a **greedy join ordering approach** with several important safeguards:
- Progressive joining: Relations are joined as they become available
- Early termination: Stops immediately on empty results
- Disjoint detection: Catches queries that would create Cartesian products
- Streaming iterators: Avoids materializing large intermediate results

This is **standard database query optimization** (similar to Selinger's algorithm from 1979), but without cost-based optimization or statistics. It's not novel, but it's correctly implemented and crucial for preventing OOM failures on complex queries.

**What makes this codebase production-ready** is the combination of multiple techniques working together:
- **Phase-based planning**: Groups patterns intelligently to avoid bad join orders
- **Early predicate filtering**: Applies filters as soon as required symbols are available
- **Streaming architecture**: Iterator-based processing throughout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wbrown/janus-datalog](https://github.com/wbrown/janus-datalog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
