---
trigger: always_on
description: This file is for agents working on **this repo**. Do not load the whole
---

# AGENTS.md

This file is for agents working on **this repo**. Do not load the whole
`ref/` tree. Start here, then open only the path listed for the task.

Human-facing docs stay in `README.md` / `README.zh-CN.md`. This file is
the project memory: why the Pro profile exists, why the Flash profile
was removed, where the measured sources live, and what not to break.

## What this package is

A [Pi](https://pi.dev) extension. It remaps Pi's prompt/tool surface
when the current model is DeepSeek V4 Pro, the same way
`pi-codex-conversion` remaps Pi onto Codex tools when the model is GPT.

| Model | Profile in this repo | Intent |
| --- | --- | --- |
| **V4 Pro** | anchored-standard | Do not let Standard's rich tool catalog derail request #1. |
| **V4 Flash** | anchored-standard (v0.3.1 default) | Same bootstrap; DeepSWE A/B measured 9/10 vs 6/10 over Pi's stock surface. |

The npm name `pi-dsh-minimal` is historical (v0.1.0 was permanent
official-minimal). v0.2.0 added a Flash weak-routing profile; v0.3.0
removed it after a controlled A/B found no lift; v0.3.1 put flash on
the Pro anchored-standard path by default (`DEFAULT_MODEL_PATTERNS`
contains both models) after that path measured +30pp on flash.

Default triggers: `deepseek-v4-pro` and `deepseek-v4-flash` → Pro
profile. `useOnAllModels` sends unknown models through Pro. Everything
else is inactive — untouched Pi surface.

## Background

DeepSeek V4 Pro's model card evaluates code-agent tasks on DeepSeek
Harness **minimal** (极简模式). Community measurements show V4 Pro
overfits the **API-visible first-request surface**:

- complete system prompt = `You are a helpful software engineer assistant.`
- tool catalog = persistent `bash` + `str_replace_editor` (official schemas)

On that surface the first thinking line is typically **We need…** /
**I need…**. On a Standard-family catalog it falls into **Let me…**.
Once request #1 has committed, later catalog expansion does not flip
the trajectory. So Pro's job is: **don't let the first step go down
the wrong track**.

V4 Flash was a different hypothesis, and it failed score-level
validation. Flash is much less sensitive to tool schema (even Minimal
persona + full Standard tools stays minimal-like) and upstream probes
showed persona/guidance sensitivity (routing %, reasoning depth,
convergence) — but those were micro-task probes with fixture tools.
Two findings killed the Flash path (v0.3.0 removal):

1. **Our controlled DeepSWE A/B** (2026-08-16; scripts under
   `scripts/deepswe/`, data in `runs/deepswe/RESULTS.md`): 10 official
   tasks × extension on/off, same pi agent + opencode-go flash,
   official verifier. 4/11 vs 5/10 solved, p=1.0. Activation verified
   per run; failures were ordinary wrong-implementations. No signal.
2. **Upstream P21**: near-field guidance is *negative* on related-task
   chains (46% vs 63% route) — real SWE sessions are related chains.
   Upstream's own P2/P9 note simple tasks saturate; hard-task
   score-level validation was never done anywhere.

Baseline sanity from the same A/B: pi + flash without the extension
solved 50% of the subset vs the official DeepSWE v1.1 leaderboard's
53% — so the harness was measuring at the right absolute level.

**2026-08-16 addendum — the Pro minimal path DOES lift Flash.**
A follow-up A/B (same harness; `runs/deepswe/RESULTS-flash-propath.md`)
forced flash through the Pro anchored-standard profile
(`useOnAllModels: true`): **9/10 solved (mean partial 1.000) vs 6/10
baseline** (+30pp, discordant pairs 3-0, McNemar p=0.25; every plugin
solve was full-score). etree / fd / ts-pattern were first-ever flash
solves. Unlike the removed weak-routing path, the Pro bootstrap has no
near-field guidance text — the P21 negative mechanism does not apply.
Open caveat: single sample, 10 tasks; replicate before hard claims.
This contradicts the older "anchored Standard does not raise Flash
score" row below (that came from Project2-style measurements).

Remember:

| | **V4 Pro** | **V4 Flash** |
| --- | --- | --- |
| First-turn tool-schema anchor | Decisive | Positive signal on DeepSWE (9/10 vs 6/10, see addendum above; replicate before trusting) |
| Anchored Standard | Strongest evidence (Project2 ~91 → 98/99) | DeepSWE: +30pp single-run signal (2026-08-16); older Project2-style data said no |
| Persona / weak self-routing | Useful but unstable | Removed in v0.3.0 — no DeepSWE lift, negative on related chains |
| Proven task lift | 91 → 98/99 | Pro path on flash: 9/10 vs 6/10 (unreplicated); weak routing: none |

**Do not re-add a Flash persona/guidance path without new score-level
evidence on a real benchmark** (probe-level route % / convergence is
not enough — that is exactly the mistake v0.2.0 made).

## Progressive disclosure — refs

`ref/` is a **local, gitignored** checkout. It is not published. If a
path is missing, clone it; do not vendor the monorepo into git.

Open **one** row. Do not browse `ref/deepseek-harness` unless the
task is official schema/string identity.

| When you are changing… | Open first | Then, only if needed |
| --- | --- | --- |
| Anything | this file + `README.md` | `CHANGELOG.md`, `NOTICE` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Averyyy/pi-dsh-minimal](https://github.com/Averyyy/pi-dsh-minimal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
