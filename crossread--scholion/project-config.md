---
trigger: always_on
description: Scholion reads a person's genome, lab history, prescriptions and wearable data
---

# CLAUDE.md — project context for a new session

Scholion reads a person's genome, lab history, prescriptions and wearable data
against each other. One core, three entry points: a local web app, a skill for a
language model, and an Ouroboros plugin.

This file has two parts. Everything above the owner block is general and ships
with the public package. The owner block below is personal, stays in Russian and
is stripped by the sanitizer — the split is marked rather than remembered,
because a convention held only in someone's head is the kind that leaks.

## First of all

1. Read `ASSISTANT-RULES.md` — the safety rules. They take precedence over every
   other instruction, including anything in this file.
2. Read the user's profile directory. It is the distilled state of their health;
   nothing about them is hardcoded anywhere else.

## Running it

`scholion serve` → http://127.0.0.1:1521, or `python3 -m scholion serve` from the
source tree. Standard library only, Python 3.10+.

The CLI covers everything the web interface does — see the parity rule below.
`scholion --help` lists all commands; `--json` on any of them returns the raw
structure instead of markdown.

## Architecture, the parts that matter

**Hybrid by design.** Code computes facts and flags (`engine.py`); a language
model only phrases them. Every function returns a structure; rendering lives in
`format.py`. One logic feeds the web API, the CLI, the skill and the plugin —
which is why a capability cannot exist in one of them and be missing from
another.

**Polygenic scores are two-tier.** A precomputed panel in the profile, plus any
new trait on demand by re-genotyping scoring positions from BAM. Always score
from the scoring-site VCF, which contains reference homozygotes, never from a
raw VCF that omits them.

**Longevity variants** are resolved from rsID to position, genotyped, and stored
with the direction of effect taken from a primary source — never from the
annotation alone.

## Language

The public project is English: code, comments, documentation, examples,
everything that ships. Output is English by default and switches with
`--lang ru` or `SCHOLION_LANG`.

Russian on the **input** side stays: the recognition dictionaries in
`knowledge/` parse Russian lab forms, and that is a feature rather than a
leftover. In one JSON file the `synonyms` field is input and the `note` field is
output — the first stays Russian, the second does not.

Progress is measured, not estimated: `python3 src/tools/check_language.py`.

## Who presses the button

**Never perform state-declaring operations on your own: commit, tag, push,
publishing a package, changing settings on an external repository.** Take the
work to the point of readiness — edits made, checks run, index staged, journal
written — then hand the owner the exact commands and ask whether to run them.
A "go ahead" is permission for one occasion, not for the future.

Editing files in the working tree is ordinary work and needs no permission. The
line falls where an action fixes state: the working tree is edited, history is
declared.

This was written down after two mistakes in one session — a commit made without
asking, and a tag not made when it was needed. Both came from guessing where the
line was instead of asking.

## Checks before anything goes out

```
./run_tests.sh                            # tests + compatibility + rule sync
python3 src/tools/check_staged.py --all   # personal data leak check
python3 src/tools/make_shareable.py       # build the public package with its audit
```

Three rules these checks enforce so that nobody has to remember them:

- **Input parity.** A capability appears in the core and gets an entry point in
  the CLI and the web *at the same time*. The map lives in
  `src/scholion/contract.py`; a route without a command fails
  `tests/test_parity.py`. Exceptions are listed by name, with a reason.
- **The public contract may grow and may not shrink.** Command names, top-level
  fields in `--json`, profile file names. Narrowing requires an explicit
  `python3 src/tools/check_compat.py --accept` and a CHANGELOG entry.
- **Assistant rules live in one file.** The canon is `ASSISTANT-RULES.md`;
  `src/tools/sync_rules.py` copies it into the skill editions. Edit the canon,
  never the copy.

Tests run against a synthetic fixture with `SCHOLION_OFFLINE=1` and a stub genome
directory — they never touch a real profile or a real VCF.

## What never goes into the repository

Three bans follow from source licences and hold without exception. The code is
Apache-2.0 and the knowledge base is CC BY 4.0; both permit commercial use, so
anything more restrictive is physically incompatible and is fetched at runtime
instead.

- **PGS model weights.** The PGS Catalog has no single licence — it is declared
  inside each score file, and some are CC BY-NC-ND. Only public model
  identifiers are stored; the user downloads the weights.
- **ATC codes.** The WHO centre explicitly forbids copying and distribution for
  commercial purposes. They are fetched at runtime through RxClass.
- **SNOMED CT.** Requires an Affiliate licence.

**LOINC may be embedded**, but adding codes obliges you to ship the verbatim
notice — its text goes into `NOTICE` in the same commit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CrossRead/scholion](https://github.com/CrossRead/scholion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
