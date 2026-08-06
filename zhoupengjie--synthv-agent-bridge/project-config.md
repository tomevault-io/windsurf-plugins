---
trigger: always_on
description: validates the complete batch and creates one SynthV undo record.
---

# Repository guidance

## Scope

This repository contains a TypeScript MCP stdio server and a persistent Synthesizer V Studio Lua executor connected by versioned file IPC.

## Invariants

- Keep the MCP server network-free by default.
- Keep track, group, and note indices 1-based at the protocol boundary.
- Validate complete write requests before calling `Project:newUndoRecord()`.
- Require current fingerprints for note edits and deletes.
- Do not parse or mutate `.svp` files directly.
- Do not log project lyrics or note data to stderr unless explicitly requested for debugging.
- Keep file IPC protocol v2 as the sole request/response envelope. Reject
  protocol v1 with `PROTOCOL_MISMATCH`; add a new version for any future
  breaking envelope change.
- Keep the public MCP surface limited to the eight compact v2 tools. Detailed
  SynthV action handlers are internal definitions exposed just in time through
  `sv_describe`; do not expose them as standalone MCP tools.
- Keep the responsibility boundary enforceable in code:
  - The Agent and user own intent, lyric/emotion/style interpretation, target
    choice, Vocal/Vocal Mode onboarding, and the requested musical values.
  - The TypeScript MCP layer owns schemas, action routing, compact projections,
    `contextId`/Guard expansion, session invalidation, and minimal
    acknowledgements. It must not invent musical values.
  - The Lua executor owns authoritative SynthV reads, host capability and
    dynamic-range checks, deterministic batch expansion, complete preflight,
    one undo boundary, and postcondition verification. It must not infer
    emotion, style, singer identity, or which notes deserve emphasis.
  - SynthV remains the project-state authority; the user remains the final
    listening and artistic authority.
- Do not probe tuning ranges at startup or at the start of each conversation.
  Treat Group Voice loudness as `-48..12`, Group Voice
  tension/breathiness/gender/tone shift as `-1..1`, Vocal Mode
  pitch/timbre/pronunciation axes as `0..150`, phoneme position/activity as
  `0..1`, and phoneme strength as `-1..1`. Phoneme `leftOffset` is a finite
  number of seconds without a Bridge-imposed bound. For automation, use the
  `definition.range` returned by the same fresh curve/phrase read instead of a
  fixed table because SynthV host and voice versions can expose different
  ranges.
- Prefer `apply_group_tuning` when one tuning pass changes Voice/Vocal Modes,
  notes or phonemes, and one or more automation curves in the same Group. It
  validates the complete batch and creates one SynthV undo record.
- Prefer `transform_notes` when every note in one freshly read scope receives
  the same explicit mechanical onset, duration, or semitone transform. With
  MCP v2, use `target: "contextNotes"` and the fresh `contextId` instead of
  repeating note indices. The Agent chooses the exact target scope and numeric
  transform; the Bridge only expands and verifies it. A seconds onset offset
  uses the fresh SynthV time axis and preserves note durations in blicks.
- Keep consecutive notes inside one lyric phrase exactly connected unless the
  user or the intended performance explicitly calls for a rest or detached
  articulation: the earlier note's end must equal the following note's onset.
  Never create tiny positive gaps to shape pronunciation or articulation,
  because they can prevent SynthV from rendering usable vocals. Use phoneme
  timing/strength and Voice or automation parameters for articulation instead.
  After duration or onset edits, reread the phrase and account for every
  remaining gap as an intentional phrase boundary or artistic rest.
- Apply that automatic note-connection rule only to notes the Agent created in
  the current task. Treat pre-existing notes, lyrics, onsets, durations, gaps,
  and rests as user-owned score structure: preserve them unless the user
  explicitly requests that specific structural change. If note provenance is
  uncertain, treat the material as user-owned. A general request to tune a
  performance authorizes the requested tuning parameters, not silent
  normalization of the user's note geometry.
- If an MCP v2 call returns `SYNTHV_SESSION_CHANGED`, do not retry its old
  `contextId` or Guard Tokens. The server has already cleared them; read the
  intended target again and build the write from the fresh context.
- On the first successful MCP/Bridge connection notice in a conversation,
  offer the optional bundled demo in one sentence: the user can reply
  `Run the Twinkle Star demo.` or `运行《小星星》Demo。` The Agent must not
  create anything until the user explicitly opts in, and it must not repeat
  the offer later in the same conversation.
- When the user starts that demo, read all of
  `examples/twinkle-star-demo.json` before acting and use it as the score,
  tuning, safety, and verification source of truth. Do not add a public MCP
  tool or move its musical decisions into TypeScript or Lua. Before each stage,
  print the matching short localized heading from `progressHeadings`, followed
  by at most one concise status sentence; do not expose raw MCP payloads:
  `Demo 1/5` checks the connection and safe location, `Demo 2/5` creates the
  isolated Note Group, `Demo 3/5` pauses for Vocal and singing-style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhoupengjie/synthv-agent-bridge](https://github.com/zhoupengjie/synthv-agent-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
