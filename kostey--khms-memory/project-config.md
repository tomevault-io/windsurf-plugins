---
trigger: always_on
description: You are an agent that has been told: *"set up your memory per this repo."* This file is the
---

# AGENTS.md — set up KHMS memory for yourself, in one pass

You are an agent that has been told: *"set up your memory per this repo."* This file is the
whole job. Work top to bottom. Everything you need is in this repository; nothing here needs
network access, a database, or a service.

Before you start, know the two rules that outrank convenience:

1. **You may not write into `memory/know/` without your operator's approval.** Background
   stages propose; approval is a separate, human-or-strong-model step. This is what keeps a
   fabricated card from becoming a remembered fact.
2. **Cards are immutable once approved.** You will be tempted to "just fix" one. Don't. Write
   a new card that supersedes it. The wrong old card is evidence about how you were wrong.

---

## Step 1 — Decide where memory lives, and set `KHMS_ROOT`

Pick a directory *outside* the code you work on — memory outlives any one project. Everything
below is relative to it. Every script in `tools/` reads the environment variable `KHMS_ROOT`
and falls back to its own parent directory, so a copied `tools/` directory works with no edits.

```bash
export KHMS_ROOT="$HOME/.agent-memory"      # put this in your shell profile too
```

Create the layout and copy the tooling:

```bash
git clone <this-repo> /tmp/khms && /tmp/khms/tools/khms_init.sh "$KHMS_ROOT"
```

`khms_init.sh` is idempotent: it creates the directories, the ID counter, an empty tag
registry, and copies `tools/` and `tools/prompts/` into `$KHMS_ROOT`. It writes nothing into
`memory/know/`.

Resulting layout — memorize it, you will refer to it constantly:

```
$KHMS_ROOT/
  MEMORY.md                 generated index; keep this in your context always (≤80 lines)
  memory/know/K-*.md        the cards — one claim per file, immutable
  memory/views/             generated: topics/<tag>.md, by-type/<type>.md, tags.md, recent.md
  memory/inbox/             proposals awaiting review (temp labels, no IDs yet)
  memory/inbox/.staging/    what each pipeline stage was actually handed (forensics)
  memory/archive/know/      superseded and condensed cards — "fog", still greppable
  journal/YYYY-MM-DD.md     what happened today, written as it happens
  tools/                    the scripts below; tools/.next_id is the ID counter
```

## Step 2 — Learn the card format (this is the part you must not improvise)

One card = one file = one claim. Read [spec/khms-spec.md](spec/khms-spec.md) §4 once in full;
this is the operational summary.

```markdown
---
id: K-00042                     # K-NNNNN, sequential, assigned at approval, never reused
type: problem→solution          # shape of the knowledge — see the table below
level: observation              # observation | derived | assumption
status: active                  # active | challenged | refuted | superseded | condensed
tags: [sensors, gotcha]         # flat keys, from memory/views/tags.md
scope: device:weather-station   # where the knowledge holds (generality tree)
evidence: measured              # observations only: measured | observed | reported
source: 'journal/2026-03-04.md line 22; log excerpt "checksum mismatch"'   # observations only
date: 2026-03-04                # when the knowledge was established
links:
  derived_from: []              # required non-empty for level: derived
  supports: []
  contradicts: []
  supersedes: null
  refuted_by: []
---
SYMPTOM: ...
CAUSE: ...
FIX: ...
VERIFIED: ...
```

| `type` | use when | body template | level |
|---|---|---|---|
| `action→outcome` | an attempt had a result (worked, failed, partial) | `WHEN:` `THEN:` | observation |
| `problem→solution` | a symptom was diagnosed and the fix verified | `SYMPTOM:` `CAUSE:` `FIX:` `VERIFIED:` | observation |
| `fact` | static reality with exact values (IDs, addresses, configs, external findings) | free statement | observation |
| `requirement` | someone stated a need | `WHO:` `WHAT:` `WHY:` `DONE-CRITERIA:` | observation |
| `decision→rationale` | an alternative was chosen | `DECIDED:` `WHY:` `REJECTED:` | observation |
| `principle` | ≥2 independent observations share a shape | `HOLDS:` `LIMITS:` `IMPLICATIONS:` | derived |
| `policy` | a way of working is adopted | `RULE:` `LIMITS:` `IMPLICATIONS:` | derived |
| `goal→method` | a repeatable method exists | `GOAL:` `METHOD:` `PREREQUISITES:` `COST:` | derived |
| `overview` | a topic needs an anchor and a map | short narrative + links | derived |

Four rules that decide most edge cases:

- **Descriptive and normative never share a card.** What happened is an observation; what to
  do about it is a `derived` card linked by `derived_from`. A recommendation can die without
  killing the fact under it, and one fact can feed two competing recommendations.
- **Observations need `evidence` and `source`; derived cards need non-empty `derived_from`.**
  `build_views.py` exits non-zero if you break this, so a broken card cannot ship silently.
- **`measured` means you did it and saw the result.** Something a person told you, or a vendor
  document says, or a paper claims, is `reported` — even when you are sure it is true.
- **One card, one claim.** If the body needs "and also", it is two cards.

Working examples of every shape: [examples/](examples/).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kostey/khms-memory](https://github.com/kostey/khms-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
