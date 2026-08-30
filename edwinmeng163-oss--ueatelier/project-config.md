---
trigger: always_on
description: This is the first-read briefing for AI agents. It keeps only core identity,
---

# UEAtelier Agent Handoff

This is the first-read briefing for AI agents. It keeps only core identity,
read-order, freshness, tool-count, file-inventory, and safety rules. Deep
operational guidance lives in `Docs/agents-guide/`.

## Project Identity

UEAtelier is an Unreal Editor MCP self-extension workbench. The main
deliverable is `Plugins/UnrealMcp`. The plugin runs inside Unreal Editor,
exposes `http://127.0.0.1:8765/mcp`, adds `Window > UEAtelier Chat`, adds
`Window > UEAtelier Workbench`, and exposes Task Atlas from the Chat panel.

Current plugin metadata:

```text
Plugins/UnrealMcp/UnrealMcp.uplugin
FriendlyName: UEAtelier
VersionName: 0.35.0
EngineVersion: intentionally omitted for cross-version source builds
Type: Editor plugin
Required plugin: PythonScriptPlugin
```

The v0.35 source line prioritizes Unreal Engine 5.7 and 5.8. UE 5.6 remains a
maintenance compile canary during the transition, not a primary release gate.
`UEvolve.uproject` is the local development host and defaults to
`EngineAssociation` `5.7`. `Examples/UEvolveExample57` (UE 5.7.4) is the
primary sample-content host and is also built/tested with UE 5.8; the full
`Examples/UEvolveExample` host remains the UE 5.6 maintenance canary. Do not
add a duplicate multi-gigabyte UE 5.8 content host.

Multi-engine discipline:

- All `#if ENGINE_*_VERSION` goes in `UnrealMcpEngineCompat.h`.
- Before switching the shared `Examples/UEvolveExample57` host between UE 5.7
  and UE 5.8, delete both the plugin and host `Binaries/` and `Intermediate/`
  directories. Partial platform/build-subdirectory cleanup is insufficient.
- Run `Tools/install_git_hooks.sh` once after clone.
- `EAiProviderKind` values are append-only; do not renumber.

## Product Goal

Target:

```text
An Unreal Editor MCP self-extension workbench that lets AI add new editor
automation capabilities under audit, dry run, backup, build, test, rollback,
RAG, and long-memory safeguards.
```

Core capabilities: call Unreal Editor tools from Chat or external MCP clients;
inspect maps, assets, actors, Blueprint graphs, Widget trees, logs, tests,
metadata, memory, skills, and knowledge cards; edit Blueprint, Widget, and
Actor state through fixed schemas; scaffold, validate, dry run, apply, build,
test, classify, and roll back new MCP tools; use RAG/tool recommendation before
creating tools; write project memory; distill repeated work into skills.

## Read Order For A New Agent

Start with `AGENTS.md`, `README.md`, `Plugins/UnrealMcp/README.md`,
`Docs/Architecture.md`, and `Docs/SecurityModel.md`.

Then read on demand:

| Task type | Read next |
|---|---|
| Architecture or module split | `Docs/Architecture.md` |
| RAG, knowledge search, recommendation | `Docs/KnowledgeRag.md`, `Tools/UnrealMcpKnowledge/README.md`, `Tools/UnrealMcpKnowledge/Evals/core_rag_eval.json` |
| Security or path safety | `Docs/SecurityModel.md` |
| Code file editing (read/search/preview/apply/rollback) | `Docs/CodeTools.md`, `Docs/agents-guide/code-tools.md` |
| Python user-tool composition with core tools | `Docs/CallTool.md`, `Docs/agents-guide/call-tool.md`, `Docs/SecurityModel.md` |
| Self-extension, scaffolds, audit, rollback, supervisor recovery | `Docs/agents-guide/self-extension.md`, `Docs/SelfExtensionPipeline.md`, `Tools/UnrealMcpSkills/mcp-self-extension/SKILL.md` |
| Task Atlas lifecycle, schemas, clustering, promotion | `Docs/agents-guide/task-atlas.md`, `Docs/TaskAtlas.md` |
| Automation tests, PIE smoke, editor diagnostics | `Docs/agents-guide/verification.md`, `Docs/Verification.md` |
| Build, release, packaging, install, deployment | `Docs/agents-guide/packaging.md`, `Docs/DeploymentTroubleshooting.md`, `Plugins/UnrealMcp/README.md`, `Tools/install_unrealmcp_to_project.py` |
| Windows compatibility or Win packaging | `Docs/agents-guide/packaging.md`, `Docs/WindowsCompatibilityLessons.md`, `Docs/WindowsPackaging.md`, `Docs/Stage2WindowsVerify.md`, `Tools/package_plugin.ps1` |
| Build hygiene or packaging-script pitfalls | `Docs/agents-guide/packaging.md`, `Docs/BuildAndPackagingPitfalls.md` |
| Tool changes | `Tools/UnrealMcpToolRegistry/tools.json`, `Plugins/UnrealMcp/Source/UnrealMcp/Private/UnrealMcpToolRegistrar.cpp`, then the category source file |

## Documentation Freshness Rule

After every meaningful project change, update the AI-facing docs before
handoff:

1. Update `AGENTS.md` when project structure, tool-extension workflow, safety
   rules, build/test commands, RAG behavior, or current project status changes.
2. Update `README.md` when user-facing install, usage, feature coverage,
   deployment, or product positioning changes.
3. Update the focused doc under `Docs/` when the change belongs to a specific
   system such as architecture, self-extension, RAG, security, supervisor, or
   deployment troubleshooting.
4. If the change adds or changes tools, update ToolRegistry metadata, tests, and
   relevant docs in the same patch.

Treat stale docs as a product bug.

## Repository Map

Important versioned paths:

```text
README.md, AGENTS.md, UEvolve.uproject, open_uevolve.command
Docs/agents-guide/
Docs/Architecture.md, Docs/CallTool.md, Docs/KnowledgeRag.md, Docs/SecurityModel.md
Docs/SelfExtensionPipeline.md, Docs/TaskAtlas.md, Docs/Verification.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edwinmeng163-oss/UEAtelier](https://github.com/edwinmeng163-oss/UEAtelier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
