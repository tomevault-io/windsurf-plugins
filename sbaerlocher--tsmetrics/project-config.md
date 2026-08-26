---
trigger: always_on
description: Tailscale Prometheus Exporter that combines API metadata with live device metrics for complete network observability.
---

# Agent Instructions — tsmetrics

## Purpose

Tailscale Prometheus Exporter that combines API metadata with live device metrics for complete network observability.

### Device Management (from Tailscale API)

- tailscale_device_count
- tailscale_device_info{device_id, device_name, os, version}
- tailscale_device_authorized{device_id, device_name}
- tailscale_device_last_seen_timestamp{device_id, device_name}
- tailscale_device_user{device_id, device_name, user_email}
- tailscale_device_machine_key_expiry{device_id, device_name}
- tailscale_device_update_available{device_id, device_name}
- tailscale_device_created_timestamp{device_id, device_name}
- tailscale_device_external{device_id, device_name}
- tailscale_device_blocks_incoming_connections{device_id, device_name}
- tailscale_device_ephemeral{device_id, device_name}
- tailscale_device_multiple_connections{device_id, device_name}
- tailscale_device_tailnet_lock_error{device_id, device_name}

### Network Configuration (from Tailscale API)

- tailscale_device_routes_advertised{device_id, device_name, route}
- tailscale_device_routes_enabled{device_id, device_name, route}
- tailscale_device_exit_node{device_id, device_name}
- tailscale_device_subnet_router{device_id, device_name}

### Connectivity & Performance (from Tailscale API)

- tailscale_device_latency_ms{device_id, device_name, derp_region, preferred}
- tailscale_device_endpoints_total{device_id, device_name}
- tailscale_device_client_supports{device_id, device_name, feature}
- tailscale_device_posture_serial_numbers_total{device_id, device_name}

## Context

- **Project:** tsmetrics - Tailscale Prometheus Exporter
- **Language:** Go with Prometheus Client Library and Tailscale tsnet
- **Architecture:** Single binary aggregates Tailscale API data + device client metrics
- **Deployment:** Docker/Kubernetes ready with optional tsnet integration
- **Entry Point:** `cmd/tsmetrics/main.go`

## Core Functionality

### What tsmetrics does

1. **Device Discovery:** Fetch all Tailscale devices via REST API (`/api/v2/tailnet/{tailnet}/devices`)
2. **API Metrics Collection:** Extract device metadata, authorization status, routing configuration,
   user assignments from API
3. **Client Metrics Scraping:** Collect live performance data from each device's metrics endpoint
   (`http://device:5252/metrics`)
4. **Metrics Aggregation:** Combine API and client data into unified Prometheus metrics
5. **Single Endpoint:** Expose all metrics at `/metrics` for Prometheus scraping
6. **Optional tsnet Integration:** Join Tailnet as device for secure internal access

### Data Sources

**Tailscale REST API:**

- Device inventory and online status
- Authorization and authentication state
- Subnet routing and exit node configuration
- User assignments and machine ownership
- Version information and OS details
- Machine key expiry dates

**Device Client Metrics (Port 5252):**

- Network traffic statistics (bytes/packets)
- Connection path information (direct/DERP)
- Health messages and connectivity status
- Route advertisement status

### Output Metrics

```
# Device Management (from Tailscale API)
tailscale_device_count
tailscale_device_info{device_id, device_name, online, os, version}
tailscale_device_authorized{device_id, device_name}
tailscale_device_last_seen_timestamp{device_id, device_name}
tailscale_device_user{device_id, device_name, user_email}
tailscale_device_machine_key_expiry{device_id, device_name}

# Network Configuration (from Tailscale API)
tailscale_device_routes_advertised{device_id, device_name, route}
tailscale_device_routes_enabled{device_id, device_name, route}
tailscale_device_exit_node{device_id, device_name}
tailscale_device_subnet_router{device_id, device_name}

# Network Performance (from device client metrics)
tailscaled_inbound_bytes_total{device_id, device_name, path}
tailscaled_outbound_bytes_total{device_id, device_name, path}
tailscaled_inbound_packets_total{device_id, device_name, path}
tailscaled_outbound_packets_total{device_id, device_name, path}
tailscaled_inbound_dropped_packets_total{device_id, device_name}
tailscaled_outbound_dropped_packets_total{device_id, device_name, reason}
tailscaled_health_messages{device_id, device_name, type}
tailscaled_advertised_routes{device_id, device_name}
tailscaled_approved_routes{device_id, device_name}

# Exporter Health
tsmetrics_scrape_duration_seconds{target}
tsmetrics_scrape_errors_total{target, error_type}
tsmetrics_api_requests_total{endpoint}
```

## Build & Development

### Build System (dde + just)

Development runs inside a container managed by **[dde](https://dde.sh)** —
a Docker dev environment manager that owns the container lifecycle and provides a shared
Traefik reverse proxy for `*.test` hostnames (app reachable at `https://tsmetrics.test`).
Project config lives in `.dde/config.yml`; compose stack in `docker-compose.yml`.

**just** only wraps multi-step or parameterized commands; trivial one-liners are invoked directly.

Bootstrap (one-time per machine): `brew tap whatwedo/tap && brew install dde && dde system:up`.

Optional git hooks via [lefthook](https://lefthook.dev) — install once with
`brew install lefthook && lefthook install`. Pre-commit auto-formats staged Go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbaerlocher/tsmetrics](https://github.com/sbaerlocher/tsmetrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
