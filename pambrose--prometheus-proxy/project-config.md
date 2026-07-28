---
trigger: always_on
description: The agent runs inside the firewall and scrapes local metrics endpoints on behalf of the proxy.
---


# Agent Configuration

The agent runs inside the firewall and scrapes local metrics endpoints on behalf of the proxy.

## Path Configs

The `pathConfigs` array defines which metrics endpoints the agent exposes through the proxy:

### Basic Configuration

```hocon
--8<-- "ConfigExamples.txt:path-config-basic"
```

### Multiple Endpoints

```hocon
--8<-- "ConfigExamples.txt:path-config-multi"
```

### With Labels

Labels are included in service discovery responses and can be used for filtering:

```hocon
--8<-- "ConfigExamples.txt:path-config-with-labels"
```

Each path config entry has these fields:

| Field    | Required | Description                                                   |
|:---------|:---------|:--------------------------------------------------------------|
| `name`   | Yes      | Human-readable endpoint name (for logs and debugging)         |
| `path`   | Yes      | Single URL segment on the proxy that Prometheus scrapes (no embedded `/`) |
| `url`    | Yes      | Actual metrics endpoint the agent fetches from                |
| `labels` | No       | JSON string of labels for service discovery (default: `"{}"`) |

!!! note "Paths are a single segment"

    A `path` is one URL segment — it must not contain an embedded `/`. The proxy serves each
    registered path at `/<path>` (a one-segment route), so a multi-segment value like
    `app/metrics` is rejected at registration (the agent logs the failure rather than reconnecting).
    Use `app_metrics` instead.

## Dynamic Target Discovery

By default `pathConfigs` is read once at startup, so adding or removing a target means editing the
config and restarting the agent. Enable dynamic discovery to have the agent reconcile its registered
paths against a **watched file** at runtime — no restart, and paths that did not change keep scraping:

```hocon
agent {
  discovery {
    enabled = false                                    // Enable dynamic discovery
    file.path = "/etc/prometheus-proxy/targets.conf"   // HOCON/JSON list of paths
    reconcileIntervalSecs = 30                         // Poll and full-resync interval
  }
}
```

The discovery file holds a `paths` list of the same `{ name, path, url, labels }` entries as
`pathConfigs`:

```hocon
paths = [
  { name = "app1", path = "app1_metrics", url = "http://app1:9090/metrics" }
  { name = "app2", path = "app2_metrics", url = "http://app2:9090/metrics" }
]
```

Every interval the agent registers newly-listed paths, unregisters removed ones, and re-registers a
path whose URL or labels changed.

Because the file is a plain list, it can be **generated** rather than hand-maintained — a Kubernetes
`ConfigMap` mounted into the agent pod (see
[the Kubernetes walkthrough](../kubernetes.md#updating-targets-without-restarting-the-agent)), an
Ansible template, or a cron job querying a service registry. Automation then needs permission only to
write a file, never to restart the agent, and the churning target list stays separate from the stable
config holding ports, TLS, and the proxy address.

A fully annotated version of this file -- required versus defaulted fields, the single-segment `path`
rule, HOCON substitutions, and the empty-versus-deleted distinction -- ships as
[`examples/discovery-targets.conf`](https://github.com/pambrose/prometheus-proxy/blob/master/examples/discovery-targets.conf)
and is shown verbatim on the [Example Configs](../examples.md) page.

| Situation                               | Behavior                                                     |
|:----------------------------------------|:------------------------------------------------------------|
| Path in both `pathConfigs` and the file | Static wins; the discovered entry is skipped (logged)       |
| File missing / unreadable / malformed   | Keeps the last-known-good set (a read failure drops nothing) |
| Valid but empty file                    | Removes all discovered paths                                |
| `pathConfigs` empty                     | Discovery-only — every path comes from the file             |

!!! note "Polling, not file-watching"

    Discovery polls the file on the interval rather than relying on OS file-change events, which are
    unreliable under Kubernetes ConfigMap updates (symlink swaps) and some bind mounts. The interval
    doubles as a full-resync safety net.

!!! note "Config-file only"

    `discovery.file.path` points at a list, so — like `pathConfigs` — it has no CLI/env equivalent.
    The scalar `enabled`, `file.path`, and `reconcileIntervalSecs` can also be set via `-D` overrides.

Dynamic target discovery is distinct from [Prometheus service discovery](../service-discovery.md),
which exposes an endpoint so *Prometheus* can find proxied targets; discovery instead lets the *agent*
pick up target changes behind the firewall without a restart.

On the proxy's [dashboard](../web-dashboard.md#the-paths-layout), each path is tagged `cfg` (static)
or `disc` (discovered), so when a target is unexpectedly present or absent you can tell at a glance
whether a human or the automation registered it.

## Proxy Connection

```hocon
agent {
  proxy {
    hostname = "proxy-host.example.com"   // Proxy hostname

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pambrose/prometheus-proxy](https://github.com/pambrose/prometheus-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
