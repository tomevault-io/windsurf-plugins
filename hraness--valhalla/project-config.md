---
trigger: always_on
description: - `crates/` – production Rust workspace crates for identity, social state, rooms consensus, games, storage, transport, the CLI, and supporting libraries.
---

# Contents

- `crates/` – production Rust workspace crates for identity, social state, rooms consensus, games, storage, transport, the CLI, and supporting libraries.
- `prototypes/` – bounded architecture and verification spikes kept outside the production dependency graph.
- `kb/` – the Wordcell/Obsidian knowledge vault containing maintained notes, plans, source captures, riffs, and scoped context hubs.
- `docs/` – operator, evidence, and architecture documentation that is not part of the knowledge vault.
- `desktop/` and `site/` – desktop companion and web product surfaces.
- `hraness/homebrew-tap` (separate repository) – the checksum-verified Homebrew formula and its generator (`tools/brew-formula.mjs`); kept out of this tree so the release security policy does not scan a Ruby surface.
- `vectors/` – frozen cross-implementation and protocol test vectors.
- `verify/` – repository verification support and proof-oriented checks.
- `Cargo.toml`, `package.json`, and `vercel.json` – Rust workspace, JavaScript tooling, and deployment entry points.

# Guidelines

<!-- hraness-public-copy:start -->
- Public copy (websites, READMEs, docs, package and GitHub descriptions, CLI help, `llms.txt`, generated pages) follows `STYLE.md`, synced from hraness/.github. Text a model writes for publication also follows `GENERATION_STYLE.md`.
- The delivery vocabulary in this file (admission, qualification, custody, receipt, bounded, lane, gate, surface, projection) is internal. Translate it into what the reader gets.
- Take one-line product and sibling descriptions from the portfolio registry and versions from the release record. Tests pin facts, not prose.
- Run `bun run check:copy` before handoff when the repository has it.
<!-- hraness-public-copy:end -->

<!-- oompa-local-efficiency:start -->
- Treat the user's request to change this repository as standing authorization for routine task-owned commits, pushes, pull requests, merges, releases, deployments, and production verification after the gates applicable to that action pass. Do not ask for duplicate confirmation. Build confidence through relevant automated checks, bounded diagnostics, and independent review, not another human approval. Passing checks does not expand task scope or authority.
- Prefer agentic service provisioning for new infrastructure. Check Vercel Marketplace for a native product that can provision the required resource first; use Stripe Projects as a supported alternative when it better covers the service or the Marketplace route only connects an existing account. Verify the current catalog, account, region, plan, recurring cost and resource capabilities before selecting a route. Prefer supported provider CLIs or APIs over browser-only setup when neither catalog fits, and explain the concrete exception. Reuse existing owner-controlled resources where appropriate; this preference alone does not authorize migrations, duplicate accounts, paid upgrades or wider access. Continue setup already authorized by the task and budget without duplicate confirmation. Keep provider credentials and generated environment files private, complete required interactive authentication, and verify deployment, persistence and recovery separately from successful provisioning.
- Separate artifact admission from live qualification and operational activation. Use applicable automated source, security, package/install, and provenance evidence for artifact admission; live provider qualification is not a universal publication prerequisite. Preserve explicit live acceptance criteria and require relevant live evidence for claims that depend on it. If publication or an artifact's install, upgrade, or default-use path activates risky unqualified behavior, keep that behavior guarded or disabled, or obtain bounded relevant evidence before shipping or activation.
- Use the repository's documented delivery workflow and preserve the identity, target, capacity, migration, and recovery guards applicable to operational activation. Replace an obsolete gate through a reviewed source and policy change with corresponding tests, never an ad hoc skip. Preserve every runtime-enforced approval, access control, branch protection, environment rule, safety policy, and required final gate. Ask for user input only when delivery needs a material product decision, missing credentials or authority, unavoidable interactive authentication, an irreversibly destructive action outside task scope, or resolution of a failure that cannot be handled safely and autonomously.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hraness/valhalla](https://github.com/hraness/valhalla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
