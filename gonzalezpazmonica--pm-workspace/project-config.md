---
trigger: always_on
description: > Auto-generated from `.claude/agents/*.md`. **Do not edit by hand.**
---

# AGENTS.md

> Auto-generated from `.claude/agents/*.md`. **Do not edit by hand.**
> Source of truth: `docs/rules/domain/agents-md-source-of-truth.md` (SE-078).

## How to use

This file is the cross-frontend mirror of Savia's agent registry. Claude Code
reads `.claude/agents/*.md` directly; OpenCode v1.14, Codex, Cursor and other
modern frontends pick up this `AGENTS.md` as freeform context. The source of
truth is `.claude/agents/*.md`; this index is regenerated automatically by
the Stop hook `agents-md-auto-regenerate.sh` whenever an agent file changes.

## Agents

| Name | Model | Permission | Tools | Description |
|---|---|---|---|---|
| architect | claude-opus-4-7 | L1 | Read,Glob,Grep,Bash | Diseño de arquitectura .NET y decisiones técnicas de alto nivel. Usar PROACTIVELY cuando: se diseña una nueva feat... |
| architecture-judge | claude-sonnet-4-6 | L1 | — | Code Review Court judge — boundaries, coupling, layer violations, patterns |
| azure-devops-operator | claude-haiku-4-5-20251001 | L1 | Bash,Read | Operaciones rápidas en Azure DevOps: consultas WIQL, actualización de work items, gestión de sprint, capacidades d... |
| business-analyst | claude-opus-4-7 | L1 | Read,Glob,Grep,Bash | Análisis de reglas de negocio, descomposición de PBIs, criterios de aceptación y evaluación de competencias del e... |
| calibration-judge | claude-sonnet-4-6 | L1 | — | Truth Tribunal judge — confidence statements match evidence strength |
| cobol-developer | claude-opus-4-7 | L3 | Read,Write,Edit,Bash,Glob,Grep | Asistencia en código COBOL/mainframe. IMPORTANTE: La mayoría de tareas COBOL deben realizarlas humanos expertos en ... |
| code-reviewer | claude-opus-4-7 | L1 | Read,Glob,Grep,Bash | Revisión de código .NET como quality gate antes de merge. Usar PROACTIVELY cuando: se completa una implementación ... |
| cognitive-judge | claude-sonnet-4-6 | L1 | — | Code Review Court judge — debuggability at 3AM, naming, complexity, logs |
| coherence-judge | claude-sonnet-4-6 | L1 | — | Truth Tribunal judge — internal consistency (sums, dates, entities) |
| coherence-validator | claude-sonnet-4-6 | L0 | Read,Glob,Grep | Verifies that generated outputs (specs, reports, code) actually match the stated objective. Use PROACTIVELY post-SDD,... |
| commit-guardian | claude-sonnet-4-6 | L4 | Bash,Read,Glob,Grep,Task | Guardian de commits: verifica que todos los cambios staged cumplen las reglas del workspace ANTES de hacer el commit.... |
| completeness-judge | claude-sonnet-4-6 | L1 | — | Truth Tribunal judge — report covers what its title/abstract promises |
| compliance-judge | claude-opus-4-7 | L1 | — | Truth Tribunal judge — PII, N1-N4b levels, format rules, confidentiality |
| confidentiality-auditor | claude-opus-4-7 | L1 | — | Audita cumplimiento de confidencialidad en PRs de pm-workspace (repo publico). Descubre dinamicamente datos sensibles... |
| correctness-judge | claude-sonnet-4-6 | L1 | — | Code Review Court judge — logic, tests, edge cases, error paths |
| court-orchestrator | claude-opus-4-7 | L4 | — | Convenes the Code Review Court, manages fix cycles, produces .review.crc |
| dev-orchestrator | claude-sonnet-4-6 | L4 | — | Analiza specs y crea planes de implementación con slices, dependencias y presupuestos de contexto |
| diagram-architect | claude-sonnet-4-6 | L1 | Read,Glob,Grep,Bash,Write,Edit | Architecture diagram specialist. Analyzes code and infrastructure to generate Mermaid diagrams, validates business ru... |
| dotnet-developer | claude-sonnet-4-6 | L3 | Read,Write,Edit,Bash,Glob,Grep | Implementación de código C#/.NET siguiendo specs SDD aprobadas. Usar PROACTIVELY cuando: se implementa una feature ... |
| drift-auditor | claude-opus-4-7 | L1 | — | Auditoría de convergencia repo: detecta drift entre docs, config y código. Usar PROACTIVELY tras cambios grandes o ... |
| excel-digest | claude-opus-4-7 | L2 | — | Digestion de hojas de calculo Excel (XLSX/XLS/CSV) — pipeline de 4 fases. Extrae estructura, formulas, patrones de ... |
| factuality-judge | claude-opus-4-7 | L1 | — | Truth Tribunal judge — factual accuracy of claims against verifiable sources |
| feasibility-probe | claude-sonnet-4-6 | L3 | — | Validates spec feasibility by attempting a time-boxed prototype. Produces viability report with score, blocking secti... |
| fix-assigner | claude-sonnet-4-6 | L2 | — | Creates fix tasks from Court findings, assigns to dev agents, triggers re-review |
| frontend-developer | claude-sonnet-4-6 | L3 | Read,Write,Edit,Bash,Glob,Grep | Implementación de código frontend (Angular y React) siguiendo specs SDD aprobadas. Usar PROACTIVELY cuando: se impl... |
| frontend-test-runner | claude-sonnet-4-6 | L4 | — | Post-commit frontend test execution — unit, component, e2e, coverage |
| go-developer | claude-sonnet-4-6 | L3 | Read,Write,Edit,Bash,Glob,Grep | Implementación de código Go siguiendo specs SDD aprobadas. Usar PROACTIVELY cuando: se implementa una feature en Go... |
| hallucination-judge | claude-opus-4-7 | L1 | — | Truth Tribunal judge — detects invented facts via SelfCheck-style consistency |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gonzalezpazmonica/pm-workspace](https://github.com/gonzalezpazmonica/pm-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
