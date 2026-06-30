---
trigger: always_on
description: - **Always squash-and-merge** when merging PRs. Use `gh pr merge --squash`.
---

# Donner Project Instructions

## Pull Requests

- **Always squash-and-merge** when merging PRs. Use `gh pr merge --squash`.
- Never use merge commits or rebase-and-merge on this repository.

## AI Comment Convention

- **Always prefix AI-generated GitHub comments with 🤖.** This applies to all PR comments, review comments, and issue comments posted by any AI agent (Claude, Codex, Copilot, etc.).
- This distinguishes human comments from AI comments, since all AI activity goes through `jwmcglynn`'s GitHub account.

## Always-Green Main

- **`main` is always green.** There is no such thing as a "preexisting test failure" — any red test blocks merge, full stop. If something on `main` breaks, the next PR is fixing it, not routing around it.
- **Run `bazel test //...` before pushing any PR.** This is the single source of truth for local validation. Our goal is that `bazel test //...` catches every regression that CI would — if CI catches something local didn't, that's a gap to fix in the test surface, not a reason to skip the local check.
- **If `main.yml`'s bazel-diff target determinator looks wrong on a PR, add the `ci:full-test` label** to force the workflow back to full `bazel test //...` coverage for that PR.
- **When touching the CMake mirror or `gen_cmakelists.py`, also run `python3 tools/cmake/gen_cmakelists.py --check --build`.** Plain `--check` is intentionally fast and static; `--build` is the opt-in local compile gate that catches real CMake drift before CI does.
- The `tiny`, `text-full`, and `geode` variant lanes now run as `*_tiny` / `*_text_full` / `*_geode` wrappers under default `bazel test //...` (see `donner_cc_test(variants=…)` in `build_defs/rules.bzl`). The transitional `tools/presubmit.sh` wrapper has been retired — `bazel test //...` is the single command that gates a PR.

## Transform Naming

- **Every `Transform2d` local, field, parameter, and struct member must be named in `destFromSource` form** — e.g., `bitmapEntityFromEntity`, `worldFromPreviousWorld`, `canvasFromDocument`. Names like `delta`, `xform`, `t`, `transform`, `mat` are banned: the destFromSource name *is* the documentation, and a value whose direction is encoded only in a comment will be composed wrong the first time it's reused. See `AGENTS.md` §"Transform Naming Convention" for the full rule and rationale.

## Formatting

- **Run `clang-format -i` on every modified C/C++ file before committing.** `git clang-format` covers staged changes. The project `.clang-format` (Google + 100-col, see `.clang-format`) is tuned so clang-format 18 and 19 produce identical output — use whichever is on your `$PATH`.

## Debugging Discipline

When debugging bugs — **especially performance or UI bugs** — write an automated test that reproduces the bug BEFORE attempting a fix. No fixes without repros.

- **A regression test is only valid if it FAILED on the broken code.** Run the test at HEAD *before* applying the fix, capture the failure output (diff PNG, pixel count, error) and verify it matches the user-reported symptom. Commit the test on its own commit first so CI records a red→green transition. If you can't get the test to fail at HEAD, the test is wrong — not the bug. "Plausible-sounding fix without a red→green transition" is an attempt, not a fix — title the commit `attempt:` / `hypothesis:` and do not mark the issue closed.
- **User pushback is automatic evidence the test was wrong.** When the user reports a bug is still present after a claimed fix, the default response is "the test that verifies my fix is wrong or missing — writing a new one." Never reply with "I don't see why it would still be broken" or ask the user to re-confirm steps. The user's repro *is* the signal; your test is what needs debugging.
- **Perf bugs**: the repro must measure the exact latency the user observes (e.g. click-to-first-pixel wall-clock, per-frame time). Put explicit budget assertions in the test (`EXPECT_LT(measured_ms, budget_ms)`) so regressions trip loudly. New perf tests should use `donner_perf_cc_test` so CPU-invariant correctness counters stay on the PR gate while runner-sensitive wall-clock budgets move to nightly `perf` targets. Don't settle for "works on my laptop" — the test itself is the verification.
- **UI bugs**: if the bug only manifests through the full editor event loop (mouse events, ImGui state, worker-thread ping-pong), write an instrumented UI-layer test that drives the live backend path (`EditorBackendCore` + `CompositorController`) with the exact request-posting sequence the editor uses. Faithfully mirror the event flow — do not fabricate a prewarm phase that the real editor doesn't fire.
- **Editor visual bugs**: follow [`docs/editor_visual_debugging.md`](docs/editor_visual_debugging.md) for `.rnr` replay, Geode direct-texture diagnostics, pixel crops, stack-layer boundaries, and failure signatures.
- **Editor path overlays must stay lockstep with presented document pixels.** A drag/zoom frame is
  wrong if the overlay uses a transform different from the shape pixels underneath it; preserve the
  same presented transform for both, or move both together.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwmcglynn/donner](https://github.com/jwmcglynn/donner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
