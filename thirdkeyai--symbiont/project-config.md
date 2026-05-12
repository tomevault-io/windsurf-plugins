---
trigger: always_on
description: Symbiont (Symbi) is a Rust-native, zero-trust agent framework for building autonomous, policy-aware AI agents. Part of the [ThirdKey](https://thirdkey.ai) trust stack: [SchemaPin](https://schemapin.org) → [AgentPin](https://agentpin.org) → **Symbiont**.
---

# Symbiont — Agent Instructions

Symbiont (Symbi) is a Rust-native, zero-trust agent framework for building autonomous, policy-aware AI agents. Part of the [ThirdKey](https://thirdkey.ai) trust stack: [SchemaPin](https://schemapin.org) → [AgentPin](https://agentpin.org) → **Symbiont**.

- **Docs**: https://docs.symbiont.dev
- **Repo**: https://github.com/ThirdKeyAI/Symbiont
- **Crate**: https://crates.io/crates/symbi

## Project Structure

```
crates/
├── dsl/              # Symbi DSL parser with Tree-sitter integration
├── runtime/          # Agent runtime (scheduling, routing, sandbox, AgentPin)
├── channel-adapter/  # Slack, Teams, Mattermost adapters
├── repl-core/        # Core REPL engine
├── repl-proto/       # JSON-RPC wire protocol types
├── repl-cli/         # Command-line REPL interface
├── repl-lsp/         # Language Server Protocol implementation
src/                  # Unified `symbi` CLI binary
```

## Build and Test

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace
cargo fmt --check
```

All four commands must pass before committing. Clippy must produce zero warnings.

## Code Style

- Rust edition 2021
- Run `cargo fmt` before committing
- Run `cargo clippy --workspace` and fix all warnings before committing
- Inline tests in source files using `#[cfg(test)] mod tests`
- ES256 (ECDSA P-256) only for AgentPin identity — reject all other algorithms
- Agent files use `.symbi` (canonical) — `.dsl` is supported indefinitely for backward compatibility. Use `dsl::is_symbi_file` / `dsl::strip_symbi_extension` for file discovery instead of inlining extension checks. New scaffolding emits `.symbi` only.

## Commit Guidelines

- Write concise commit messages focused on the "why"
- No mention of AI assistants or co-authoring in commit messages
- Use `date` command to determine the current date when adding dates to docs

## Security

- Zero-trust by default: all inputs are untrusted
- Cryptographic audit trails for agent actions
- Policy engine enforces runtime constraints via the Symbi DSL
- AgentPin integration for domain-anchored agent identity
- SchemaPin integration for tool schema verification
- Private keys (`*.private.pem`, `*.private.jwk.json`) must never be committed

## Docker

- Image: `ghcr.io/thirdkeyai/symbi:latest`
- Base: `rust:1.88-slim-bookworm` (builder), `debian:bookworm-slim` (runtime)
- The Dockerfile uses dependency caching with stub sources; cleanup globs must catch `libsymbi*` and `.fingerprint/symbi*`

## Releasing

See `.claude/RELEASE_RUNBOOK.md` for the full release process, including:

- How to determine which crates need version bumps
- Cross-crate version reference update checklist
- CI verification steps before tagging
- Docker build cache pitfalls
- crates.io publish order

## OSS Sync

Private repo is on Gitea. Public mirror is `github.com:ThirdKeyAI/Symbiont.git`.

```bash
bash scripts/sync_oss_to_github.sh --force
```

The script exits with code 1 during cleanup even on success — this is a known quirk.

## DSL Quick Reference

Agent definitions live in `agents/*.symbi` (legacy `.dsl` is also recognized for backward compatibility). Key block types:

```
metadata { version "1.0", author "team", description "What this agent does" }

with { sandbox docker, timeout 30.seconds }

schedule daily_report { cron: "0 9 * * *", timezone: "UTC", agent: "reporter" }

channel slack_support { platform: "slack", default_agent: "helper", channels: ["#support"] }

webhook github_events { path: "/hooks/github", provider: github, agent: "deployer" }

memory context_store { store markdown, path "data/agents", retention "90d" }
```

Parse agent definitions with `symbi dsl -f agents/<name>.symbi`. (The `symbi dsl` subcommand name is intentionally preserved — it's a stable CLI surface, even though the file extension flipped.)

## Sandbox Tiers (all OSS)

The tiers form a monotonically increasing host-isolation ladder:

| Tier  | Backend              | Selection                          | Prerequisites |
|-------|----------------------|------------------------------------|---------------|
| tier0 | None (dev only)      | `with { sandbox = "none" }` / SYMBIONT_ALLOW_UNISOLATED=1 | — |
| tier1 | Docker               | default                            | `docker` daemon |
| tier2 | gVisor (`runsc`)     | `with { sandbox = "gvisor" }`      | `runsc` registered as Docker runtime |
| tier3 | Firecracker microVM  | `with { sandbox = "firecracker" }` | `firecracker` binary + operator-supplied vmlinux + rootfs.ext4 |

All three host-isolation tiers ship in the OSS runtime — no "Enterprise" gating on gVisor or Firecracker. Per-agent tier comes from the DSL `with { sandbox = "..." }` block; project default lives in `[sandbox] tier = "..."` in `symbiont.toml`.

For Tier 3 setup (kernel + rootfs prep, in-VM init contract, hardening checklist), see `docs/firecracker-setup.md`. Scaffold a tier3 project with:

```bash
symbi init --profile assistant --sandbox tier3 \
  --firecracker-kernel /path/to/vmlinux \
  --firecracker-rootfs /path/to/rootfs.ext4
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThirdKeyAI/Symbiont](https://github.com/ThirdKeyAI/Symbiont) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
