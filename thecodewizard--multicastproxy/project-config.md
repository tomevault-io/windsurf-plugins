---
trigger: always_on
description: Build and maintain a Windows service that relays multicast traffic between two Layer 3 networks by:
---

# AGENTS.md

## Project purpose
Build and maintain a Windows service that relays multicast traffic between two Layer 3 networks by:
1. receiving multicast locally,
2. tunneling payloads as unicast to a remote peer,
3. re-emitting them as multicast on the remote network,
4. optionally rewriting subnet references inside the payload.

This project must prioritize reliability, safe operations, simple configuration for non-technical users, and maintainable production code.

---

## Primary implementation goals
- Implement this as a Windows Service, not as a console-only utility.
- Prefer .NET Worker Service hosting with Windows Service integration.
- The service must be able to run unattended and recover cleanly from transient network failures.
- The same codebase should support both relay directions through configuration, not separate binaries unless clearly justified.

---

## Non-negotiable functional requirements
- Multicast listener must support one or more configured multicast source ports.
- Valid port ranges are 0-65535; reject invalid configuration with clear logs.
- A unicast tunnel port must be configurable for agent-to-agent forwarding.
- A destination IP must be configurable for the remote agent.
- Payload rewrite settings are optional:
  - If no rewrite source and destination subnet are configured, payloads must be forwarded unchanged.
  - If rewrite is enabled, rewrite only the intended subnet text and preserve all other payload bytes where possible.
- Do not assume a single multicast use case or hardcode customer-specific addresses in production code.

---

## Configuration rules
- Configuration must be simple for non-technical operators.
- Prefer `appsettings.json` plus environment-specific overrides.
- All important settings must be documented in a sample config file.
- Never hardcode production IPs, ports, secrets, or environment-specific paths.
- Validate configuration at startup and fail fast with human-readable Event Log messages.
- Use explicit typed options classes for configuration.

Suggested configuration model:
- `MulticastPorts`: array of integers
- `TunnelPort`: integer
- `DestinationIP`: string
- `MulticastGroup`: string if needed
- `ListenInterfaceIP`: optional string
- `SendInterfaceIP`: optional string
- `PayloadRewriteSourceSubnet`: optional string
- `PayloadRewriteDestinationSubnet`: optional string
- `Logging:LogLevel`

---

## Security and public-repo rules
- This repository is public. Never introduce secrets, API keys, tokens, credentials, private endpoints, or customer-specific addressing into source control.
- Assume all config files except example templates may contain sensitive environment details; avoid logging sensitive config values unless explicitly running in debug mode.
- Do not add any network-driven code execution, remote script download, or unsafe binary replacement logic.
- Keep all public examples on RFC documentation ranges and documentation-only values.

---

## Deployment and upgrade rules
- This service must not self-update.
- New versions are distributed as MSI packages and installed manually or through external deployment tooling.
- Do not implement GitHub polling, release checking, updater helpers, or in-place binary replacement unless explicitly requested later.
- Keep the runtime service focused on packet relay, configuration validation, and logging.

---

## Networking and protocol rules
- Use UDP sockets carefully and explicitly document socket options.
- Be conservative with multicast behavior; do not broaden multicast scope unless required.
- Avoid packet amplification, loops, or duplicate forwarding.
- Protect against relay loops between two agents. Add a loop-prevention strategy if traffic can return to the sender.
- Packet handling should be resilient: malformed packets should be logged at debug/warning level and should not crash the service.
- Preserve raw bytes unless a rewrite is explicitly configured and safely applicable.
- Do not assume payloads are always ASCII unless the protocol guarantees it. If text rewrite is attempted, handle non-text payloads safely.

---

## Windows service and operations rules
- Service must integrate cleanly with Windows Service Control Manager.
- Log operational events to Windows Event Log.
- Logging must be configurable so normal operation is quiet, while debug mode provides detailed packet-flow troubleshooting.
- Expected logs:
  - startup/shutdown
  - config validation failures
  - listener bind/join failures
  - remote send failures
  - packet counts / packet summaries at debug level
- Do not spam Event Log with per-packet informational logs in normal mode.

---

## Code quality rules
- Prioritize production-ready code over proof-of-concept shortcuts.
- Break the code into small services/classes with clear responsibilities.
- Avoid giant Program.cs implementations.
- Prefer the following separation:
  - configuration/options
  - multicast receive service
  - unicast tunnel service
  - multicast re-emission service
  - payload rewrite service
  - Windows service host/bootstrap
  - logging/event log integration
- Keep customer-specific constants out of business logic.
- Use dependency injection and typed options.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecodewizard/MulticastProxy](https://github.com/thecodewizard/MulticastProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
