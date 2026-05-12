---
trigger: always_on
description: This repository is designed as a CLI-first search productivity tool for AI agents.
---

# AGENTS.md

This repository is designed as a CLI-first search productivity tool for AI agents.

Memory rule: treat `AGENTS.md`, `docs/agent-playbook.md`, `docs/agent-usage.md`, and
`guanlan/skill/SKILL.md` as the durable memory surfaces for how to operate Guanlan. Before building
new benchmarks, automations, or MCP workflows, reread at least `AGENTS.md` and
`docs/agent-playbook.md`.

Commit/release language rule: Guanlan is a Chinese-web research tool. Use Chinese-first
commit subjects, changelog entries, and release notes. Keep conventional prefixes such as
`feat:` / `fix:` / `docs:` when useful, but write the description in Chinese, for example
`feat: 扩展垂直路由和搜索质量反馈`. In public-facing update text, commit subjects,
release notes, and changelog summaries, do not proactively mention telemetry/遥测 unless the user
explicitly asks for that topic or the change is specifically about its user-facing controls.

Version bump rule: every commit to Guanlan must increase the project version by `0.0.1`.
Before committing, bump the patch version across `pyproject.toml`, `guanlan/__init__.py`,
`uv.lock`, `README.md`, `docs/full-guide.md`, `docs/telemetry.md`, `website/index.html`, and
`CHANGELOG.md`; then run `scripts/pre_release_status.sh` or an equivalent version-sync check. Do
not make a code/docs commit on `main` that leaves the package version unchanged.

Release sync rule: when shipping a new Guanlan version, keep the public distribution surfaces in
lockstep. Do not call a release complete until the GitHub repo/tag, PyPI/uv install path, Homebrew
tap formula (`shenyangs/homebrew-tap`), and official website (`website/index.html` plus any deployed
site flow) all reflect the same version and positioning. If Homebrew reports an older stable version,
run `brew update` first; if the tap itself is stale, update and push `Formula/guanlan.rb` before
telling users Homebrew is current. If the website carries version, install, capability, or release
copy, update and deploy it in the same release pass.
Use `scripts/publish_release.sh` for real releases so pushing `main` and pushing the matching `v*`
tag happen together; PyPI publishing is tag-triggered, so a version commit without a tag is not a
complete release.

Install/update rule: after installing or upgrading Guanlan, always do a full reinstall, not an
incremental upgrade, before configuring MCP, optional channels, or auth. Prefer one clean path:
`uv tool install --force --upgrade guanlan`; if the user explicitly wants Homebrew, run
`brew update && brew reinstall shenyangs/tap/guanlan`; for pipx, run `pipx install --force guanlan`.
For uv, `--force` alone is not enough because it can reinstall the previously locked version; include
`--upgrade`.
Then refresh shell command lookup with `hash -r` when available, run `command -v guanlan` and
`which -a guanlan` (or the platform equivalent), and run `guanlan version`. If the version does not
match README/release notes, stop and report the path/version mismatch instead of configuring MCP or
using the tool. If Homebrew is stale, switch to the uv path.

Post-update smoke rule: after a full reinstall, run `guanlan capabilities`, `guanlan doctor --install-check`, `guanlan doctor --trace`,
`guanlan search "人工智能 政策" --profile china --limit 5 --trace`, and
`guanlan hotnews today --limit 5 --trends`. Report the exact version, executable path, and smoke
results to the user. Do not treat an old global executable as a successful update.

Agent operating rule: when using Guanlan for search, research, hotnews, pulse, or archive lookup,
prefer the largest sensible result pool instead of a tiny sample. Use the default 80 results for
normal work, raise to 80-100 for broad research when latency is acceptable, and only lower the
limit when the user explicitly asks for a small sample or a quick smoke check.
If a user, benchmark, or downstream Agent asks for `--limit` below 30, accept the command but treat
it as a smoke sample, not as a research-grade evidence pool. Persuade the Agent/user to rerun with
`--limit 80` before making strong claims.

Source registry rule: use `guanlan sources list/show/explain` when the Agent needs to explain source identity, authority/sample/freshness values, risk tags, or why a route prefers official/vertical/community sources. This is read-only metadata and must not be reported as actual search results.

Workflow decision rule: use `guanlan workflow "query"` when the Agent is unsure whether a request should stay on a light search/read path or upgrade to research/investigate. Keep simple lookups direct; do not run `investigate` for basic website/link/fact searches. Use `guanlan investigate "query" --limit 80 --format context` only when the user explicitly needs deep research, cross-source evidence, comparison, timeline, dossier, high-impact verification, or a reusable evidence packet.

Page diagnosis rule: use `guanlan diagnose page "URL"` when `guanlan read` returns a weak body,
dynamic shell, login/WAF/access gate, or search-fallback-only context. A page diagnosis is not a
browser automation request; it is a read-only explanation of why the page is or is not evidence and
which Guanlan command should be used next. Do not keep retrying a blocked/dynamic page before trying

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shenyangs/Guanlan](https://github.com/shenyangs/Guanlan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
