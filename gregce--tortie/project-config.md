---
trigger: always_on
description: Electron + tmux shell for agentic coding. Product philosophy + name: docs/ZEN-OF-TORTIE.md. Architecture authority: docs/audits/2026-08-20-electron-typescript-architecture.md. Design authority: DESIGN.md + docs/DESIGN-SPEC.md. Work queue: docs/BACKLOG.md.
---

# Tortie — agent conventions

Electron + tmux shell for agentic coding. Product philosophy + name: docs/ZEN-OF-TORTIE.md. Architecture authority: docs/audits/2026-08-20-electron-typescript-architecture.md. Design authority: DESIGN.md + docs/DESIGN-SPEC.md. Work queue: docs/BACKLOG.md.

## The name (Phase 16.5, bundle id changed again in Phase 27)
The product is **Tortie** (`com.itavero.tortie` since Phase 27, `com.specstory.tortie` before that, `~/Library/Application Support/Tortie`). Tortie belongs to Ita Vero, LLC, the operator's company; the SpecStory INTEGRATION (the bundled specstory binary, src/main/specstory, the capture surfaces) keeps its name because it is a separate product Tortie talks to — never "finish off" that rename. The data directory follows `app.setName`, not the bundle id, so the Phase 27 id change moved no data. It was `gmux` until Phase 16.5, and much of the codebase's PROSE still says so — that is fine and deliberate. What is NOT prose, and must never be "finished off" by a later cleanup, is the set of identifiers live data is bound to: the tmux socket `-L gmux`, `resources/gmux-tmux.conf`, the `@gmux-*` session options, the `GMUX_SESSION_ID`/`GMUX_MANAGED` pane env, the inner `<userData>/gmux/` directory, the `window.gmux` bridge, the `gmux-asset:` scheme, `gmux.*` localStorage keys and `gmux-*` CSS classes. Renaming any of the first five strands sessions that are running right now. DEVELOPMENT.md has the full table and the reasons (README.md is product-facing). **User-visible copy is the only place the name may appear, and there it is always "Tortie".**

## Architecture invariants
- Sessions live in the PRIVATE tmux server (socket `-L gmux`, config resources/gmux-tmux.conf). The app is a disposable client. Never move durability-critical state into the app.
- Address live tmux sessions by immutable `$-id` (or `=`-exact name match), never bare names.
- The manifest (SQLite, main/manifest) is the source of truth for restore: argv + resume_argv always use ABSOLUTE binary paths. Agents are nonetheless LAUNCHED by bare name (Phase 12.7 F3): an absolute argv[0] made every durable gmux agent the one process on the machine that `pkill -f "$(command -v claude)"` matches. tmux's execvp finds the binary because the login-shell PATH is injected into the server env.
- Sessions are addressed by IDENTITY, never by name: `@gmux-id` (plus the `GMUX_SESSION_ID` pane-env stamp as the second source). A live session that carries neither is NOT OURS — never adopt it, never kill it.
- tmux SAFETY: only ever `tmux -L gmux`. Never touch the user's default tmux server, ~/.tmux.conf, or kill sessions you didn't create.

## Tortie never loads third party code (Phase 23) — the permanent refusals
These bind every future round the way the tmux safety rules above do. They are the outcome of docs/research/31-extensions.md, which examined bb, Zed and pi, wrote four competing architectures and had three adversaries attack each one. Eleven of the twelve reviews came back fatal. The single line that ended all of them is the first refusal.

The boundary, and it is the whole design:
> Configuration selects from choices the compiled world already contains, or names an executable the user has personally confirmed.

1. **No third-party JavaScript, TypeScript, WebAssembly or native code executes in any Tortie process.** Not main, not the renderers, not the preload, not a worker, not a `utilityProcess`.
2. **No `tortie.d.ts`, no SDK package, no contribution-point registry.** If a proposal begins "we will expose an interface so extensions can…", it is this refusal. bb froze 65 component prop types into a public contract and deleted it the next day.
3. **No marketplace, no store UI, no in-app browse-and-install, no update badge, no extension count on the activity rail.**
4. **No configuration mechanism may implement, replace, decorate or intercept** Explorer, SCM, search, the terminal, the tab spine, the manifest, the tmux layer, or Context's own data.
5. **No configuration mechanism may set a session's status.** Status semantics are in the UI rules below and they do not move.
6. **No third-party native code inside the signed bundle.** It would need `com.apple.security.cs.disable-library-validation` app-wide and permanently, against a configuration whose note reads "ZERO entitlements are needed".
7. **The main renderer's CSP is never relaxed.** Third-party HTML, if ever hosted, gets its own `session` partition and its own served CSP. `build/assert-preview-containment.mjs` asserts this at build time.
8. **Nothing may cause a process to start on a configuration change alone.** A human confirms the bytes, out of band of any agent turn, and the agreement is bound to a hash of the fields that decide what runs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gregce/tortie](https://github.com/gregce/tortie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
