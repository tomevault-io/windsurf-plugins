---
trigger: always_on
description: This project uses **Linked Literate Programming (LLP)** ([upstream](https://github.com/ccheever/llp), v0.5.2). Design
---

# Agent Instructions

This project uses **Linked Literate Programming (LLP)** ([upstream](https://github.com/ccheever/llp), v0.5.2). Design
and research decisions live in numbered documents under `llp/`, and code points at them with
`@ref LLP NNNN#anchor — gloss` comments. Start with [LLP 0000](./llp/0000-fluiddb.explainer.md).

## Map

- TypeScript is the primary implementation: `packages/`, `evals/`, `examples/`, `scripts/` at the root.
- Python is archived intact under `deprecated/python/`; run historical lab commands from that directory.
- Start development with `bun install --frozen-lockfile`; verify with `bun run check`, `bun run test:runtime`,
  `bun run verify:packages`, and `./ref-check`. `bun run eval` exercises TypeScript with free synthetic fixtures.
- `llp/0000` is the root: what FluidDB is, where things live, the constraints, and a table of every document.
- `llp/0015` describes the whole design as the evidence stands; read it before changing an architectural part.
- Archived Python subsystems:
  - engine: 0004
  - write path: 0005
  - read path: 0006
  - background passes: 0007
  - Jev: 0008
  - evaluation: 0009
  - interfaces: 0010
  - the memory with several stores (`deprecated/python/lab/memory/`): 0013
- Research:
  - how research is done: 0001
  - the findings ledger: 0002; round records 0002.000-0002.003 (rounds 1-4)
  - round 5: 0003 (RFC) and 0003.000 (results)
  - round 6: 0012 (RFC) and 0012.000 (results)
  - round 7: 0013 (RFC) and 0013.000 (results)
  - round 8: 0014 (RFC) and 0014.000 (results)
  - round 9: 0016 (RFC) and 0016.000 (results)
  - round 10, real conversations (private data under `LAB_PRIVATE_DIR`): 0017 (RFC) and 0017.000 (results)
  - round 11, other people's conversations (OpenAI only via `LAB_PROCESSORS=openai`): 0018 (RFC) and 0018.000 (results)
  - round 12, memory for conversation (meaning search, statements, a dossier): 0019 (RFC) and 0019.000 (results)
  - round 13, search and creation done properly, Jev picks (replay: `deprecated/python/lab/bench/replay.py`): 0020 (RFC) and 0020.000 (results)
  - round 14, a better picker, every wording kept, a live replay: 0021 (RFC) and 0021.000 (results)
  - round 15, the re-telling detector: 0022 (RFC) and 0022.000 (results)
  - SDK, database adapters, MCP and parity: 0023.001 and 0023.001.000
  - HiveNet feedback and portable memory skills: 0023.002 and 0023.002.000
  - Merge review, erasure generations and npm preview: 0023.003; release steps in `docs/releasing.md`.
  - the agenda: 0011
- TypeScript conversation-memory package and Cloudflare service (`packages/`): 0023
- BetterMind backend integration and memory-viewer assessment: 0023.000 (contracts, gaps and evaluation gates).
- Sole-runtime BetterMind integration, SDK source/forget APIs and regression evidence: 0023.004.
- Communication preferences, occasional check-ins, Jev boundary and technical feedback routing: 0023.005 (proposal).
- TypeScript replay evaluation: 0024 (protocol), 0024.000 (verification), `evals/README.md` (commands).
- Swappable SDK components and comparisons: 0024.001 (protocol), 0024.001.000 (real-provider results).
  `bun run eval:compare` is free; `@fluiddb/fluiddb/eval` exports the portable grader/comparison API.

## LLP documents

- Documents live flat in `llp/` as `NNNN-slug.type.md`. Sub-documents nest by dotted number: `0003.000-…` is the first
  child of LLP 0003. Numbers are never reused.
- `llp/current/` holds links to what is in play; `llp/foundation/` holds links to the kernel (empty until documents
  are `Active`). Orient by reading `foundation/`, then `current/`, then the `@ref`s in scope.
- New documents take the next number and the metadata header (`Type`, `Status`, `Systems`, `Author`, `Date`).
  - Types used here: RFC for proposals with hypotheses, Research for results and the findings ledger (LLP 0002),
    Principles, Explainer.
- Documents are living: update them when the system changes. Mark replaced ones `Superseded` or `Tombstoned` in the
  header.
- Provenance on rationale:
  - `[confirmed]` (named person, date) for what the maintainer said
  - `[observed]` (with a path) for what the code or results show
  - `[inferred]` for hypotheses; a document containing `[inferred]` stays `Draft`

## @ref annotations

- Add `# @ref LLP NNNN#section — gloss` above code that implements a non-obvious documented decision: code an agent
  might "simplify" in a way that breaks the design.
- When changing code that carries a `@ref`, check that the section still applies; update the document or the
  reference.
- `./ref-check` validates references and metadata (vendored from LLP v0.5.2, MIT, with exclusions for this project's
  private `.context`, model `.cache` and generated `.wrangler` directories). Run it before committing.

<!-- BEGIN LLP SKILLS MANAGED BLOCK -->
Before editing a subsystem with documented design, orient first: read its
governing LLP, and for non-trivial work invoke `llp-orient` to assemble a
context pack of the constraints the change must respect.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheMind-AI/fluid-db](https://github.com/TheMind-AI/fluid-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
