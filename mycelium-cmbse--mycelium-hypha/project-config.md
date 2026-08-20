---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two things in one repository, scoped to the **OMG SysML v2 / KerML** standards (not UML):

1. The **`hypha` Claude plugin** — `.claude-plugin/`, `skills/`, `agents/`, `commands/`, `hooks/`, and
   the `knowledge/` base the plugin reads at runtime.
2. The **generation pipelines** in `tools/` that build `knowledge/` from upstream OMG sources in
   `sources/`. The pipelines are not shipped with the plugin.

## Build & test

**.NET** (`tools/metamodel-gen`, `tools/knowledge-gen` and `tools/hypha-cli`, `net10.0`, NUnit), from
repo root:

```sh
dotnet build mycelium-hypha.sln
dotnet test mycelium-hypha.sln
dotnet test mycelium-hypha.sln --filter "FullyQualifiedName~MetamodelClosureTests"   # one fixture/test
dotnet test mycelium-hypha.sln --filter "TestCategory=Live"   # [Explicit]: really fetches from GitHub
```

Nothing reaches the network in a normal run. The `Live` category is `[Explicit]`, so it is opt-in and
never in CI; set `GITHUB_TOKEN` before running it (the anonymous API allows 60 requests an hour).

**Python** (`tools/spec-extract`, Python ≥ 3.12), from `tools/spec-extract/`:

```sh
python -m venv .venv && . .venv/Scripts/activate   # or source .venv/bin/activate
pip install -e .[dev]
ruff check .
pytest
pytest tests/test_clauses.py::test_detects_clauses_and_groups_body   # one test (or: pytest -k <expr>)
```

## The CLI generates, the tests verify

Generation is driven by **`tools/hypha-cli`** (the `hypha` command), never by the test runner. No
test writes into `knowledge/` — running `dotnet test` leaves the working tree clean, and a test that
dirties it is a bug.

```sh
dotnet run --project tools/hypha-cli/Hypha.Tools -- generate            # all artifacts, all releases
dotnet run --project tools/hypha-cli/Hypha.Tools -- generate metamodel --tag 2026-05
dotnet run --project tools/hypha-cli/Hypha.Tools -- fetch --tag 2026-05 # sources/<tag>/
```

**The orchestration lives in the library, not in the fixtures.** Each artifact is an
`IKnowledgeGenerator` (`Artifact`, `Order`, `GenerateAsync(tag)`), registered as a collection, so a
full run is a loop over `Order` — metamodel (10), grammar references (20), textual notation (30),
cross-references (40, last, because it reads the index and the generated examples). The CLI resolves
that collection; it does not know the artifact names, so adding a generator adds its verb.

Inputs are read from `RepositoryRoot`, output is written to `OutputRoot` (`--output`), which defaults
to the same folder. That separation is what lets `KnowledgeRegenerationTests` regenerate everything
into a scratch folder and compare it **byte for byte** against the committed files: the **committed
files are the golden**, and proving they are still reproducible no longer means rewriting them.
After an intended format change, regenerate with `hypha generate`, review the diff, and run the
`[Explicit]` `Bless_*` tests for the metamodel expectations.

- `spec-extract`'s `test_generate.py` writes the git-ignored `knowledge/<tag>/spec/` from
  `sources/<tag>/specs/*.pdf`. That is now all Python does here, and it is the one place the old
  "tests write it" convention survives — because that output is never committed.
- A generator whose inputs are absent returns `Skipped` with a reason, so CI stays green without the
  copyrighted/optional inputs. Inputs that are present but would yield **wrong** output throw instead
  — a missing metamodel index, a clause title reaching the cross-references, a slug collision.
- "Interesting" metaclasses are discovered via `ModelInspector`, never hardcoded.

## Committed vs git-ignored (and why)

- **Committed:** `knowledge/metamodel/`, `knowledge/textual-notation/`; `sources/xmi/` and
  `sources/textual/` (both EPL-2.0).
- **Git-ignored — never commit:** `sources/specs/*.pdf` and the generated `knowledge/spec/`. These are
  **full verbatim OMG specification text**; the OMG license forbids redistributing it, so it is
  regenerated locally only. A SessionStart hook (`hooks/check-spec-pdfs.py`) tells the user when the
  PDFs are missing.
- Exact upstream sources, commits and licenses are recorded in `sources/README.md` and `NOTICE`.
  (Metamodel XMI ← `SysML-v2-Pilot-Implementation`; PDFs + textual sources ← `SysML-v2-Release`.)

## Determinism of committed artifacts

Anything generated and committed must be byte-stable: sort collections with `StringComparer.Ordinal`,
emit LF (`.gitattributes` pins `knowledge/** eol=lf`), and include no timestamps, machine paths, or
culture-sensitive formatting. JSON uses `System.Text.Json` with fixed options and `\r\n`→`\n`
normalization. Golden tests compare against the committed files, so non-deterministic output breaks CI.

## Knowledge-base shape (what the skills/agents consume)

- `knowledge/metamodel/elements/<Name>.md` — one file per metaclass / enumeration / primitive: front
  matter, Generalizations/Specializations, **Owned features**, an **Inherited features** table giving
  the *full effective* feature set (read it directly — do not re-walk the generalization chain), and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mycelium-cmbse/mycelium-hypha](https://github.com/mycelium-cmbse/mycelium-hypha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
