---
trigger: always_on
description: Enterprise BPMN 2.0 Generator: JSON Logic-Core → Validation → ElkJS Layout → BPMN 2.0 XML + SVG.
---

# CLAUDE.md — BPMN Generator

## Project Context

Enterprise BPMN 2.0 Generator: JSON Logic-Core → Validation → ElkJS Layout → BPMN 2.0 XML + SVG.
OMG BPMN 2.0.2 compliant (ISO/IEC 19510:2013). Compatible with bpmn.io / Camunda Modeler.

Used as a Claude Code Skill (SKILL.md) — the LLM extracts Logic-Core JSON from natural language, the pipeline handles layout and serialization. The LLM NEVER touches coordinates.

## Glossary

- **Logic-Core**: the JSON intermediate format between LLM and pipeline. Schema in `references/input-schema.json`, prose in `references/logic-core-schema.md`. Example: `tests/fixtures/simple-approval.json`.
- **BPMN 2.0.2**: OMG standard (ISO/IEC 19510:2013) for business process notation. We emit XML compatible with bpmn.io and Camunda Modeler.
- **BPMNDI**: BPMN Diagram Interchange — the `<bpmndi:*>` namespace that carries graphical layout (coordinates, waypoints) alongside the semantic XML.
- **Pool / Lane**: a Pool is a participant in a collaboration (its own process boundary); Lanes partition a Pool into roles/actors. Pools communicate via Message Flows.
- **WF-Net**: Workflow-Net — a restricted Petri-Net with one source and one sink. Used for soundness analysis (WF01–WF03 rules).
- **Soundness**: a process is sound if every case can reach the end state, no dead activities, no proper deadlocks. Petri-Net property.
- **Sugiyama**: layered graph drawing algorithm (Sugiyama et al., 1981). ElkJS implements a variant; we use it via the `org.eclipse.elk.layered` algorithm.
- **ElkJS Layered**: JavaScript port of the Eclipse Layout Kernel's layered algorithm. Our auto-layout engine — see `scripts/layout.js`.
- **Bruce Silver Method & Style**: industry-recognized style conventions for BPMN diagrams. Most M-layer rules (M01–M10) derive from this work.
- **MCP**: Model Context Protocol — the protocol Claude Code uses to talk to external tools. We expose the generator via `scripts/mcp-bpmn-server.js`.
- **MaD**: Model-and-Data sanity check used by the robustness subsystem to validate synthetic fixtures.
- **Golden file**: an `.expected.bpmn` (or `.expected.svg`) committed alongside a fixture; tests fail if output diverges.

## Architecture

23 core-pipeline + 5 agent + 9 robustness modules under `scripts/`. Verify current inventory with `find scripts -name '*.js' -not -path '*/node_modules/*' -not -name '*.test.js' | wc -l`.

```
Core Pipeline (run on every generate call)
  pipeline.js (Orchestrator, public API runPipeline)
    ├── validate.js          ← rules.js
    ├── rules.js             ← types.js, workflow-net.js
    ├── workflow-net.js      ← types.js
    ├── topology.js          ← types.js
    ├── layout.js            ← types.js, utils.js, topology.js, elkjs
    ├── coordinates.js       ← types.js, utils.js
    ├── visual-refinement.js ← coordinates.js (opt-in compaction passes)
    ├── bpmn-xml.js          ← types.js, utils.js, topology.js, icons.js
    ├── svg.js               ← types.js, utils.js, icons.js
    ├── icons.js             ← utils.js
    ├── dot.js               ← types.js
    ├── types.js             (no deps)
    └── utils.js             (reads config.json)

Standalone tooling
  import.js                  BPMN XML → Logic-Core (DOM parser)
  moddle-import.js           BPMN XML → Logic-Core (bpmn-moddle path)
  http-server.js             HTTP API (/api/v1/generate, /orchestrate)
  mcp-bpmn-server.js         MCP server entry point
  evaluate-slm.js            Pipeline evaluation runner
  prepare-training-data.js   Training-data prep script
  audit.js                   Append-only JSONL audit log
  delivery.js                Webhook delivery + dead-letter
  orchestrator.js            Multi-agent orchestration

Agent subsystem (scripts/agents/)
  compliance.js, layout.js, llm-provider.js, modeler.js, reviewer.js

Robustness subsystem (scripts/robustness/)
  cli.js, curate-mad.js, failure-classifier.js, fixture-persister.js,
  graph-isomorphism.js, mad-validator.js, report-generator.js,
  stress-tester.js, synthetic-generator.js
  (+ seed-catalog.json, config.json, README.md)
```

**Guiding principle:** Each pipeline step is independently replaceable, configurable, and testable.

## Key Files

| File | Purpose |
|------|---------|
| `scripts/pipeline.js` | Orchestrator + CLI + Public API (`runPipeline`) |
| `scripts/rules.js` | Rule Engine: 28 rules, 4 layers (Soundness/Style/Pragmatics/Workflow-Net); M05/M06 severity=OFF. Verify count: `grep -c '^\s*id:' scripts/rules.js` |
| `scripts/validate.js` | Thin wrapper around `runRules()` |
| `scripts/types.js` | `isEvent`, `isGateway`, `isArtifact`, `bpmnXmlTag` |
| `scripts/utils.js` | `loadConfig`, `CFG`, constants, `esc`, `wrapText` |
| `scripts/topology.js` | `inferGatewayDirections`, `sortNodesTopologically`, `orderLanesByFlow` |
| `scripts/layout.js` | `logicCoreToElk`, `runElkLayout` (ElkJS Sugiyama) |
| `scripts/coordinates.js` | `buildCoordinateMap`, `clipOrthogonal`, pool width balancing |
| `scripts/bpmn-xml.js` | `generateBpmnXml` — OMG-compliant BPMN 2.0 XML + DI |
| `scripts/svg.js` | `generateSvg` — SVG rendering of all BPMN elements |
| `scripts/icons.js` | Event markers, task icons, bottom markers (Loop, MI, Ad-Hoc) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stieges/bpmn-generator](https://github.com/Stieges/bpmn-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
