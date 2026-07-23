---
trigger: always_on
description: All paths below are relative to the git root. When working inside a component, also read that component's `CLAUDE.md` — each is self-contained and covers its own build, style, and test conventions.
---

# aztec-packages

All paths below are relative to the git root. When working inside a component, also read that component's `CLAUDE.md` — each is self-contained and covers its own build, style, and test conventions.

<components>
`yarn-project/` is the TypeScript monorepo containing the node, client SDK (`aztec.js`), PXE/wallet, sequencer, prover, p2p stack, and tooling — the main entrypoint for most day-to-day work; see `yarn-project/CLAUDE.md`.

`barretenberg/` is the C++ ZK proving system (Honk, Chonk, ECCVM); see `barretenberg/CLAUDE.md` and `barretenberg/cpp/CLAUDE.md`. `barretenberg/cpp/src/barretenberg/vm2/` is the AVM (Aztec Virtual Machine) for public execution; see its `CLAUDE.md`. `barretenberg/sol/` is the Solidity on-chain verifier; see `barretenberg/sol/CLAUDE.md`. `barretenberg/ts/` contains the TypeScript bindings for barretenberg (bb.js).

`avm-transpiler/` transpiles Noir bytecode to AVM bytecode (Rust). `noir/` is the Noir compiler, a git submodule pointing to noir-lang/noir. `noir-projects/` holds the protocol circuits and contract libraries written in Noir; see `noir-projects/aztec-nr/CLAUDE.md`.

`l1-contracts/` holds the Solidity L1 rollup contracts (a Foundry project). `docs/` is the developer documentation site (Docusaurus); see `docs/CLAUDE.md`. `spartan/` holds Kubernetes deployment infrastructure (Helm charts + Terraform); see `spartan/CLAUDE.md`. `bb-pilcom/` is the PIL compiler for AVM relation codegen. `ci3/` contains CI infrastructure scripts.
</components>

<build_system>
Dependencies flow barretenberg → noir → l1-contracts → yarn-project. From the git root, use `make <target>`: `make fast` builds everything needed for development, `make yarn-project` runs the full TS build chain (which builds bb, noir, and l1-contracts first), `make bb-cpp-native` builds barretenberg C++ native only, `make noir` builds the Noir compiler, and `make l1-contracts` builds the Solidity contracts via Foundry. For individual components, run `./bootstrap.sh` inside each directory.

When a change spans multiple components, rebuild in dependency order: first `barretenberg/cpp/` with `cmake --preset default && cd build && ninja`, then `barretenberg/ts/` with `./bootstrap.sh` (which generates TS bindings from C++), then `noir/` with `./bootstrap.sh` if noir changes are needed, then `noir-projects/` to compile contracts, then `l1-contracts/` with `forge build`, and finally `yarn-project/` with `yarn build` from inside `yarn-project/` (not the git root).

The noir-projects build scripts default `$NARGO` to `noir/noir-repo/target/release/nargo`. Do not override this with a globally installed nargo — version mismatches produce opaque bytecode failures in downstream components.
</build_system>

<bumping_noir>
To bump the Noir compiler version (e.g. a request like "bump the noir compiler version to X"), run `noir/scripts/bump_noir_compiler.sh <ref>` — the single source of truth, also surfaced via the `noir-sync-update` skill. It bumps the `noir/noir-repo` submodule to `<ref>` (a noir-lang/noir ref: release tag `v1.0.0-beta.23`, nightly `nightly-2026-06-02`, branch, or commit), refreshes `avm-transpiler/Cargo.lock` and `yarn-project/yarn.lock`, reformats `noir-projects`, and stages everything. Do not bump the submodule by hand; skipping any of these leaves the tree inconsistent and fails CI. The script does not commit — verify with `git status` from the repo root, then commit as `chore: update Noir to <ref>`.
</bumping_noir>

<git_workflow>

<critical_never_assume_master>
Never assume the base branch is `master` or `main`. Most branches target `next` or a `merge-train/*` branch, and defaulting to `master` produces incorrect diffs and broken PR comparisons. Determine the actual base before diffing or opening a PR.

If a PR is already open, that is authoritative:

```bash
gh pr view --json baseRefName -q '.baseRefName'
```

Otherwise infer from the component being worked in:

| Component | Base branch |
|---|---|
| `barretenberg/**` | `merge-train/barretenberg` |
| `yarn-project/**` | `merge-train/spartan` |
| `barretenberg/cpp/src/barretenberg/vm2/**` | `merge-train/avm` |
| everything else | `next` |

The bases above target the `next` line. For work scoped to the v5 release line, use `merge-train/spartan-v5` (which targets `v5-next`) in place of `merge-train/spartan`.

`spartan` in these branch names refers to the deployment infrastructure in `spartan/`; it is not a release channel, network, or SDK version. Never present it, `merge-train/*`, `next`, or `v5-next` to a user as something to install or migrate to — releases are version tags (e.g. v5) and networks are named (e.g. testnet).

Use the discovered base in `git diff origin/<base>...HEAD` and `git log origin/<base>..HEAD`. Always `git fetch` before creating branches so the base is not stale.
</critical_never_assume_master>

<commits_and_prs>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AztecProtocol/aztec-packages](https://github.com/AztecProtocol/aztec-packages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
