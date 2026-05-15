---
trigger: always_on
description: githut is a terminal TUI for discovering, browsing, and acquiring GitHub repositories.
---

# githut — CLAUDE.md

## What is githut?

githut is a terminal TUI for discovering, browsing, and acquiring GitHub repositories.
It is NOT a local git manager (that's lazygit). It talks to the GitHub API.

Primary use case: you're on a headless server, SSH session, or just don't want to open
a browser. You want to find a repo, read its README, clone it or fork it — all from
the terminal, keyboard-driven.

## Stack

- Language: Rust (stable)
- TUI: ratatui + crossterm
- Async: tokio
- GitHub API: octocrab
- Git operations: git2 (regular clone) + git CLI shell-out (sparse clone)
- HTTP: reqwest (raw file download fallback for files >1MB)
- Markdown rendering: pulldown-cmark (custom ratatui renderer in src/markdown.rs)
- Error handling: anyhow
- Serialization: serde + serde_json
- URL encoding: urlencoding
- Base64 decoding: base64

## Project Structure

```
githut/
  src/
    main.rs          -- entrypoint, boots tokio runtime, initializes terminal, runs app
    app.rs           -- App struct, central state machine
    types.rs         -- shared types (Repo, SearchResult, AppState, etc.)
    git.rs           -- local git operations (clone, sparse-clone)
    api/
      mod.rs         -- re-exports GithubClient
      auth.rs        -- shells out to `gh auth token` to get token
      client.rs      -- GitHub API calls (search, readme, contents, file) via octocrab
    tui/
      mod.rs
      ui.rs          -- all ratatui rendering logic
      events.rs      -- keyboard input handling, event loop
    markdown.rs      -- custom markdown → Vec<Line> renderer (pulldown-cmark)
    input.rs         -- TextInput struct: cursor movement, arrow keys, path expansion
  Cargo.toml
  flake.nix
  CLAUDE.md
```

## Layout

```
+----------------------------------+-----------------------------+
| [ search: ______________ ]        |                             |
+----------------------------------+  README preview              |
| owner/repo-name      Rust  4.2k  |  (rendered markdown)        |
| short description here...        |                             |
| > owner/selected-repo  Go  891   |                             |
|   description of selected...     |                             |
+----------------------------------+-----------------------------+
 /:search  j/k:nav  J/K:scroll  l:files  c:clone  o:browser  q:quit
```

Left pane: search bar + scrollable results list (50% width)
Right pane: README auto-loads on selection (50% width), debounced 300ms
Bottom bar: keybindings hint

File browser mode (activated with `l`):
```
+----------------------------------+-----------------------------+
| Files — owner/repo/src/          |  file content / preview     |
+----------------------------------+                             |
| ▶ api/                           |                             |
| ▶ tui/                           |                             |
|   main.rs                        |                             |
| > app.rs                         |                             |
+----------------------------------+-----------------------------+
 j/k:nav  J/K:scroll preview  l:open  h:up/back  Esc:back  q:quit
```

## Keybindings

Controls are consistent across all modes.

| Key         | Action                                        |
|-------------|-----------------------------------------------|
| /           | Focus search input                            |
| Enter       | Confirm search                                |
| j / k       | Navigate list (repos or files)                |
| J / K       | Scroll preview pane (readme or file content)  |
| l / Enter   | Open file browser / enter dir / preview file  |
| h           | Go up one dir; at root, back to repo list     |
| Esc         | Back / close overlay                          |
| c           | Clone selected repo / save file (in FileBrowsing) |
| C           | Sparse-clone (prompts path + dirs)            |
| f           | Fork selected repo                            |
| s           | Star / unstar selected repo                   |
| o           | Open selected repo in browser                 |
| r           | Refresh / re-fetch results                    |
| ?           | Toggle help overlay                           |
| q           | Quit                                          |

## Auth

Auth is handled by shelling out to `gh auth token`.
This avoids any OAuth setup — user just needs `gh` installed and authed.

```rust
// auth.rs pattern
let token = std::process::Command::new("gh")
    .args(["auth", "token"])
    .output()?;
let token = String::from_utf8(token.stdout)?.trim().to_string();
// pass to octocrab builder
```

If `gh` is not installed or not authed, githut should show a clear error and exit.

## GitHub API Usage

All calls go through octocrab. Key endpoints:

- Search repos: `GET /search/repositories?q=...`
- Get README: `GET /repos/{owner}/{repo}/readme`
- Get contents (dir listing): `GET /repos/{owner}/{repo}/contents/{path}`
- Get file content: `GET /repos/{owner}/{repo}/contents/{path}` (single file)
- Fork repo: `POST /repos/{owner}/{repo}/forks`
- Star repo: `PUT /user/starred/{owner}/{repo}`
- Unstar repo: `DELETE /user/starred/{owner}/{repo}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karimKandil0/githut](https://github.com/karimKandil0/githut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
