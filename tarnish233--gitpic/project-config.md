---
trigger: always_on
description: `gitpic` is a Rust CLI. Source lives in `src/`:
---

# Repository Guidelines

## Project Structure & Module Organization
`gitpic` is a Rust CLI. Source lives in `src/`:
- `main.rs` — entry point, subcommand dispatch, exit codes.
- `cli.rs` — clap argument/subcommand definitions.
- `config.rs` — config model + XDG path resolution (`~/.config/gitpic/config.toml`).
- `auth.rs` — the credential: reads/writes the 0600 `auth.toml` that `gitpic auth login` produces, and is the only source there is. No secret is held in `Config`.
- `oauth.rs` — GitHub device flow: the wire protocol behind `gitpic auth login`.
- `github.rs` — GitHub Contents API client (upload, dedup, health checks).
- `naming.rs`, `link.rs`, `imageproc.rs`, `output.rs`, `error.rs` — path/hash, URL/markdown, compression, human/JSON output, error types.
- `history.rs` — the upload log the app's 历史 pane reads.
- `release.rs` — the update check behind `gitpic update check`: version parsing and comparison, the `releases/latest` fetch, and the release's assets (name, size, download URL, GitHub's `digest`) that the app installs an update from. The origin is a compile-time constant on purpose, pinned by a test — its text is rendered inside GitPic's own window, so nothing configurable may choose it, and download URLs come from the API rather than a template for the same reason.
- `install_source.rs` — which of the three ways this binary was installed (inside GitPic.app, `cargo install`, or unknown), so `gitpic update` prints the one upgrade command that install actually wants instead of two to choose between. Canonicalises `current_exe()` first: the app links `~/.local/bin/gitpic` into the bundle, so the commonest install of all is invoked through a symlink, and on Apple platforms the un-canonicalised path is that symlink — which classifies it as neither an app nor a cargo bin and prints "download it again" for the one install that updates itself.
- `testutil.rs` — `#[cfg(test)]` only: the loopback stub server, a canned response, and the request reader shared by `github`'s and `release`'s tests. One `sock.read` is not a whole request; the module says what that cost twice.
- `commands/` — one module per action (`upload`, `auth_cmd`, `repos`, `branches`, `doctor`, `list`, `config_cmd`, `completion`, `skill`, `update`).

Docs: `README.md` (中文, default), `README.en.md`, `skills/gitpic/SKILL.md` (agent
usage), `CHANGELOG.zh-CN.md` (中文, Release source), and `CHANGELOG.md` (English). Keep
both changelogs aligned for every release. CI lives in `.github/workflows/`.

**The `release-notes-end` marker splits two audiences, and the half above it is for
people deciding whether to upgrade.** Everything above becomes the GitHub Release body
*and* the app's update sheet; everything below stays in the file. So above the marker:
one theme heading and **two or three bullets of roughly one line each** — what changed,
in the user's terms, no mechanism. Aim for ~40 characters of 中文 or ~100 of English per
bullet; a released 0.20.3 bullet ran to 246 characters, which is a paragraph pretending
to be a summary. Everything that made it worth doing — the measurement, the design that
was rejected, the test that was wrong — goes *below* the marker and into the commit
message, which is where someone reading the code will look for it. Nothing is lost by
being brief up top; it is only moved to the reader who wants it.

**Distribution is one path: the DMG, then the app updates itself.** There is no Homebrew
cask, no `gitpic_cli` formula and no `tarnish233/homebrew-tap` — all three were retired
together, and nothing transitional was left behind because the only user is the author. Do
not reintroduce a brew install path without first saying why the in-app updater is not
enough.

The app asset is `GitPic-<version>-macos-arm64.dmg` — a disk image with an `/Applications`
symlink beside the app, so installing is the usual drag-across. It was a `.zip` up to 0.13.1.

**The quarantine flag is the one manual step, and it is not optional.** The app is ad-hoc
signed and not notarised, so a freshly downloaded copy refuses to open *at all* until
`xattr -dr com.apple.quarantine /Applications/GitPic.app`. The cask's `preflight` used to do
this silently and nothing does it now, which is why the release notes and both READMEs say it
immediately after the drag rather than as a footnote. Self-update is unaffected —
`SelfUpdateInstall` copies with `ditto --noqtn` and strips the attribute itself — so only a
fresh manual install ever needs the command.

**`releases/latest` is the single thread the whole thing hangs from.** It is what the app's
own update check polls, so a release flagged prerelease is a release no installed app can
see, silently and indefinitely. See constraint 3 in `release.yml`'s header.

**The terminal `gitpic` command comes from the app, on request.** 设置 ▸ 通用 ▸ 命令行 links
`~/.local/bin/gitpic` at the copy inside the bundle and writes three completions
(`~/.zfunc/_gitpic`, `~/.local/share/bash-completion/completions/gitpic`,
`~/.config/fish/completions/gitpic.fish`). Linking rather than copying is what keeps the
command and the app from ever being at different versions — the one property the cask
genuinely provided. `~/.local/bin` rather than `/usr/local/bin` because it is user-owned:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tarnish233/gitpic](https://github.com/tarnish233/gitpic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
