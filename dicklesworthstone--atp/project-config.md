---
trigger: always_on
description: > Guidelines for AI coding agents working in this repository.
---

# AGENTS.md — atp (standalone distribution repo)

> Guidelines for AI coding agents working in this repository.

---

## What This Repo Is (and Is Not)

This is the **standalone product/distribution repo** for the `atp` CLI — the
fountain-coded file-transfer tool. It contains **no Rust source code** and it
never should. It holds:

| File | Purpose |
|------|---------|
| `README.md` | The product documentation for `atp` |
| `install.sh` | Prebuilt-binary installer (curl-one-liner entry point) |
| `UPSTREAM_REV` | The exact asupersync commit releases are built from |
| `scripts/build-atp.sh` | Local/pinned build helper |
| `.github/workflows/release.yml` | Cross-platform release builds + GitHub release |
| `upstream` (symlink, gitignored) | Local convenience link to the asupersync checkout |

**The canonical ATP source lives in
[`Dicklesworthstone/asupersync`](https://github.com/Dicklesworthstone/asupersync)**
(locally: `/data/projects/asupersync`, symlinked here as `./upstream`):

- CLI binary: `src/bin/atp.rs` (feature `atp-cli`)
- Control/application layer: `src/atp/` (manifest, delta, verify, proof, daemon)
- Wire/transport layer: `src/net/atp/` (`transport_tcp`, `transport_rq`, `transport_quic`)
- RaptorQ codec: `src/raptorq/` (RFC 6330)
- Benchmarks vs rsync: `scripts/atp_bench/`, spec `docs/atp_bench_matrix_spec.md`,
  append-only evidence ledger `docs/atp_rq_beat_rsync_ledger.md`

**Never copy ATP source files into this repo.** If the code needs changing, do
the work in asupersync (its own `AGENTS.md` governs that work), land it on
asupersync `main`, then bump `UPSTREAM_REV` here.

---

## Ground Rules (inherited from the asupersync project)

1. **NO FILE DELETION** without express permission from the user.
2. **No destructive git/filesystem commands** (`git reset --hard`, `git clean -fd`,
   `rm -rf`, force-pushes) without the user supplying the exact command and
   explicit consent in the same message.
3. **`main` is the only branch.** No feature branches, no worktrees, no PRs from
   agents. Commit directly to `main`.
4. **No script-based bulk edits** of files in this repo; make changes manually.
5. Keep this repo tiny. The bar for adding new files is high.

---

## Release Process — dsr ONLY (GitHub Actions is DISABLED)

**USER DIRECTIVE (2026-07-10): GitHub Actions must not be used for ANYTHING in
this repo.** Both workflows are `disabled_manually` via `gh workflow disable`;
the YAML files remain in-tree as reference build recipes only. Never re-enable
them. Releases are built and published with
[dsr](https://github.com/Dicklesworthstone/doodlestein_self_releaser) from the
config in `~/.config/dsr/repos.d/atp.yaml`, which builds from the `UPSTREAM_REV`
pin via `scripts/build-atp.sh --pinned` (never from the shared asupersync
working tree).

```bash
set -euo pipefail
VERSION=0.3.8
TAG=v$VERSION

# 0. Preflight. repos.d/atp.yaml is authoritative; `dsr repos info atp`
#    reads the legacy three-target registry and is not a release gate.
export DSR_CONFIG_DIR="$HOME/.config/dsr"
assert_actions_disabled() {
  local states
  states=$(gh api repos/Dicklesworthstone/atp/actions/workflows \
    --jq '.workflows[].state')
  test -n "$states"
  test -z "$(printf '%s\n' "$states" | grep -vx disabled_manually || true)"
}
wlap_ps() {
  local engine="$1" script="$2" encoded
  encoded=$(printf '%s' "$script" | iconv -f UTF-8 -t UTF-16LE | base64 -w0)
  ssh wlap "$engine -NoLogo -NoProfile -NonInteractive -EncodedCommand $encoded"
}
command -v iconv >/dev/null
command -v base64 >/dev/null
command -v minisign >/dev/null
dsr doctor
dsr health check trj --no-cache
dsr health check mmini --no-cache
dsr health check wlap --no-cache
# minisign on trj/mmini is optional — it only enables the best-effort signature
# check during the Linux/macOS installer E2E. The Windows installer does not use
# minisign at all, so wlap needs nothing here.
ssh trj 'command -v minisign >/dev/null'
ssh mmini 'test "$(command -v minisign)" = /opt/homebrew/bin/minisign && minisign -v'
yq -e '.act_job_map == null and (.targets | length == 7)' \
  "$DSR_CONFIG_DIR/repos.d/atp.yaml"
assert_actions_disabled

# 1. Pick the asupersync commit to ship (must be pushed to origin/main there)
git -C upstream fetch origin main
UPSTREAM_SHA=$(git -C upstream rev-parse origin/main)
test "${#UPSTREAM_SHA}" -eq 40
test -z "${UPSTREAM_SHA//[0-9a-f]/}"
git -C upstream merge-base --is-ancestor "$UPSTREAM_SHA" origin/main

# 2. Update the pin, run the distribution gates, stage exactly this release's
#    intended files, commit, assert a clean tree, and push main + master.
printf '%s\n' "$UPSTREAM_SHA" > UPSTREAM_REV
bash -n install.sh scripts/build-atp.sh scripts/test-install.sh scripts/test-build-atp.sh
shellcheck -S warning install.sh scripts/build-atp.sh scripts/test-install.sh scripts/test-build-atp.sh
bash scripts/test-install.sh
bash scripts/test-build-atp.sh
RELEASE_FILES=(
  AGENTS.md
  README.md
  UPSTREAM_REV
  install.ps1
  install.sh
  scripts/test-install.ps1
  scripts/test-install.sh
  skills/atp/references/OPERATIONS.md
)
EXPECTED_RELEASE_FILES=$(printf '%s\n' "${RELEASE_FILES[@]}" | sort)
git add -- "${RELEASE_FILES[@]}"
test "$(git diff --cached --name-only | sort)" = "$EXPECTED_RELEASE_FILES"
test -z "$(git diff --name-only)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/atp](https://github.com/Dicklesworthstone/atp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
