---
trigger: always_on
description: Zotero 10 plugin that adds voices to Zotero's built-in **Read Aloud**: OpenAI
---

# CLAUDE.md — zotero-tts

Zotero 10 plugin that adds voices to Zotero's built-in **Read Aloud**: OpenAI
(or any OpenAI-compatible server), Azure Speech, and a local Kokoro-FastAPI.
Zotero's own Standard/Premium voices keep working; ours join the Local tier as
`TTS-<Provider>-<voice>`. Also: shortcuts for the speed (Shift+Z/X/C) and for
skipping by sentence / paragraph (arrows / Shift+arrows), one voice and speed
across documents, settings backup/restore (file or WebDAV), highlight colors.

- `README.md` — user-facing docs, kept short and scannable (header with
  badges, a GIF, emoji feature list, install, providers table, settings;
  long detail goes into `<details>` blocks or `tutorials/`, never into the
  body). **After editing any `.md` under the repo, run `npm run docs`** — it
  renders every doc to `docs/` (gitignored), the source tree mirrored; tell
  the user which `docs/….html` to open.
  `PHILOSOPHY.md` — the plugin is an enhancer for Read Aloud, not a
  replacement; the yardstick for new features.
  `tutorials/` — Azure's free tier, Kokoro-FastAPI and Chatterbox-TTS-Server
  in Docker, remote access through Cloudflare. Provider and server how-tos
  go there, not into the README.
- `notes/NOTES.md` — engineering notes: every production incident, and the Zotero
  internals verified by reading its source. **Read it before touching Read
  Aloud internals; append to it (English) whenever something non-obvious is
  learned or broken.** `notes/NOTES_SUPPLEMENTARY.md` — one-time setup logs and
  overtaken lists moved out of NOTES.md; history, not required reading.
  Both live in `notes/`.

## Working with the user

- Reply in **Chinese**, always, whatever language the input (logs, source,
  instructions) is in. Everything else is **English only, American
  spelling** (color, gray, license, -ize) — code, identifiers, comments,
  commit messages, README, NOTES.md. Verbatim
  product strings (Azure's "多语言" voice names, the rendered "多语种"
  label) and test fixtures for the Chinese collation are data, not prose,
  and stay as they are.
- Architecture-level forks: present the options with concrete costs and a
  recommendation, then wait. Implementation details: pick the sane default,
  state it in one line, move on.
- Research first when asked to: report findings, do not change code.
- Commit only when told ("commit"); push only when told. Simple work
  (docs, wording, one-liners) goes directly on `main`; only hard work
  (bug fixes, features) gets a `feat/...`/`fix/...` branch, merged into
  `main` with `--ff-only` when told. Commit
  messages: `feat: …` / `fix: …` / `docs: …` / `chore: …`, as short as
  possible — usually the subject line alone; a body only when the why is
  not obvious, and then a line or two, never a long description. No
  `Co-Authored-By` trailer.
- Before any push: scan the history for keys (`sk-…`, 32-hex, `ghp_…`).
  Never echo an API key into the conversation or a file. Zotero's
  `prefs.js` holds the user's keys in plaintext — grep it only for the exact
  pref you need, never print whole lines.
- **Git housekeeping is delegated** (settled 2026-08-28): committing what
  is in the working tree, deleting merged branches locally and on origin,
  tagging, pushing, and `--ff-only` merges run in the `git-chores` agent
  (`.claude/agents/git-chores.md`: Opus at maximum reasoning effort, this
  file's rules) — `Agent` with `subagent_type: "git-chores"`, told exactly
  what to commit, with which message, and what to leave in the working
  tree. The main session confirms its report against `git log` / `git
  status`. Only a merge that does not fast-forward — conflicts, a
  diverged `main` — stays in the main session, and a resolution that is a
  judgment call is asked about. Agent definitions under `.claude/agents/`
  are read when a session starts: one added mid-session is "not found"
  until the next session — then use `general-purpose` with
  `model: "opus"` and the agent file's rules pasted into the prompt.

## Commands (Node 22, ESM)

```
npm test              # vitest, ~340 tests, includes test/build.test.ts which runs the build
npm run typecheck     # tsc --noEmit
npm run build         # esbuild → addon/content/zotero-tts.js, zip → build/zotero-tts.xpi
npm run docs          # pandoc → docs/*.html, the Markdown rendered for the browser
```

After any change: tests, typecheck, build — then the xpi goes into the
running Zotero through the zotero-dev bridge (`zotero_plugin_install`
upgrades it in place, no restart) and the live verification below runs
there.

**Test builds carry a beta version.** Every xpi built for testing sets
`addon/manifest.json`'s `version` to the next version plus `-beta`
(`1.7.4-beta`), and a second test build of the same version counts up
(`-beta2`, `-beta3`, …), so Tools → Plugins shows at a glance which build is
actually installed. `package.json` stays at the released version — it is not
what Zotero displays, and leaving it alone keeps the lock file out of it.

**Every build is verified by its mechanism, not only by its visible
effect** — a diagnostic that proves the code path ran, with the expected
output stated before it runs. A fix that merely looks right may be working

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xujialiu/Zotero-TTS](https://github.com/xujialiu/Zotero-TTS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
