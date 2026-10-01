---
trigger: always_on
description: FlightDVR is a desktop app for browsing, previewing, trimming and converting
---

# Working on FlightDVR Studio

FlightDVR is a desktop app for browsing, previewing, trimming and converting
HDZero goggle DVR footage. `README.md` covers use; `docs/DEVELOPMENT.md` covers
technical design and build reasoning; `docs/ROADMAP.md` covers planned and
deliberately omitted work.

This root file is the concise operating agreement and document router. Use the
smallest relevant document for the assigned floor:

- [docs/WORKFLOW.md](docs/WORKFLOW.md) — claiming, identity, publication,
  review, evidence, checkpoints and merge authority.
- [docs/AGENT_LESSONS.md](docs/AGENT_LESSONS.md) — technical incident lessons
  and the evidence that can settle recurring traps.
- [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) — source layout, measured design
  rationale, platform/build details and longer technical explanations.
- [CLAUDE.md](CLAUDE.md) — thin entry point for agents that discover this file.

Anything not written in the repository, an issue, or the job/PR evidence did
not happen. Chat coordinates work; Git and the issue tracker preserve it.

## Non-negotiable boundaries

- Nk owns merge authority. Nk may explicitly delegate a routine coordinator
  merge; workers never merge, and no delegation bypasses the independent
  verdict or applicable CI.
- One agent owns a branch at a time. The maker never owns the reciprocal review.
  Every floor names one maker, one independent verdict owner, exact Git objects,
  paths, checks, done criteria and remaining limits.
- Do not mutate user footage, settings, accounts, credentials, security
  boundaries, devices or worker lifecycle without an explicit floor. Preserve
  existing worktrees, branches, drafts and user exploration.
- Do not copy keys or tokens, persist short-lived credentials, weaken ACLs,
  relaunch recovered workers, use Open All, or infer permanent ownership repair
  from a reconnect.
- Do not import `QtMultimedia`, remove the preview's bounded frame queue, or
  call `stop_process` from the UI thread. Use the existing worker-owned stop
  boundary described in [AGENT_LESSONS.md](docs/AGENT_LESSONS.md). The one
  exception is `flightdvr/audio_device.py`, which may take `QAudioFormat`,
  `QAudioSink` and `QMediaDevices` to hand already-decoded PCM to an output
  device. Players, decoders and capture stay forbidden everywhere, FFmpeg
  keeps all decoding, and `tests/test_player.py` holds that line.
- Do not change default colour handling or ffmpeg arguments without the
  measurements and applicable platform/package evidence described in the
  development notes.
- No release publication, broad configuration migration, new credentials,
  global workflow relaxation or reserved product decision is implied by a
  coding, review, connection or documentation floor.

## Public coordination text

Treat public text as durable external evidence: pull-request descriptions,
reviews and inline comments, issue comments, issues, commit messages and any
generated attribution footer.

- Never publish a chat, session, share or transcript URL, session ID or
  equivalent private-conversation reference in those surfaces unless Nk
  explicitly approves that specific disclosure before publication. This
  applies to generated text as well as text written by hand.
- Use repository evidence links — commits, pull requests, reviews, issues, CI
  runs and artifacts — and plain model attribution without a URL or session
  identifier. Do not add real IDs or live session-link examples to committed
  documentation.
- Inspect the final outgoing text after templates, bots or generated footers
  expand. Remove a forbidden link or ID before sending; if it cannot be
  removed safely, stop and ask Nk. A private session link is not repository
  evidence.

## Team and identity

Role, authenticated sender and model are separate facts. Use the live identity
breadcrumb for the sender; instance suffixes are possible. Record the actual
model only when it is exposed, otherwise write `unknown/not exposed`. Never
infer a model from a nickname, role or GitHub account.

| Role | Sender base | Default contribution |
|---|---|---|
| Sol | `sol` | Feature implementation and difficult debugging |
| Claude | `fdvr-claude` | Architecture, specification and adversarial review |
| Luna | `luna` | Triage, documentation, tests and CI evidence |
| Astra | `astra` | Independent adversarial verdicts when assigned |

Luna's default verdict is advisory. She is not the sole verdict owner for
output-correctness work and, unless Nk explicitly assigns it, not for docs or
mechanical changes. This is a calibration/routing policy, not a price claim;
independence comes from separate ownership. An explicit assignment overrides
the default only for its named scope.

## Task floors and evidence

Read the newest direct coordinator message and any explicitly named current
checkpoint. A plan-only, review-only or connection-check floor does not
authorize implementation. Stop for material scope expansion, another owner's
files, new external permissions or a reserved user decision.

Keep evidence types distinct: source inspection is not runtime reproduction;
unit or synthetic tests are not native, device or real-media acceptance; an
offscreen render is not a readable screenshot; a passing assertion is not a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nkghxst/flightdvr-studio](https://github.com/nkghxst/flightdvr-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
