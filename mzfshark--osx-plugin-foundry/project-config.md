---
trigger: always_on
description: Foundry template for building, testing, and deploying Aragon OSx plugins with comprehensive tooling.
---

# Copilot Instructions — Aragon OSx Plugin (Foundry)

Foundry template for building, testing, and deploying Aragon OSx plugins with comprehensive tooling.

## Big Picture

- **Plugin variants**: UUPS upgradeable (`MyUpgradeablePlugin.sol`), cloneable (`MyCloneablePlugin.sol`), and static (`MyStaticPlugin.sol`) implementations available.
- **Aragon OSx integration**: Plugins integrate with DAO framework via `PluginSetup` contracts that define install/uninstall/update lifecycles.
- **Custom plugins**: Harmony-specific voting plugins (HIP, Delegation, Validator Opt-In) under `src/harmony/`; use as reference for production implementations.
- **Deployment flows**: Three strategies—simple plugin repo, full DAO with plugins, and trustless factory-based—choose via `DEPLOYMENT_SCRIPT` in `Makefile`.
- **Dependencies**: Aragon OSx libraries (`@aragon/osx`, `@aragon/osx-commons`), OpenZeppelin contracts, and Forge standard library.

## Planning & Issue Tracking Workflow

**CRITICAL: After completing planning and BEFORE starting implementation:**

1. **Generate Plan Document**: Create `PLAN.md` at repository root containing:

   - [ ] Clear task breakdown with checkboxes
   - [ ] Implementation steps and guidelines
   - [ ] Dependencies and integration points
   - [ ] Expected outcomes and acceptance criteria

2. **Sync with GitHub Project**: Using GitHub CLI (`gh` - already authenticated as mzfshark):

   ```bash
   # Create issue from PLAN.md
   gh issue create --title "[Plan] <descriptive-title>" --body-file PLAN.md --project "https://github.com/users/mzfshark/projects/5"
   ```

3. **Update Plan Progress**: As tasks complete, update checkboxes in `PLAN.md` and sync with issue:
   ```bash
   # Update the issue body with current PLAN.md
   gh issue edit <issue-number> --body-file PLAN.md
   ```

**IMPORTANT**: Never run `git commit` or `git push` automatically. Always ask the user before any git operations.

**Never start implementation without a documented plan in `PLAN.md` and corresponding GitHub issue.**

## Tool Restrictions

**FORBIDDEN: Do NOT use `codacy_get_pattern` tool** — This tool is incompatible with WSL environments and will fail. Use alternative Codacy tools for code quality analysis.

## Language Standards

**MANDATORY: All public-facing content MUST be in English:**

- **Code comments**: All comments in source code must be written in English
- **Logs and console output**: All log messages, debug output, and error messages must be in English
- **GitHub Issues**: All issue titles, descriptions, and comments must be in English
- **Commit messages**: All git commit messages must be in English following conventional commits format
- **Documentation**: All README files, inline docs, and API documentation must be in English
- **Variable/function names**: Use English for all identifiers in code

**Examples:**
```bash
# ✅ CORRECT
git commit -m "feat: implement Band Oracle integration for HIP plugin"

# ❌ INCORRECT
git commit -m "implementa integração com Band Oracle para plugin HIP"
```

**Note**: This standard ensures international collaboration and maintainability. Internal planning documents (like `PLAN.md` for local work) may use Portuguese if needed, but all published content must be English.

## Key Paths

- Plugin implementations: `src/*.sol` (template variants), `src/harmony/*.sol` (production Harmony plugins).
- Setup contracts: `src/setup/MyPluginSetup.sol` (defines installation logic and permissions), `src/harmony/*Setup.sol`.
- Deploy scripts: `script/Deploy*.s.sol`—choose via `DEPLOYMENT_SCRIPT` in `Makefile`.
- Test files: `test/*.t.sol` (Solidity tests), `test/*.t.yaml` (YAML test definitions converted via Bulloak).
- Test builders: `test/builders/SimpleBuilder.sol` (local), `test/builders/ForkBuilder.sol` (fork testing with OSx factories).
- Config: `foundry.toml`, `.env` (copy from `.env.example`), `Makefile`.
- Artifacts: `artifacts/` (deployment outputs), `deployed_contracts_harmony.json` (Harmony addresses).

## Install, Build, Test

- **Setup**: `make init` (checks dependencies, runs first build). Copy `.env.example` to `.env` and configure.
- **Build**: `forge build` or `make` (via Makefile).
- **Test**: `make test` (unit), `make test-fork` (fork tests using `RPC_URL`), `make test-coverage` (HTML report).

## Terminal Timing (tests/build/type-check)

After running `test`, `type-check`, or `build` commands, wait 120 seconds before attempting to read terminal output.
- **Test scaffolding**: `make sync-tests` (scaffolds/syncs `.t.yaml` → `.t.sol` via Bulloak), `make check-tests` (checks sync status).
- **Test tree**: `make test-tree` (generates `TESTS.md` from `.t.yaml` files).

## Deployment Workflow

1. **Pre-deploy simulation**: `make predeploy` (dry-run).
2. **Deploy**: `make deploy` (deploys, verifies, writes to `artifacts/`).
3. **Retry/resume**: `make resume` (retry failed txs).
4. **Verification**: `make verify-etherscan`, `make verify-blockscout`, `make verify-sourcify`.
5. **Update registry**: Use `script/update-deployed-contracts.sh <contract> <address> <tx_hash>` to update `deployed_contracts_harmony.json`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mzfshark) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
