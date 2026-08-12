---
trigger: always_on
description: This file defines rules, constraints, and architecture context for AI agents working in the `axios/` monorepo.
---

# Axios Agent Configuration

This file defines rules, constraints, and architecture context for AI agents working in the `axios/` monorepo.

## Tooling and Agent Skills

This workspace relies on globally installed agent skills and plugins rather than custom, project-specific frameworks.

> [!IMPORTANT]
> **Heed Installed Agent Skills & Environments:** Review available globally installed agent skills, tools, and local development environments before starting work:
>
> - **Rust Development:** Look for and adhere to installed Rust coding/linting agent skills for Rust-specific idioms and patterns.
> - **Nix Tooling:** Check if the `IN_NIX_SHELL` environment variable is set. If it is set, assume all development environment tools are already available and do not redundantly invoke `nix-shell`.
> - **Git Workflows & Commits:** Check for workflow-specific agent skills. Automated commits are permitted and encouraged _only if_ they strictly adhere to the instructions of an installed commit hygiene agent skill. If no such skill is available, do not perform automated commits.
> - **Formatting & treefmt:** Before any commit, format the workspace using `treefmt` to keep the codebase clean intra-commit.
> - Balance global tooling with project constraints: Heed installed agent skills for general workflows and language conventions, but always prioritize this monorepo's architectural layers, terminology constraints, and local invariants. Do not ignore your tooling, but ensure its application aligns with the project context.

---

## Spec-Driven Development

This project is strictly spec-driven. Agents must regularly consult the specification documents inside the [docs/specs/](docs/specs) directory to ensure design correctness.

- **Review Specifications First:** Before implementing features or making modifications, locate and read the corresponding spec file (e.g. L1/Atom specs, L3/Eos specs, or L4/Ion specs).
- **No Ad-Hoc Decisions:** Do not make assumptions or ad-hoc design decisions if a specification is unclear, ambiguous, or missing details.
- **Surface Unknowns:** If you encounter a gap or ambiguity in the specifications, halt and surface the unknown immediately so that it can be explicitly discussed, resolved, and documented.
- **Ground-Truth Direction (L2/HTC):** The composition substrate has a landed
  ADR and SAD but no spec yet (spec authorship is P3/P4 work). Until then,
  treat [ADR-0005](docs/adr/0005-hermetic-transactional-composition.md) and
  [htc-sad.md](docs/architecture/htc-sad.md) as the normative direction for
  anything touching build execution, composition, or the atom-DAG re-scope —
  they take precedence over any stale evaluation/derivation framing still
  present elsewhere in this tree.

---

## Project Overview

Axios is a decentralized, content-addressed source publishing stack decomposed
into three independent Cargo workspaces mapped to a six-layer architecture:

```
L5  Plugins    Plugin crates extending ion (future)
L4  ion/       Frontend: CLI, manifests, resolution
L3  eos/       Engine: builds, stores, scheduling
L2  HTC        Build-execution & composition substrate: CAS, compositions,
                interface manifests, build records, fetch-proxy execution,
                closure computation, materialization (skeleton workspace: htc/)
L1  atom/      Protocol: identity, addressing, publishing
L0  Cyphr      Cryptographic substrate (external; future)
```

Dependencies flow strictly downward: ion → eos → atom, with eos dispatching
build execution through HTC's executor trait (no crate dependency yet — see
[htc-sad.md](docs/architecture/htc-sad.md)). Each workspace is an
independent Cargo workspace with path-based inter-workspace deps.

For architecture details, see:

- [ADR-0001](docs/adr/0001-monorepo-workspace-architecture.md)
- [Roadmap](ROADMAP.md)
- [Formal Model](docs/models/publishing-stack-layers.md)

---

## Terminology Glossary

> [!CAUTION]
> Use **only** the canonical terms below. Legacy terms from the original eka
> codebase must **never** appear in new code, documentation, or conversation.
> If you catch yourself using a deprecated term, stop and correct it.

| Canonical Term         | Definition                                                                                                                                                                                                             | Deprecated Aliases (NEVER use)    |
| :--------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------- |
| **Anchor**             | Cryptographic commitment establishing atom-set identity: `czd(charter₀)`, the coz digest of the set's signed founding charter transaction (backend-agnostic; immutable across succession)                              | genesis, root, Root               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axiosoph/axios](https://github.com/axiosoph/axios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
