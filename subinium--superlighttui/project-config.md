---
trigger: always_on
description: - Respond in Korean (한글) by default
---

# SuperLightTUI — Project Instructions

## Language
- Respond in Korean (한글) by default
- All code, commit messages, PR descriptions, and code comments in English

---

## STOP — Release Workflow Checklist (I Keep Forgetting This)

Every release goes through ALL of these steps. No exceptions. No "probably fine". No "CI will catch it".
Patch (x.y.Z), minor (x.Y.0), major (X.0.0) all follow the same flow — only the version bump differs.

### 1. Pre-CI (LOCAL, on release branch)
Run the full Core Gate AND Extended Gate locally BEFORE pushing. The branch must be green on your machine first.
- [ ] `cargo fmt -- --check`
- [ ] `cargo check --all-features`
- [ ] `cargo clippy --all-features -- -D warnings`
- [ ] `cargo test --all-features`
- [ ] `cargo check --examples --all-features`
- [ ] `typos`
- [ ] `cargo check -p superlighttui --no-default-features`
- [ ] `cargo check -p slt-wasm --target wasm32-unknown-unknown`
- [ ] `cargo hack check -p superlighttui --each-feature --no-dev-deps`
- [ ] `cargo audit`
- [ ] `cargo deny check`

If ANY fails: fix first. Never push a known-red branch hoping CI will catch it.

### 2. Version bump + changelog
- [ ] Bump `Cargo.toml` version
- [ ] Update `CHANGELOG.md` — group all commits since last tag (Added / Changed / Fixed / Perf / Docs)
- [ ] Update any README / doc badges that show the version

### 3. Branch + commit + push
- [ ] Branch named `release/vX.Y.Z` (already created at start of work, verify)
- [ ] Single atomic commit titled `feat: vX.Y.Z — <short summary>` (or `fix:` for hotfix)
- [ ] `git push -u origin release/vX.Y.Z`

### 4. PR + wait for CI
- [ ] `gh pr create` with full release notes in body (closes #issue-numbers)
- [ ] **WAIT** for CI on the PR — `gh pr checks --watch` or `gh run list --branch release/vX.Y.Z`
- [ ] ALL required checks must be `completed success`. No merging yellow-X.

### 5. Merge PR
- [ ] Merge with **squash** — body becomes the squashed commit body
- [ ] `git checkout main && git pull`
- [ ] Verify `git log -1` shows the squashed commit as expected on main

### 6. Tag + push tag
- [ ] `git tag -a vX.Y.Z -m "vX.Y.Z"` on the merged main commit
- [ ] `git push --tags`
- [ ] **WAIT** for `release.yml` workflow — `gh run list --workflow=release.yml --limit 1`
- [ ] Workflow must be `completed success` before proceeding

### 7. Post-release verification
- [ ] `gh release view vX.Y.Z` shows artifacts
- [ ] crates.io: `cargo search superlighttui` shows new version (may take a few minutes)
- [ ] docs.rs: `https://docs.rs/superlighttui/X.Y.Z` built successfully
- [ ] Smoke: `cargo install superlighttui --version X.Y.Z` succeeds; run a trivial example
- [ ] Homebrew tap (if applicable): formula `url` + `sha256` updated; `brew install --build-from-source` works

### 8. Only now announce
Only after steps 1–7 are all green. Not before.

### Red flags that mean STOP
- "Probably fine" → no, run the gate
- "Just a docs change" → still run `cargo check --examples --all-features` (examples import the library)
- "CI will catch it" → no, locals catch it first, CI is a last line, not a first
- "I'll tag now and fix if anything breaks" → no, no broken tags. A tag is a public artifact.
- "The fmt check is being pedantic" → run `cargo fmt` and move on
- "This test was already flaky" → diagnose root cause, do NOT mark `--skip`

---

## Pre-Release Quality Gate (MANDATORY — NO EXCEPTIONS)

Before ANY commit, PR, or release, run ALL checks below.
**If ANY check fails, DO NOT proceed. Fix first.**

### Core Gate (every commit)
```bash
# 1. Format check (THIS IS THE ONE YOU KEEP FORGETTING)
cargo fmt -- --check

# 2. Compilation
cargo check --all-features

# 3. Clippy (deny all warnings)
cargo clippy --all-features -- -D warnings

# 4. Full test suite
cargo test --all-features

# 5. Examples compile
cargo check --examples --all-features
```

If `cargo fmt -- --check` shows diffs, run `cargo fmt` to fix, then re-run all.

### Extended Gate (before PR or release)
These match CI jobs that block merge. Run after the core gate passes.
```bash
# 6. Typo check
typos

# 7. No-default-features (catches missing #[cfg] gates)
cargo check -p superlighttui --no-default-features

# 8. WASM target
cargo check -p slt-wasm --target wasm32-unknown-unknown

# 9. Feature combinations (needs cargo-hack: cargo install cargo-hack)
cargo hack check -p superlighttui --each-feature --no-dev-deps

# 10. Dependency audit
cargo audit

# 11. Deny check (licenses, bans, sources)
cargo deny check
```

### CI Job Reference
| Job | Command | Blocks merge? |
|-----|---------|---------------|
| Format | `cargo fmt -- --check` | Yes |
| Check (stable) | `cargo check --all-features` | Yes |
| Check (MSRV 1.88) | `cargo check --features async,serde` (toolchain 1.88) | Yes |
| Clippy | `cargo clippy --all-features -- -D warnings` | Yes |
| Test | `cargo test --all-features` | Yes |
| Typos | `typos` | Yes |
| Check (no-default) | `cargo check -p superlighttui --no-default-features` | Yes |
| Check (WASM) | `cargo check -p slt-wasm --target wasm32-unknown-unknown` | Yes |
| Feature Combinations | `cargo hack check --each-feature --no-dev-deps` | Yes |
| Audit | `cargo audit` | Yes |
| Deny Check | `cargo deny check {advisories,bans,licenses,sources}` | Yes (advisories soft) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [subinium/SuperLightTUI](https://github.com/subinium/SuperLightTUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
