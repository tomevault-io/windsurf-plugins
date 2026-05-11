---
trigger: always_on
description: Intelligent UPS monitoring daemon for NUT (Network UPS Tools). Orchestrates graceful shutdown of VMs, containers, remote servers, and local systems during power events.
---

# Eneru

Intelligent UPS monitoring daemon for NUT (Network UPS Tools). Orchestrates graceful shutdown of VMs, containers, remote servers, and local systems during power events.

## Development Setup

**CRITICAL: NEVER run `pip`, `pip3`, `python -m pip`, `python`, `pytest`, or any other dev/Python tooling directly against the system Python. ALL Python work — install, uninstall, run, test, version-check — MUST happen inside a `uv` virtualenv. No exceptions.**

This rule applies to *every* operation, including:

- Installing packages (`pip install ...`)
- **Uninstalling packages (`pip uninstall ...`) — even to "clean up" or fix broken state.** A system-wide `pip uninstall eneru` will rip out files claimed by both pip and the deb/rpm package (e.g. `/usr/local/bin/eneru`), breaking the package install. If the system has stale pip-installed Python packages owned by Eneru, the only correct cleanup is to reinstall the deb/rpm to restore the package's files, then leave the pip remnants alone, *or* delete only the pip-owned site-packages directory by hand after confirming nothing else needs it. Never invoke pip itself.
- Running the test suite (`pytest`)
- Running ad-hoc scripts (`python -c '...'`)
- Editable dev installs (`pip install -e .` — use `uv pip install -e .` inside the venv only)

If you need to verify the installed deb/rpm package, invoke the package's own entry point (e.g. `/usr/local/bin/eneru version`, `python3 /opt/ups-monitor/eneru.py version`) — these read from `/opt/ups-monitor/`, not from system Python paths, so no venv is required.

```bash
# Create and activate virtualenv (disposable tmp folder)
uv venv /tmp/eneru-venv
source /tmp/eneru-venv/bin/activate

# Install package with all dev dependencies
uv pip install -e ".[dev,notifications,docs]"
```

## Commands

```bash
# Testing (always inside virtualenv)
pytest                              # Run all tests
pytest -m unit                      # Unit tests only
pytest -m integration               # Integration tests only
pytest --cov=src/eneru              # With coverage

# Development
python -m eneru validate --config examples/config-reference.yaml
python -m eneru run --dry-run --config examples/config-reference.yaml

# Documentation
mkdocs serve                        # Local docs preview
```

## Project Structure

```
src/eneru/                      # Main package
  AGENTS.md                     # Module map + mixin pattern (agent context)
  __init__.py                   # Public API exports
  __main__.py                   # CLI entry point (python -m eneru)
  version.py                    # Version string (single source of truth)
  config.py                     # Configuration dataclasses + ConfigLoader
  state.py                      # MonitorState dataclass
  logger.py                     # TimezoneFormatter + UPSLogger
  notifications.py              # NotificationWorker (Apprise integration)
  utils.py                      # Helper functions (run_command, etc.)
  actions.py                    # REMOTE_ACTIONS templates
  monitor.py                    # UPSGroupMonitor core: init, polling, orchestration, main loop
  multi_ups.py                  # MultiUPSCoordinator (thread-per-group)
  cli.py                        # CLI argument parsing + main()
  shutdown/                     # Per-phase shutdown mixins
    vms.py                      # VMShutdownMixin (libvirt)
    containers.py               # ContainerShutdownMixin (docker/podman + compose)
    filesystems.py              # FilesystemShutdownMixin (sync + unmount)
    remote.py                   # RemoteShutdownMixin (SSH-based remote servers)
  health/                       # Health-monitoring mixins
    voltage.py                  # VoltageMonitorMixin (thresholds, AVR, bypass, overload)
    battery.py                  # BatteryMonitorMixin (depletion rate, anomaly detection)

tests/                          # pytest tests
  conftest.py                   # Shared fixtures
  test_constants.py             # Shared test constants (sample webhook URLs, etc.)
  test_config_loading.py        # Config: defaults + YAML file parse
  test_config_notifications.py  # Config: legacy Discord, avatar handling
  test_config_filesystems.py    # Config: mount path parsing
  test_config_vm_containers.py  # Config: compose files, container runtime
  test_config_remote.py         # Config: remote servers, ordering, safety margin
  test_config_validation.py     # Config: cross-field validation, edge cases
  test_*.py                     # Unit/integration tests for non-config modules
  e2e/                          # End-to-end tests
    docker-compose.yml          # E2E test environment
    config-e2e*.yaml            # E2E test configs
    nut-dummy/Dockerfile        # NUT server simulator
    ssh-target/Dockerfile       # SSH target container

docs/                           # MkDocs documentation (ReadTheDocs)
  index.md                      # Homepage
  getting-started.md            # Installation guide
  configuration.md              # Config reference
  triggers.md                   # Shutdown triggers
  notifications.md              # Apprise setup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m4r1k/Eneru](https://github.com/m4r1k/Eneru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
