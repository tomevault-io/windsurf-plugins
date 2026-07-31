---
trigger: always_on
description: `comptime` is Neotype’s compile‑time evaluator — a very limited analog to Zig’s `comptime`. It takes a Scala expression (via macros), lowers
---

# Comptime Agent Notes

## What this module is
`comptime` is Neotype’s compile‑time evaluator — a very limited analog to Zig’s `comptime`. It takes a Scala expression (via macros), lowers
it to a small IR, and evaluates it with a rule engine so that validation and construction can be done at compile time. The intent is to
pre‑optimize certain expressions beyond what `inline` alone can do, while keeping the surface area deliberately small and well‑tested.
The behavior is driven by a whitelist of supported language constructs and stdlib rules (see `modules/comptime/SUPPORTED.md` and the tests
under `modules/comptime/shared/src/test/scala/comptime`).

## Principles
- Prefer concise, table‑driven rules (typed pairs) and `RulesFor` helpers.
- Avoid code generation unless it clearly pays off; keep rules as plain Scala.
- Preserve rule ordering semantics: the first matching rule wins.
- Performance changes should be measured (see `modules/comptime/PERF_PLAN.md`).

## Rule authoring
- Prefer `RulesFor` helpers (`ops*`, `byName_*`, `rules`) at call sites.
- When using typed rule tables, use `opsList` / `ops1List` / `ops2List` / `ops3List` and `byName_*List` helpers to avoid `: _*`.
- Introduce a helper before repeating `compileRecv*` or manual `Eval.Apply*` blocks.
- Keep shared tables local unless reused across multiple rule groups.

## Naming conventions

### S/L Pattern for lazy (by-name) arguments
Methods that handle by-name arguments use an S/L suffix where:
- `S` = Strict (evaluated immediately)
- `L` = Lazy (by-name, evaluated on demand)

The pattern describes argument positions left-to-right:
```
byName_L      # 1 arg, lazy
byName_SL     # 2 args: strict, then lazy (e.g., getOrElse)
byName_LS     # 2 args: lazy, then strict (e.g., fold)
byName_SLL    # 3 args: strict, then 2 lazy
```

For curried arities, prefix with the arity pattern:
```
byName1_1_LS  # curried (1)(1), lazy first, strict second
byName1_1_SL  # curried (1)(1), strict first, lazy second
```

### Arity constants
Use underscores to separate curried arg counts:
```
A0, A1, A2, A3     # flat arities
A1_1, A1_2, A2_1   # curried: A1_1 = (1 arg)(1 arg)
```

### Internal helpers
- `compileByName_SL`, `compileByName_LS`, `compileByName_SLL` - compile helpers
- `ruleRecvByName_SL`, `ruleRecvByName_LS` - rule builders

## Performance
- Rule lookup is indexed by name in `CallRuleEngine` (via `RuleEngine`); keep this invariant when editing.
- Only add recv/arity indexing if measurements show a hotspot.

## Debugging
- Use `ComptimeDebug` for rule dispatch logs when diagnosing failures.
- Add minimal, targeted tests rather than broad rewrites.

## Tests
- Main comptime tests live in `modules/comptime/shared/src/test/scala/comptime`.
- Update `modules/comptime/SUPPORTED.md` when feature coverage changes.

## Adding new stdlib rules

### Quick reference for common patterns

| Pattern | Table type | Helper | Use case |
|---------|-----------|--------|----------|
| `recv.op()` | `A => R` | `opsList` | Basic accessors (head, tail, reverse) |
| `recv.op(arg)` | `(A, B) => R` | `ops1List[B]` | Single-arg ops (take, drop) |
| `recv.op(a, b)` | `(A, B, C) => R` | `ops2IntIntList` | Two-arg ops (slice) |
| `recv.op(f)` with `f: Any => Boolean` | `(A, F) => R` | `ops1PredList` | Predicate ops (exists, filter) |
| `recv.op(f)` with `f: Any => Any` | `(A, F) => R` | `ops1FnList` | Transform ops (map, groupBy) |
| `recv.op(f)` with `f: (Any,Any) => Boolean` | custom | `sortWithAny` | Binary predicates (sortWith) |
| `recv.op(f)` with `f: (Any,Any) => Any` | custom | `reduceAny` | Binary ops (reduce, foldLeft) |
| `recv.op()` with implicit | `A => R` | `anyArityOpsList` | Ops with implicits (sum, flatten) |

### Receiver matching

When the call owner differs from expected (e.g., `SeqOps` vs `Seq`), add it to `seqRecv`:
```scala
private val seqRecv = Recv.union(
  "scala.collection.Seq",
  "scala.collection.SeqOps",  // sortWith, sortBy
  "scala.collection.StrictOptimizedIterableOps",  // flatten
  // ...
)
```

### Arity matching for implicits

Methods with implicits (like `sum`, `sorted`, `flatten`) use `anyArityOpsList` because:
- Scala resolves implicits at compile time → arity appears as A0
- But sometimes they show up as A1 or A1_1
- `ASet(Set.empty)` matches any arity

For curried methods with implicits (like `sortBy(f)(implicit ord)`):
- Use `ruleRecv1AnyArity` which tries A1 first, then A1_1
- Or skip and document as unsupported (complex cases)

### Table vs inline rules

Prefer tables for:
- Groups of similar methods (all predicates, all transforms)
- Simple receiver→result patterns

Use inline rules for:
- Methods needing special logic (type dispatch like sum/max)
- By-name arguments (use byName_ helpers)
- Curried methods (use ruleRecv11Or2)
- **Methods with overloads that differ by return type** — need access to `call.targs` (see "Using AST type information" section)

### Runtime typeclass dispatch (TypeClassMaps)

Operations requiring `Ordering` or `Numeric` (like `sum`, `max`, `min`, `minBy`, `maxBy`) use runtime type dispatch via `TypeClassMaps` in `StdlibCollectionHelpers.scala`:

```scala
// Look up Ordering/Numeric based on element's runtime class

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kitlangton/neotype](https://github.com/kitlangton/neotype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
