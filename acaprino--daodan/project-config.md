---
trigger: always_on
description: The Daodan is the symbiote that augments its host. This repository is the Daodan of coding agents: one marketplace of agents, skills, and workflows that augment a base model into a specialized toolkit covering development workflows, code quality, AI tooling, scraping, trading, observability, and more. It ships to four hosts from one source. Remote: `acaprino/daodan` on GitHub.
---

# Daodan

The Daodan is the symbiote that augments its host. This repository is the Daodan of coding agents: one marketplace of agents, skills, and workflows that augment a base model into a specialized toolkit covering development workflows, code quality, AI tooling, scraping, trading, observability, and more. It ships to four hosts from one source. Remote: `acaprino/daodan` on GitHub.

## Project structure

`plugins/<name>/` is a plugin's **content kernel**, and it is the only hand-authored plugin source alongside `adapters/`. Each kernel holds `plugin.toml` (the neutral control plane), `roles/` (agent bodies), `workflows/` (entry-point bodies plus one TOML sidecar each), `skills/`, `contracts/` and `policies/`. Those are also the only directories the compiler ships: a `references/`, `scripts/` or `mcp/` directory at the kernel root reaches no host package, so anything a body reads at runtime lives under `skills/<name>/`, and `scripts/lint_bundled_paths.py` fails a `${CLAUDE_PLUGIN_ROOT}` reference that the generated package does not contain. A kernel that ships an MCP server declares it under `[[mcp.servers]]` in `plugin.toml` (name, command, args, the server file under a skill) and requires the `mcp.servers` capability; the Claude adapter renders it as the package's `.mcp.json` and the catalog entry's `mcpServers` pointer, and Codex and Copilot, where no probe has shown a host starting a plugin-declared server, ship the file and open every workflow with a note giving the exact command to register. `peer-review` is the one plugin that declares one. Markdown is always the behaviour; TOML is always declarative metadata and never carries a prompt.

`adapters/<host>/` is what one host can do: `capabilities.toml` binds every neutral capability to a host mechanism, `coordination.toml` orders the coordination strategies that host supports, `layout.toml` says where each component lands in its packages and what the two Claude placeholders a kernel may write (`${CLAUDE_PLUGIN_ROOT}`, `$ARGUMENTS`) become on that host, `templates/` holds the harness wrappers, `policies/` holds this host's implementation of a neutral policy, and `overrides/` holds fingerprinted semantic divergences.

**Everything under `exports/` and every root marketplace manifest is generated.** `scripts/daodan_build.py` compiles kernels plus adapters into `exports/claude/`, `exports/copilot/`, `exports/codex/` and `exports/pi/`, and into `.claude-plugin/marketplace.json`, `.github/plugin/marketplace.json`, `.agents/plugins/marketplace.json` and the root `package.json`. That last one is the odd member and it is not a Node project: Pi has no marketplace, it installs a package and reads the manifest at that package's root, so the repository root is where its catalog has to live. Never hand-edit any of them; edit the kernel and rebuild.

`AGENTS.md` and the four repository workflow skills under `.agents/skills/` are native Codex adaptations of this file and `.agents/skills/`. Generate them with `python scripts/sync_codex_instructions.py`; `--check` is the parity gate. Edit the Claude-side canonical copies, then run the synchronizer. The transformation is deliberately limited to instruction paths, cache paths, install commands and the host-relative downstream description.

The compiler is `scripts/daodan/`: `model.py` and `load.py` (strict TOML loading), `validate.py` and `trust.py` (semantic and secret validation), `adapter.py` (capability parity and strategy selection), `overrides.py` (the fingerprint gate), `render.py`, `templates.py`, `provenance.py`, `catalogs.py` and `report.py`. Standard library only, no third-party dependency anywhere in the toolchain.

`codebase-xray` was named `deep-dive-analysis` until marketplace 14.0.0, and its analysis artifact directory kept the old name, `.deep-dive/`, until marketplace 27.0.0. It is now `.codebase-xray/`, matching the plugin. That directory is the stable downstream contract, declared as the plugin's `write-confinement` policy: every consumer reads it by that path, so a rename is a marketplace-wide change and never a local one. There is no fallback to the old path, by decision: a reader that silently accepted both would leave two contracts alive and nothing would ever name which one it read.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acaprino/daodan](https://github.com/acaprino/daodan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
