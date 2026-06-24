---
trigger: always_on
description: This file is the operating guide for Claude Code in this repository. Keep it factual and current. If it conflicts with the current worktree or with C++ source, the current worktree plus C++ source wins.
---

# CLAUDE.md

This file is the operating guide for Claude Code in this repository. Keep it factual and current. If it conflicts with the current worktree or with C++ source, the current worktree plus C++ source wins.

## Project And Source Of Truth

RustyCore is a Rust port of a TrinityCore-derived World of Warcraft Wrath/Cata-classic private server. The port target is full functional parity with the legacy C++ server, not a smaller compatible subset.

- Rust repo: `/home/server/rustycore`
- Legacy C++ reference: `/home/server/woltk-trinity-legacy`
- Remote: `https://github.com/alseif0x/rustycore.git`
- Main work branch: `develop`
- `main` is kept fast-forward synced at stable checkpoints.
- Rust toolchain: Rust 1.85, edition 2024.
- `protoc`: `/home/cdmonio/.local/protoc/bin/protoc`

Do not trust existing Rust, old AI summaries, or migration docs as correctness proof. Always contrast behavior against the C++ source before implementing or approving a change.

### Reference Priority

The legacy C# server and older C#-based notes are secondary historical references only. They are useful for finding intent, old diagnostics, or previous packet experiments, but they are not an authority for this port.

For protocol, gameplay, database, map/runtime, and persistence behavior, the final implementation must be anchored to the C++ source under `/home/server/woltk-trinity-legacy` or to a real client/server packet capture when C++ is incomplete or ambiguous. Do not approve a layout, field order, bit count, opcode response, or runtime rule merely because a Rust comment says "C# format", "C# ref", or "matches C#".

When touching code that still cites C#:

1. Treat the C# citation as suspect until checked.
2. Locate the equivalent C++ packet/class/function.
3. Update the comment to cite C++ once verified.
4. If C++ and C# disagree, stop and document the discrepancy before changing Rust.
5. If keeping C# behavior intentionally, explain why C++ does not answer the case and add the packet capture/client-build evidence.

## Current Checkpoint

As of the last audited port state before this documentation refresh:

- Last audited port base: `1af9223 Add honest progress audit (R8-entities)`
- At that base, `develop`, `origin/develop`, `main`, and `origin/main` all pointed at `1af9223`.
- Tree expected clean on `develop`.
- Latest documented coverage count: `736/759 = 96.97%`.
- Latest handoff item: `TEST-DEBT / #NEXT.R8.ENTITIES.765`.

Start every session with:

```bash
cd /home/server/rustycore
git status --short --branch
git log --oneline --decorate -8
head -n 20 docs/migration/current-session-handoff.md
```

If HEAD has moved beyond `1af9223`, audit the commits instead of trusting their messages. A documentation-only commit that updates this file is not a new port base; code-bearing commits must still be reviewed against C++:

```bash
git log --oneline 1af9223..HEAD
git diff --stat 1af9223..HEAD
git diff 1af9223..HEAD
```

Only promote a newer commit to the reliable base after reviewing it against C++ and validating tests/docs.

## Mandatory Porting Method

Every implementation slice must follow this sequence:

1. Inspect current repo state and latest handoff.
2. Pick a real documented gap from `docs/migration/current-session-handoff.md` or the inventory files.
3. Locate exact C++ source anchors in `/home/server/woltk-trinity-legacy`.
4. Compare existing Rust against C++ before editing.
5. Implement the smallest faithful Rust change that moves the full port forward.
6. Add focused tests, preferably positive and negative branches.
7. Update migration docs/checklists with the new `#NEXT.R8.ENTITIES.xxx` item when closing a represented implementation gap.
8. Recalculate progress honestly.
9. Run validation.
10. Commit on `develop`, push, fast-forward `main`, push, and return to `develop` only at stable closure points.

Do not do "bulk close" inventory edits. A closed `#NEXT` item must correspond to real code and tests, with exact C++ refs, Rust targets, checks run, and remaining boundaries stated. Discovering or documenting a gap is useful, but it is not an implementation closeout.

Do not mark anything `manual-test-ready` unless it has actually been installed/restarted and exercised manually against the client/runtime.

## Build And Test

Use `PROTOC` explicitly for any command that may compile protobuf-dependent crates:

```bash
PROTOC=/home/cdmonio/.local/protoc/bin/protoc cargo check -p world-server
PROTOC=/home/cdmonio/.local/protoc/bin/protoc cargo test -p wow-world --lib
PROTOC=/home/cdmonio/.local/protoc/bin/protoc cargo test -p wow-map --lib
```

Fast iteration commands:

```bash
cargo fmt --check
cargo fmt --all -- --check
cargo test -p wow-world some_test_name --lib
cargo test -p wow-map some_test_name --lib
cargo clippy -p wow-map -p wow-world --all-targets
git diff --check
```

TSV inventory files must keep 9 tab-separated columns:

```bash
awk -F '\t' 'NF != 9 { print FNR ":" NF ":" $0; bad=1 } END { if (bad) exit 1; print "TSV_OK" }' docs/migration/inventory/r8-entities-miniphase.tsv
```

Current useful baselines from recent handoff:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alseif0x/rustycore](https://github.com/alseif0x/rustycore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
