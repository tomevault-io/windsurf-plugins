---
trigger: always_on
description: Turns saved Instagram reels and carousels, and saved X posts, into a
---

# Reels To Knowledge Base

Turns saved Instagram reels and carousels, and saved X posts, into a
searchable, actionable knowledge base: SQLite + FTS5 for the pipeline, an
Obsidian vault as the readable view.

## Why the pipeline is split

Everything mechanical runs locally and free — download, frame sampling,
transcription, OCR. Only the judgement step needs a model, and it runs on the
Claude Code subscription rather than per-token API billing.

```
1. rkb prepare   download, sample frames, transcribe, OCR      (plain Python)
2. extraction    read the context, write structure back        (you)
3. rkb vault     render the database as markdown               (plain Python)
```

SQLite is the source of truth. The vault is a generated view, so `rkb vault` is
safe to re-run after every batch.

## Where posts come from

Three doors, all landing in the same `posts` table at status `new`:

```
rkb import      the Meta export           (Instagram, bulk backfill)
rkb bookmarks   the live X bookmark feed  (X, bulk, needs cookies)
rkb add <url>   one link, any platform    (either; also what the bot calls)
```

Two front doors put `add` within reach of your phone, both landing in the same
place. `rkb telegram` runs a bot (stdlib only, no library): forward a post from
the share sheet and it lands here. `rkb ingest --serve` is the same thing over
HTTP, for an iOS Shortcut or a curl.

**A front door is not a stage.** Neither one downloads, transcribes or reads
anything. Saving costs milliseconds; preparing costs a video download, ffmpeg,
whisper and OCR on this machine — so they are separate acts with separate
triggers. A forwarded link lands at `new` and waits. Draining the queue is
deliberate: `rkb prepare`, `/prepare` in the chat, or the button the bot puts
on its reply. The extraction pass is still yours in Claude Code, on the
subscription, rather than an LLM provider billing per token.

One batch runs at a time (`worker.run_batch`). Two would fight over the same
rows and fetch in parallel, which is what `prepare`'s sleep exists to prevent.

`rkb/platforms.py` is the only place a platform is described. Adding a third
source is a row in that table plus a branch in `acquire`, not a new pipeline.

## Your job: the extraction pass

When asked to extract, process or ingest a batch:

1. `./bin/rkb pending -n 25` — list posts awaiting extraction.
2. For each, read `<media_dir>/context.md`. It contains the caption, the
   transcript, and **the verbatim OCR text of every frame or slide**. A
   text-only X post has none of those: `media_kind` is `text`, and the tweet
   text under `## Tweet text` is the entire post. There are no frames to open
   and nothing is missing — do not file it as under-prepared.
3. Read images from `<media_dir>/frames/` only when the OCR is ambiguous — a
   blurred URL, a layout you can't reconstruct from text alone. OCR is cheaper
   and more literal than a vision read; prefer it.
4. Emit one JSON object per post and write them back in one call:
   `./bin/rkb record batch.json`
5. `./bin/rkb vault` to refresh Obsidian.
6. `./bin/rkb triage` to see what carries no payload — anything without a link
   lands in the user's review queue, which is theirs to decide, not yours.

### Output schema

```json
[
  {
    "shortcode": "Dbm7X5IAuEe",
    "summary": "one line — what this post is actually about",
    "detail": "2–4 sentences: what the tool/technique does and how it was used",
    "tools": ["Claude Code", "mlx-whisper"],
    "links": ["https://github.com/owner/repo"],
    "prompt": null,
    "tags": ["agents", "local-llm"],
    "actionable": "clone the repo and run the example, or null",
    "confidence": "high"
  }
]
```

## Rules that matter

**The link is the payload.** Hunt for the repo or URL before anything else —
check the OCR text, every frame including the last, and the caption. Every post
you leave without a link goes to the user's review queue, where they read the
raw OCR themselves, so an empty `links` is a claim that you looked and found
nothing.

**Do not compose prompts.** The verbatim on-screen text is already captured in
`ocr_text` and rendered in full on every vault note and dashboard card — the
user reads it there and lifts what they want. Leave `prompt` null. Do not
rewrite a checklist into a "drill", add framing, reorder items, or drop half of
it to make a nicer artefact. Your job is to find the link and describe the post,
not to design study material out of it.

**When the address bar is covered, read the chrome around it.** A screen
recording usually leaks the repo owner somewhere else in the frame: a browser
tab title, an adjacent tab, a window title, a terminal prompt, a git remote, a
Docker image name, a GitHub sidebar avatar. This is a real miss from the first
pass — FreeLLMAPI's owner (`tashfeenahmed`) was legible in a neighbouring tab
while the URL bar sat behind a caption overlay, and the post was filed as "no
link". Exhaust the frame before declaring a link unrecoverable.

**Resolve a named tool to its canonical repo** when you are certain
(`Appsmith` → `https://github.com/appsmithorg/appsmith`). If you are not
certain, leave `links` empty and set `confidence: medium`.

**Never invent.** A missing link is recoverable from the review dashboard; a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Manas-Taneja/ReelsToKnowledgeGraph](https://github.com/Manas-Taneja/ReelsToKnowledgeGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
