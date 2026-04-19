---
trigger: always_on
description: For major design changes, always ask the maintainer (@imran-siddique) before proceeding:
---

# Copilot Instructions for agent-governance-toolkit

## Decision Escalation

For major design changes, always ask the maintainer (@imran-siddique) before proceeding:
- New packages or modules that change the repo structure
- Cross-cutting changes spanning 3+ packages
- Security model changes (identity, trust, policy engine)
- Breaking API changes to public interfaces
- New framework integrations or SDK additions
- Changes to CI/CD pipeline architecture

Do NOT auto-merge large feature PRs without maintainer review.

## External Contribution Quality Gate

When external contributors open issues or PRs proposing integration with their own project/tool/library, apply these quality checks before investing review time:

- **Minimum credibility threshold**: The referenced project should have meaningful community adoption (e.g., 50+ GitHub stars, multiple contributors, evidence of production usage). One-person repos with <10 stars and no community traction do not warrant integration effort.
- **Self-promotion filter**: Issues or PRs that primarily serve to promote the contributor's own low-profile project — rather than adding genuine value to AGT — should be deprioritized. Politely acknowledge but do not fast-track.
- **Verify claims**: If the PR cites benchmarks, adoption numbers, or production deployments, spot-check them. Unverifiable claims are a red flag.
- **Scope proportionality**: A small or unknown project requesting a large integration surface (new package, new dependency, new CI pipeline) is disproportionate. Suggest they contribute as an example or community link instead.
- **Dependency risk**: Adding a dependency on an obscure package creates supply chain risk. Prefer vendored examples or optional integrations that don't add to the core dependency tree.

## PR Merge Workflow

When merging PRs, follow this sequence for EACH PR (do not batch):

1. **Review** — run all mandatory checks below
2. **Update branch** — merge latest main into the PR branch (`update-branch` API or UI button)
3. **Approve pending workflows** — fork PRs may have `pull_request_target` workflows waiting for maintainer approval; approve them in the Actions tab
4. **Approve the PR** — submit an approving review
5. **Enable auto-merge** — set squash auto-merge so it merges once CI passes
6. **Move to next PR** — don't wait; auto-merge handles the rest

This prevents PRs from stacking in the merge queue behind stale branches.

## PR Review — Mandatory Before Merge

NEVER merge a PR without thorough code review. CI passing is NOT sufficient.

Before approving or merging ANY PR, verify ALL of the following:

1. **Read the actual diff** — don't rely on PR description alone
2. **Dependency confusion scan** — check every `pip install`, `npm install`, `cargo add` command in docs/code for unregistered package names. The registered names are:
   - **PyPI:** `agent-os-kernel`, `agentmesh-platform`, `agent-hypervisor`, `agentmesh-runtime`, `agent-sre`, `agent-governance-toolkit`, `agentmesh-lightning`, `agentmesh-marketplace`
   - **PyPI (local-only, not published):** `agent-governance-dotnet`, `agentmesh-integrations`, `agent-primitives`, `emk`
   - **PyPI (common deps):** `streamlit`, `plotly`, `pandas`, `networkx`, `aioredis`, `pypdf`, `spacy`, `slack-sdk`, `docker`, `langchain-openai`
   - **npm:** `@microsoft/agent-os-kernel`
   - **crates.io:** `agentmesh`
3. **New Python modules** — verify `__init__.py` exists in any new package directory
4. **Dependencies declared** — any new `import` must have the package in `pyproject.toml` dependencies (not just transitive)
5. **No hardcoded secrets** — no API keys, tokens, passwords, connection strings in code or docs
6. **No plaintext config in pipelines** — ESRP Client IDs, Key Vault names, cert names go in secrets, not YAML
7. **Verify PR has actual changes** — check `additions > 0` before merging (empty PRs have happened)
8. **MIT license headers** — every new source file (`.py`, `.ts`, `.js`, `.rs`, `.go`, `.cs`, `.sh`) must have the license header. This is the #1 most common review finding.

## Security Rules

- All `pip install` commands must reference registered PyPI packages
- All security patterns must be in YAML config, not hardcoded
- All GitHub Actions must be SHA-pinned (use `action@<sha> # vX.Y.Z` format, never bare tags like `@v46`)
- All workflows must define `permissions:`
- Use `yaml.safe_load()`, never `yaml.load()`
- No `pickle.loads`, `eval()`, `exec()`, `shell=True` in production code
- No `innerHTML` — use safe DOM APIs
- No `unwrap()` in non-test Rust code paths (use `?` or explicit error handling)
- Docker images must use pinned version tags or SHA digests (never `:latest`)

## Supply Chain Security (Anti-Poisoning)

### Version Selection
- **7-Day Rule:** Never install a package version released less than 7 days ago. Prefer versions with at least one week of stability and consistent download metrics.
- **Fallback:** If the latest version is < 7 days old, pin to the previous stable release.
- **Verification:** Check release timestamps via `npm view <package> time` or `pip index versions <package>`.

### Version Locking
- **Exact versions only:** Use exact versioning in `package.json` (e.g., `"axios": "1.14.0"`). Prohibit `^` or `~` ranges.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/agent-governance-toolkit](https://github.com/microsoft/agent-governance-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
