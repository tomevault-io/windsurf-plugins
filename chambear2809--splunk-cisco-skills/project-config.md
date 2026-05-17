---
trigger: always_on
description: enables mutating execution for all clients connected to that server process.
---

# Splunk Platform and Cisco Skills — Codex Context

This repository is a working library of Cursor, Codex, and Claude Code agent
skills, MCP tooling, intake templates, reference docs, and shell automation for
planning, rendering, installing, configuring, validating, and handing off Splunk
Platform, Splunk Cloud, Splunk Observability Cloud, Cisco, and adjacent
operational integrations.

It goes well beyond Technology Add-on setup. The catalog covers Cisco product
onboarding, Splunk apps and TAs, Enterprise Security and the broader Splunk
security portfolio, ITSI, SOAR, On-Call, Observability Cloud integrations,
dashboards, detectors, OpenTelemetry collectors, Kubernetes APM
auto-instrumentation, Browser RUM and Session Replay, AWS, ThousandEyes, and
Galileo integrations, HEC, ACS allowlists, PKI, SmartStore, federated search, workload
management, Monitoring Console, license management, indexer clusters, Edge
Processor, Stream, SC4S, SC4SNMP, Universal Forwarders, Linux Splunk Enterprise
hosts, self-managed Kubernetes runtimes, and external-collector topologies.
Most workflows are render-first and validation-heavy, with explicit apply
phases and secret-file guardrails for production changes.

## How To Use This Repo With Codex

When the user asks about a Cisco product or Splunk app/workflow, find the matching
skill in the table below and read `skills/<skill-name>/SKILL.md` for complete
instructions. If more detail is needed, also read `skills/<skill-name>/reference.md`.

The user can also invoke skills directly as slash commands (e.g. `/cisco-catalyst-ta-setup`).

## Skill Catalog

| Skill | Target | Main purpose |
|-------|--------|--------------|
| `cisco-product-setup` | Cisco product catalog workflow | Resolve a Cisco product name from SCAN, classify gaps, and delegate install/configure/validate to the matching Cisco setup skill |
| `cisco-scan-setup` | `splunk-cisco-app-navigator` | Install and validate the Splunk Cisco App Navigator (SCAN) catalog app; trigger catalog sync from S3 |
| `cisco-catalyst-ta-setup` | `TA_cisco_catalyst` | Configure Catalyst Center, ISE, SD-WAN, and Cyber Vision inputs |
| `cisco-catalyst-enhanced-netflow-setup` | `splunk_app_stream_ipfix_cisco_hsl` | Install and validate optional Enhanced Netflow mappings for extra dashboards |
| `cisco-appdynamics-setup` | `Splunk_TA_AppDynamics` | Configure AppDynamics controller and analytics connections, inputs, and dashboards |
| `splunk-appdynamics-setup` | AppDynamics suite router | Route AppDynamics requests, enforce taxonomy coverage, orchestrate child skills, and emit doctor reports |
| `splunk-appdynamics-platform-setup` | AppDynamics On-Premises / Virtual Appliance | Render Enterprise Console, Controller, Events Service, EUM Server, Synthetic Server, HA, upgrade, and secure platform runbooks |
| `splunk-appdynamics-controller-admin-setup` | AppDynamics Controller administration | Configure and validate API clients, OAuth, users, groups, roles, SAML, LDAP, permissions, licensing, and license rules |
| `splunk-appdynamics-agent-management-setup` | Smart Agent / Agent Management | Render Smart Agent remote install, upgrade, rollback, and managed .NET, Database, Java, Machine, and Node.js agent plans |
| `splunk-appdynamics-apm-setup` | AppDynamics APM | Configure and validate business applications, tiers, nodes, business transactions, endpoints, remote services, information points, snapshots, metrics, and app-server snippets |
| `splunk-appdynamics-k8s-cluster-agent-setup` | AppDynamics Cluster Agent | Render Cluster Agent, Kubernetes auto-instrumentation, Splunk OTel Collector, and workload rollout validation assets |
| `splunk-appdynamics-infrastructure-visibility-setup` | AppDynamics Infrastructure Visibility | Render Machine Agent, Server Visibility, Network Visibility, Docker/container visibility, service availability, tags, and infrastructure health rules |
| `splunk-appdynamics-database-visibility-setup` | AppDynamics Database Visibility | Render Database Agent readiness, redacted Database Visibility API collector payloads, and DB server/node/event validation |
| `splunk-appdynamics-analytics-setup` | AppDynamics Analytics | Render Transaction, Log, Browser, Mobile, Synthetic, Connected Devices Analytics, ADQL, Events API schemas, publish, and query validation |
| `splunk-appdynamics-eum-setup` | AppDynamics EUM / RUM | Render Browser RUM, Mobile RUM, IoT RUM, app keys, JS injection, mobile snippets, Session Replay, source maps, and beacon validation |
| `splunk-appdynamics-synthetic-monitoring-setup` | AppDynamics Synthetic Monitoring | Render Browser Synthetic, Synthetic API Monitoring, hosted and private agents, PSA Docker/Kubernetes/Minikube assets, Shepherd URLs, and run validation |
| `splunk-appdynamics-log-observer-connect-setup` | AppDynamics Log Observer Connect | Render LOC setup, legacy Splunk integration detection/disablement, Splunk service-account handoffs, and deep-link validation |
| `splunk-appdynamics-alerting-content-setup` | AppDynamics alerting content | Render health rules, schedules, policies, actions, email digests, suppression, import/export, rollback, and validation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chambear2809/splunk-cisco-skills](https://github.com/chambear2809/splunk-cisco-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
