---
trigger: always_on
description: `dirextalk-deployer` is a cross-platform deployment product and agent skill, not a Linux-only script collection. Maintain it as a portable orchestration layer driven by Git Bash on native Windows and Bash on Linux/macOS/WSL while deploying a Linux-based Dirextalk server.
---

# AGENTS.md

`dirextalk-deployer` is a cross-platform deployment product and agent skill, not a Linux-only script collection. Maintain it as a portable orchestration layer driven by Git Bash on native Windows and Bash on Linux/macOS/WSL while deploying a Linux-based Dirextalk server.

## Development

- This vNext repository is being built from zero. Implement the best target
  contract directly. Do not retain historical-version compatibility code.
- Do not add dual paths, version negotiation, compatibility shims, or fallback
  branches for superseded designs. Replace the old contract and migrate test or
  development data directly; a frozen external boundary requires an explicit
  product decision, not a second production path.
- Prioritize core product behavior and real device/node evidence. Do not add
  anti-counterfeit or exhaustive adversarial-observer machinery beyond the
  required release provenance unless a concrete product threat or gate needs it.
- Review the final production path strictly; model and fixture tests do not
  substitute for executable integration evidence.

## Product Scope

- Deploy, resume, verify, destroy, and locally wire a production Dirextalk message server.
- Install the independently released `YingSuiAI/dirextalk-updater` host binary from the deployer-owned immutable version/commit/SHA pin. The deployer does not embed or build updater Go source.
- Own the production split Compose topology, host lifecycle and image-update
  wrappers under `scripts/cloud-init/split/runtime`, together with their
  operational tests and canonical bundle. Message Server and Agent repositories
  own only their application image builds and formal version-tag publication.
- Fresh deployment installs the updater control plane with its resident watchdog disabled. Direct-version upgrades are client initiated; do not reintroduce the retired daily updater GitHub discovery timer or service.
- Treat `SKILL.md` as the compact agent-facing entrypoint. Detailed runbooks belong in `references/`; `scripts/` are the stable implementation entrypoints.
- `SKILL.md` is a user-facing runbook that must remain usable by less capable models. Its Freshness Gate, step-by-step onboarding, semantic confirmation policy, AWS promotional/billing reminders, and repeated safety guidance are intentional product behavior; preserve them unless the product owner explicitly changes that onboarding contract.
- The supported local conversation bridge is `dirextalk-connect`, installed from `dirextalk-connect@latest` by default or built from `YingSuiAI/dirextalk-connect`.
- MCP support is capability-driven and is separate from bridge-agent support. Declared MCP consumers connect directly to the deployed message server's HTTP endpoint; unknown runtimes never receive a generic fallback.
- Supported local agent targets are the dirextalk-connect agent providers, treated as peers: `acp`, `antigravity`, `claudecode`, `codex`, `copilot`, `cursor`, `devin`, `gemini`, `iflow`, `kimi`, `opencode`, `pi`, `qoder`, `reasonix`, and `tmux`.
- Do not reintroduce legacy local gateway installation flows or third-party chat platform wiring.
- Do not hard-code one developer's home directory, shell, agent executable path, AWS region, domain, node id, token, or password.

## Platform Law

Every deployer change must classify paths and commands by the platform that will consume them:

- **Remote server paths** are Linux paths inside EC2/cloud-init/Docker, such as `/var/dirextalk-message-server`, `/var/dirextalk-message-server/p2p/bootstrap.json`, and `/etc/dirextalk-message-server`.
- **Deployer execution paths** are used by the orchestration engine. On Git Bash, normalize paths before passing them to Windows-native Node.js, AWS CLI, curl, or agent executables. Do not rely on implicit MSYS argv conversion: parent runtimes may set `MSYS_NO_PATHCONV=1`, and `/tmp` redirections otherwise diverge from native-tool file arguments.
- **Local bridge paths** are consumed by `dirextalk-connect` and the local agent process. On Windows they must be Windows-compatible paths, not `/mnt/c/...` or Git Bash-only `/c/...` paths.
- **Documentation paths** must be portable examples using `$HOME`, `%USERPROFILE%`, `$env:USERPROFILE`, `<service_id>`, or `<domain>`, not machine-specific absolute paths.

If a change writes a path into `state.json`, `credentials.json`, `dirextalk-connect/config.toml`, docs, or printed commands, verify which process will read that path and format it for that process. Do not generate an artifact without a current consumer.
Use `scripts/lib/git-bash.sh`, `scripts/lib/local-paths.sh`, and `scripts/lib/paths.sh` for the Git Bash platform check and path conversion. These helpers must lexically recognize `C:\Users\alice`, `C:/Users/alice`, `/mnt/c/Users/alice`, `/cygdrive/c/Users/alice`, and `/c/Users/alice` before calling shell-specific conversion tools.

## Entrypoints

- All supported hosts run `bash scripts/orchestrate.sh`, `bash scripts/destroy.sh`, `bash scripts/update.sh`, and `bash scripts/reset-app-data.sh`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YingSuiAI/dirextalk-deployer](https://github.com/YingSuiAI/dirextalk-deployer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
