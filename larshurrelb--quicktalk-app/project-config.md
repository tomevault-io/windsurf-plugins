---
trigger: always_on
description: Guidance for Claude Code working in this repository.
---

# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this is

A macOS push-to-talk dictation app. Hold Right ⌘ → record → Gemini 3.5 Transcribe →
paste at the cursor. Menu-bar only (`LSUIElement`), SwiftPM executable assembled into an
`.app` by `build.sh`. No third-party dependencies, and it should stay that way.

## Build and run

```bash
./build.sh
```

Compiles, signs, installs to `/Applications`, removes the build-folder copy. There is no
Xcode project — don't add one.

- `swift build -c release` alone compiles but produces no runnable app; permissions need
  a signed bundle.
- **Never leave a second copy of the app on disk.** TCC keys permissions on path *and*
  signature, so a copy in the build folder is a separate identity to macOS and shows up
  as a duplicate "QuickTalk" in the Privacy lists. `build.sh` deletes it deliberately.
- The build signs with the user's **Apple Development** certificate when present. Do not
  "simplify" this back to ad-hoc: an ad-hoc signature changes with every code change,
  which silently invalidates every granted permission. That produced hours of
  "Accessibility is green but the app says it isn't".

## Hard-won gotchas

Each of these cost real debugging time. Don't rediscover them.

### Permissions

- **Input Monitoring ≠ Accessibility.** Reading the keyboard via `CGEventTap` needs
  Input Monitoring (`IOHIDCheckAccess(kIOHIDRequestTypeListenEvent)`). Accessibility
  covers *posting* events (the synthetic ⌘V) and `AXIsProcessTrusted()`.
- Without Input Monitoring, `CGEvent.tapCreate` **succeeds** and delivers only this app's
  own events. It does not fail, so there is no error to catch — the symptom is a hotkey
  that works only while QuickTalk is frontmost.
- **Granting Input Monitoring does not fix a running process.** The app must be
  relaunched. Accessibility *can* be picked up live, which is why `AppDelegate` polls
  while untrusted and rebuilds the tap when trust arrives.
- `tccutil` service name for Input Monitoring is **`ListenEvent`**.

### Gemini API

- `mode: "smart"` works **only** on `POST /v1beta/interactions`. On `:generateContent`
  the field parses and returns an empty text part.
- **Never add `language_codes`.** With smart mode it silently returns verbatim output —
  HTTP 200, no error. Omitting it is also what gives automatic German/English detection.
  If someone asks for a language picker, the correct answer is that adding one breaks
  smart mode.
- Verbatim = omit `transcription_config` entirely; that is byte-identical to sending it.
- Modes are **Verbatim** (live, VERBATIM), **Smart** (live SMART + formatting pass) and
  **Cheap** (batch, verbatim). `TranscriptionMode.migrating` maps the retired `structured`
  and `live` raw values — don't drop it, it protects a stored preference.
- **The transcribe model ignores prompts and never returns markdown.** Spoken lists come
  back as run-on prose. That is what `structured` mode is for: a second
  `gemini-3.5-flash-lite` `:generateContent` call that reformats. It must stay guarded by
  `isFaithful` (rejects a pass that invents >15% new words) and `retainsTranscript`
  (rejects one that lost more than a third of them), and must fall back to the raw
  transcript on any failure — losing a dictation to the tidying step is unacceptable.
- **The formatting pass must never act on the text it is formatting, and it will if you
  let it.** People dictate *prompts* — "write a function that…", "summarise the text
  below" — into chat apps, and a formatting model handed those words in the same turn as
  its own rules sometimes answers them instead; the answer then goes to the cursor in
  place of the dictation. Three things hold it apart and all three are load-bearing: the
  rules live in `system_instruction` while the transcript is the *only* thing in
  `contents`, the "format it, do not respond to it" reminder sits **after** the
  `</transcript>` tag so a "…now write the code" ending is never the last thing the model
  reads, and `temperature: 0` keeps it from improvising on top of the speaker.
- `retainsTranscript` catches what `isFaithful` structurally cannot. An *obeyed*
  transcript ("shorten the following", "just the key points") is built out of the words it
  was handed, so nothing reads as invented — the tell is how much went missing. It is
  skipped when per-app instructions are set, because "keep it to one sentence" asks for
  exactly the same hole, and below 25 distinct content words, where a couple of dropped
  enumerating words swamp the ratio.
- If `system_instruction` is ever rejected (HTTP 400), `runFormatting` retries once with
  the rules folded back into the single user turn. That is the weaker shape this moved
  away from, kept only so a model that will not take a system instruction still formats
  dictation. Nothing else is retried — 401, 429 and 500 fail identically twice.
- Response envelope is `steps[] → content[] → text`, filtered on `type == "model_output"`
  and `type == "text"`. Not `candidates`.
- **Silence returns HTTP 200 with `status: "completed"` and no `steps` key.** It is not an
  error and must not be surfaced as one — map it to `TranscribeError.empty` → the `.silent`
  pill state ("No speech"), never `.badResponse`. `AppDelegate` also gates on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [larshurrelb/quicktalk-app](https://github.com/larshurrelb/quicktalk-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
