---
trigger: always_on
description: This monorepo builds filegrc, a Git-native GRC system for SOC 2 work. It has two Node.js packages:
---

# filegrc Repository Instructions

## Purpose

This monorepo builds filegrc, a Git-native GRC system for SOC 2 work. It has two Node.js packages:

- `filegrc`: the zero-dependency filegrc engine, which validates, searches, edits, and renders GRC data.
- `create-filegrc`: the filegrc scaffolder, which creates a standalone SOC 2 repository.

The generated repository is the product. Keep it understandable to an engineer who opens it without prior context.

## Agent-facing product surface

Treat headless use as a first-class interface. An agent with no filegrc context must be able to discover the right record type, inspect current relationship candidates, create or update JSON and Markdown through one validated payload, complete scheduled and event work, prepare an audit, and verify the result without opening the renderer.

- Keep the generated root `AGENTS.md` as the program and Git guide.
- Keep `data/AGENTS.md` as the universal record workflow. Add collection-level `AGENTS.md` files only where a wrong action has material compliance, privacy, or audit consequences.
- Keep `filegrc guide`, `types`, `list`, `get`, `references`, `scaffold`, CRUD, `content`, obligations, events, program readiness, audit readiness, and evidence packets model-driven.
- Scaffold files are prompts, not compliance facts. They must keep incomplete work in a non-final state and make missing required values obvious.
- Browser and CLI mutations must use the same domain functions and the same `{ record, content }` shape.
- Every resource type must pass automated guide and scaffold coverage. Test first-class multi-record workflows through the CLI as well as their domain functions.

## Product principles

- Git is the system of record. GRC records live as plain, reviewable files under `data/`.
- Git exclusively supplies version-control facts: file history, authors, commit timestamps, diffs, commit messages, revisions, renames, and prior file versions. Do not mirror them in FileGRC records or maintain a parallel change log.
- Domain events still need explicit dates. Do not replace dates such as `occurredOn`, `approvedOn`, or `completedOn` with Git metadata.
- Do not store a second change log or duplicate Git-derived fields such as `createdAt`, `updatedAt`, `createdBy`, or `updatedBy`.
- Store each mutable program fact or decision in one authoritative record and reference it by ID. Policies state durable rules, categories, and required outcomes; they do not copy current people, vendors, systems, reporting addresses, schedules, recovery targets, or other inventories that have their own resource.
- Roll up an Obligation occurrence when one queue-level owner, window, population rule, and reconciliation conclusion govern the work. Keep member-level operating records, evidence, completion, exception, and non-applicability facts inside that occurrence. Split work into separate Obligations or Action Items only when a member needs its own queue owner, deadline, conclusion, or follow-up lifecycle.
- The engine must work locally, in CI, and in a basic server environment with only a supported Node.js release and Git.
- The current repository state must remain useful without a network connection.
- Data files are authoritative. Rendered pages, indexes, caches, and reports are derived output.
- Never fetch external references automatically. A user may open or import one explicitly.
- Keep the model generic. Organization-specific fields belong in namespaced extensions.
- Keep the default starter Security-only and as simple as the Security Common Criteria permit. Do not turn optional Trust Services Categories or common implementation choices into default records or readiness gates. Require category-specific details, including numeric recovery objectives, only when management selects that category or an approved commitment or risk decision requires them.
- Prefer explicit, inspectable behavior over automation that changes audit records without review.
- UI, HTTP, and CLI workflows must call the same domain functions so headless agents receive the same calculations, validation, and output as browser users.

## Standards alignment

- Use AICPA SOC 2 terms for the assurance subject matter and use NIST OSCAL as the main interoperability reference for machine-readable GRC structure.
- Preserve OSCAL's useful separation among catalogs and requirements, profile-like applicability and tailoring, bounded Systems, Components, inventory items, control implementation, assessment plans, and assessment results. Keep FileGRC's flat, Git-reviewable records and typed ID relationships instead of reproducing OSCAL's nested document structure.
- Use OSCAL names only when the FileGRC concept has the same meaning. In particular, reserve `Profile` and `tailoring` for control selection or modification, `parameter` for a defined requirement placeholder and its assigned value, and `Component`, `Information Type`, and `inventory item` for their established model roles. Do not rename a broader FileGRC workflow to an OSCAL term merely because the records overlap.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alignbase/filegrc](https://github.com/Alignbase/filegrc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
