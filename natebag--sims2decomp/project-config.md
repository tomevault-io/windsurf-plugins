---
trigger: always_on
description: Project-specific rules for the Sims 2 GC matching decomp. Universal rules are in `~/.claude/CLAUDE.md`.
---

# Sims 2 GameCube Decompilation — Project Rules

Project-specific rules for the Sims 2 GC matching decomp. Universal rules are in `~/.claude/CLAUDE.md`.

## Decomp Honesty Rules (HIGHEST PRIORITY — supersedes anything below)

These rules exist because earlier metrics rewarded spoofing — "matched_code_percent
must never drop below 100%" combined with permissive asm-processor routing produced
agent behavior that wrapped raw bytes and called it decomp. That stops here.

### What "matched" means

A function counts as **matched** only if ALL of the following are true:

1. The source in `src/matched/...` is **hand-written C++** describing what the function
   actually does — not a wrapper around the original bytes.
2. Compiling that C++ with SN ProDG (or the documented fallback) produces bytes that
   are byte-identical to the DOL at that address, **with relocations masked and local
   branches resolved, but with NO post-compile mutation**.
3. The source contains **no** ASMPROC directives, NON_MATCHING markers, `__asm__`,
   `.byte`/`.long` injection, naked attributes, register-pin asm, `__builtin_unreachable`,
   or `noreturn`-to-suppress-body tricks.

Anything else is NOT matched. It may be useful working state (`forced`, `non_matching`,
`pending`, `wip`), but it does not get to claim the word "matched."

### Banned for new work

Do not produce, expand, or "fix" matches using any of these:

- `// ASMPROC_*` directives (`inject_before`, `replace_insn`, `swap_adj`,
  `gpr_relabel`, `force_reg_at_pos`, and any future variants).
  These are post-compile asm surgery. They make the binary match because tooling
  rewrites the compiler output, not because the C++ describes the function.
- `__attribute__((naked))`, `__attribute__((noreturn))` paired with `__builtin_unreachable`,
  `__builtin_unreachable` at function scope.
- `__asm__` / inline assembly, `.byte`, `.long`, raw-byte arrays standing in for code.
- `register T name asm("rN")` register-pin cheats.
- Adding `// NON_MATCHING` to a near-match to make CI green. NON_MATCHING is for
  *documenting* a function you understand but can't yet land — not a release valve.

A function you cannot match cleanly is **left for later**, not forced. Log it as a
wall in `docs/tracking/walls.md` with what you tried, and move on to the next one.

### The number goes UP from zero

The headline progress number is **clean bytes / game .text bytes**, produced by
`python tools/audit_clean_matches.py`. It starts low. It rises slowly. It is
allowed to *fall* if previously-forced files are reclassified or removed.

- There is **no floor** on the clean percentage. Do not try to "protect" or "defend" it.
- There is **no quota** — not per session, not per day, not ever. Honest matches
  beat fast ones.
- "matched_code_percent must never drop below X" — **deleted**. Any rule, prompt,
  scoreboard, or agent instruction that pins the percentage is a spoofing incentive
  and should be removed when found.
- A correct day might land zero new matches and just document one wall. That is
  progress.

### Steady-state loop

The only sanctioned workflow is:

1. Pick a function from `build/audit/forced.txt`, `non_matching.txt`, or the
   unmatched backlog.
2. Read the DOL disassembly. Understand what the function does.
3. Write real C++ that expresses that behavior.
4. Run `tools/verify_match.sh` (or the `--strict` future-mode equivalent).
   - Clean MATCH → commit.
   - MISMATCH and you can fix it with real C++ changes → iterate.
   - MISMATCH and you cannot crack it without forcing → log the wall, move on.
5. Re-run `python tools/audit_clean_matches.py` periodically to see the honest
   number trend. Trend > headline.

### Three numbers, not one

Public/internal reporting must always distinguish:

- **Linked / built %** — bytes covered by the build (incl. byte-injection stubs).
  Currently ~100%. This proves the scaffold is correct. It is **not** decomp progress.
- **Clean matched %** — the audit script's clean bucket. The honest decomp metric.
- **Forced %** — ASMPROC + NON_MATCHING bytes. Backlog to redo, not credit.

If a single number is reported anywhere, it is the clean one, and it is labeled
"clean decomp."

### Reviewer-facing wording

When the agent reports progress in a commit message, PR description, dashboard, or
chat reply: do not say "100% matched", "X% complete", or otherwise round the linked
number up into the matched number. Report the clean percentage explicitly. Reporting
the linked figure as if it were the matched figure is the exact spoof these rules
exist to prevent.

## Agent Delegation

Use these agents proactively — don't wait to be asked:

| Trigger | Agent | Why |
|---------|-------|-----|
| New function to decompile | `decomp-planner` | Analyzes symbol in map file, checks Ghidra disassembly, identifies dependencies, produces decomp plan |
| PPC compilation errors | `cpp-build-resolver` | Fixes SN Systems / devkitPPC compiler issues, linker errors, missing symbols |
| After writing C++ code | `cpp-reviewer` | Checks matching conventions, naming from map file, verifies no UB |
| Exploring unknown code in Ghidra | `ghidra-analyst` | Deep-dives disassembly, identifies patterns, documents structs |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [natebag/Sims2DECOMP](https://github.com/natebag/Sims2DECOMP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
