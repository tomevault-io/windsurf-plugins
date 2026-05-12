---
trigger: always_on
description: > AigisCode is a native Rust whole-codebase analyzer. This file gives AI coding
---

# AGENTS.md

> AigisCode is a native Rust whole-codebase analyzer. This file gives AI coding
> agents the current project contract.

## Deployment

- Website host: `21.davidstrejc.cz:/home/agisicode.com/public_html`
- Website build: `cd website && npm run build`
- Domain: `aigiscode.com`

## Product Direction

- `GOAL.md` is the top-level product contract. Read it before making product,
  graph, detector, plugin, or reporting decisions.
- `ZEUS_SHIELD.md` defines the guardian doctrine for preventing architectural
  drift, dependency cancer, and AI-generated incoherence. Use it when making
  decisions about doctrine, enforcement, convergence, and codebase guidance.
- The implementation target is native Rust end-to-end.
- Do not add new Python bridges, Python tooling, or Python-hosted runtime paths.
- The website may describe roadmap items, but only the Rust CLI is a supported
  product surface today.

## Current CLI

```bash
aigiscode analyze /path/to/project
aigiscode agent /path/to/project
aigiscode agent-run /path/to/project
aigiscode agent-spider /path/to/project
aigiscode report /path/to/project
aigiscode info /path/to/project
aigiscode plugins
aigiscode tune /path/to/project
aigiscode surface /path/to/project
aigiscode --version
```

`analyze` writes:

- `.aigiscode/deterministic-analysis.json`
- `.aigiscode/semantic-graph.json`
- `.aigiscode/dependency-graph.json`
- `.aigiscode/evidence-graph.json`
- `.aigiscode/contract-inventory.json`
- `.aigiscode/doctrine-registry.json`
- `.aigiscode/deterministic-findings.json`
- `.aigiscode/ast-grep-scan.json`
- `.aigiscode/external-analysis.json`
- `.aigiscode/architecture-surface.json`
- `.aigiscode/review-surface.json`
- `.aigiscode/convergence-history.json`
- `.aigiscode/guard-decision.json`
- `.aigiscode/aigiscode-handoff.json`
- `.aigiscode/agentic-review.json`
- `.aigiscode/graph-packets.json`
- `.aigiscode/repository-topology.json`
- `.aigiscode/aigiscode-report.json`
- `.aigiscode/aigiscode-report.md`

## Recommended Agent Workflow

1. Run `aigiscode analyze /repo`.
2. Parse `.aigiscode/aigiscode-report.json` for the consolidated machine contract.
3. Read `.aigiscode/deterministic-findings.json` for raw detector output.
4. Use `.aigiscode/ast-grep-scan.json` when you want provenance-rich structural rule hits from the secondary scanner plane. Today it covers loop-local expensive-operation clues for `AlgorithmicComplexityHotspot`, dangerous-API clues that can reinforce native `SecurityDangerousApi` findings (`eval`, `exec/system`, unsafe deserialization, unsafe HTML output), and narrow framework-misuse clues for both raw env access outside config/bootstrap boundaries and raw container/service-locator lookup outside provider/bootstrap or injection boundaries that can reinforce native `SanctionedPathBypass` findings. The engine stays generic, while framework-specific rule catalogs can now contribute findings with explicit provenance such as `ast_grep.pattern.laravel` and `ast_grep.pattern.django`; this artifact is evidence, not graph truth. `aigiscode-report.json.summary` and `architecture-surface.json.overview` now also expose family-level scanner counts so the scanner mix is visible without loading the raw scanner artifact.
5. Use `.aigiscode/dependency-graph.json` for low-noise architecture queries.
6. Use `.aigiscode/evidence-graph.json` for detailed call-site and runtime evidence.
7. Use `.aigiscode/contract-inventory.json` for declared routes, hooks, env/config keys, and symbolic runtime contracts.
8. Use `.aigiscode/doctrine-registry.json` for the machine-readable guardian doctrine and default clause disposition.
9. Use `.aigiscode/architecture-surface.json` and `.aigiscode/review-surface.json` for topology and triage. On cropped/scoped analyses these artifacts now distinguish confirmed orphan debt from `boundary-truncated` files whose callers may live outside the analyzed slice.
10. Use `.aigiscode/convergence-history.json` to compare the current run against the previous artifact baseline in the same output directory.
11. Use `.aigiscode/guard-decision.json` for the current allow/warn/block judgment and required review radius.
12. Use `.aigiscode/aigiscode-handoff.json` when handing the repository to another agent.
13. Use `.aigiscode/agentic-review.json` or `aigiscode agent /repo` when you want the graph-backed AI review contract, prompts, diff-aware task packets, evidence chains, bounded typed multi-path graph traces, bounded code flows, explicit source/sink paths, bounded semantic state-flow evidence for mutable carriers, artifact priorities, and the adapter catalog for execution. The contract now also treats `.aigiscode/graph-packets.json` and `.aigiscode/repository-topology.json` as first-class agent context, not side artifacts.
14. Use `.aigiscode/graph-packets.json` when you want bounded doctrine-aware graph neighborhoods for agents or reviewers without loading the entire graph artifact family. Fallback focus-file packets now also carry bounded traces, code flows, source/sink paths, and semantic state-flow evidence instead of empty shells.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Draivix/aigiscode](https://github.com/Draivix/aigiscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
