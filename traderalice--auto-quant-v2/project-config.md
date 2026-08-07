---
trigger: always_on
description: AutoQuant V2 is a pre-alpha, Agent-native quantitative research workbench. Its
---

# AutoQuant V2 contributor guide

AutoQuant V2 is a pre-alpha, Agent-native quantitative research workbench. Its
job is to turn quantitative research into a file-backed, versioned, testable
workflow that coding Agents can operate and humans can inspect.

The same repository must work independently and unchanged as a specialized
Workspace desk inside OpenAlice or another Agent Harness. Host-specific
Session orchestration, communication, provenance, and live trading are optional
surrounding capabilities; they do not define AutoQuant Core.

Domain correctness, reproducible evidence, Agent operability, and a coherent
project model take priority over backward compatibility while V2 is taking
shape. Auto-Quant Classic files were retired from the current tree; Git history
is their archive and current code must not load or reinterpret them under V2
semantics.

Use Python 3.11 and `uv` for repository code and scripts. Keep projects
self-contained: a Workspace owns project discovery and a standardized Harness,
while each Project owns its research question, source inputs, strategies or
models, Runs, and durable artifacts.

The repository root is the default Workspace. Start with `aq project list .`
and `aq orient .`; use the checked-in `projects/sample-research-desk` to learn
the complete desk without treating its example question as a real assignment.
Create or continue sibling Projects in the root `projects/`. A framework
developer may have an ignored `autoquant-workspace.local.json` that explicitly
redirects effective Project discovery outside the repository; inspect CLI or
Studio configuration-source output before assuming which Projects are active.

For a new standalone desk, `aq workspace init <directory>` owns an absent or
empty target by default. Keep request/dataset packaging in a sibling staging
directory, or pass `--adopt-existing` explicitly when caller/host files are
already inside the intended desk. Adoption preserves those surrounding files
but refuses any existing Workspace manifest, local override, or `projects`
entry; never move or delete caller material merely to satisfy initialization.
Dataset asset paths resolve from the directory containing the package
manifest. When raw files already live under `staging/raw-ohlcv/`, put
`dataset-package.json` at `staging/` and use paths such as
`raw-ohlcv/AAPL.csv`; do not copy the raw bytes beside a deeper manifest.
Core still creates the intentional Project-local normalized, content-locked
snapshot. Never use `..`, absolute paths, or symlinks to bypass the manifest
root. In the strict Research Request `source`, set both `artifactPath` and
`artifactRevision` when an exact caller artifact is known, or set both to JSON
`null`; never fill only one. A local immutable artifact may use an explicit
content digest as its revision claim.

Workspace staging and Project `data/`/cache are persistent local working
evidence, not ordinary Git research state. The repository desk ignores
`staging/`, and every Project ignores its normalized data/cache by default.
Do not use `git add -f` to copy normal market data into history merely because
the checked-in teaching Project tracks its small deterministic fixture; that
sample is the explicit exception. Commit briefs, request/Study contracts,
candidate source, Runs, Sessions, Reports, Reviews, Dossiers, and other durable
research records. Track market bytes only when the caller explicitly requests
a distributable fixture and its source terms permit it.

Before adding a host integration or public surface, ask:

1. Can a coding Agent use the same capability in a standalone Workspace?
2. Is durable truth recoverable from files, manifests, Git, and immutable
   evidence rather than private conversation context?
3. Does the change preserve one Core contract across CLI, JSON, Studio, and
   any host projection?

## Starting a quantitative assignment

Do not begin with data downloads, candidate edits, model training, or
backtests. First establish the assignment as an English Markdown research
brief that another Agent can recover from the filesystem.

1. Inspect the Workspace and existing Projects. Continue the Project when the
   assignment belongs to its evolving research question; do not create a new
   Project merely because a new coworker or conversation arrived.
2. When the assignment is genuinely new, create its construction site with
   `aq project create <workspace> <project-id>`. First inspect
   `aq project templates`; use its fit and anti-fit contracts rather than
   guessing from template names. Use `--template blank` while the appropriate
   research method is still unclear. Factor evidence that must feed Portfolio
   or governed RL belongs in `ohlcv-research-desk`, not a standalone Portfolio
   Lab.
3. Before quantitative work, read and update the Project-root `research.md`.
   Rewrite the incoming assignment in English and preserve useful source
   context, including a verbatim caller statement when precision requires it.
4. Make the research decision, question, motivation, asset scope, horizon or
   cadence, available evidence, material constraints, evaluation meaning,
   expected deliverable, assumptions, open questions, and proposed bounded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TraderAlice/Auto-Quant-V2](https://github.com/TraderAlice/Auto-Quant-V2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
