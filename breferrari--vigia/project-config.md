---
trigger: always_on
description: A live diff **monitor** for the terminal. Not a review tool.
---

# vigia

A live diff **monitor** for the terminal. Not a review tool.

`vigia` is Portuguese: a watchman, and in nautical use a porthole. The window you look through. Also the verb: `vigia .` reads as "watch this."

## What this is, in one paragraph

You run an AI coding agent in one pane and `vigia` in the pane beside it. It shows the working-tree diff as it changes, continuously, without being touched. It is ambient. It is closer to `btop` than to a git client: you read state from shape and colour, glance away, and glance back.

## The product class is the whole thesis

`vigia` is **monitor-class**: already open, rarely touched, correct with zero input, cheap for days, glanceable. It is not a **reviewer**: something you launch per changeset to step through, annotate and decide on.

That distinction is not stylistic. It generates every budget in `SPEC.md`, and those budgets are the product.

**If a change makes `vigia` a better reviewer at the cost of an invariant, the change is wrong.** Reject it and say why.

## Stack — settled, do not relitigate

| Layer | Choice | Why |
|---|---|---|
| TUI | `ratatui` + `crossterm` | `bottom`, a btop-class system monitor, runs exactly this pair. `crossterm` is the only backend with Windows plus cross-platform mouse. `termion` is Unix-only |
| Git | `gix` | In-process diff. No `git diff` subprocess per tick |
| Watch | `notify` | Native FS events per platform, which I1 requires instead of a timer. Pure Rust: no `cc` on any tier-1 target. Coalescing is ours, not its debouncer crate |
| Highlighting | `syntect` | Pure Rust, so no C toolchain in CI. What `delta` and `bat` use |
| Release | `cargo-dist` | Cross-platform binaries + Homebrew formula + GH workflow |

Everything above is pure Rust on purpose: `--target x86_64-unknown-linux-musl` gives a static binary with no cross-toolchain, and macOS/Windows are tier-1. **Choosing tree-sitter over `syntect` reintroduces a C toolchain**: that is a spec change, not an implementation detail.

**Reading a dependency's source: address it, never search for it.** `ratatui` 0.30 splits into `ratatui-core`, `ratatui-crossterm` and `ratatui-widgets`, so the types this project draws against most often (`Buffer`, `Cell`, the layout solver) live in a **transitive** crate that is not in this workspace. Its source is in the Cargo registry cache, behind a hash-suffixed index directory and a version-suffixed crate directory. Both of these are instant:

```bash
ls ~/.cargo/registry/src/*/ratatui-core-*/src/buffer/          # the sources, addressed directly
cargo metadata --format-version 1 | jq -r '.packages[] | select(.name=="ratatui-core") | .manifest_path'
```

**Never run `find /` on Windows.** Under Git Bash `/` is the MSYS root, which mounts the Windows registry as directories under `/proc`, so the walk never terminates, and `~/.cargo` is not under it anyway. `.claude/scripts/scan-guard.mjs` refuses the call. Bound any exploratory scan with `timeout`, and prefer Glob or Grep pointed at an explicit path.

`gix` was the least-precedented dependency here, so Phase 1 proved it before anything was built on top. **Proven 2026-07-30:** hunk boundaries match `git diff -U3` exactly and every Phase 1 budget holds with room. Evidence and the one constraint it came with are in `SPEC.md` §10.

## Method: spec-driven, drift-enforced

`SPEC.md` is the source of truth. Code is written against it.

1. If the code and `SPEC.md` disagree, **stop.** One of them is wrong. Decide which, and change that one deliberately, in its own commit.
2. Every invariant in `SPEC.md` has a test that fails when it is violated. An invariant without a failing test is a wish. `SPEC.md` and `RULINGS.md` have no size cap: `package.rs` fails only when an invariant id leaves the table. This file keeps a hard byte ceiling there.
3. Performance budgets are tests, not aspirations. The thesis is a measurable claim, so a regression past budget **fails the build**.

Do not add a dependency, a flag, or a subcommand that `SPEC.md` does not name. Propose it, get it into the spec, then build it.

## Where design decisions live

Design rationale for this project is recorded outside this repo, reachable through the **`vigil` MCP server**. This repo declares its identity as `vigia` in `.om-project`, which is what scopes reads and writes to this project.

### Reading

| Tool | Use it for |
|---|---|
| **`search`** | The full written record: why the product class is what it is, why each dependency was chosen, which alternatives were rejected and on what evidence, what each budget was set against. **Start here.** |
| **`expand`** | Once you have a specific note, see what it links to and what links back. Cheaper and more exact than searching again for the neighbourhood. |
| **`recall`** | Short durable lessons scoped to this project. Pass `explain: true` when something you expected is missing: it distinguishes "scoped away" from "never existed". |
| **`reason`** | A judgement that needs several notes weighed against each other, for example "is what I am about to do consistent with what was decided". It spawns a second session, so it is slower and costs more. Use it only when `search` returned the notes but not the answer. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [breferrari/vigia](https://github.com/breferrari/vigia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
