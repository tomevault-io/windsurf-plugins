---
trigger: always_on
description: This file is an AI-consumable package guide for agents that need to use GraphK without re-reading the full codebase.
---

# GraphK Agent Guide

This file is an AI-consumable package guide for agents that need to use GraphK without re-reading the full codebase.

Scope:

- explain what GraphK is
- explain the runtime model
- explain the public package surface exported by `graphk`
- explain how to compose and execute pipelines correctly

## Purpose

GraphK is a graph-execution core for Python. It provides a small set of primitives for building executable graphs composed of nodes, pipelines, branch selectors, execution policies, and runtime sessions.

GraphK is not a full application framework. It is the execution kernel underneath graph-based workflows.

The public package is intended to be imported as:

```python
import graphk
```

## Mental Model

The core execution model is:

- `Store` is hierarchical scoped state
- `Matcher` evaluates rules against a store
- `Session` carries runtime data across execution
- `Belief` carries scoped execution state for nodes and pipelines
- `Policy` carries rule sets that validate entry and exit conditions
- `Node` is the smallest executable unit
- `Pipeline` composes nodes and nested pipelines
- `SequencePipe` executes in order
- `BranchPipe` selects one matching branch
- `MultiPipe` executes multiple matching branches
- `Runner` is the preferred end-to-end execution engine
- `Emitter` wraps a runner into a request/response API

Preferred public execution entry points:

1. `Runner` for pipeline execution
2. `Emitter` for request/response style usage

## Package Map

The package currently exports:

- `Store`
- `Matcher`
- `Logger`
- `Persistence`
- `Session`
- `Belief`
- `Policy`
- `Node`
- `Pipeline`
- `Runner`
- `Emitter`
- `BranchPipe`
- `MultiPipe`
- `SequencePipe`
- `demo`
  Demo node namespace:
  - `demo.ApplyNode`
  - `demo.IncrementNode`
  - `demo.AccumulateNode`
  - `demo.ConcatNode`
  - `demo.TextConcatNode`
  - `demo.EchoNode`
  - `demo.RouteNode`

## Scoped Key Syntax

GraphK state access is built on scoped keys. This is essential for correct use of `Store`, `Session`, and belief/policy resolution.

Supported key forms:

- `value`
  Meaning: resolve through the current store and then upward through the parent chain.

- `./value`
  Meaning: resolve only in the current store.

- `../value`
  Meaning: resolve only in the direct parent store.

- `@Scope/value`
  Meaning: resolve in the nearest store whose scope name matches `Scope`.

- `@Scope`
  Meaning: open-scope lookup used by operations like `contains()` when checking whether that scope exists in the chain.

Use `Store.get()` and `Store.set()` with these forms rather than manually walking parent links.

## Core Types

### `Store`

Purpose:

- base hierarchical key/value state container
- parent-linked
- scope-aware

Important behavior:

- inherits from `dict`
- supports scoped reads and writes
- supports flattening, saving, restoring, and cloning

Important methods:

- `rescope(scope)`
  Change the store scope name.

- `attach(parent)`
  Attach this store to a parent store.

- `detach()`
  Remove the parent link.

- `contains(key)`
  Check whether a scoped key or scope exists.

- `get(key, default=None)`
  Resolve a scoped key.

- `set(key, value)`
  Assign a value through scoped resolution rules.

- `scopes()`
  Return scope names from current store to root.

- `levels()`
  Return the number of scope levels.

- `root()`
  Return the root store.

- `flatten(top_down=False)`
  Return a resolved merged view of the store chain.

- `save()`
  Return a snapshot of local state.

- `restore(saved)`
  Restore local state from a snapshot.

- `clone(all=False)`
  Clone the store.

- `to_dict()`
  Export a resolved dictionary view.

Usage guidance:

- use `Store` as the base abstraction for hierarchical state
- use `Session` and `Belief` instead of raw `Store` when runtime semantics matter
- prefer `get()` and `set()` instead of direct dictionary access when hierarchy matters

### `Matcher`

Purpose:

- evaluate rules against a `Store`
- supports exact match, numeric comparison, wildcard, regex, callable predicates, and global predicates

Important behavior:

- inherits from `Store`
- stores rules in the same hierarchical model as other state containers

Important methods:

- `match(state)`
  Evaluate the matcher against a target `Store`.

- `to_dict()`
  Return the effective resolved rule set.

Important constants:

- matching strategy:
  - `MATCH_ALL`
  - `MATCH_ANY`

- resolver strategy:
  - `RESOLVER_BOTTOMUP`
  - `RESOLVER_TOPDOWN`
  - `RESOLVER_ROOT`
  - `RESOLVER_BOTTOM`
  - `RESOLVER_FLAT_BOTTOMUP`
  - `RESOLVER_FLAT_TOPDOWN`

Usage guidance:

- use `Matcher` directly when you need rule-based state matching
- use `Policy` when the matcher is intended to govern execution entry or exit
- `_` may be used as a global predicate key whose value is one callable or a list of callables

### `Session`

Purpose:

- runtime state flowing across nodes during execution

Important behavior:

- inherits from `Store`
- has execution status fields:
  - running
  - completed
  - cut
  - error

Important properties:

- `is_running`
- `is_completed`
- `is_cut`
- `is_error`
- `code`
- `message`

Important methods:

- `start()`
  Mark the session as running.

- `completed()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kochf1/graphk](https://github.com/kochf1/graphk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
