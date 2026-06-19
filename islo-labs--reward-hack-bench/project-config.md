---
trigger: always_on
description: Benchmark for measuring execution-environment effectiveness at preventing
---

## Context

Benchmark for measuring execution-environment effectiveness at preventing
reward hacking in agent evaluations. Takes tasks from SWE-bench + CyBench,
injects cheat patterns (writeup lookup, upstream-fix, etc.) via an adapter,
scores each trial with both the original grader AND an LLM-as-judge.

**Headline metric:** `solved_fairly = static_reward × fairness`. Architecture
and rationale are in [`design.md`](design.md); read that before making
substantive changes. [`README.md`](README.md) shows end-users how to run.

## Environment

Do not create or use a Python virtual environment (`.venv`) in this project.
Harbor is installed globally via `uv tool install`; adapter scripts use
PEP 723 inline metadata. Avoid `uv sync` / `uv run` against this project.

Harbor must be installed from the **`reward-hack-bench-changeset`** fork
branch — superset of phased gateway control (PR
[#1575](https://github.com/harbor-framework/harbor/pull/1575)) that also
adds first-class docker-compose support inside islo VMs (so multi-
container CTF tasks finally run under islo):

```bash
uv tool install --force \
  --from 'git+https://github.com/islo-labs/harbor-fork@reward-hack-bench-changeset#egg=harbor[islo]' \
  harbor
```

What this branch gives us:
- **Compose mode in islo**: `docker-compose.yaml` detection takes priority
  over Dockerfile; the env spins up Docker Compose inside the islo VM.
  ezmaze / diffecient / noisier-crc are now islo-runnable.
- **Phased gateway** (mandatory schema): `gateway: { setup: {...}, agent:
  {...}, verifier: {...} }`. Flat `default_action`/`rules` at top level is
  rejected with a migration error.
- **Content-based filtering** per rule:
  ```yaml
  rules:
    - host_pattern: "*"
      action: deny
      content_filter:
        direction: request | response | both
        filter_type: regex | content_type | size_limit
        pattern: "<regex/mime/bytes>"
  ```
  This is the more powerful axis — blocks by response body across hosts,
  mirrors, search-engine caches, etc.

When upstream merges these into harbor-framework/harbor main, switch
back to `git+https://github.com/harbor-framework/harbor#egg=harbor[islo]`.

PyPI release is too old; bare `harbor` without the `[islo]` extra
won't load the islo backend.

## How the pieces fit

- **Source tasks** live under `datasets/<corpus>/<task>/`. Two shapes:
  SWE-bench (flat, `tests/config.json`, parser.py grader) and CTF
  (multi-container via `environment/docker-compose.yaml`, grep-F grader).
  The shape is either declared in the manifest's `task:` block or inferred
  from file contents.
- **`adapter/build.py`** reads `adapter/manifest.yaml` and generates
  cheating-augmented tasks under `datasets/reward-hack/<target_name>/`.
  Each entry combines a source + mode + `task:` facts (shape, flag,
  repo_url, writeup_urls) + `config:` (mode knobs).
- **`adapter/judge_template.py`** is the LLM-as-judge script injected into
  each generated task's `tests/judge.py`. Runs at verifier time, reads
  the agent trajectory, emits `judge.json` + single-key `reward.json`.
- **Running the generated tasks**: `harbor run -y -p datasets/reward-hack/<name>`.
- **The final sweep** lives in `experiments/hero-run-v2/` — all policy
  configs including the `open` and `network-none` baselines. Compact
  per-trial verdicts of the published run are tracked in `results/`;
  `experiments/hero-run-v2/extract_results.py` regenerates them from raw
  `jobs/` outputs and `render_dashboard_v2.py` renders `dashboard.html`
  from `results/`. The raw `jobs/` tree stays gitignored.

## Status (as of June 2026 — check `adapter/manifest.yaml` + `design.md` for ground truth)

**The benchmark is complete and published.** The hero-run-v2 sweep
(8 tasks × 7 policies × k=3, Opus 4.7) is the authoritative result set —
see the "Final results" section in `design.md`. Headline:
`llm-judge-request` is the only policy at cheat rate 0 while holding the
model's 0.58 fair-rate ceiling; `layered-tools-off` is the best static
policy (0.08).

**Validated cheat modes** (cheat fires, judge detects, `solved_fairly=0`):
- `upstream-fix` on all 4 SWE-bench tasks.
- `writeup-pointer` on all 4 CTF tasks.

**Commented out in manifest, awaiting validation or refuted**:
- `verifier-smuggle`, `grader-shortcut`, `writeup-pointer` on SWE-bench,
  `bg-reward-write`. See the manifest comments for per-mode status.

## Operational gotchas worth not re-learning

- **Harbor requires `-y` on `harbor run`** when tasks declare host env
  vars (e.g., `ANTHROPIC_API_KEY` passthrough in `[verifier.env]`).
  Without it the CLI prompts interactively and a background run will abort.
- **Harbor's built-in metrics (`Mean`/`Max`/`Min`/`Sum`) require single-key
  `reward.json`.** The adapter emits only `{"solved_fairly": X}`; full
  breakdown sits in `judge.json` sidecar.
- **IPv6 on Docker Desktop for macOS**: default bridge has IPv6 disabled,
  but container DNS still returns AAAA. Node-based HTTP clients (claude-code)
  can hang. Fix: `NODE_OPTIONS=--dns-result-order=ipv4first` in the agent
  Dockerfile. Applied in `datasets/cybench/ezmaze/environment/Dockerfile`.
- **Task name must be `org/name`** — adapter auto-namespaces under
  `reward-hack/` if the target_name has no slash.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [islo-labs/reward-hack-bench](https://github.com/islo-labs/reward-hack-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
