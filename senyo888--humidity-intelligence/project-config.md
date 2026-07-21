---
trigger: always_on
description: ![Humidity Intelligence agent header](assets/agent-banner.png)
---

![Humidity Intelligence agent header](assets/agent-banner.png)

# Humidity Intelligence Agent Guide

This is the public, repository-safe operating guide for AI agents working on Humidity Intelligence. It is intentionally concise and complements the tracked public architecture contract.

## Project Identity

Humidity Intelligence is a deterministic Home Assistant environmental control engine packaged as a HACS custom integration. It performs runtime-driven environmental orchestration, resolves one control decision per cycle, and generates truth-based dashboards from backend telemetry, mappings, and diagnostics.

## Source of Truth

- Public repo correctness must be reviewable from tracked files in this repository.
- `ARCHITECTURE.md`, `AGENTS.md`, `README.md`, `CHANGELOG.md`, tracked docs, runtime code, tests, generated UI templates, and `ui-gallery/` are the public review contract.
- Maintainers may keep `DESIGN_BRIEF.md`, `PROJECT_SUMMARY.md`, `ROADMAP.md`, and `PROPOSALS.md` as ignored local planning or release-preparation documents.
- If architecture, runtime behavior, security posture, release flow, contributor expectations, or documentation expectations materially change, update the relevant tracked public docs in the same work.
- Ignored local docs may be mirrored separately, but must not be required for public contributor correctness.
- Maintainers may keep local-only instructions in `AGENTS.local.md`. That file is intentionally ignored and must not be required for public contributors or public repo correctness.

## Repository Workspace Boundary

- Release truth must come from a git-managed Humidity Intelligence checkout or a worktree created from that checkout.
- Unmanaged mirror folders, scratch folders, and local planning surfaces are not release authority.
- Local-only planning, lab, credential, generated-export, and agent-private files must remain ignored unless publication is explicitly approved.
- Do not require machine-specific local paths for public repo correctness, validation instructions, release notes, issue templates, or PR descriptions.

## Codex Pet Memory Architecture

- Codex pet memory is a first-class subsystem under `.codex/memories/`.
- Canonical pet memory paths use the placeholder pattern `.codex/memories/pets/<PetName>/`.
- Shared project memory belongs under `.codex/memories/project/`; shared terminology belongs under `.codex/memories/shared/`.
- Pet identity pointers may exist under `.codex/pets_pointer/`. `.codex/pets/` is historical or possible app identity space only; pet memory, history, canon, and reporting rules must not be stored there.
- Repository memory must stay public-safe: no secrets, credentials, private entity IDs, private MCP configuration, or machine-specific local paths.

## Non-Negotiable Architecture Rules

- Preserve the integration name, domain, HACS identity, and public package positioning unless explicitly instructed otherwise.
- Keep deterministic control authoritative: one selected ventilation lane per evaluation cycle.
- Keep humidity targets season-aware and profile-relative.
- Keep generated dashboards and reason panels aligned with backend truth only.
- Do not add hidden automations, hidden service paths, or parallel output writers.
- Optional frontend cards and UI dependencies must never block backend functionality.
- Unknown, unavailable, incomplete, or unmapped inputs must degrade safely and explainably.

## Deterministic Runtime Rules

- Preserve lane priority: CO emergency, humidity danger, mould danger, mould risk, condensation danger, condensation risk, zone 1, zone 2, AQ, normal.
- CO emergency is always the highest-priority runtime lane.
- Humidity, mould, and condensation alerts must resolve source, room, and zone before applying zone-bound control.
- Humidity danger thresholds are derived from the active target profile, not legacy static alert values.
- Humidifier lanes remain independent from ventilation lane resolution.
- Global gates must be respected and surfaced truthfully in runtime telemetry and UI.
- Missing outputs or failed optional service calls must be logged, skipped, and exposed without crashing the control loop.

## UI/Card Generation Rules

- Do not invent placeholder entities.
- Do not use private entity IDs, device IDs, room names, telemetry values, or user-specific helpers in published cards, tests, docs, screenshots, or examples.
- Do not ship malformed Lovelace structures, empty card containers, invalid conditionals, or unresolved self-mapped placeholders.
- Dashboard chips must map to backend telemetry, entity mapping, diagnostics, or runtime truth.
- Current Air Control chips are display surfaces only. They must not create or alter lane decisions.
- Alert chipsets should stay concise: active lane/status plus resolved source context.
- Optional chip rows and optional frontend dependencies must hide or degrade cleanly when unavailable.
- After UI template, mapping, chip, or card-generation changes, validate exported/generated cards before completion.

## Home Assistant and HACS Compatibility Rules

- Keep config flow, options flow, entity registry behavior, services, translations, diagnostics, and generated files compatible with supported Home Assistant versions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [senyo888/humidity-intelligence](https://github.com/senyo888/humidity-intelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
