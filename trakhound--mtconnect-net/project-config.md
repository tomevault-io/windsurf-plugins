---
trigger: always_on
description: `MTConnect-Agent` is the standalone HTTP agent host. It loads an `agent.config.yaml` (or `agent.config.json`) configuration file, builds an `MTConnectAgentBroker`, instantiates every configured agent module, and runs them inside a single process that can be invoked from a terminal, hosted as a Windows service, or run as a systemd unit.
---

# Agent CLI

`MTConnect-Agent` is the standalone HTTP agent host. It loads an `agent.config.yaml` (or `agent.config.json`) configuration file, builds an `MTConnectAgentBroker`, instantiates every configured agent module, and runs them inside a single process that can be invoked from a terminal, hosted as a Windows service, or run as a systemd unit.

The CLI surface is defined by `MTConnectAgentApplication.Run(args)` in `agent/MTConnect.NET-Applications-Agents/MTConnectAgentApplication.cs`. The shipped `mtconnect.net-agent` binary in `agent/MTConnect.NET-Agent/` is a thin entry point that calls `Run(args, true)` on this class. Embedders that scaffold their own agent from the `MTConnect.NET-Agent-Template` NuGet template get the same CLI surface for free.

## Synopsis

```text
mtconnect.net-agent [help|install|install-start|start|stop|remove|debug|run|run-service] [configuration_file]
```

If no command is given, the agent runs in `run` mode (foreground, info-level logging). If no configuration file is given, the agent looks for `agent.config.json` first, then `agent.config.yaml`, then copies `agent.config.default.yaml` into place if neither file exists.

## Commands

| Command | Description |
|---|---|
| `help` | Print usage information and exit. |
| `run` | Run the agent in the foreground at info-level logging. Default when no command is supplied. The process blocks until killed with `Ctrl+C` or SIGINT. |
| `debug` | Run the agent in the foreground at **debug-level** logging on the console target. Same lifecycle as `run`. |
| `trace` | Run the agent in the foreground at **trace-level** logging on the console target. Same lifecycle as `run`. |
| `run-service` | Run the agent as a Windows service (non-blocking). On non-Windows operating systems the agent logs that the command is unsupported and exits. |
| `install` | Install the agent as a Windows service. Windows only. Stops and removes any existing service of the same name first. |
| `install-start` | Install the agent as a Windows service and immediately start it. Windows only. |
| `start` | Start the previously installed Windows service. Windows only. |
| `stop` | Stop the Windows service. Windows only. |
| `remove` | Stop and remove the Windows service. Windows only. |
| `reset` | Clear the durable observation buffer, the durable asset buffer, and the file-index directory at `durableBufferPath`. Use after a corrupted shutdown or when a fresh-state restart is wanted. |

## Arguments

| Argument | Description |
|---|---|
| `configuration_file` | Path to the agent configuration file. Absolute or relative to the executable's directory. If omitted, the agent looks for `agent.config.json`, then `agent.config.yaml`, in the executable's directory, and copies `agent.config.default.yaml` into place if neither exists. |

## Example invocations

Run in the foreground against an explicit config file:

```bash
./mtconnect.net-agent run /etc/mtconnect/agent.config.yaml
```

Run with debug-level console logging while developing a new module:

```bash
./mtconnect.net-agent debug
```

Install the agent as a Windows service named per `serviceName` in the config and start it immediately:

```powershell
.\mtconnect.net-agent.exe install-start C:\ProgramData\MTConnect\agent.config.yaml
```

Clear the durable buffer (e.g. after schema-incompatible config changes):

```bash
./mtconnect.net-agent reset
```

Run from source against an embedded `agent.config.yaml` during development:

```bash
dotnet run --project agent/MTConnect.NET-Agent -- debug
```

## Configuration file

The agent reads YAML by default (CamelCase keys). JSON is also accepted; when both are present, JSON takes precedence and is converted to YAML on first run. Default filenames live alongside the agent binary:

| File | Role |
|---|---|
| `agent.config.yaml` | The active configuration. |
| `agent.config.json` | Alternative active configuration, JSON format. |
| `agent.config.default.yaml` | Shipped default. Copied to `agent.config.yaml` on first run if neither active file exists. |
| `NLog.config` | NLog configuration. Copied from `NLog.default.config` on first run if missing. |
| `Devices.xml` | Device-information-model file (or directory) loaded into the agent. Path comes from the `devices` key in the config. |

The active configuration is reloaded when `monitorConfigurationFiles: true` is set (the default) and the file is edited at runtime, with a minimum gap of `configurationFileRestartInterval` seconds between restarts.

### Top-level keys

Backed by `AgentApplicationConfiguration` and the inherited `AgentConfiguration` base.

| Key (YAML) | Type | Default | Description |
|---|---|---|---|
| `changeToken` | string | new GUID per save | Internal token regenerated whenever the agent rewrites the file. Surfaces in the Agent device's `mtconnect:ChangeToken` data item so consumers can detect config rewrites. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrakHound/MTConnect.NET](https://github.com/TrakHound/MTConnect.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
