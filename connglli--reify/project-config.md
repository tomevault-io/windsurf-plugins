---
trigger: always_on
description: + Knowledge Background: Optimizing Compilers, Program Generation, SMT theories
---

# Reify: Agent Guideline

+ Knowledge Background: Optimizing Compilers, Program Generation, SMT theories
+ Implementation Language: C++ 20 (source: ./src)
+ Scripting Language: Python 3.12 (source: ./scripts, virtualenv: ./venv)

## Project Overview

This project implements a random program generator called Reify.

Unlike generators such as Csmith and YARPGen that generate programs based on syntactic rules only, Reify generate programs centering on semantic rules. It distinguishes two distinct semantics:
(1) compile-time semantics (what a program can do), represented by the control flow graph (CFG), and
(2) runtime semantics (what a program actually does), represented by execution paths (EP) within the CFG.

For a CFG $g$ and an EP $\pi$ on it, Reify constructs a program $P$ guaranteed to be well-behaved with respect to a specific input $i$ and output $o$. This means that $P$'s CFG is $g$ and when executing with $i$, $P$ deterministically follows the $\pi$ to produce the expected output $o$.

Reify separates the generation process into two stages:
1. Leaf Function Generation: Generate a leaf function and its input/output satisfying a CFG and an EP on it.
2. Whole Program Generation: Compose leaf functions into a whole program via peepwhole rewrite.

Reify implements these with help of a symbolic intermediate language called SymIR. SymIR incorporates symbols to represent unknown values. Reify leverage symbolic execution to reason about program behaviors with symbols. Solving the collected constraints via SMT solvers to find concrete values for symbols to satisfy desired behaviors.

Technical details of the two stages and the IR are presented in [./DOCS.md](./DOCS.md).

## Design Principles

The SMT solver takes most of Reify's running time. So every design choice should first ask: **does this make the solver's job easier?**

But Reify is a random program generator. If we only do what makes the solver happy, we generate boring programs that don't stress compilers. So the real rule is:

> **Spend the solver's effort on the shape of the program (CFG and path), not on the math inside each statement.** Make each statement cheap to solve. Get interesting programs by combining cheap statements along complex paths, not by writing one fancy statement.

### SymIR: pick operations the solver can handle fast

- **Keep expressions in a form the solver finds cheap.** Linear shapes (sums of small terms) solve fastest. Don't push toward richer expression forms just to look more interesting — richness should come from longer paths and bigger CFGs.
- **Ration non-linearity between program variables.** The existing `coef * var` term form (where `coef` is itself a symbol the solver picks) already puts a product of two unknowns into every query — that's the baseline, not a violation. The rule is about *adding more*: operations like `var * var`, `var / var`, `var % var`, and symbolic shifts should be used sparingly when introduced. Prefer one symbolic side and a small constant on the other. Don't put them in every expression.
- **Keep symbolic shift amounts in a small set of values.** The bigger the set, the more the solver has to try.
- **Keep symbolic indexing as simple as possible.** Prefer concrete indices, or pin a symbolic index to a single value. Avoid general array-theory reasoning unless we really need it.
- **Don't widen bit-widths without a reason.** A wider type makes every operation cost more inside the solver. Use the smallest width that still triggers the bug class we care about.
- **Every partial op needs a guard.** Operations like division, remainder, and shift have inputs that cause UB. They must ship with constraints that rule those inputs out.
- **A new op has to pay rent.** Before adding one, ask: can we already get the same bugs by combining existing ops along a more interesting CFG or path? If yes, do not add the op.

### Symbolic Execution: keep constraints simple

Some constraint shapes make the solver work much harder. Use them as little as possible:

- **ITE (if-then-else):** each ITE forces the solver to try both arms.
- **Large disjunctions / OR chains:** each branch is another case to try.
- **Quantifiers** (rules of the form "for all X" or "there exists X"): often push the query into undecidable territory.
- **Cardinality constraints** (e.g., "at most K of these are zero"): expensive even when K is small.
- **Fresh symbols:** every new symbol adds a dimension to the search space.
- **Mixing theories** (bit-vectors + arithmetic + arrays in one query): the solver pays for theory combination.

Concrete reminders:

- **Gate values with the path condition, not by merging branches.** Single-path symbolic execution naturally avoids most ITE. Don't drift toward branch merging "for elegance".
- **Simplify ITE while building it, not later.** Patterns like `ite(true, a, b) → a`, `ite(c, true, b) → c or b`, `ite(c, a, false) → c and a` should be done by helpers in the encoder, so the solver never sees them.
- **Pull shared parts out of ITE arms.** `ite(c, f+a, f+b)` should be written as `f + ite(c, a, b)`. Smaller arms, more sharing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [connglli/Reify](https://github.com/connglli/Reify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
