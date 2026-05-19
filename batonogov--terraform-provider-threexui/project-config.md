---
trigger: always_on
description: manages self-hosted panels that frequently use self-signed certificates;
---

# Repository Guidelines

This file is the source of truth for coding agents working in this repository.

## Purpose

Terraform provider for the [3x-ui](https://github.com/MHSanaei/3x-ui) panel,
implemented in Go with `terraform-plugin-framework`. The Go module is
`github.com/batonogov/terraform-provider-threexui`; the Terraform Registry
address is `batonogov/threexui`.

## Project Structure & Module Organization

```text
provider/              - all provider code
  provider.go          - ThreeXUIProvider (framework): Metadata, Schema, Configure, Resources, DataSources
  client.go            - HTTP client for 3x-ui API (cookie auth, auto re-login)
  types.go             - Inbound, ClientTraffic, APIResponse, ParseJSONField
  resource_inbound.go  - threexui_inbound resource (CRUD, Reality, settings defaults)
  resource_inbound_client.go - threexui_inbound_client resource (mutex, UUID)
  resource_settings_tabs.go  - panel_general/security/telegram/subscription (typed attributes)
  resource_panel_user.go     - threexui_panel_user resource (admin credentials change)
  resource_xray_settings.go  - CRUD for xray_basics/dns/routing/balancers/reverse/outbounds (typed attributes)
  xray_basics_schema.go      - model, schema, expand/flatten for xray_basics (log, policy, api, stats)
  xray_dns_schema.go         - model, schema, expand/flatten for xray_dns (servers, hosts)
  xray_routing_schema.go     - model, schema, expand/flatten for xray_routing (rules)
  xray_balancers_schema.go   - model, schema, expand/flatten for xray_balancers
  xray_reverse_schema.go     - model, schema, expand/flatten for xray_reverse (bridges, portals)
  xray_outbounds_schema.go   - model, schema, expand/flatten for xray_outbounds (per-protocol settings)
  inbound_settings_schema.go - model, schema, expand/flatten for per-protocol settings (vless, trojan, ss, http, socks, mixed, wg, dokodemo, hysteria)
  inbound_stream_settings_schema.go - model, schema, expand/flatten for stream_settings (tcp, ws, grpc, httpupgrade, xhttp, kcp, hysteria, reality, sockopt)
  inbound_sniffing_schema.go - model, schema, expand/flatten for sniffing
  settings.go          - buildSettingsJSON(map[string]any), flattenSettings(string), expand/flatten clients/fallbacks/peers
  stream_settings.go   - buildStreamSettingsJSON(map[string]any), flattenStreamSettings(string), expand/flatten per-transport
  sniffing.go          - buildSniffingJSON(map[string]any), flattenSniffing(string)
  settings_helpers.go  - mergeSettings
  list_helpers.go      - typesListToAnySlice, typesListInt64ToAnySlice, anySliceToTypesList
  default_settings.go  - default settings per protocol, applyDefaultInboundSettings
  resource_xray_version.go - threexui_xray_version resource (install/manage Xray core version)
  data_source_*.go     - data sources (inbounds, server_status, settings, xray_config, xray_versions, online_clients)
  testdata/            - round-trip fixtures for corpus_test.go; see provider/testdata/README.md to refresh
examples/              - example TF configs for manual testing
docs/
  index.md             - provider docs landing page (Terraform Registry)
  resources/           - per-resource Registry docs
  data-sources/        - per-data-source Registry docs
  guides/              - operational walkthroughs (backup-as-code, server-migration, bulk-clients)
README.md              - English README; localized in README.ru_RU.md, README.fa_IR.md, README.ar_EG.md, README.zh_CN.md, README.es_ES.md
3x-ui-<version>/       - 3x-ui source snapshots (in .gitignore, for reference/diffing)
docker-compose.yaml    - 3x-ui on port 2053 (version via THREEXUI_VERSION env, default v3.0.2)
Taskfile.yml           - task build / test / fmt
.github/FUNDING.yml    - GitHub Sponsors funding config (github: batonogov)
.github/workflows/
  ci.yml               - lint, govulncheck, unit tests, acceptance tests, compatibility matrix (PR + push main)
  docs.yml             - docs/examples validation: terraform fmt, markdownlint, yamllint (PR + push main)
  release-please.yml   - Release Please + GoReleaser (conventional commits -> semver tag -> build + sign + publish)
  flake-tracking.yml   - weekly compat matrix with continue-on-error, posts per-version results table as GitHub issue
```

## Provider Resources

| Terraform Resource | File | Description |
| --- | --- | --- |
| `threexui_inbound` | resource_inbound.go + inbound_*_schema.go | Inbound (vless/vmess/trojan/ss/http/mixed/wg/tunnel/hysteria). Typed blocks for settings/stream_settings/sniffing |
| `threexui_inbound_client` | resource_inbound_client.go | Client within an inbound. Typed attributes |
| `threexui_panel_general` | resource_settings_tabs.go | Panel settings (web, LDAP). Typed attributes |
| `threexui_panel_security` | resource_settings_tabs.go | 2FA. Typed attributes |
| `threexui_panel_user` | resource_panel_user.go | Admin credentials change. Write-only (no read API) |
| `threexui_panel_telegram` | resource_settings_tabs.go | Telegram bot. Typed attributes |
| `threexui_panel_subscription` | resource_settings_tabs.go | Subscriptions. Typed attributes |
| `threexui_xray_basics` | resource_xray_settings.go + xray_basics_schema.go | Base Xray config (merge root). Typed blocks |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [batonogov/terraform-provider-threexui](https://github.com/batonogov/terraform-provider-threexui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
