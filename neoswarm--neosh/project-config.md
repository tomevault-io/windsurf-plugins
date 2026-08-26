---
trigger: always_on
description: A terminal-first agent workspace: Rust core, TypeScript plugins, "the extensibility model of Neovim
---

# Working on neosh

A terminal-first agent workspace: Rust core, TypeScript plugins, "the extensibility model of Neovim
and the feature set of Cursor".

The thesis is that **extensibility is the product**. Every capability ships on the same public API a
third-party plugin author would use. If a feature needs a private escape hatch, the API is wrong and
the API is what to fix.

Which means the user owns the workspace, not us: every part of it is theirs to replace, and a plugin
of their own is how they do it. New windows, panels and floats; new commands and keys on any of
them; different colours, different layout, a sidebar that is nothing like ours — none of that is a
feature we grant, it is what the API is for. Ours are the defaults, and a default is something you
turn off.

## Layout

| Crate | Owns |
|---|---|
| `neosh-proto` | Every wire type. `#[derive(TS)]` on all of them; generated `.ts` is committed. |
| `neosh-core` | Buffers, windows, floats, extmarks, highlights, focus, keymaps, the single-writer `Editor`. No I/O. |
| `neosh-script` | The `deno_core` host and the op surface. The only crate that knows deno exists. |
| `neosh-provider` | `trait Provider`, the shared SSE parser, and one driver per transport. |
| `neosh-agent` | `Session`, `Turn`, the tool registry, the permission layer, the agent loop. |
| `neosh-syntax` | Syntax highlighting: what a token *is*, never what colour it is. The only crate that knows a grammar exists. |
| `neosh-tui` | Terminal frontend. Owns **all** display-width maths. |
| `neosh` | The binary. Wires the tasks, owns config, hosts the chat — and is both the workspace (`daemon.rs`) and the terminal that views it (`client.rs`). |
| `neosh-plugins` | `plugins/` itself. One file of Rust holding the `include_dir!`s, because the tree an `include_dir!` reads has to be under the package root or it embeds here and publishes empty. |

`plugins/api/` is `@neosh/api` — generated protocol types plus a hand-written ergonomic layer.
`plugins/builtin/` are ordinary plugins that happen to ship in the binary. The directory is also a
crate (`plugins/Cargo.toml`, `plugins/lib.rs`) and nothing moved to make it one: `cargo package`
carries only what is under a package root, so a `neosh-script` embedding `../../plugins` built fine
in a checkout and would have uploaded a crate with no plugins and no API source in it. Publishing
is `docs/releasing.md`.

## Rules that are not negotiable

- **Anything neosh can put on screen, a plugin can put on screen — and take off it.** Windows,
  splits, floats, docked panels, extmarks, highlight groups, status segments, commands, keymaps and
  options are all public API, and every bundled plugin is written against exactly that: the sidebar,
  the model switcher, the palette, git, approvals. The test for a new feature is "could a third
  party have written this, and can they replace it?" — if the answer needs an op nobody else may
  call, the op is what to fix, not the feature. In practice that means a new capability lands as
  `ApiCall` + generated TS + a plugin that uses it, in that order; anything the binary draws for
  itself must also be disable-able (`plugins.disabled`) and overridable by a plugin that loads
  after it and wins.
- **Unicode width is not optional.** `unicode-width` + `unicode-segmentation`. Column maths on
  `char` counts corrupts layout on day one. Columns on the wire are UTF-8 byte offsets; motion steps
  by grapheme cluster; every display-width question is answered in `neosh-tui`.
- **No `unwrap()` on anything touching plugin input or a network response.**
- **Secrets never touch disk in plaintext and never appear in logs or event payloads.** They are
  `secrecy::SecretString`, read at request time, and structurally absent from `UiEvent`.
- **Every public API has a TS type generated from or checked against the Rust side.** Drift fails
  the build.
- **Streaming markdown caches to the last *complete* block boundary** and re-parses only the
  trailing partial block.
- **Which way a diff line went is a background; what the code says is the foreground.**
  `ExtmarkOpts::line_hl_group` bands the whole row and every ranged group is patched over it. A
  `Diff.*Line` group that sets a foreground silently un-highlights the code on it. Colours for code
  come from `neosh-syntax`, which maps grammar scopes onto `Syntax.*` and never loads a theme of its
  own — the palette owns colour, here as everywhere.
- **A panel is a surface, not a program.** Anything a bundled plugin draws, a third party must be
  able to add to and take keys in without forking it. Three mechanisms, and a new panel of ours uses
  all three or it is not finished: a buffer **kind** (`buf.create({ kind })`), which is what
  `KeymapScope::BufKind` binds against and what `win.list` finds it by; **contribution points**
  (`ext.contribute`/`ext.list`), which is how rows and verbs arrive from plugins we have never heard
  of; and **shared vars** (`vars`, scoped to the workspace, a conversation or a project) for anything
  about a thing rather than about us — `state` stays private and a favourite is not private. Every
  key in a panel is an ordinary binding pointed at a named command, so `^Z` lists it and `init.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neoswarm/neosh](https://github.com/neoswarm/neosh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
