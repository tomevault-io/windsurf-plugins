---
trigger: always_on
description: - Use existing .venv python if available, otherwise create a venv and install using `uv pip install -e[solvers,rust,pypcode,dev]`
---

# Knuckledragger Development Guide

- Use existing .venv python if available, otherwise create a venv and install using `uv pip install -e[solvers,rust,pypcode,dev]`
- Try to follow the style that already exists in the repo
- Simple and non verbose is much preferred
- Solver instability and solve time are ALWAYS a concern. Track them.
- Recommend changes to improve clarity of error messages
- Recommend changes to make more consistent with Lean/Rocq/Isabelle/ACL2 conventions
- Recommend new features that this system is missing that Lean/Rocq/Isabelle/ACL2/Mizar might have
- Record new learnings in this file to avoid repeating failures
- Actually run the file using `time python` to make sure proofs go through. A file should not take longer than 0.5s
- Do not just fill up on junk theorems with little content. Make theorems if useful, but sheer bulk of content is bad since I need to manually review it.
- Some problems are things that should be improved. Do not always just silently find a workaround. Parser errors in valid lean syntax should be improved.
- You may sometimes need new axioms (rarely), but ask me before you add them.
- Add comments into proofs to explain the rough lines of what you are doing
- Try to simplify proofs by factoring any common moves like `unfold` or `rw` above the branches of cases or splits.
- Big jumps for the solver are slow, but so is too many little steps. There is a balance.
- DO NOT try to be sneaky to get a proof though (mutating stuff, using python craziness). Knuckledragger relies on proper usage to remain sound.

## Library Usage

Knuckledragger is a python library for interactive theorem proving backed by Z3.

### Core Types

- `kd.Proof` - A proven theorem (frozen dataclass with `thm: smt.BoolRef`)
- `smt.BoolRef` - Z3 boolean formula (theorems/propositions)
- `smt.ExprRef` - Z3 expression (terms)
- `smt.SortRef` - Z3 sorts/types (e.g., `smt.IntSort()`, `smt.RealSort()`)

Many proof and tactic functions can also take a string in simple Lean-like syntax. This can improve readability and reduce verbosity

### Z3 Basics (from `kdrag.smt`)

Common functions:

- `smt.Const(name, sort)` or `smt.Ints("x y z")`, `smt.Reals("x y z")`
- `smt.ForAll(vs, body)`, `smt.Exists(vs, body)`
- `smt.And(...)`, `smt.Or(...)`, `smt.Implies(p, q)`
- `smt.Function(name, *arg_sorts, ret_sort)` - declare uninterpreted function

Operators naturally overload: `+`, `-`, `*`, `/`, `==`, `!=`, `<`, `>`, `<=`, `>=`

### Simple Proofs

`kd.prove(thm, by=[lemma1, lemma2], timeout=1000)` - Prove a theorem in one Z3 call

- Returns a `kd.Proof` object
- Use `by=[...]` to provide lemmas as assumptions
- Throws `kd.kernel.LemmaError` if unprovable

### Proof State Tactics (Preferred: `@Theorem` decorator)

**Modern style** (preferred):

```python
@kd.Theorem(smt.ForAll([x], x + 0 == x))
def add_zero(l):
    """x + 0 == x"""
    x = l.fix()
    l.auto()
```

**Old style** (less preferred):

```python
l = kd.Lemma(smt.ForAll([x], x + 0 == x))
x = l.fix()
l.auto()
add_zero = l.qed()
```

For development, use `@kd.PTheorem` which prints next goal instead of erroring.

### Tactics Reference

Tactics are methods on `ProofState` (returned by `kd.Lemma` or passed to `@Theorem`):

**Opening quantifiers/assumptions:**

- `l.fix()` - open one ∀, returns the fresh variable
- `l.fixes()` - open multiple ∀s, returns list
- `l.intros()` - move implication hypothesis into context. Returns that new context formula
- `l.obtain(n)` - open ∃ in hypothesis `n`, returns witness constants

**Proving subgoals:**

- `l.auto(**kwargs)` - call Z3 on current goal with context
- `l.show(thm, by=[...])` - prove goal equals `thm` using lemmas
- `l.exact(pf)` - close goal with proof `pf`

**Manipulating context/goal:**

- `l.have(thm, by=[...])` - add lemma to context (must be implied by context)
- `l.rw(rule, at=None, rev=False)` - rewrite using equality `rule` (at goal or hyp index)
- `l.unfold(*decls, at=None)` - unfold definitions
- `l.split()` - split `∧` in goal or `∨` in hypotheses
- `l.apply(n)` - apply implication from hypothesis `n`
- `l.exists(*ts)` - provide witnesses for ∃ goal

**Induction:**

- `l.induct(x, using=None)` - induct on variable `x` (auto-detects or use custom principle)
  - **Important**: After `l.induct(x)` on an inductive datatype, the constructor case generates `ForAll` for ALL constructor fields
  - For `Cons(head, tail)` you must use `head, tail = l.fixes()` not `l.fix()`
  - Example: After inducting on a list, the `Cons` case needs `fixes()` to open both head and tail

**Advanced:**

- `l.specialize(n, *ts)` - instantiate universal in hypothesis `n` with terms
- `with l.sub() as l1: ...` - create subgoal

**Finish:**

- `l.qed()` - complete proof and return `kd.Proof`

### Definitions and Axioms

- `kd.define(name, vars, body)` - Define a function. Returns function with `.defn` attribute
- `kd.axiom(thm)` - Assert axiom (use sparingly!)
- `kd.FreshVar(name, sort)` or `kd.FreshVars("x y z", sort)` - Create schema variables

### Algebraic Datatypes

```python
List = kd.Inductive("List")
List.declare("Nil")
List.declare("Cons", ("head", smt.IntSort()), ("tail", List))
List = List.create()
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [philzook58/knuckledragger](https://github.com/philzook58/knuckledragger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
