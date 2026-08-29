---
trigger: always_on
description: A Claude Code **plugin marketplace**. `.claude-plugin/marketplace.json` lists the
---

# CLAUDE.md — patrickdappollonio/claude-plugins

A Claude Code **plugin marketplace**. `.claude-plugin/marketplace.json` lists the
plugins under `plugins/`.

## Hard rules

- **Never add Claude/AI attribution** to commits or PRs — no `Co-Authored-By:
  Claude`, no "Generated with Claude Code". This is a standing, non-negotiable
  constraint from the repo owner.
- **Commit/push only when asked.** Work on a branch, not `main`.
- **Bump the plugin's version on every meaningful change**, in its
  `plugins/<name>/.claude-plugin/plugin.json`, following
  [semver](https://semver.org/): **patch** for fixes and doc/wording tweaks,
  **minor** for new backwards-compatible capability (a new command, fence, or
  skill behavior), **major** for breaking changes (removed/renamed commands,
  changed file formats or defaults users rely on). Pure repo chores that don't
  touch a plugin (root README, CI) don't need a bump. The version lives **only** in that
  `plugin.json` — there are no git tags, GitHub releases, or per-plugin
  versions in `marketplace.json` to check or update, so don't go looking.
- **Both install paths must stay equivalent: Claude Code marketplace *and*
  `npx skills`.** A Claude Code install brings a whole plugin (every skill under
  `plugins/<name>/skills/`); `npx skills add … --skill <name>` installs **one
  skill directory and nothing above it**. So a skill must be **self-contained**:
  everything it needs lives inside its own `skills/<skill>/` directory, and
  nothing in `SKILL.md` may reach outside it — no `../` paths, no "see the other
  skill's file", no reliance on a sibling skill being present, and no symlinks
  (a symlink out of the directory arrives as a plain text file on a Windows
  checkout with `core.symlinks=false` — a silent, broken install). Duplicating
  shared prose or code into each skill is the correct trade-off, as visual-docs
  does with the renderer. A plugin may hold several skills; each must work alone.
  When you add a skill, also add it to the root README's `--skill` name table,
  and phrase cross-skill references so they degrade to a name a reader can
  install ("the full `adversarial-review` skill"), never a relative path.
- **Skills: extract whatever can be read on demand, but always keep the
  general guideline inline — otherwise the extraction is moot.** The whole
  `SKILL.md` loads into context on every invocation, so keep it under ~500
  lines. It holds what steers behavior on *every* pass — principles, the
  process outline, decision tables, rationalizations, red flags, the checklist
  — plus a short, self-sufficient summary of every extracted topic (the rule,
  the threshold, the one-line procedure). The comprehensive version — full
  procedures, routing tables, handoff templates, digests of other skills — goes
  in a `.md` file **in the same skill directory**, and the summary says exactly
  when to read it ("read `cyclomatic-complexity.md` before splitting a
  function"). An agent that never opens the side file must still do the right
  thing from the summary alone. Load-bearing discipline rules stay inline in
  full — an agent under pressure will skip a side file. Reference sibling
  files by bare filename, never a path, so both install paths ship them.
  And the skill must **insist that the agent read every companion file on
  first use in a session, before any other step** — a dedicated section near
  the top listing the files, plus a red flag and a checklist item — and re-read
  the named file at its step. Summaries are reminders of text already read,
  never a substitute. `code-simplification` is the worked example.
- The visual-docs server has **no authentication**. `--host` / `0.0.0.0` binding
  must stay opt-in and documented as trusted-network-only.
- **Agents must never have to write code — not even a small script — to operate
  visual-docs.** Every operation an agent performs (resolve a directory, serve,
  read comments, set status, lint, stop) must be a first-class `node
  visual-docs-server.js --<command>` / `visual-docs-lint.js` invocation that
  prints **ready-to-read, formatted text** (never JSON the agent must parse, and
  never something it has to `curl`/`jq`/`python`/`node -e` to consume). Keep these
  commands **OS-agnostic as much as possible**: do the platform-specific work
  (temp dirs via `os.tmpdir()`, backgrounding via detached `spawn`, HTTP via
  `fetch`) *inside* the Node CLI, so the skill only ever runs `node <script>
  <args>` — no `nohup`/`&`, no `/tmp` paths, no shell-only constructs. When you
  add an agent-facing capability, add a `--command` for it; don't make the agent
  improvise glue.

## visual-docs architecture

A zero-dependency reader you serve locally. Nothing is remote.

**Where the code lives — read this before editing anything under
`plugins/visual-docs/`.** The canonical renderer is `extras/visual-docs/`
(`server/` + `shared/`), which sits outside every plugin so it is never itself
installed and never has to be chosen between. Each skill that needs it carries a
**real, committed copy** at `plugins/visual-docs/skills/<skill>/{server,shared}`.
That duplication is deliberate: `npx skills` installs a skill directory and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickdappollonio/claude-plugins](https://github.com/patrickdappollonio/claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
