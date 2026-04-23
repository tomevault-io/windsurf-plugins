---
trigger: always_on
description: Lean 4 (v4.28.0) + Mathlib formalization of log-ratio convergence for (u,v)-flower
---

# CLAUDE.md — fd-formalization

## Project

Lean 4 (v4.28.0) + Mathlib formalization of log-ratio convergence for (u,v)-flower
graphs. Headline theorem: `flowerDimension` in `FlowerDimension.lean`. Zero sorry,
zero custom axioms on main branch.

## Build & verify

```bash
lake build --wfail      # primary check — warnings are errors
lake lint                # Mathlib linter suite
lake build FdFormal.Verify  # axiom dashboard (#print axioms)
```

Pre-commit hooks enforce: trailing whitespace, EOF newline, merge conflicts,
copyright headers on all `.lean` files.

## Lean style (Mathlib conventions)

### Naming

- **Prop terms** (theorems): `snake_case` — `mul_comm`, `flowerVertCount_pos`
- **Types/Props/Sorts** (structures): `UpperCamelCase` — `FlowerVert`, `HasLogRatioDimension`
- **Other Type terms**: `lowerCamelCase` — `hub0`, `edgeSrc`, `instFintypeFlowerEdge`
- **UpperCamelCase inside snake_case**: becomes `lowerCamelCase` — `neZero_iff` not `NeZero_iff`
- **Conclusion-first**: `lt_of_le_of_ne` (conclusion `lt`, hypotheses `le` and `ne`)
- **`_of_` pattern**: hypotheses joined by `_of_` in order: `C_of_A_of_B` for `A → B → C`
- **American English**: `factorization` not `factorisation`

### Formatting

- **100-char line limit** (linter-enforced)
- **`by` at end of preceding line**, never on its own line
- **2-space indent** for proof bodies; **4-space** for multi-line statements
- **No empty lines** inside declarations (linter-enforced)
- **Focusing dots** `·` flush with current indent, tactics indented beneath
- **`:`, `:=`, infix ops** at end of line, not start of next
- **`fun x ↦`** not `λ x ↦`; **no `$`** (use `<|` if needed)

### Tactics

| Goal type | Preferred tactic |
|-----------|-----------------|
| Linear ℕ/ℤ arithmetic | `omega` |
| Numerical evaluation | `norm_num` |
| Decidable props | `decide` |
| Positivity (0 ≤ x, 0 < x) | `positivity` |
| Monotonicity/congruence | `gcongr` |
| General simplification | `simp` (last resort) |
| Nonlinear arithmetic | `nlinarith [hint]` |
| ℕ subtraction → ℤ | `zify [h1, h2]` |

- **Terminal `simp`**: do NOT squeeze (maintenance burden from lemma renames)
- **Non-terminal `simp`**: MUST be `simp only [...]`
- **One tactic per line** (semicolons only for short single-idea sequences)
- **Set equality**: `ext v; simp [...]` is canonical — don't build manual
  `constructor`/`rcases`/`absurd` chains when `simp` with the right lemma set closes it
- **Skip `ext`** when `simp` alone closes a set equality (e.g., `ball_zero`)
- **`simp` before `tauto`**: `simp` + commutativity lemmas (`adj_comm`, `or_comm`)
  often closes goals that look like they need `tauto`
- **`.rfl`** not `Iff.rfl` — dot notation for constructor terms
- **Named lemmas over `show ... from rfl`**: e.g., `one_add_one_eq_two.symm`
  not `show (2 : ℕ∞) = 1 + 1 from rfl`

### Attributes

- `@[simp]`: equations/iff where LHS is more complex than RHS; must not loop
- `@[ext]`: extensionality lemmas
- `@[simps]`: auto-generate projection simp lemmas for structures
- `@[gcongr]`: congruence lemmas of form `f x₁ ∼ f x₂` given `x₁ ∼ x₂`

### Types and definitions

- **`Type*`** not `Type _` (performance requirement)
- **`where` syntax** for instances, not braces
- **Named instances**: `instance instFintypeFlowerEdge : Fintype (FlowerEdge u v g)`
- **`variable` blocks** for shared parameters — don't repeat `(u v g : ℕ) (hu : 1 < u)`
- **Hypotheses left of colon** — `(h : 1 < n) : 0 < n` not `: 1 < n → 0 < n`
- **`abbrev`** (reducible) requires justification; `@[irreducible]` requires justification
- **Classical by default** — don't thread `Decidable` instances unless the type requires them
- **Definition argument order**: choose to minimize downstream `symm`/`comm` calls —
  e.g., `{v | G.edist v c < r}` not `{v | G.edist c v < r}` when most API lemmas
  state the varying argument first

### Documentation

- **Module docstring** (`/-! ... -/`) required after imports: title, summary,
  Main definitions, Main statements, Implementation notes, References, Tags
- **Definition docstrings** (`/-- ... -/`) required on every `def` (linter: `docBlame`)
- **References**: cite as `[AuthorYear]`, anchor in `docs/references.bib`

### Imports

- **Granular imports only** — never `import Mathlib`
- Import hierarchy: Algebra → Order → Topology → Analysis (no cross-category violations)
- Files under ~1000 lines; split along natural boundaries

## Aristotle prover

**Role: leaf-lemma grinder and dependency detector, not theorem architect.**

### When to use

- Cast-control lemmas (ℕ → ℝ), positivity/nonzeroness, algebraic reshaping
- Squeeze bounds, pow/log simplification, recurrence-to-closed-form algebra
- High success on algebraic/order-theoretic leaves

### When NOT to use

- Headline theorems, design decisions, anything where definitions are still moving
- If you can't explain in one sentence why the lemma is true, don't submit it

### Submission protocol

1. **Freeze the statement** — hand-design def + statement, compile to sorry, then submit
2. **Each sorry = one leaf** — one concept, one obvious target, short dependency cone
3. **Proof-shaped files** — short helpers first, named intermediates, minimal imports
4. **Batch by type**: positivity → algebra → squeeze → limits → cleanup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Project-Navi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
