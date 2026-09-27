---
trigger: always_on
description: These instructions apply to the entire repository.
---

# MailFathom Development Instructions

These instructions apply to the entire repository.

The product name is `MailFathom`. The service carries a solution of its own, `backend/MailFathom.slnx`, and `frontend/` stands beside it holding the client. **`frontend/` is a pnpm workspace of React and TypeScript**: the Uno Platform client that stood there was withdrawn — the platform did not work out for this project — and what replaced it is two packages under `frontend/src/`, `Client.Backend` and `Client.App`, with `frontend/README.md` as their page. `frontend/src-tauri/` beside them is the desktop head's Rust crate — the shell that wraps what those two build, and one of the two places a difference between the web head and the desktop head is allowed to exist, the other being a stated rule in the client's one stylesheet; `frontend/src/AGENTS.md` § *The two heads* holds that rule. The client's unit suite runs on Vitest, and a test in it sits beside the source it covers rather than in a tree of its own, so `frontend/tests/` holds the contract that governs all four suites beside the three that drive a built client and belong to neither package — `frontend/tests/AGENTS.md` says why, which of those three reaches a real deployment, and which of them drives the desktop head's own WebView. What this file holds is written for both stacks, and what governs the client alone is `frontend/AGENTS.md` beside `frontend/src/AGENTS.md`. Project directory and file names use short boundary names such as `Domain`, `Application`, and `Host`, while `backend/Directory.Build.props` applies the `MailFathom.*` prefix to assembly names and root namespaces.

## Project status

MailFathom is released. Tags are pushed, the image and the Helm chart are published, and the newest GitHub release carries the schema artifact that brings a database to it beside the `mfctl` binaries, so every file here describes something a reader can install rather than something being prepared for them. The major version is still `0`, and `<VersionPrefix>` in `Version.props` names the release being worked toward rather than the last one shipped. What follows is how a change against that is judged.

- A breaking change to the MCP tool contract, the configuration schema, the database schema, or the deployment contract is permitted, because the major version is `0`. ADR 0004 narrows SemVer's `0.y.z` clause to exactly that: a minor may break any of the four public surfaces, a patch may break none of them, and no deprecation window exists below `1.0.0`. So what a change decides is which shape of a contract is right, never whether the break may be taken — a wrong key name, a wrong tool argument, or a wrong column is not kept because somebody would have to act on an upgrade.
- MailFathom is under active development, and until `1.0.0` a release owes no compatibility with any earlier one — the root `README.md` tells every reader so before they install it. That reaches as far as complete incompatibility: a configuration that has to be rewritten rather than migrated, a tool contract replaced rather than extended, a deployment that has to be reinstalled. So a change is designed for the contract it should have, and never shaped, narrowed, or postponed to spare an existing installation an upgrade step.
- What a break does cost is the record of it. Name it in the issue and in the pull request, against the surface it breaks and with the operator's action rather than only the fact, because the release pull request composes `CHANGELOG.md` from that reading and nothing else writes the file. A break nobody wrote down is the defect here; the break itself is not.
- The database schema is append-only, which is the one place the permission above stops being a licence: a change adds a migration and never regenerates a baseline, and a release is deployable over the previous release's data unless its changelog entry says otherwise.
- Documentation, deployment assets, and the third-party register describe a product somebody is running, so an inaccuracy in one is a defect against a user rather than a note to fix before release.
- Still refused, and unchanged by any of the above: compatibility shims, deprecation machinery, versioning scaffolding, and migration paths for versions that never existed. Owning the current contract is not the same as inventing history behind it.

None of this relaxes the rest of these instructions.

## Where the rest of the contract lives

This file is loaded into every agent session in either stack, so it holds what has to be true before a file is read and what a session in one stack still owes the other: the non-negotiables, the two roles, the privacy and governance posture, the licensing obligations, the workflow and verification entry points, and the principles that outlive whichever language a stack is written in. A rule a session in the other stack could not act on is not one of them, so it lives in the stack that acts on it. Every rule lives in exactly one place, and no rule is stated twice. A row below is a pointer to the whole rule, never a summary of one.

| Where | Read when | What it holds |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Krzysztof318/MailFathom](https://github.com/Krzysztof318/MailFathom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
