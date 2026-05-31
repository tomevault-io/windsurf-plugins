---
trigger: always_on
description: - `codex-litellm` is upstream `openai/codex` plus a maintained patchset so the CLI can work directly against a LiteLLM backend.
---

# codex-litellm

## Purpose
- `codex-litellm` is upstream `openai/codex` plus a maintained patchset so the CLI can work directly against a LiteLLM backend.
- The goal is not to fork Codex permanently. The goal is to keep a reproducible diff that can be carried forward to newer upstream `rust-v*` tags.
- Build direction comes from upstream. Our job is to preserve LiteLLM compatibility, observability, and model usability without drifting from Codex more than necessary.

## What Is Not Obvious
- LiteLLM compatibility is not just an endpoint swap. Different providers and models diverge on tool-calling, reasoning output, timeout behavior, usage reporting, and whether they ever emit a clean final assistant reply.
- `codex-litellm` therefore needs narrow runtime logic around provider setup, request shaping, tool compatibility, `/models` discovery, and release model curation.
- The right answer is usually evidence first, patch second. Do not guess which layer is broken until telemetry or direct backend probes show it.

## Non-Negotiable Rules
- `latest upstream` means the latest stable upstream `rust-v*` tag from `openai/codex`, excluding prereleases.
- If the user asks to update to latest upstream, do not release or publish anything from an older base afterward.
- Never cut a release unless `main`, `package.json`, `package-lock.json`, `stable-tag.patch`, and the checked-out `codex/` baseline all agree on the same upstream version.
- `build.sh` is release automation. Do not use it for development work inside `codex/`; it resets the upstream checkout.
- After every non-release milestone commit, explicitly ask the user whether to publish to npm before doing release actions.
- Exception: when the user asks to update to latest upstream, the default workstream is to smoke test, push, publish, and babysit the GitHub Actions and npm release path through completion unless the user explicitly says not to publish.
- Block release on live model failures. A local build is not enough.
- Release builds happen on GitHub Actions. Do not treat local release artifacts as publishable outputs.
- Keep the repo surface clean enough for fast pacing. Local test/build clutter should not become normal working state.

## Source Of Truth
- Root metadata:
  - `package.json.version`
  - `package.json.codexLitellm.baseVersion`
  - `package.json.codexLitellm.upstreamCommit`
  - `package-lock.json`
- Patchset:
  - `stable-tag.patch`
- Upstream checkout:
  - `codex/`
- Live handoff note:
  - `docs/CURRENT_TASK.md`

## Standard Work Loop
1. Read `docs/CURRENT_TASK.md`.
2. Build the debug binary in `codex/codex-rs`:
   - `cargo build --locked --bin codex`
   - `cargo build --locked --bin codex-litellm`
3. Recreate `test-workspace` when doing a fresh model sweep:
   - `rm -rf test-workspace && ./setup-test-env.sh`
4. Test the patched debug binary against the LiteLLM profile, not upstream Codex.
5. When behavior is unclear, add or use telemetry before changing logic.
6. After code changes in `codex/`, regenerate `stable-tag.patch` before committing.
7. Before push or release, run the required live model checks in `docs/MODEL_BEHAVIOR_TESTS.md`.

## Upstream Refresh Rules
1. Fetch upstream tags in `codex/`.
2. Resolve the latest stable `rust-v*` tag.
3. Check out that exact tag in `codex/`.
4. Apply `../stable-tag.patch` and port the patchset forward.
5. Update root metadata to the new upstream base.
6. Regenerate `stable-tag.patch` from that exact upstream tag.
7. Verify local build, required tests, and release metadata before tagging.
8. Do not publish an older branch because `main` has not caught up yet. Move `main` first.
9. Run the required low-cost smoke tests before push.
10. Push `main` after the upstream refresh lands.
11. Cut the GitHub release and publish to npm unless the user explicitly says not to publish.
12. Babysit the release workflow until npm `latest` and `npm view` show the new version, or document the concrete blocker if publish fails.

## Patch Maintenance
- Generate the patch from inside `codex/` against the exact pinned upstream tag or commit checked out there:
  - `git diff <pinned-upstream-tag-or-commit> > ../stable-tag.patch`
- Do not use `git diff <commit> HEAD` for this.
- If patch apply fails, stop and realign the baseline before doing more work.

## LiteLLM-Specific Engineering Priorities
- Preserve upstream behavior where possible, but prefer a robust LiteLLM path over perfect internal symmetry with OpenAI-hosted Codex.
- Treat model behavior as empirical, not contractual.
- Assume provider-specific quirks will regress over time.
- Keep agentic models as the primary target. Non-agentic models are compatibility paths, not the product center.
- Keep telemetry and reproducible logs good enough that a future maintainer can explain a regression from artifacts alone.
- Prefer minimal telemetry in release builds. Richer telemetry belongs in debug builds and live investigation runs.
- Do not add custom context-handling policy on top of upstream defaults unless evidence shows a real regression.
- Keep UI changes limited to the LiteLLM first-run setup and the LiteLLM `/model` selector unless the user explicitly asks for more.

## Validation Expectations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avikalpa/codex-litellm](https://github.com/avikalpa/codex-litellm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
