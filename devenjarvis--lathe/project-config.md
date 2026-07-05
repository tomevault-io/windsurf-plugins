---
trigger: always_on
description: Orientation for AI coding agents working in this repo.
---

# AGENTS.md

Orientation for AI coding agents working in this repo.

## What this is

Lathe is a Go CLI plus a set of coding-agent skills (the `SKILL.md` format is now a cross-tool standard — Claude Code, Cursor, Codex, Gemini CLI, opencode, Cline, Windsurf) that together generate, store, serve, verify, and extend hands-on technical tutorials. See `README.md` for user-facing docs.

The boundary is strict: **skills generate content; the CLI owns durable state.** All model work — generating, verifying, extending, answering reader questions, and authoring voices — runs in the user's **interactive** coding-agent session via user-invoked skills (`/lathe`, `/lathe-verify`, `/lathe-extend`, `/lathe-ask`, `/lathe-tag`, `/lathe-voice`, `/lathe-work`). The Go binary never drives a model itself — it spawns no `claude`/agent subprocess (which also keeps Lathe off metered headless runs like `claude -p`, metered as of 2026-06-15; interactive sessions are not). Don't move generation logic into Go, and don't have skills write to `~/.lathe/` directly — they call `lathe` commands (`lathe store`, `lathe verify-result`, `lathe extend-start`/`extend-commit`, `lathe voice add`) instead. The one skill→CLI **read** path is `lathe voice show` (the active voice spec) — still consistent with the boundary: the CLI stays the sole owner of the voice files and config; the skill only asks for text.

The web buttons close the copy-paste gap without crossing that boundary. When a `/lathe-work` worker session is connected (it long-polls `GET /-/work`, so `internal/queue` knows it's live), Ask/Verify/Extend **enqueue a job** the worker claims and runs in its interactive session; with no worker they fall back to the same paste-able handoff as before. The model still only ever runs in the interactive session — the queue is just an in-memory bridge between the browser and that session. MCP, if ever added, would be only an alternate transport for this same queue, not a way to run a model inside Go.

## Layout

```
main.go                           cobra entrypoint
cmd/
  root.go                         rootCmd ("lathe")
  list.go, open.go, rm.go, serve.go, store.go    one subcommand per file
  verify.go, extend.go            print the /lathe-verify, /lathe-extend handoff command
  verify-result.go                lathe verify-result — skill records verify status/result
  extend-start.go, extend-commit.go    lathe extend-{start,commit} — skill reserves/records a part
  work.go                         lathe work next/answer/done — the /lathe-work worker loop's CLI (reads ~/.lathe/serve.json)
  tag.go                          lathe tag — skill sets/adds/removes a tutorial's search tags
  version.go                      lathe version — prints buildinfo.String() (alias for --version)
  skills.go                       lathe skills install/list — write embedded skills to Claude Code / Cursor / Codex / Gemini / opencode / Cline / Windsurf
  voice.go                        lathe voice list/show/set-default/add/rm — manage writing voices (parent + subcommands, one file)
internal/
  buildinfo/                      Version/Commit/Date vars (ldflags-injected) + Resolve()/String()
  frontmatter/                    Parse()/Strip() — shared name:/description: frontmatter scanner (used by skills + voice)
  skills/                         embedded skills (//go:embed data) + catalog (skills.go), Cursor translation (cursor.go)
  voice/                          embedded voice presets (//go:embed data) + List/Resolve/Add/Remove + fixed guardrail Preamble (voice.go)
  config/                         TutorialsDir(), VoicesDir(), ConfigDir() → ~/.lathe; config.json (ReadConfig/WriteConfig, DefaultVoice/SetDefaultVoice); serve.json runtime file (ServeRuntime Read/Write/Remove) so `lathe work` finds the running server
  queue/                          in-memory job queue + worker presence (queue.go): Enqueue/Claim(long-poll+ctx)/Done/SetAnswer/Get, reclaim guard, MarkWorkerSeen/WorkerConnected — bridges the web UI and a /lathe-work session
  store/
    metadata.go                   Tutorial struct (incl. Repo/RepoBranch + Tool/Tools + Sources + Voice + Model), Status enum, Read/WriteMetadata, RepoDisplay, VerifyResult
    store.go                      Store()/StoreOptions, Delete(), Normalize{Tags,Sources,Repo,Tools,Voice}, copyDir/copyFile, detectParts, SlugToTitle, PromoteIndexToPart
  serve/
    server.go                     net/http handlers (list, tutorial, part, delete); handleList renders a flat newest-first list; list.html does client-side search/filter/sort
    ask.go, verify.go, extend.go  POST endpoints: enqueue a job when a /lathe-work worker is connected, else return a paste-able skill command (handoff.go: writeQueued/writeHandoff)
    work.go                       worker bridge endpoints: GET /-/work (long-poll claim), POST /-/work/{id}/{answer,done}, GET /-/work/{id} (browser ask-answer poll), GET /-/worker (presence)
    renderer.go                   goldmark + chroma markdown rendering
    layout.html, list.html        embed.FS page templates
    components.html               shared {{define}} partials (head, badge, themeToggle, liveNudge — the last carries its own <script>, included on both pages)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devenjarvis/lathe](https://github.com/devenjarvis/lathe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
