---
trigger: always_on
description: > Canonical contributor guide for AI agents (Claude Code, Cursor, Copilot, Codex, Aider, …) and humans alike. Read once before touching the codebase.
---

# AGENTS.md — ontology-atlas

> Canonical contributor guide for AI agents (Claude Code, Cursor, Copilot, Codex, Aider, …) and humans alike. Read once before touching the codebase.

## Project overview

`ontology-atlas` is **a local-first ontology workbench for understanding a product/system from business core to implementation evidence**. The `.md` frontmatter inside the vault *is* the nodes and edges — frontmatter is self-approving, no separate review step. Planners, marketers, C-level decision-makers, developers, and AI agents should be able to read the same graph: business/product domains, capabilities, ownership, dependencies, evidence, and impact. Developers edit via CLI (`ontology-atlas` 54 commands — vault scaffold, agent setup repair, agent-file drift readout, agent activity heartbeat, MCP verify, deterministic graph compile, standard-format interop export, bounded path enumeration, transitive reachability, relation preflight + write, commit preflight, git snapshot, agent handoff, growth/maintenance queue, daily exploration, graph-level deep dive) or web UI (`/ontology`, `/docs`); AI agent (Claude Code, Codex, Cursor) reads/writes the same `.md` files through the `mcp/` server's current runtime inventory. **The macOS app carries that server inside its own bundle** — installing the app installs the agent surface, and the in-app connect button writes the client config with real absolute paths. There is no npm package; environments without the app run the server from a source checkout.

Atlas does not try to replace CodeGraph, grep, AST indexes, language servers,
or source search. Those tools answer structural code questions. Atlas gives
coding agents the durable meaning layer above them: the task starting point,
domain/capability context, implementation evidence, impact boundary, and
verification path that explain why a code artifact matters.

In this project, **ontology** means the executable meaning model of a
business/product and the codebase that realizes it. The five authorable kinds,
reserved reader kind, exact relation support layers, `is_a` test, and
RDF/OWL/SKOS/SHACL non-conformance boundary have one authority:
`docs/ONTOLOGY-ATLAS-SPEC.md` §2/§5. Do not maintain another kind/relation
glossary here; this guide owns contributor workflow, not the public meta-model.

**Identity (2026-07): agent-native, human-sovereign.** Not "memory for agents"
(machine-only stores lose to automation scale) and not another wiki (human-only
docs rot instantly). One meaning layer both audiences read and write: agents
are first-class users who keep it fresh through MCP/CLI; humans stay the
arbiters of meaning through plain markdown, git diffs, and their own disk as
the source of truth. The marketing hook opens with the agent pain; the product
substance is the shared layer. Every surface must pass both tests: can an
agent consume it (typed facts, handoff) and can a human read and judge it
(plain language, visual hierarchy)?

**Two surfaces, one folder (2026-07-27 — `docs/DECISIONS.md`).** The macOS app
is the vault's home: the workbench where a person judges the map and connects
the agents. The web is first a **gateway** (open the map with no install —
demo, first five minutes, a shareable link) and second a **second-best
workbench** where no app exists yet (Chromium on Windows/Linux). They do **not
promise the same screens**, and desktop capabilities ship without a web
backfill. What is shared is the folder: same markdown on disk, one parser
contract, and every cross-surface record written inside the vault
(`.ontology-atlas/*.jsonl`). One codebase, one build — the split is the four
capability bridges plus honest degradation, never a fork. Full contract,
including the web smoke gate that keeps the unattended surface alive:
`.claude/rules/surfaces.md`.

**Two gates stand before implementation** — the PO gate for product/UX/graph/
MCP/CLI/workflow/macOS-shell changes, then the design gate for anything visual.
Shipped output is not product progress here unless it improves a real human or
AI-agent ontology workflow. Both are specified under *Working principles* below;
don't start a non-trivial change without them.

For direction, see `docs/PRODUCT-DIRECTION.md`. For features users can use right now, see `docs/FEATURES.md`.

The single guiding principle (v3, R11 fire #25):

> **One product/system, one ontology, that people and their AI agents grow together.**

Markdown frontmatter is the graph. The git repo is the source of truth. No backend. No login. The developer + AI-agent loop keeps the ontology fresh; the macOS app and topology are the shared decision surface for planners, marketers, leadership, developers, and agents.

## Quick start

```bash
pnpm install && pnpm dev          # localhost:3000 — pick a markdown folder and you're in
pnpm --dir mcp install            # mcp/ carries its own lockfile — root install skips it
pnpm checks:changed               # start here: the focused checks for what you touched
```

**Re-run the `--dir mcp` line after any pull that touches `mcp/package.json`.**
It is the only way `mcp/node_modules` reaches the version that file names, and a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wlsdks/ontology-atlas](https://github.com/wlsdks/ontology-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
