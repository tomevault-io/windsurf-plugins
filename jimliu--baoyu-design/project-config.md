---
trigger: always_on
description: The harness-specific tools `system-prompt.md` relies on, for when you are running inside **Claude Code**. The main prompt only names capabilities ("ask the user", "preview", "screenshot", "debug"); this doc gives the exact Claude Code tool, signature, and call pattern. Generic tools (`Bash`, `Read`/`Write`/`Edit`/`Glob`, `gh`) are the same everywhere and aren't covered here.
---

# Claude Code tools — reference

The harness-specific tools `system-prompt.md` relies on, for when you are running inside **Claude Code**. The main prompt only names capabilities ("ask the user", "preview", "screenshot", "debug"); this doc gives the exact Claude Code tool, signature, and call pattern. Generic tools (`Bash`, `Read`/`Write`/`Edit`/`Glob`, `gh`) are the same everywhere and aren't covered here.

## Web tool → Claude Code tool map

The upstream prompt references Claude.ai web tools that do not exist in Claude Code. Substitute as follows everywhere — prose and code alike:

| Web tool | Claude Code equivalent |
|---|---|
| `questions_v2` | `AskUserQuestion` (returns answers inline; up to 4 questions/call, follow-up call if more needed) |
| `done`, `fork_verifier_agent` | `SendUserFile` + the Claude Preview MCP; an `Agent` subagent (prompt: [`../agents/fork-verifier-agent.md`](../agents/fork-verifier-agent.md)) for thorough checks — see "Verification & debug" below |
| `write_file` (and its `asset:` param) | `Write` — drop the "asset review pane" concept entirely |
| `copy_files` | `Bash cp` |
| `read_file`, `list_files`, `view_image` | `Read` (it renders images too), `Glob` / `Bash ls`, `Grep`; before using `Read` on image files, run the vision probe below |
| `show_to_user` | `SendUserFile` (or `open <path>` for a self-contained file); for final deliverables also give the served `http://localhost:<port>/...` URL. Surface screenshots only after the vision probe passes; otherwise provide screenshot file paths without reading them (see "Showing files & preview"). |
| `eval_js`, `eval_js_user_view`, `run_script` | `Bash`; the Claude Preview MCP `preview_eval` for in-page JS |
| `web_fetch`, `web_search` | `WebFetch`, `WebSearch` |
| `copy_starter_component` | `Bash cp starter-components/<file> designs/<project>/` (or `Read` + adapt) |
| `invoke_skill("X")` / `invoke the "X" skill` | `Read` the matching `built-in-skills/<file>.md` |
| `gen_pptx` | `Bash`: serve the deck over HTTP, write the gen_pptx input object to a JSON file, then `node <skill>/agents/gen-pptx/dist/cli.mjs --url <servedDeckUrl> --config <jsonPath> --out <dir>` — see "Exporting to PPTX" below |
| `/projects/<projectId>/<path>` | ordinary filesystem paths (relative to cwd, or absolute) |

_Video export has no web-tool equivalent — see "Exporting to video" below._

## AskUserQuestion (clarifying questions)

Replaces `questions_v2`. `AskUserQuestion` **returns the user's answers inline** — ask, then continue once they respond. It shows up to 4 questions per call; for a large new project, ask a focused round and make a follow-up call if you need more.

- A remembered preference may be offered as a *suggested* default inside a question, but the user still confirms.
- Prefer it over listing choices as text bullets in your reply.
- The project-setup prompts — **where to save** the project and **which design system(s)** to use (a multiSelect; see [`use-design-system.md`](../built-in-skills/use-design-system.md)) — are ordinary `AskUserQuestion` calls.

## Showing files & preview

To surface a deliverable, use `SendUserFile` with the file path (works for any file type — HTML, images, text). Reading a file does NOT show it to the user.

**For final design/prototype deliverables, treat the preview as part of delivery, not only private validation.** Claude Code has no shared, user-visible browser to flip on (the Claude Preview MCP is agent-driven), so make the result visible by handing it off: `SendUserFile` the deliverable and give the user the served `http://localhost:<port>/<project>/<file>.html` URL so they can open and interact with the live prototype in their own browser. If the vision probe passes, also surface a final `preview_screenshot` (it renders inline in the transcript). If the probe does not pass, save any screenshot to disk and report its path without reading or embedding it. Do this after verification, unless the user asked you not to.

To open a prototype in a browser — whether for the user to interact with or for you to preview/screenshot it — **always serve it over HTTP and load the `http://localhost:<port>/<project>/<file>.html` URL; do not open the HTML directly from `file://`.** A multi-file prototype (an HTML entry that loads `<script type="text/babel" src="…jsx">` components) only works over HTTP — the browser blocks cross-origin local script reads — and self-contained single files go through the same served URL so preview and screenshots stay consistent.

Serve the whole `designs/` directory once (one server for all projects) and reuse it. Preview through the Claude Preview MCP, which serves from a named config in `.claude/launch.json`: define a single `designs` server that serves the whole `designs/` directory (`python3 -m http.server 4311 --directory designs`) so every project shares one server.

## Vision input probe


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JimLiu/baoyu-design](https://github.com/JimLiu/baoyu-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
