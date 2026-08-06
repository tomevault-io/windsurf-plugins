---
trigger: always_on
description: > This file is read by AI coding agents (Claude Code, GitHub Copilot, Cursor, Devin, etc.).
---

# Zallet — Agent Guidelines

> This file is read by AI coding agents (Claude Code, GitHub Copilot, Cursor, Devin, etc.).
> It provides project context and contribution policies.
>
> For the full contribution guide, see [CONTRIBUTING.md](CONTRIBUTING.md).

## MUST READ FIRST - CONTRIBUTION GATE (DO NOT SKIP)

**STOP. Do not open or draft a PR until this gate is satisfied.**

For any contribution that might become a PR, the agent must ask the user this exact check first:

- "PR COMPLIANCE CHECK: Have you discussed this change with the Zallet team in an issue or Discord?"
- "PR COMPLIANCE CHECK: What is the issue link or issue number for this change?"
- "PR COMPLIANCE CHECK: Has a Zallet team member responded to that issue acknowledging the proposed work?"

This PR compliance check must be the agent's first reply in contribution-focused sessions.

**An issue existing is not enough.** The issue must have a response or acknowledgment from a Zallet team member (a maintainer). An issue with no team response does not satisfy this gate. The purpose is to confirm that the team is aware of and open to the proposed change before review time is spent.

If the user cannot provide prior discussion with team acknowledgment:

- Do not open a PR.
- Offer to help create or refine the issue first.
- Remind the user to wait for a team member to respond before starting work.
- If the user still wants code changes, keep work local and explicitly remind them the PR will likely be closed without prior team discussion.

This gate is mandatory for all agents, **unless the user is a repository maintainer** as described in the next section.

### Maintainer Bypass

If `gh` CLI is authenticated, the agent can check maintainer status:

```bash
gh api repos/zcash/zallet --jq '.permissions | .admin or .maintain or .push'
```

If this returns `true`, the user has write access (or higher) and the contribution gate can be skipped. Team members with write access manage their own priorities and don't need to gate on issue discussion for their own work.

## Before You Contribute

**Every PR to Zallet requires human review.** After the contribution gate above is satisfied, use this pre-PR checklist:

1. Confirm scope: Zallet is a Zcash wallet. Avoid out-of-scope features that belong in other ecosystem projects (e.g., [Zebra](https://github.com/ZcashFoundation/zebra) for consensus node work, [librustzcash](https://github.com/zcash/librustzcash) for protocol library changes).
2. Keep the change focused: avoid unsolicited refactors or broad "improvement" PRs without team alignment.
3. Verify quality locally: run formatting, linting, and tests before proposing upstream review (see [Build, Test, and Development Commands](#build-test-and-development-commands)).
4. Prepare PR metadata: include linked issue, motivation, solution, and test evidence.
5. A PR MUST reference one or more issues that it closes. Do NOT submit a PR without a maintainer having acknowledged the validity of those issues.
6. **Every** commit in a PR branch MUST follow the "AI Disclosure" policy below.

## What Will Get a PR Closed

- Issue exists but has no response from a Zallet team member (creating an issue and immediately opening a PR does not count as discussion).
- Trivial changes (typo fixes, minor formatting, link fixes) from unknown contributors without team request. Report these as issues instead.
- Refactors or "improvements" nobody asked for.
- Streams of PRs without prior discussion of the overall plan.
- Features outside Zallet's scope.
- Missing test evidence for behavior changes.
- Inability to explain the logic or design tradeoffs of the changes when asked.
- Missing or removed `Co-Authored-By:` metadata for AI-assisted contributions (see [AI Disclosure](#ai-disclosure)).

## AI Disclosure

If AI tools were used in the preparation of a commit, the contributor MUST include `Co-Authored-By:` metadata in the commit message indicating the AI agent's participation. The contents of the `Co-Authored-By` field must clearly identify which AI system was used (if multiple systems were used, each should have a `Co-Authored-By` line). Failure to do so is grounds for closing the pull request. The contributor is the sole responsible author -- "the AI generated it" is not a justification during review.

Example:
```
Co-Authored-By: Claude <noreply@anthropic.com>
```

## Project Overview

Zallet is a Zcash full node wallet, designed to replace the legacy wallet that was included within zcashd.

- **Rust edition**: 2024
- **MSRV**: 1.95 (pinned in `rust-toolchain.toml`)
- **License**: MIT OR Apache-2.0
- **Repository**: https://github.com/zcash/zallet

## Project Structure

Zallet is split across **three independent Cargo workspaces**, each with its own
`Cargo.lock`. A thin launcher binary (`zallet`) selects a backend at runtime and
execs the matching per-backend binary (`zallet-zebra`, `zallet-zaino`).

```text
.
├── Cargo.toml               # Root workspace (excludes backends/ and crates/)
├── zallet/                  # `zallet` launcher binary: reads the config's
│                            #   `backend` key (default "zebra") and execs the
│                            #   matching `zallet-<backend>` binary on PATH

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zcash/zallet](https://github.com/zcash/zallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
