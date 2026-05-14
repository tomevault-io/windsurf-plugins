---
trigger: always_on
description: **Project Context**: This is a verification-focused coding project across Lean 4, Dafny, and Verus.
---

# VeriCoding Project Overview

**Project Context**: This is a verification-focused coding project across Lean 4, Dafny, and Verus.

Remember that you are run across multiple cycles, so focus on iterating and making incremental progress on a solution rather than getting desperate to one-shot it.

## MCP

- Lean MCP: `uvx lean-lsp-mcp`. You'll need its search capabilities since your training date cutoff is very out of date for Lean 4.

### Lean LSP-MCP Tool Usage

The lean-lsp-mcp tools provide real-time feedback on your Lean code:

#### 1. `lean_diagnostic_messages`
Get all diagnostic messages (errors, warnings, infos) for a Lean file.

```text
Example output:
l20c42-l20c46, severity: 1
simp made no progress

l21c11-l21c45, severity: 1
function expected at h_empty term has type T ∩ compl T = ∅
```

#### 2. `lean_goal`
Get the proof goals (proof state) at a specific location in a Lean file. Main tool to understand proof state evolution!

```text
Example output:
Before:
S : Type u_1
inst✝¹ : Fintype S
inst✝ : Nonempty S
P : Finset (Set S)
hPP : ∀ T ∈ P, ∀ U ∈ P, T ∩ U ≠ ∅
⊢ P.card = 2 ^ (Fintype.card S - 1)
After:
no goals
```

#### 3. `lean_term_goal`
Get the expected type (term goal) at a specific location.

- `lean_hover_info`: Retrieve hover information (documentation) for symbols, terms, and expressions in a Lean file (at a specific line & column).

Example output (hover info on a `sorry`):

```text
The `sorry` tactic is a temporary placeholder for an incomplete tactic proof,
closing the main goal using `exact sorry`.

This is intended for stubbing-out incomplete parts of a proof while still having a syntactically correct proof skeleton.
Lean will give a warning whenever a proof uses sorry, so you aren't likely to miss it,
but you can double check if a theorem depends on sorry by looking for sorryAx in the output
of the #print axioms my_thm command, the axiom used by the implementation of sorry.
```

`lean_completions`: Code auto-completion: Find available identifiers or import suggestions at a specific position (line & column) in a Lean file. Use this to fill in program fragments.

#### 6. `lean_multi_attempt`
Try multiple Lean code snippets at a line and return goal state and diagnostics for each.

- `lean_declaration_file`: Get the file contents where a symbol or term is declared. Use this to find the definition of a symbol.

#### 8. Search Tools:
- `lean_leansearch`: Natural language and Lean term search (limit: 3req/30s)
- `lean_loogle`: Search by constant, lemma name, type shape, or conclusion (limit: 3req/30s)
- `lean_state_search`: Search theorems based on proof state (limit: 3req/30s)
- `lean_hammer_premise`: Search premises using Lean Hammer (limit: 3req/30s)

## General Programming Philosophy

Programming is about onomastics (naming), composition (functoriality), and caching. Think conformally (at every scale and across scales).

Build a pit of success: internal systems that grow as a whole outwards, never allowing the fallible external world to leak in except at boundaries. Meet the external world at well-defined interfaces.

When solving problems, write tooling/linters/auto-fixers to widen the pit of success. Use rigid compiler error messages and linter warnings to guide future users (**including** AI) toward correct solutions.

Lean's mut notation is really useful, don't be afraid to use it.

## Project Structure

- `.mcp.json` - MCP server configuration for Lean development tools
- `CLAUDE.md` - This file, containing project instructions and Lean development guidelines
- Additional Lean files and experiments to be created as needed

## Development Commands

For Lean development, the key command is:
- `lake build` (use frequently for constant feedback). It can take filenames as arguments to build them separately.

The lean-lsp-mcp tools are already configured in `.mcp.json` and available through the MCP interface.

## Experiment Tracking

Uses Weights & Biases (wandb) for tracking verification experiments, failure analysis, and LLM usage metrics

## Lean 4 Development Guidelines


- Wrap reserved names in «guillemets» when needed
- Implement "notation typeclasses" like `GetElem`, `Add`, etc where appropriate.
- Practice "sorry-friendly programming": Instead of a comment you put down a spec, but it is only "proved" with `sorry`. If it should compile, use a named hole instead. This is strictly better than a comment, because the typechecker will use it for program generation.
- Decompose proofs until tools like `canonical`, `grind`, and `simp` dissolve the pieces. Use them to do the "how", the AI should do the "what".
- Don't use `i` and `j` as variable names when you could use `r`(ow) and `c`(olumn) instead. Ditto for `m` and `n` as matrix dimensions. Use `R` and `C`.
### Import and Module Structure

- Imports MUST come before any syntax elements, including module and doc comments
  - [ ] TODO set extensible error messages to suggest a fix for AI. Then remove this admonishment.
- Set `linter.missingDocs = true` and `relaxedAutoImplicit = false` in `lakefile.lean`.

### Common Errors and Solutions

- **"unexpected token 'namespace'"**: Module/doc comment placed incorrectly (should be after imports)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Beneficial-AI-Foundation/vericoding](https://github.com/Beneficial-AI-Foundation/vericoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
