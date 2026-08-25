---
trigger: always_on
description: Project-specific rules for working in this repo. These are learned conventions, not
---

# NO XMFD — Development Practices

Project-specific rules for working in this repo. These are learned conventions, not
guesses — most exist because a past session violated them.

## Git workflow

- **Bug fixes** → commit directly to `main`. No branch.
- **Features** → a dedicated feature branch (e.g. `feature/wpn-weapon-select`), merged
  into `main` when ready. **Delete the branch (local + remote) immediately after
  merging** — do this in the same sitting as the merge, before moving on to
  versioning/release work.
- **"push" and "merge to main" are separate authorizations — "push" never implies the
  other.** "push" (on a feature branch) pushes that branch only, exactly as-is; it does
  not fast-forward-merge into `main`, and does not push `main`. Merging into `main`
  needs its own explicit instruction ("merge to main", "merge it in") given
  separately, even right after a "push" in the same turn. Default assumption on any
  feature branch is "stays a branch" until that explicit merge instruction arrives.
- **Never push unless the user's current message literally contains the word "push"**
  (or "upload"/"publish to remote"). This is the single most-violated rule in this
  project (10 documented incidents). None of the following authorize a push: "yes",
  "ok", "looks good", approving a proposed plan, or a content instruction like "update
  the readme" — even if that plan's last step was a push.
  - `git push` must never appear in the same tool call as `git add`/`git commit` —
    not even separated by a heredoc terminator with no `&&`. Before submitting any git
    command, scan the whole string for "push"; if it co-occurs with add/commit, split
    it out and drop the push.
  - Default end-of-turn state for any git-touching turn is **"committed, not
    pushed"** — say so explicitly and wait for the word.
  - A "push" said earlier in the session does not carry forward to later turns, even
    minutes later in the same conversation.
- **No pull requests.** Don't open one, don't offer one, and after a push don't
  mention/paste the "Create a pull request" link GitHub prints — just report the push
  landed.

## Scope discipline

- Build only what was asked. An "open question" noted in a planning doc is a question
  to *ask* the user — not license to pick an answer and ship it.
- No abstractions, boilerplate, or features beyond the explicit request, even when
  adjacent and seemingly obvious (e.g. don't add a new toggle/config/UI control that
  wasn't asked for while implementing a related feature).

## Build & preview

- The frontend lives in real files under `src/web/` (`shell/`, `pages/<x>/`,
  `shared/`) — not embedded in C# strings. `dotnet build` only validates C# routes and
  the embedded-resource manifest; it does **not** validate JS/CSS.
- Verify any frontend edit by loading it in the `tools/serve_web.py` HTTP harness
  (default port 8782, `.claude/launch.json` config `hud-web`). `tools/build_preview.py`
  is stale/obsolete for this purpose.
- Launch serve_web with PowerShell `Start-Process` (a Bash background job dies when
  the tool call ends):
  ```
  Start-Process -FilePath "python" -ArgumentList "tools/serve_web.py","--port","8782" -WorkingDirectory "<repo-root>" -WindowStyle Hidden -PassThru
  ```
  Start it once per session and leave it running — it reads files off disk per
  request, so edits are picked up live. Never stop it as a "cleanup" step after
  verifying; restart only if `serve_web.py` itself changed. If a port check is needed,
  use `Get-NetTCPConnection -LocalPort 8782 -State Listen` (port-based) to find the
  real owner — process-name filters can miss it (e.g. Windows Store Python).
- Use the shared `src/web/shared/theme.css` `var(--no-*)` color tokens instead of
  hardcoding hex values (`--no-green`, `--no-red`, `--no-amber`, `--no-bg`,
  `--no-panel-bg`, `--no-panel-border`, `--no-ink`, etc.). Deliberate literal-color
  exceptions are fine when called out (e.g. a true-black power-off state).
- Perf A/B session reports go in `_scratch/perf-sessions/*.txt` (gitignored) — never
  commit or push them. The PerfLogging feature code and its docs are committed
  normally.
- After a significant code change, `dotnet build -c Release` and let the
  `DeployToGame` MSBuild target copy the fresh DLL into the real game's
  `BepInEx/plugins` folder without waiting to be asked. This is a reversible, local
  action (overwrites a DLL on disk) — it's a separate gate from `git push`, which
  still needs the literal word per the Git workflow rules above.

## Docs & comments

- Present tense, current-state only. Don't describe past changes or migrations ("X
  used to live here") — just state what's true now.
- No reader-directed meta-commentary ("this is the part people miss", "you'd be
  surprised how often"). State the fact plainly; justifying a design choice is fine,
  editorializing about the reader is not.
- Comments explain **why**, not what: a non-obvious choice, a runtime/engine/API
  constraint, an invariant a future edit must preserve, or a threading/lifecycle/
  ordering/caching assumption. A workaround's comment says when it can be removed.
  Don't restate what the code already says, don't excuse unclear code instead of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roke77/NOXMFD](https://github.com/roke77/NOXMFD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
