---
trigger: always_on
description: OpenCharaAgent is a runtime that lets an AI character (a **chara**)
---

# OpenCharaAgent — project memory for Claude Code

## What this is, and who it serves (the product philosophy — read first)

OpenCharaAgent is a runtime that lets an AI character (a **chara**)
**live in a computer**: a persistent digital being with its own sandbox, memory,
goals, pace of life, and real agency (shell, files, tools) behind an
allowlisted, audited gateway. Three audiences, in priority order:

1. **Character creators** — people who want their character alive QUICKLY
   (POSITIONING, owner 2026-07-17: we no longer CLAIM "original character"
   in outward copy — the market imports any card; tagline 「和你的角色一起创作 /
   Create with your characters」, one-liner in README hero),
   to chat with it and to watch it do its own things. The path from
   *inspiration → living chara* must be short: AI-assisted card creation
   (draft the card, a small SVG avatar, a theme color — like the web deck
   already does) while the human keeps FULL control over every detail of who
   their character is.
2. **The charas themselves** — they live at their own rhythm in step with
   reality: they think, pursue their goals, browse what interests them, rest
   when they choose, and *decide* when something is worth telling their human
   (the `speak` tool). The engine's job is respect: neutral guidance about how
   to use tools and unattended time — suggestions, never orders. This is where
   most of the remaining design effort lives (see Roadmap: the chara
   curriculum).
3. **Developers / agent users** — strip the persona and a chara degenerates
   cleanly into a hermes/openclaw-style workhorse: cards, packs, MCP, skills,
   headless `run -p`, the JSON-RPC gateway.

It draws on several projects — the clones under `reference/` (gitignored) are
**hermes-agent, AstrBot, cc-switch, openclaw, pi**; **always consult them when
designing**. (pi = earendil-works/pi, added 2026-07-17: the minimal-harness
counterpoint — the agent extends its own harness via runtime extensions instead
of the core growing features; consult it on extensibility questions.) (SillyTavern is the card/world-book FORMAT spec we stay compatible
with, not a clone on disk.)

- **NousResearch/hermes-agent** — the most important. Agent runtime, context
  management, prompt-cache discipline, skills, plugin/registry patterns.
  RULE (owner, 2026-06-13): before building or fixing any COMMODITY
  subsystem (streaming, tool loop, PTY, dashboards, session hygiene —
  anything that isn't the chara-life innovation core), read the hermes
  counterpart first and port its solution shape AND its edge cases —
  hermes's scars are the maturity we lack. Architecture stays ours; never
  inherit its fallback-model logic. Parity checklist:
  `docs/OPEN-WORK.md` (Part 1).
  CLARIFIED (owner, 2026-06-19): **"behave apple-to-apple with hermes" is the
  default for the WHOLE tool/harness layer**, not just the four context
  subsystems — including behavioral GUARDS (hard-block a foreground long-running
  command, NOT an advisory note; a real PTY for interactive programs; parallel
  sub-agent fan-out). These are mechanical, value-NEUTRAL harness decisions with
  mature solutions, so adopt hermes's directly. The NEUTRALITY principle (below)
  is about the **chara's WORLDVIEW/VALUES only** — keeping a chara free of a
  built-in value-direction so it can play ANY role — it does NOT license
  re-shaping the harness toward a "gentler" agent. The ONLY legitimate tool
  divergences are ARCHITECTURALLY FORCED and must preserve hermes's capability +
  contract: one-process-one-chara maps hermes's GLOBAL `~/.hermes/{skills,
  memories}` to PER-CHARA storage (a host runs many charas; global would
  cross-contaminate); macOS sandbox-exec `deny network*` makes execute_code use
  hermes's OWN file-RPC transport instead of a UDS; the macOS/Linux-only scope
  drops Windows fallbacks. Everything else tracks hermes — when in doubt, match it.
  STRENGTHENED (owner, 2026-06-18): four subsystems are now **apple-to-apple
  IDENTICAL** to hermes — copy the algorithm, the numbers, and the prompt text
  verbatim, then run a comparison agent each pass until they match: (1) the
  **compaction trigger** (threshold ratio, protect-first/last, anti-thrash
  guard), (2) the **summary template** (the full structured `## Active Task…`
  sections, the iterative-update framing, the REFERENCE-ONLY handoff prefix,
  the deterministic static fallback), (3) **cache_control** (the
  `system_and_3` breakpoint placement), (4) **reasoning replay**
  (reasoning_content padding + `reasoning_details`/signature round-trip + the
  per-provider echo tiers). The ONLY edit allowed while porting these is
  de-branding the MODEL-FACING text: no literal "hermes"/"Hermes"/"the VM"
  may appear in any string the model sees — system prompts, tool descriptions,
  skill bodies, summary instructions (use neutral wording — "this runtime",
  "your environment"). OS NAMES ARE NEUTRAL, NOT BRAND (owner, 2026-06-19):
  "Linux", "macOS", etc. are plain factual content and may appear in
  model-facing text — they need NOT be scrubbed (e.g. a terminal tool
  description saying "shell commands on a Linux environment" is fine). Only
  the hermes brand / "the VM" framing is off-limits. Source-code COMMENTS may

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenChara/OpenCharaAgent](https://github.com/OpenChara/OpenCharaAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
