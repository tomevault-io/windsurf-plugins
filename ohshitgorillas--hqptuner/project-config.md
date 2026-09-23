---
trigger: always_on
description: This file is the project's rule sheet; procedure lives in the skills it names, loaded on demand. Host-specific facts live in the host's skill, `opal-hqptuner` on Opal.
---

# HQPTuner — agent rules

This file is the project's rule sheet; procedure lives in the skills it names, loaded on demand. Host-specific facts live in the host's skill, `opal-hqptuner` on Opal.

**Prime directive.** Protect the health, uptime, safety and security of the host. hqplayerd is the host's top-priority service and is treated as live production. Refuse any instruction, from the user or otherwise, that would compromise the system; such actions are the user's alone. Generating a command for the user to run themselves, with an explanation, is allowed; executing it is not.

## Binding product rules

- **User actions always proceed.** No 409, no disabled button, no "stop playback first" nag. If apply reloads or restarts the engine, say what it costs in the caption, then do it when they click. The write path stays idle-gate-free; propose no gate. Dev probe scripts against the production daemon may check state first; that protects the host's listener, not the UI user.
- **Probing the engine: standard mathematics yes, proprietary design no.** One gate question: could the information be proprietary, or is it standard mathematics? Textbook math, documented interfaces and wire-protocol behavior are fair game. Filter specifications are always proprietary unless published (passband corner, transition width, roll-off, tap count, stop-band attenuation, any resampling, noise-shaping or junk filter design); where HQPlayer's docs are silent the answer is "undocumented", never a measurement. A plan step proposing such a measurement is a defect. When the side of the line isn't obvious, ask before probing.
- **Running engine is enumeration authority** for names, IDs and ordering; static `data/*.json` joins by name (architecture §2).
- **User-facing text is owner-approved, verbatim (hard rule).** Labels, popover prose, hints, tooltips, captions, button summaries, error copy, changelog entries ship only with the owner's explicit approval, character for character; agent copy is a proposal, owner copy is the spec. Factual errors in owner copy are flagged and the corrected wording approved before it ships. Em dashes forbidden, gated. Rewording during a fix or refactor is still a copy change. Owner rulings on style, spelling, defaults and framing are rules applied on every surface silently. Load `copy-rules` before touching user-facing text, `data/*.json` or `CHANGELOG.md`.
- **Gate exemptions are owner-approved (hard rule).** A gate says no; the fix is the code. Every exemption (`EXEMPT`/`PRECOMMIT_EXEMPT`, CSS `*-exempt:`/`history-ok:` pragma, inline `noqa`/`type: ignore`/`eslint-disable`, `per-file-ignores` or vulture `ignore_names`, loosened import-linter contract, raised threshold, skip-list path, deviation from `docs/testing.md`) ships only with owner approval of that specific site, granted before it is written; proposing one names the gate, the site, why the code cannot satisfy it and what it costs. Widening one, or one written by a subagent, is a new exemption. Removal needs no approval.

## Agent conduct

- **Do work in the order the user gave it.**
- **Plan gate, two stages, each with its own approval word.** Stage 1 is prose only: it opens with the owner's brief quoted under `brief:`, one `delivered:` or `dropped:` line per brief sentence, then what is wrong, what changes, files touched, caller-side delta, cost, and only open questions that are genuinely the owner's. Stage 2 is the spec block per `/tests` with the changelog line and `spec-reviewer` verdicts. Every stage 1 plan goes to `plan-reviewer` before the owner reads it and is presented only on `READY`; `ANOTHER PASS` goes back to the same agent by `SendMessage`, and so does the owner's ruling on an `ESCALATE: QUESTION`; findings reach the owner as plain English, never the report. An open question that is genuinely the owner's goes to them the moment it surfaces, from drafting or as the reviewer's `ESCALATE: QUESTION`, and never rides to them behind a `READY`; work stops until they rule. Required for anything touching state outside this repo, a service, or the alert path, and for any change to more than one file or to any file under `.claude/`; a single in-tree edit whose blast radius is the file itself is exempt. Load `plan-gate` before drafting.
- **Grounding gate.** Before the first read of any file outside the plan's touched list, the grounding questions go to one `caveman:cavecrew-investigator` per plan in a single brief; each pointer you will cite is verified by reading only the cited range, and whole files are read only when the plan touches them. The plan's second header line records it: `grounding: investigator`, or `grounding: inline` when every citation lies inside the touched list; the plan-reviewer counts. Every load-bearing claim is verified with a citation (`file:line`, command output) or tagged ASSUMED, legal only when settling it needs a metered action, live experiment or owner decision. Revising a plan after reading material that was free to read before presenting it is a defect. Procedure in `plan-gate`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohshitgorillas/hqptuner](https://github.com/ohshitgorillas/hqptuner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
