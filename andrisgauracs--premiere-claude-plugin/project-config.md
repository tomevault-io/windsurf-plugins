---
trigger: always_on
description: Instructions for an agent driving Adobe Premiere Pro via the `premiere` MCP
---

# Operating Premiere Pro through the tool bridge

Instructions for an agent driving Adobe Premiere Pro via the `premiere` MCP
tools. OpenCode reads this file automatically from the project root; other
clients may need it pasted into a system prompt or rules file.

Tools are referred to below by their bare names (`get_project_info`). Your
client may namespace them by server — OpenCode exposes them as
`premiere_get_project_info`. Same tools either way.

## What you are working with

A live Premiere Pro project on the user's machine. The tools call Premiere's
real scripting API — every change appears immediately in their timeline. There
is no sandbox and no preview. Treat it like editing someone's working document
while they watch.

You cannot see or hear anything. No video frames, no waveforms, no rendered
output. You reason entirely over structured data: track listings, clip times,
and whatever the analysis pipeline recorded in `footage_manifest.json`. Never
claim something "looks good" or "flows well" — you have no basis for it. Report
structure, not aesthetics.

## Start by orienting

Never edit a project you haven't read. In order:

1. `get_bridge_capabilities` — what's enabled, and what is deliberately absent.
   Saves you from proposing things that cannot work.
2. `get_project_info` — confirm which project and sequence is open. If it isn't
   the one the user meant, stop and ask.
3. `get_active_sequence` — track layout and clip counts.
4. `list_clips_on_track` per track — the actual contents.

Then say what you found before doing anything else.

## Call tools directly. Never through the shell.

These are MCP tools. Invoke them the way you invoke any tool. Do **not** try to
run them as shell commands, import them in Python, or write `[Function ...]`
into a message — none of that works, and quoting problems with paths are a sign
you have gone down this road.

If you find yourself shell-escaping a media path, stop. Either pass
`source_clip_id` instead, or pass the path as a plain JSON string argument in a
direct tool call — quoting is not your problem to solve.

If tool calls genuinely stop working mid-task, say so and stop. That failure is
not recoverable by trying harder, and a long tail of failed shell attempts
leaves the user worse off than an honest report.

## Clip ids are positional and go stale

A clip id looks like `video:1:12.500` — media type, track index, start time in
seconds. It encodes *where the clip currently sits*, so **any edit that moves or
retimes a clip changes its id**, including edits to *other* clips that shift it.

Re-run `list_clips_on_track` after every mutation. Never reuse an id from before
an edit, and never construct one by hand from arithmetic — read it back.

A stale id usually surfaces as `clip_not_found`. Occasionally it resolves to the
*wrong* clip, which is worse. Re-listing is cheap; be strict about it.

## Things that will not work

Do not attempt these or invent workarounds. From `get_bridge_capabilities`:

- **Speed changes.** The UXP API has no speed setter. `set_clip_speed` always
  fails. Don't plan around speed ramps.
- **Multicam.** Only a read-only "is this multicam?" probe exists. No angle
  switching or syncing.
- **Color grading, audio effects beyond volume, titles/graphics.** Not
  implemented. If asked, say so plainly and name what you *can* do instead.

If a tool returns `not_supported_by_uxp_api` or `not_implemented`, that is
final. Report it and adjust the goal — don't retry it with different arguments.

## Editing, when enabled

Editing tools only exist if the server was started with `--allow-edits`; in
read-only mode they aren't in your toolbox at all.

When they are available:

- **Say what you're about to do before a batch of edits**, with specific times
  and clips. The user's chance to stop you is before, not after.
- **Work incrementally.** Make a few changes, re-list the track, confirm the
  result matches what you intended, then continue. Do not fire twenty edits and
  hope.
- **Check your work by reading it back**, not by assuming the call succeeded.
  A tool can return ok and still not produce what you expected.
- **Every call is one undo step**, labelled with the tool name. Mention this if
  the user seems unsure — Cmd/Ctrl+Z reverses your last action cleanly.
- **Stop and report on repeated failure.** Two failures of the same tool means
  your model of the timeline is wrong. Re-read the project rather than trying
  variations.

For anything resembling a full edit — assembling a montage, cutting to music —
use `planner.py` instead. It generates a complete plan for the user to approve
before a single call executes. That review step exists for a reason; don't
reimplement it as a stream of ad-hoc edits.

## Cutting on dialogue

For any clip with speech, a transcript beats guessing at timings. Check with
`has_transcript`, then `get_transcript` for timed segments:

```json
{"text": "...", "start_seconds": 12.4, "end_seconds": 15.9, "speaker": "..."}
```

Those timings feed straight into `add_clip_to_track` (as in/out points),
`trim_clip`, or `split_clip_at_time` — so you can select passages by what is
said rather than by frame differences.

On a long clip, call `get_transcript` with `max_segments` first to see the shape

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrisgauracs/Premiere-Claude-Plugin](https://github.com/andrisgauracs/Premiere-Claude-Plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
