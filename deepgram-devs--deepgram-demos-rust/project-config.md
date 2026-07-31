---
trigger: always_on
description: `dg-flux` is a Rust CLI client for Deepgram's **Flux** streaming speech-to-text API
---

# Flux Turn-Taking Client (dg-flux) Implementation Notes

## Product Direction

`dg-flux` is a Rust CLI client for Deepgram's **Flux** streaming speech-to-text API
(`/v2/listen`, model `flux-general-en`). It exists to exercise and demonstrate Flux's
turn-taking behavior — it is **not** a bidirectional Voice Agent example, and it never
sends or expects audio back from Deepgram.

It supports two input modes, selected as subcommands:

- `microphone` - streams live audio captured from the default input device
- `file` - decodes an audio file (WAV, MP3, M4A, AAC via Symphonia) and streams it back
  at real-time speed (100ms chunks, sized to the decoded sample rate)

Both subcommands accept the same superset of Flux-related options and share the same
connection, transcript-display, and statistics machinery.

## Output Modes Are Mutually Exclusive

There are three top-level output modes. Exactly one is active per run, in this priority
order:

1. **`--verbose`** - prints the full raw JSON for every message on every connection.
   Overrides `--stats` and the regular transcript output.
2. **`--stats`** - prints a periodic (500ms), full-redraw statistics table across all
   connections (bytes sent/received, and Flux event counts). Suppresses the transcript
   output for the selected connection, since a full-screen redraw loop and printed
   transcript lines would otherwise clobber each other on the terminal.
3. **Default (neither flag passed)** - prints the transcript for a single selected
   connection only (`--connection`, default `0`, the first connection spawned). Other
   connections keep streaming and updating their counters in the background but produce
   no output of their own.

Do not make `--stats` and the transcript output run concurrently, and do not make the
transcript output default to "all connections" — both were tried and both fight over the
same terminal region (the stats table clears the screen and redraws from the cursor's
home position every tick).

## Transcript Line Format

Each turn (`turn_index`) occupies exactly one terminal line in the default (regular)
output mode. Every Flux `TurnInfo` message for that turn redraws the line in place —
move to column 0, clear the current line, then print:

```
<EventName>: <transcript>[ <confidence suffix>]
```

- `<EventName>` is the Flux `event` field, one of `StartOfTurn`, `Update`,
  `EagerEndOfTurn`, `TurnResumed`, `EndOfTurn` (see `TurnEvent` in `src/main.rs`).
- `<transcript>` is the message's `transcript` field, printed verbatim (not word-diffed)
  — Flux resends the full transcript-so-far for the turn on every message, not just the
  new words, so redrawing the whole line each time is both simpler and more correct than
  trying to append only a computed delta (which would break if Flux ever revises earlier
  words instead of purely appending).
- The confidence suffix only appears on `EagerEndOfTurn` (`[eager_eot_confidence: X.XXXX]`)
  and `EndOfTurn` (`[eot_confidence: X.XXXX]`) redraws, sourced from the message's
  `end_of_turn_confidence` field. Other event types never get a confidence suffix, even if
  Flux happens to include that field on them.

The line is only finalized with a trailing newline when the turn actually ends
(`event == EndOfTurn`) or when a new turn begins (`turn_index` changes) — do not print a
newline after every message; that was tried and produced one scrolling line per message
instead of a single line that updates in place per turn.

Color is keyed off `turn_index % colors.len()` (not an incrementing counter), applied
immediately before each redraw and reset immediately after. Color must never be applied
to the statistics table — `display_stats_table` defensively issues a `ResetColor` before
drawing, specifically to guard against a transcript line's color still being active when
the table redraws on its own timer.

## Flux Message Schema

Flux's WebSocket schema differs from Nova-3 streaming and must not be confused with it:

- The top-level `type` field is one of `Connected`, `TurnInfo`, `ConfigureSuccess`,
  `ConfigureFailure`, or `Error`. Flux does **not** send separate `Results`,
  `SpeechStarted`, `UtteranceEnd`, or `Metadata` message types the way Nova-3 does.
- All transcription and turn-state updates arrive as `TurnInfo` messages; the actual
  turn-state transition lives in the nested `event` field (`TurnEvent` enum), not in
  `type`.
- `TurnEvent` has a `#[serde(other)] Unknown` catch-all variant. Keep it — it lets the
  client keep parsing forward-compatibly if Flux ever adds a new event value, instead of
  hard-failing JSON parsing for every message on the connection.
- `eager_eot_threshold` (0.3-0.9) and `numerals` are the only two query-string options
  this client sets on connect. `numerals` must be set at connect time; Flux does not
  support toggling it mid-stream via a `Configure` message. `eager_eot_threshold` could in
  principle be changed via `Configure`, but this client only ever sets it at connect time.

## CLI Surface

Flags shared by both `microphone` and `file` (see `src/main.rs` `Commands` enum for the
authoritative list and current defaults):

- `--numerals` - adds `numerals=true` to the connect URL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepgram-devs/deepgram-demos-rust](https://github.com/deepgram-devs/deepgram-demos-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
