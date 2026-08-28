---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## What this is

A reimplementation of Diablo II 1.14d's item description engine, reconstructed from the
disassembly. A C++ producer captures a unit's stat lists as JSON; a C# consumer renders the exact
tooltip the game would draw. See `README.md` for the split and `docs/writers.md` for the spec.

Correctness here means **byte-identical to the original**, including its oddities. This is not a
reimplementation "in the spirit of" the game.

## The rules that matter

### The disassembly is the only truth

Not IDA's names, not its function bounds, not the C# code, not comments, not D2MOO, not this file.
Verify against the binary before changing behaviour.

**Never fix a bug by pattern-matching the symptom.** A parity failure tells you *that* something is
wrong, never *what*. Read the function in IDA, work out what it actually does, and implement that —
then cite the address. A change that makes the output match without a traced explanation is a guess
that happens to fit today's sample, and it will be wrong on the next one. This applies to captured
game output too: real data proves a divergence exists, it does not tell you which side is right.

If the disassembly is unavailable, say so and stop, rather than reasoning from plausibility.

Concretely, in this binary:

- **IDA function names are frequently lies.** `ITEMS_nullsub` is not a nullsub — it is the
  level-requirement fold. `szPrintfBuffer` is the weapon-damage writer. `ModifyPotionValueByDifficulty`
  never reads difficulty. `ITEMMODS_CalculateRepairCost` is a by-time interpolator.
- **IDA mis-reports `start_ea`.** It has been wrong at least four times here — verify the prologue
  (`55 8b ec`, usually after `0xCC` padding) rather than trusting the reported entry.
- **D2MOO can merge two distinct functions into one.** It decompiles `STATLIST_CalcCombinedStatValue`
  cases 1 and 13 through a single helper whose 1.14d counterparts are different functions reading
  different arrays. Following it there makes a correct fix look impossible.
- **The decompiler loses register arguments** on `__usercall`/`__fastcall` and drops varargs. Use
  `disasm` and read the pushes for anything involving argument passing or colour markers.

### Addresses

Every `0x...` is an offset into the **merged single-binary 1.14d build** — one `0x400000`-based
`.text`. They are **not** 1.10f DLL-relative. D2MOO annotates the same offsets as `1.14d: Game.0x...`.

### Game data

Comes from an MPQ extraction of the **1.14d** MPQs. Never a CASC extraction (those are D2R-era) and
never a re-exported or modded tree. The embedded copies under `data/` are byte-identical to that
extraction, and `tools/DataSmoke` takes an extraction path so you can check they have not drifted.

`STRUCT_CreateBinFieldExcelAndFillData` (real entry `0x6bd640`) deletes any row whose first cell is
exactly `Expansion`. Most `.bin` files therefore hold one fewer record than the `.txt`, and **every
literal row index in the binary is post-splice**. Verify per file — `ItemStatCost.txt` has no such
row, `ItemTypes.txt` does.

### Reachability is where this project goes wrong

Across eight audit rounds, the *mechanism* claims were reliable and the *"can a player see it"*
claims were wrong in **every single round**, in both directions:

- over-claimed three times (inflated affix counts, cell counts passed off as row counts)
- under-claimed once — a bug dismissed as unreachable for four rounds turned out to affect fifteen
  real items, because "every class-restricted shield is a paladin shield" was simply false

So: **count rows against the shipped files, not cells, and not from memory.** A dismissal is where
the next bug hides. If something is unreachable, say why, with the count.

### Comments

Only for genuinely non-obvious behaviour, and cite the address it models. No narrative comments, no
banners, no restating what the code plainly does. Much of this code looks odd because the original
is odd — that is what earns a comment; ordinary code does not.

### Parsers

Any data-parsing helper is written in **C#**, not PowerShell.

## Working method

The find → verify → fix loop is how every bug here was found, and the verify step caught something
in **every** round: a finding refuted outright, an expected string wrong in three ways, a proposed
fix of the wrong shape, a stated cause that concealed a larger defect underneath.

When changing behaviour:

1. Trace it in the disassembly yourself and cite the addresses.
2. Establish reachability by counting against the shipped data.
3. Fix, and pin it with a test asserting the **exact** rendered string.
4. Change **both** implementations, or the differential will fail — and it is the only thing
   holding them together.
5. Update `docs/writers.md` — it is the living spec, and a stale line there has caused bugs.

After changing C#, run the ReSharper inspection and read the SARIF. After changing TypeScript, run
ESLint **and** Prettier — they are separate gates, and `npm run lint` does not include formatting.

Be sceptical of a passing test suite. Four separate times a test was found to be asserting the bug

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResurrectedTrader/D2ItemToolkit](https://github.com/ResurrectedTrader/D2ItemToolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
