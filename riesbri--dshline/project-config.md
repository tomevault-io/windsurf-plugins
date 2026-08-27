---
trigger: always_on
description: Instructions for an agent — or a person — working **on** this repository. If you want to *use* the interface, start at [`README.md`](README.md). If you want to send a change, read [`CONTRIBUTING.md`](CONTRIBUTING.md) as well.
---

# AGENTS.md

Instructions for an agent — or a person — working **on** this repository. If you want to *use* the interface, start at [`README.md`](README.md). If you want to send a change, read [`CONTRIBUTING.md`](CONTRIBUTING.md) as well.

Read [`docs/design.md`](docs/design.md) before changing anything about drawing, keyboard decoding, or text escaping. This file is the short version: the rules, the commands, and the mistakes that are easy to make.

## What this project is

A terminal interface for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness). It runs as a plugin inside the agent's own process, rather than as a client connecting over a network. There are two packages:

```
packages/renderer      @dshline/renderer   widths, keys, input line, boxes, screen — knows nothing about agents
packages/dshline       @dshline/dshline    the plugin: session loop, transcript, harness integration, view registry
packages/dshline/bin   dshline             a launcher wrapper, and deliberately nothing more
```

`bin/dshline.mjs` exists so that using this does not require remembering two things (`dsh`, `--profile dshline`). It finds the harness launcher, adds the profile and the working folder, and hands over the terminal with `stdio: 'inherit'`. It must never grow session logic: one implementation of the frontend is the point, and a wrapper that started doing its own work would be a second one.

That split is not just tidiness. **The renderer must never import from the harness, and must never gain a dependency or a peer dependency.** Having no dependencies is what lets this plugin add nothing to a user's setup, and it is why every rule below about widths, cutting, and escaping can be tested without a terminal and without a model.

## Commands

```sh
pnpm install
pnpm build       # tsc -b for both packages
pnpm test        # the full suite, no terminal and no model required
pnpm typecheck   # tsc -b, same project graph
pnpm security    # the dependency and workflow checks CI runs
```

Nothing outside this repository is needed. The harness's real service types come from the registry's `next` release tag, so a fresh clone type-checks with no second checkout.

## One trap: build before you test by hand

`packages/dshline` imports the renderer **by package name**. That name resolves through `exports` to the compiled `lib/` folder, not to `src/`.

- **Tests are fine.** A vitest alias points the package name at `src`, so tests run the code you just wrote.
- **A harness profile that installed this plugin from a path is not.** It loads `lib/`. After any change to source, run `pnpm build` before starting the interface again, or you are testing the previous version.

This has already caused one silent failure: before the alias existed, a renderer change was invisible to the plugin's tests, so a test could pass against code that no longer existed.

## Rules that are easy to break

Breaking one of these usually produces a failure somewhere unrelated, which is why they are listed rather than left to be rediscovered.

1. **Make text safe before adding color, never after.** `escapeControls` neutralizes the escape character itself. Run it over already-colored text and it destroys the color; run it over only some parts and control sequences get through everywhere else. Text from a model, a tool, a log, or a paste is all untrusted.
2. **Apply color to one row at a time, after adding the gutter mark.** `paint()` and `style()` both put a reset code at the end of whatever they wrap. Color a multi-line string in one call and every row except the last is left with color still switched on, which then leaks into whatever is drawn next to it.
3. **Colour is chosen by role, never by name, and the closer is always the full reset.** `paint(text, 'error')`, not `style(text, 'red')` — a colour named at a call site is a decision no palette can revisit, which is how a failed tool and a deleted diff line ended up unable to differ. `style` stays exported as the primitive and as a test fixture, and a check fails the build if it regrows a call site. Two roles that share a colour today stay separate when they mean different things: giving them one value later is free, splitting them apart again is not. The reset at the end must stay `CSI 0 m`; the foreground-only `CSI 39 m` renders the same and is read as an OPENING sequence by every wrap, which replays it onto each continuation row and never clears. Each layer owns its own roles: the renderer declares only the ones it draws itself, and this frontend adds the rest by augmenting `PaletteRoles` from its own package — a role like `assistant` or `pressure-warn` in the renderer would make it domain-aware, which rule 9 exists to prevent. How much colour a terminal shows is Node's `getColorDepth`, not a rule table here.
4. **Every write to the terminal goes through `Screen`.** Finished rows are committed to real scrollback and never rewritten; the bounded live area must stay last. That is the terminal model, not an implementation detail.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [riesbri/dshline](https://github.com/riesbri/dshline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
