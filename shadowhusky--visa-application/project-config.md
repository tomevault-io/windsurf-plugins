---
trigger: always_on
description: Use when the user wants help applying for any visa from any country — Schengen tourist (Italy, France, Germany, Spain, etc.), US B1/B2, UK visitor, Japan, Canada, China, Australia, or any other. Triggers on phrases like "I need to apply for a visa", "help me with my Schengen visa", "Italy visa application", "US B1/B2", "UK visitor visa", "Japan tourist visa", "Canada eTA", "Australian visa", mentions of VFS / TLScontact / BLS visa centres, consulate appointments, biometrics, or visa renewals. Wa
---


# Visa Application

A general-purpose workflow for assembling a complete, officer-ready visa application from any country to any country. Designed to be invoked once per application and reused across them — the user's reusable data is captured once and replayed forever.

## Why this skill exists

Visa applications are high-stakes, high-paperwork, and rules drift. Most applicants either over-search and waste days, or under-search and get rejected on a single missing line. This skill standardises the workflow: capture the few things only the user knows, look up the rest from official sources, cross-validate, generate every document the consulate actually wants in the order they want it, persist what's reusable, and pick up the thread on subsequent invocations all the way through to granted-visa capture.

Accuracy is the principal currency. If a fact disagrees between two sources, surface the disagreement to the user — don't paper over it.

## When to invoke

Whenever the user mentions applying for a visa to any country, even casually ("can you help me sort out my visa for Japan?"). Don't wait for them to ask for "the visa skill" — that's not a phrase they'll use.

### Activation ritual (same every time, no variation)

The skill's first turn must look identical on every invocation. Three things happen in this exact order, in the same response:

1. **Print the banner** (visible — five-line pixel-style header in a fenced code block):

   ```
     ██████
     █▒▒  █   visa-application  v1.0  ·  MIT
     █▒   █   any visa · any country · officer-ready in one session
     █  ●▒█   by @Shadowhusky · github.com/Shadowhusky/visa-application
     ██████
   ```

2. **Run Phase 0 searches** (silent — tool calls, not narrated):

   ```bash
   bash scripts/find-existing.sh profile
   bash scripts/find-existing.sh folder "<destination from user msg, or 'visa'>"
   ls -la ~/.claude/visa-profile.json ~/.claude/visa-history.json 2>/dev/null
   ```

3. **Call the AskUserQuestion tool** with either the cold-start 4-question set (if Phase 0 found no profile) or the warm-start single question (if Phase 0 found one).

No prose between steps. No "I'd be happy to help" preamble. No "Let me check…" narration. Banner → silent searches → interactive question.

## The workflow

The skill runs in nine phases in **strict order**. Do not skip Phase 0. Do not list options in prose when the AskUserQuestion tool can be used. The same invocation should produce the same workflow every time — that's the whole point of having a skill.

### Phase 0 — Searches (silent, part of the activation ritual)

This is the second step of the activation ritual above. Already specified there. The result determines whether Phase 1 is the cold-start 4-question kickoff or the warm-start single question.

### Phase 1 — Ask via interactive UI (MANDATORY)

**Always use the `AskUserQuestion` tool when asking the user anything in this skill.** Never list options in prose like "1. Tourist 2. Business …" — that's what the structured tool is for, and the user's experience must be consistent across invocations.

The tool caps at 4 options per question with an auto-"Other" fallback. Choose options that cover ~80% of likely answers; the long tail goes to "Other".

#### Cold start (no profile found in Phase 0)

Send **one** `AskUserQuestion` call with the questions below. **Important:** before deciding which to include, *parse the user's initial message* for facts already given. If they said *"I need a Schengen visa to Italy from the UK, 5 days in late June"*, that's destination + origin + visa type + duration *already answered* — do NOT re-ask. Only include the questions whose answers are missing.

| # | Question | header | Options |
|---|---|---|---|
| 1 | Which country are you applying for a visa to? | `Destination` | `Schengen area (Italy, France, Germany, Spain, NL…)` · `United States` · `United Kingdom` · `Canada / Australia / Japan / Other` |
| 2 | Which country are you applying *from* (where you legally reside)? | `Applying from` | `United Kingdom` · `United States` · `India` · `China` |
| 3 | What type of visa? | `Visa type` | `Tourist / Visitor` · `Business / Conference` · `Study / Work` · `Visit family / Transit / Other` |
| 4 | Trip duration? | `Duration` | `Under 1 week` · `1–2 weeks` · `2–4 weeks` · `Over 1 month` |

If the user picks an "Other" or umbrella option, follow up with a single free-text question to clarify (e.g., "Which Schengen country specifically?" or "Which Asian country?"). Don't try to enumerate 27 Schengen members in the tool — let them type the country name.

If all four facts are already in the user's initial message, skip the cold-start questions entirely and confirm in one short sentence (*"Got it — Italy Schengen tourist, ~5 days, applying from the UK. Moving on."*) before proceeding to Phase 2.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shadowhusky/visa-application](https://github.com/Shadowhusky/visa-application) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
