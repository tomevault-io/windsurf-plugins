---
trigger: always_on
description: All paths below are relative to the git root. When working inside a component, also read that component's `CLAUDE.md` — each is self-contained and covers its own build, style, and test conventions.
---

# aztec-node

All paths below are relative to the git root. When working inside a component, also read that component's `CLAUDE.md` — each is self-contained and covers its own build, style, and test conventions.

<components>
`yarn-project/` is the TypeScript monorepo containing the node, client SDK (`aztec.js`), PXE/wallet, sequencer, prover, p2p stack, and tooling — the main entrypoint for most day-to-day work; see `yarn-project/CLAUDE.md`.

`noir-projects/` holds the contract libraries written in Noir; see `noir-projects/aztec-nr/CLAUDE.md`.

`docs/` is the developer documentation site (Docusaurus); see `docs/CLAUDE.md`. `spartan/` holds Kubernetes deployment infrastructure (Helm charts + Terraform); see `spartan/CLAUDE.md`. `ci3/` contains CI infrastructure scripts.
</components>

<build_system>
From the git root, use `make <target>`: `make fast` builds everything needed for development, and `make yarn-project` runs the full TS build chain. For individual components, run `./bootstrap.sh` inside each directory.

When a change spans multiple components, rebuild in dependency order: first `noir-projects/` to compile contracts, then `yarn-project/` with `./bootstrap.sh` from inside `yarn-project/` (not the git root). `yarn build` only recompiles TypeScript: it skips the code generation bootstrap runs, so generated inputs such as `yarn-project/constants/src/constants.gen.ts` stay stale and surface as type errors about missing members. Barretenberg is not built from this repo: the `bb` binaries come pre-built via `labs-aztec-toolchain`, and the `@aztec-foundation/bb.js` (and related) npm packages are consumed as published releases pinned in `yarn-project/package.json` resolutions.

The noir-projects build scripts default `$NARGO` to `labs-aztec-toolchain/bin/nargo`, which is provisioned from the versions pinned in `labs-aztec-toolchain/bootstrap.sh`. Do not override it with a globally installed nargo — version mismatches produce opaque bytecode failures in downstream components.
</build_system>

<git_workflow>

<base_branch>
The default branch and standard PR base is `main`. If a PR is already open, its base is authoritative:

```bash
gh pr view --json baseRefName -q '.baseRefName'
```

Port PRs (branches named `port-<PR>-to-<target>`) target the port's target branch rather than `main`. Never present internal branch names, or `spartan` (the deployment infrastructure in `spartan/`), to a user as something to install or migrate to — releases are version tags (e.g. v5) and networks are named (e.g. testnet).

Use the base in `git diff origin/<base>...HEAD` and `git log origin/<base>..HEAD`. Always `git fetch` before creating branches so the base is not stale.
</base_branch>

<commits_and_prs>
Follow Conventional Commits: `fix:`, `feat:`, `chore:`, `refactor:`, `docs:`, `test:`. During development just create normal commits — do not amend unless explicitly asked.
</commits_and_prs>

<git_staging>
When staging files, prefer `git add -u` or name specific files rather than `git add -A` or `git add .`. The aggregate flags will pick up unrelated untracked working directories (e.g. personal scratch projects at the repo root) and quietly stage them. Subagents must always name specific files in `git add` — never `-u`, `-A`, or `.` — because they lack the main conversation's context for judging which changes belong to the current task.
</git_staging>

<lockfile_discipline>
Never bulk-update lockfiles (`Cargo.lock`, `yarn.lock`). Use targeted updates only: `cargo update --precise <version> --package <name>` for Rust, and `yarn up <package>@<version>` in the relevant workspace for TypeScript. Bulk updates drag in unrelated transitive changes that make review impossible and frequently break reproducibility.
</lockfile_discipline>

<standard_contract_repin>
Never run `noir-projects/noir-contracts/bootstrap.sh pin-standard-build` on your own initiative. The pin exists so ordinary source or bytecode changes do NOT move the standard contracts' canonical addresses, and CI does not fail when the bytecode drifts. A re-pin is a deliberate redeploy decision for a human to make: if a change seems to need one, leave the pin, rebuild against it, and ask. See the comment on `pin-standard-build` for why re-pinning is breaking.
</standard_contract_repin>

</git_workflow>

<code_formatting>
Each language's formatter is documented in the relevant subdir `CLAUDE.md` — TypeScript conventions live in `yarn-project/CLAUDE.md` and Noir in `noir-projects/aztec-nr/CLAUDE.md`. A post-edit hook runs the appropriate formatter automatically, so there is normally no need to invoke one by hand.
</code_formatting>

<red_green_testing>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aztec-labs-eng/aztec-node](https://github.com/aztec-labs-eng/aztec-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
