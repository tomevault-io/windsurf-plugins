---
trigger: always_on
description: hwatu is a headless verification harness for coding agents: a warm
---

# hwatu for coding agents

hwatu is a headless verification harness for coding agents: a warm
WebKit daemon where opening, driving, and closing a real rendered
page costs about as much as running `ls`. Jcode drives it natively as
its `browser` backend; everything else connects over MCP or CLI. Plan
of record: the [AI verification roadmap](roadmaps/verification.md).

It is not a scraping browser. If you need to crawl the web at scale,
use a headless-Chrome fleet or Lightpanda. hwatu is for the inner
loop of frontend development: an agent edits code, opens the page
headlessly, checks it, and moves on, dozens of times an hour, on the
same machine the human is working on.

## Why agents like it

- **13-16 ms window spawn** from a warm daemon, measured medians
  ([benchmarks](benchmarks.md)). Verification loops spawn and discard
  windows constantly; hwatu keeps the whole loop (open, load, read,
  screenshot, close) under ~200 ms with zero setup.
- **One shared engine, zero supply chain.** N windows share one
  WebKit network process and a prewarm pool (~56 MB per extra
  window). One static binary plus the distro's webkitgtk: no Node,
  no npm package, no per-version browser download.
- **Real rendering.** Full WebKit: layout, CSS, WebGL, media.
  Screenshots show what a user would see. (Contrast with
  render-less automation engines, which are fast but blind.)
- **Headless by default for agents.** `--headless` never maps a
  window; `--background` maps one without an activation request. The
  human keeps typing while the agent verifies. The CLI defaults to
  headless when it detects a coding-agent environment (markers like
  `CLAUDECODE`, `JCODE_SOCKET`, `CURSOR_AGENT`), so a forgotten flag
  never puts a window in the user's WM; `--focus` opts back in, and
  `HWATU_AGENT_MODE` / `"agent_mode"` in
  `~/.config/hwatu/config.json` tune the agent default
  (`normal` | `background` | `headless`).
- **Human hand-off.** Every headless/background window is a live
  session. `hwatu focus <id>` promotes it to a normal window in the
  user's tiling WM: the human watches or takes over, then closes it.
  Headed and headless are a property of a *window*, not of the
  browser launch.
- **Terse, JSON-native protocol.** One newline-delimited JSON
  request per Unix-socket connection. No tool schema, no session
  objects, no WebSocket. Cheap for token budgets, trivial to drive
  from any language.

## What the agent gets

| primitive | what it answers |
|---|---|
| `snapshot` | what's on this page, what can I click (JSON, ~tokens not pixels) |
| `diff --other/--baseline` | how close are these two renders, where do they differ, as a score + worst-first regions (`worst_region`, `significant_regions`) + heatmap |
| `clone` | a self-contained local copy of a live page (rendered DOM + assets), verified against the original with a measured pixel-match report |
| `motion` | every animation as numbers: duration, delay, easing, keyframes |
| `seek` | pin all animations at time t; two shots at the same t are byte-identical |
| `expect` | assert page state in one call (polls, structured pass/fail) |
| `render --stdin` | see generated HTML rendered, no temp file, no server |
| `shot` / `shot --full` | what a user would see (real GPU-composited WebKit render) |
| `click` / `type` / `scroll` / `upload` | real pointer/input events, structured errors on misses |
| `console` | JS errors, console output, failed requests since last check |
| `net` | structured per-window request log: method, url, status, type, timing |
| `challenge` | is this a CAPTCHA / anti-bot wall, should a human take over |
| `resize` | verify responsive layouts across viewport widths |
| `focus <id>` | materialize any headless session as a real window for the human |

Ambiguity is an error with a match count, never a silent wrong click.
Refs from `snapshot` are live element handles; staleness is a clear
error, not a mystery.

## The protocol

Socket: `$XDG_RUNTIME_DIR/hwatu.sock` (fallback
`/tmp/hwatu-$UID.sock`). One request per connection: connect, write
one JSON line, read one JSON line, disconnect.

```sh
printf '{"cmd":"open","url":"http://localhost:3000","mode":"headless"}\n' \
  | socat - UNIX-CONNECT:$XDG_RUNTIME_DIR/hwatu.sock
```

Or use the CLI, which is the same protocol with argv parsing:

```sh
hwatu --headless localhost:3000     # open without a window (returns id)
hwatu --background localhost:3000   # open mapped but unfocused
hwatu check localhost:3000 --eval 'document.title' --shot=/tmp/c.png
                                    # one-shot: open+wait+eval+shot+close
hwatu wait-load                     # block until the load settles
hwatu wait-load --until dom         # release at DOMContentLoaded (faster)
hwatu snapshot                      # text + interactables, cheaper than a shot
hwatu expect '#status' --text ready # assert page state (polls up to 5s)
hwatu expect '#status' --text ready --watch # stream initial state + truth flips until navigation
hwatu eval 'document.title'         # id-less: follows the window you opened
hwatu click a --contains "Sign in"  # real pointer-event click
hwatu click --ref 4                 # click interactable #4 from the snapshot
hwatu type 'input[name=q]' hi --enter   # fill and submit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hongnoul/hwatu](https://github.com/hongnoul/hwatu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
