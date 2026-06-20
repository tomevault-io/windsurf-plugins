---
trigger: always_on
description: Find the bugs your tests miss. Define what your Solana program must guarantee in a .qedspec; QEDGen validates it, generates tests and proofs, and scaffolds agent-fill Rust code. Trigger when the user asks for "qedgen", "qedspec", "verify my code", "prove correctness", formal verification, property testing, generated Kani/proptest/Lean artifacts, or Solana program correctness.
---


# QEDGen

## Trigger And Mission

Use this skill when the user wants to verify Solana program behavior, write or review a `.qedspec`, generate verification artifacts, onboard an existing Anchor program, or keep generated artifacts in sync.

**Preflight (once):** if `qedgen --help` fails, the CLI binary isn't set up yet — run `bash install.sh` from this skill's directory. It downloads the platform binary and links it onto your `PATH` (safe to re-run). Everything below assumes `qedgen` resolves.

Mission:
- Read the source before writing the spec.
- Treat `.qedspec` as the single source of truth.
- Use `qedgen check` to validate the spec.
- Use `qedgen codegen` to scaffold generated artifacts.
- Fill generated Rust handler TODOs as an agent task, then build and test.
- Use `qedgen verify` and drift gates to keep proofs and code synchronized.

Do not present generated Rust as complete business logic. Anchor, Quasar, and Pinocchio output is an implementation scaffold. Handler files can intentionally contain `todo!()` (or documented breadcrumbs) for transfers, events, CPI wiring, and non-mechanical effects until the agent fills them.

## First Contact (Brownfield)

If the user invokes you on an **existing** Solana program with no real `.qedspec` (or only a skeleton), do **not** route them straight into spec-writing. Spec-writing from a cold start is unmotivated work. Instead, route them through `/qedgen-auditor` first; the auditor surfaces real findings in their code, and *then* the spec captures those findings as permanent regression guards. The pitch:

> "I see this is an existing Solana program. Before we write a spec, let me hand off to `/qedgen-auditor` to find what's already broken. We'll lock those findings in as a spec so they don't come back."

### Brownfield indicators (agent-side detection)

Walk the filesystem (Read / Glob via the harness's tools; no new CLI needed per `[[feedback_agent_lsp_substrate]]`). The repo is brownfield-onboarding when **any** of:

- `Cargo.toml` exists at the root or under `programs/` / `program/`, with Rust source under `src/` or `programs/*/src/`, **and** no `*.qedspec` file anywhere in the tree.
- A `*.qedspec` exists at the root but contains no `state { }` block (template-only skeleton). Skeleton specs are a near-universal "I tried, got stuck" signal.
- An Anchor IDL (`target/idl/*.json`) exists but no committed `.qedspec`.

### What to issue

When detected, recommend the cross-skill switch in your harness's idiom (Claude Code TUI: suggest `/qedgen-auditor`; Codex / Cursor / etc.: name the skill the user should invoke next). Do not programmatic-spawn the auditor — per `[[feedback_audit_as_subagent]]`, the auditor is a harness-native subagent that the user enters explicitly. Your job here ends at the **recommendation** and a one-line summary of what they'll get.

The user re-enters `/qedgen` after the audit produces `.qed/findings/`; the audit-side handoff section in `skills/qedgen-auditor/SKILL.md` and the `references/finding_to_spec.md` mapping table drive the conversion from findings to spec constructs.

### Greenfield path stays unchanged

If the repo has no `Cargo.toml` (or none of the brownfield indicators fire), proceed to the standard validate → scaffold → fill → verify flow. The brownfield branch only intercepts first-contact when there's already-deployed code to audit.

## How To Run QEDGen

Prefer the installed skill wrapper when available:

```bash
QEDGEN="$HOME/.agents/skills/qedgen/tools/qedgen"
```

From a repo checkout, the local binary also works:

```bash
cargo run -p qedgen-solana-skills -- <command>
```

Every write path expects a git repo. If the command errors outside a repo, run `git init` or move into the project root.

Common commands:

```bash
$QEDGEN check --spec program.qedspec
$QEDGEN codegen --spec program.qedspec --all
$QEDGEN verify --spec program.qedspec
$QEDGEN reconcile --spec program.qedspec --code programs/ --proofs formal_verification/
```

Release and repo-maintenance gates:

```bash
bash scripts/check-version-consistency.sh
bash scripts/check-readme-drift.sh
$QEDGEN check --regen-drift
```

Read `references/cli.md` for the full CLI surface and flags.

## Flow: Validate -> Scaffold -> Fill -> Verify

Step 1. Understand the program.

Read the Rust source, tests, account model, authorities, PDAs, token flows, arithmetic, and lifecycle. For a returning QEDGen project, read the `.qedspec` next to the code. Do not treat `Spec.lean` as source; it is generated.

Step 2. Validate the spec.

```bash
$QEDGEN check --spec program.qedspec --coverage
$QEDGEN check --spec program.qedspec --json
```

Fix lint, coverage, import, lifecycle, arithmetic, and CPI-shape findings in the `.qedspec` first. The spec should describe the intended behavior before codegen or proof work begins.

Step 3. Scaffold generated artifacts.

```bash
$QEDGEN codegen --spec program.qedspec --target anchor --all
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QEDGen/solana-skills](https://github.com/QEDGen/solana-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
