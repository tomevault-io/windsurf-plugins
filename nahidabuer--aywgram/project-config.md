---
trigger: always_on
description: This guide defines repository-wide instructions for coding agents working with the Telegram Desktop codebase.
---

# Agent Guide for Telegram Desktop

This guide defines repository-wide instructions for coding agents working with the Telegram Desktop codebase.

## Fork Lineage and Upstream Integration


```text
AywGram Desktop
  -> AyuGram Desktop
    -> Telegram Desktop
```

Preserve the distribution-specific patches and the existing AyuGram feature set as the
product baseline. Upstream integration should move the fork closer to AyuGram
and Telegram Desktop only where the incoming changes are useful and compatible
with that baseline; upstream parity by itself is not a reason to remove or
weaken fork-specific behavior.

For recurring upstream maintenance:

- Use the repository-owned `.agents/skills/ayugram-upstream-sync` workflow for
  recurring audits and integrations, and preserve it when resolving upstream
  agent-workflow conflicts. Continue to ignore unrelated skills or agent
  pipelines introduced by upstream.
- Start from a clean worktree and fetch the relevant remotes before comparing
  commits. In this checkout, `ayugram` is the public AyuGram source, `telegram`
  is the official Telegram Desktop source, and `origin` is the distribution
  fork's publishing remote. Verify their URLs and record the fetched branch
  tips in each maintenance report
- Review AyuGram and Telegram Desktop separately. Select changes based on the
  requested product needs, security and correctness fixes, compatibility, and
  the value of reducing future divergence. Do not bulk-merge an upstream branch
  without first understanding its unique commits and affected files.
- Integrate on a dedicated branch in stages: distribution patches first, then the
  selected AyuGram changes, then the selected Telegram Desktop changes and
  follow-up compatibility fixes. Adjust the order when commit ancestry requires
  it, and explain the reason in the plan.
- Before resolving conflicts, identify the intent on all sides. Preserve the
  current distribution and AyuGram behavior while adapting it to upstream APIs and
  structural changes. Never resolve a conflict by blindly choosing one side for
  a file containing fork-specific features.
- Treat submodule pointers, generated sources, API schemas, settings
  serialization, styles, and localization as explicit integration risks. Keep
  related main-repository and submodule changes aligned, and report any
  referenced submodule commit that is unavailable from its configured remote.
- After each integration stage, inspect the resulting diff for lost AyuGram or
  distribution features and run focused checks appropriate to the touched areas.
  Follow the repository build restrictions in this guide; do not build unless
  the user explicitly overrides the repository-wide no-build instruction.
- Deliver a maintenance summary containing the compared commit ranges, grouped
  incoming changes, selected and deferred patches with reasons, conflicts and
  resolutions, feature-preservation checks, unresolved risks, and the proposed
  next integration baseline.

If you're asked to create a Pull Request, then clearly state in PR description that it was AI generated.

## Build and Platform Instructions

Avoid building the project unless the user explicitly authorizes a build.
Build Release when no configuration is named; build Debug only when explicitly
requested or after approval for a stated diagnostic need. Before configuring,
building, packaging, or troubleshooting, read only the guide for the target
platform: [Windows](docs/agent-building-windows.md),
[macOS](docs/agent-building-macos.md), or
[Linux](docs/agent-building-linux.md).
Keep all generated output under the repository-root `out/` directory.
Treat a successful build as compilation evidence, not UI or release approval.

## Text File Format

- On Windows, keep project text files with CRLF line endings.
- Do not save source, header, build/config, style, or localization files as UTF-8 with BOM. Use UTF-8 without BOM.
- When rewriting project text files for normalization, preserve file content otherwise and do not introduce a BOM.
- Never hard-wrap Markdown prose or list items to an 80-column limit. Keep each paragraph or list item on one line unless Markdown syntax requires a deliberate line break.

## Commits

- Follow the Conventional Commits format:

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- Subject: one concise, plain-language line summarizing the change, ~50-60 characters, matching the style of recent `git log` subjects. This is usually the entire message.
- Add a short plain-language body only when the subject can't carry it (what was done, not the technical how) — a line or two at most.
- Never add a `Co-Authored-By:` line or any tool/assistant attribution trailer.
- Never add `Autotask:`/attempt or other workflow markers — commits read like normal history.

## Local Storage Serialization

Both app-level (`Core::Settings`) and session-level (`Main::SessionSettings`) use sequential binary serialization via `QDataStream`. Key rules:

- New fields must ALWAYS be appended at the **end** of the stream, never inserted in the middle
- Reading new fields must be guarded with `!stream.atEnd()` and provide a meaningful default/fallback

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NahidaBuer/Aywgram](https://github.com/NahidaBuer/Aywgram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
