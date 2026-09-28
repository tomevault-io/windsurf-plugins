---
trigger: always_on
description: @.claude/rules/branch.md
---

# CLAUDE.md — LoopBoard

@.claude/rules/branch.md
@.claude/rules/tests.md

VSCode extension: renders workspace `.loopboard/` tracker as interactive board, writes edits back
to markdown, spawns model-specific Claude Code loop terminals. Decisions: `decisions/` (index
`DECISIONS.md`); verification status: `VERIFICATION.md`.

Storage: everything under `.loopboard/` — `TODO.md` (slim task index, grammar v5), `DONE.md`
(accepted, lazy), `LOOP.md` (rules + loop worker instructions), `tasks/<id>.md` (per-task detail).

## Non-negotiable

1. Docker-only toolchain — NEVER install on host (no `npm`/`brew`/`pip`/`apt`; never type
   `npm`/`npx`/`node` directly). Everything runs via `make` → `docker run node:22`. Host has only
   Docker, `make`, git, VSCode. Tool missing from node:22 gets its own image.
2. Zero runtime dependencies. devDependencies = exactly `typescript` + `@types/vscode`. No
   `@types/node`, no bundler, no frameworks; webview = vanilla HTML/CSS/JS.
3. `.loopboard/` markdown = source of truth. Index (`TODO.md`, grammar v5) carries only
   id/phase/model/groomer/questions/feedback per entry; every other field lives in `tasks/<id>.md`.
   Parse tolerantly, write back canonical on every save, preserve unparseable lines verbatim
   (flagged in UI). Grammar + task-file format documented in `LOOP.md`.
4. ALL markdown IO goes through `src/store.ts` — only module that knows `.loopboard/` paths; merge
   logic in exactly one place (`merge.ts`, `patchTarget` routes index vs detail). Saves are
   field-level patches on ONE file: re-read disk, re-parse, apply one field, serialize whole file,
   atomic write (temp + rename). Same-field conflict → disk wins + toast.
5. Board performs ONLY three human actions: promote (New→Backlog on tick), accept (Review→DONE.md
   on tick), demote (Backlog→New, immediate button click, non-destructive). Everything else is a
   field patch the loop reacts to. Never auto-move tasks optimistically.

## Commands

```
make install         # npm install in Docker
make build           # tsc -> out/                 (extension host)
make test            # tsc tsconfig.test.json -> out-test/, node --test 'test/*.test.js'
make package         # vsce package -> loopboard-todo-<version>.vsix
make check           # build + test (no .vsix) — MUST pass before any commit
make check PACKAGE=1 # build + test + package — opt-in packaging check on the same command
make clean
```

Any `src/**` change requires `make test` + `make check` green before it counts as done.

## Architecture

- Pure modules (unit-tested in Docker; must NEVER import `vscode` or node typings): `parser.ts` +
  `writer.ts` (index file), `taskfile.ts` (per-task detail file), `model.ts`, `merge.ts`,
  `gates.ts`, `loop.ts`, `view.ts` — compiled by `tsconfig.test.json` (`types: []`) into
  `out-test/`.
- VSCode-touching (manual F5 verification only): `extension.ts`, `store.ts`, `controller.ts`,
  `panel.ts`, `sidebar.ts`, `terminals.ts`, `webview.ts` — main `tsconfig.json` → `out/`.
- Webview assets: `media/board.{html,css,js}`, `media/sidebar.{html,css,js}` — vanilla JS, CSP
  nonce, VSCode theme variables only.
- Keep new logic pure/testable; wrap `vscode` imports as thinly as possible.

## Critical learnings (do not rediscover)

- No `@types/node`: tests are plain CommonJS `.js` in `test/` requiring `out-test/`; main tsconfig
  adds lib `DOM` (not @types/node) for `setTimeout`/`TextDecoder`.
- `node --test` needs glob `'test/*.test.js'` — bare `test/` dir arg is treated as module path
  (Node 22).
- Core invariant: parse→write→parse is a fixpoint for BOTH files; `serializeTodo(parseTodo(x))` and
  `serializeTaskFile(parseTaskFile(x))` idempotent as text. Any parser/writer change keeps fixture
  suite green (incl. index fixture with an HTML-comment template — task-like `- [ ]` lines inside
  comments must not parse as entries).
- Emoji canonicalization: index parser strips leading ❓ from question text; task-file parser strips
  ⚠️ from `## Feedback`; writers re-add them.
- Task-file heading order: `## Meta` → `## Problem` → `## Description` → `## Goals` → `## Worklog`
  → `## Delivered` (t-2191). Problem/Goals are groomer-owned free markdown (no length/shape
  validation) and the yardstick review judges `## Delivered` against; all three story sections share
  ONE board renderer (`renderDetailSection`/`DETAIL_SECTIONS`), attachments stay Description-only.
- Task file `tasks/<id>.md` is eager-scaffolded on draft create (t-6ab4: `store.createDraft` writes
  a skeleton — `added: <today>`, everything else empty and so omitted by `serializeTaskFile`, same
  shape the writer already canonicalizes an empty detail to). Missing file (only possible for
  pre-t-6ab4 entries or one deleted out-of-band) still parses as empty detail and card still shows
  the "no detail file yet" hint — no longer fires for ordinary new drafts. Writer rewrites H1 from
  index title on every task-file save (index title wins on divergence).
- Webview vs concurrent loop writes: board defers incoming refresh while a field is focused
  (`pendingBoard`, flushed on focusout). Model select normalizes `default (opus)` → `''` before
  patching so an unchanged default never trips a false same-field conflict.
- Index `## Tasks` heading + HTML-comment extras round-trip verbatim via `getTasksHeading` /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SinnConsulting/LoopBoard](https://github.com/SinnConsulting/LoopBoard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
