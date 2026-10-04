---
trigger: always_on
description: Primary briefing for coding agents. Read this before copying any workflow.
---

# AGENTS.md — adopt these CI templates in a consumer repo

Primary briefing for coding agents. Read this before copying any workflow.
Humans: [README.md](./README.md) and [docs/adopt-checklist.md](./docs/adopt-checklist.md).

## 1. Purpose & how to use this repo

This repository holds **sanitized GitHub Actions workflow templates** and
short ops docs. It is not CI for a product app.

Your job when adopting:

1. Copy selected files from `workflows/` into
   `YOUR_ORG/YOUR_REPO/.github/workflows/` (drop the `.example` suffix only
   when the consumer project has the matching scripts and secrets).
2. Replace every placeholder: project name, cache paths, npm/cargo scripts,
   path filters, labels, runner labels, model ids, and doc links.
3. Wire secrets/vars (names in §3; details in [docs/secrets.md](./docs/secrets.md)).
4. Confirm self-hosted macOS expectations (§4) before marking `check` required.
5. Do **not** assume the consumer monorepo layout matches any example path
   in comments (`apps/…`, `scripts/…`, etc.). Those are illustrations.

If a template step references a script that does not exist in the consumer
repo, delete or rewrite the step — do not invent a no-op that always passes.

## 2. Workflow catalog — when to adopt each

| Adopt when… | Template | Priority |
|---|---|---|
| You need a hard verify gate on Apple silicon / macOS-native tooling | `workflows/check.yml` | Core |
| You have no self-hosted runner and need a hosted verify gate | `workflows/check-hosted.yml` | Core (hosted) |
| The app has React/TSX and you want PR-diff issue comments without blocking day one. Optional escalate job past a new-issue threshold | `workflows/react-doctor.yml` | Core if React |
| You want automated Cursor CLI review + optional autofix on same-repo PRs | `workflows/cursor-review.yml` | Optional, cost-aware |
| You want agent review with zero push risk (start here, graduate to `cursor-review.yml`) | `workflows/cursor-review-readonly.yml` | Optional, cost-aware |
| Jobs run on a small self-hosted pool and can sit `queued` with no `timeout-minutes` signal | `workflows/queue-stall-alarm.yml` | Strongly recommended with self-hosted |
| Media/toolchain behavior depends on an ffmpeg major-version floor you must prove on Linux | `workflows/ffmpeg-floor.yml.example` | Only if you have a floor matrix script |
| Maintainers want `@droid` in issues/PRs to run Factory Droid | `workflows/droid.yml.example` | Optional |

Adoption order that usually works: `check` → `queue-stall-alarm` (if
self-hosted) → `react-doctor` → `cursor-review` → optional examples.
When you have no self-hosted runner, start with `check-hosted` instead of
`check`. Prefer `cursor-review-readonly` over `cursor-review` until you
need autofix.

## 3. Required secrets / vars (names only)

Configure in the **consumer** repo (or org). Full notes:
[docs/secrets.md](./docs/secrets.md).

### Almost always

| Name | Kind | Used by |
|---|---|---|
| `GITHUB_TOKEN` | Automatic | All workflows (default permissions still matter) |

### Cursor review pipeline

| Name | Kind | Used by |
|---|---|---|
| `CURSOR_API_KEY` | Secret | `cursor-review.yml` |
| `CURSOR_PUSH_TOKEN` | Secret (fine-grained PAT, contents:write on same repo) | Autofix push that must retrigger workflows |
| `CURSOR_REVIEW_MODEL` | Secret **or** Variable (optional override) | Review agent model id |
| `CURSOR_AUTOFIX_MODEL` | Secret **or** Variable (optional override) | Autofix agent model id |

### Readonly review pipeline (no push)

`CURSOR_PUSH_TOKEN` is not needed for `cursor-review-readonly.yml`.

| Name | Kind | Used by |
|---|---|---|
| `CURSOR_AGENT_REVIEWS` | Variable (kill switch) | Every agent job |
| `REACT_DOCTOR_ESCALATION_THRESHOLD` | Variable (optional, default 10) | `react-doctor.yml` escalate |
| `CURSOR_ESCALATION_MODEL` | Variable (optional) | `react-doctor.yml` escalate |

### Factory Droid

| Name | Kind | Used by |
|---|---|---|
| `FACTORY_API_KEY` | Secret | `droid.yml.example` |

### Project-specific (only if your `check` needs them)

| Name | Kind | Notes |
|---|---|---|
| Registry / cloud tokens | Secret | Only if install or E2E hits private registries |
| `VERIFY_SKIP_E2E` | Env set **in the workflow**, not a repo secret | Pattern: skip heavy E2E on `push`, run on PR/nightly |

Never commit key material. Never put PATs in `vars.*`.

## 4. Self-hosted macOS runner expectations (generic)

Details: [docs/self-hosted-macos-runner.md](./docs/self-hosted-macos-runner.md).

Agents configuring a consumer repo should assume:

- Runner labels match the YAML, typically `[self-hosted, macOS]`. Add extra
  labels only if every machine that should take the job has them.
- Toolchain on `PATH` for non-interactive jobs: Node (version your lockfile
  needs), Rust/cargo if applicable, Homebrew tools your gate asserts
  (e.g. `ffmpeg` / `ffprobe`), Xcode CLT or full Xcode if Swift/iOS steps exist.
- **Persistent cargo/target cache lives outside the checkout workspace**, e.g.
  `$HOME/.cache/<project>-cargo-target-$RUNNER_NAME`.
  `actions/checkout` clean/reset must not delete it.
- **One cache directory per `RUNNER_NAME`.** Concurrent runners must not share
  a target dir if any job can `rm -rf` the cache (wipe races).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RayFernando1337/ray-fernando-actions-templates](https://github.com/RayFernando1337/ray-fernando-actions-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
