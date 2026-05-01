---
trigger: always_on
description: AgenticOps uses a multi-agent architecture where an orchestrator routes user queries to specialist agents. Each specialist has access to specific MCP tools and skills relevant to its domain.
---

# AgenticOps Agent Definitions

## Agent Overview

AgenticOps uses a multi-agent architecture where an orchestrator routes user queries to specialist agents. Each specialist has access to specific MCP tools and skills relevant to its domain.

## Agent Graph Structure

```
                            ┌──────────────┐
                            │ Orchestrator │
                            └──────┬───────┘
                                   │ (conditional routing)
                  ┌────────────────┼────────────────┬────────────┬────────────┐
                  ▼                ▼                ▼            ▼            ▼
         ┌────────────┐   ┌───────────┐   ┌──────────┐   ┌───────────┐   ┌──────────┐
         │Troubleshoot│   │ Compliance│   │ Security │   │ Discovery │   │ Topology │
         └─────┬──────┘   └─────┬─────┘   └────┬─────┘   └─────┬─────┘   └────┬─────┘
               │                │                │               │              │
               │                │                ▼               ▼              ▼
               │                │          ┌──────────┐   ┌──────────┐   ┌──────────┐
               │                │          │ Testing  │   │Remediation│   │          │
               │                │          └────┬─────┘   └────┬─────┘   │          │
               │                │               │              │          │          │
               └────────────────┴───────────────┴──────────────┴──────────┴──────────┘
                                                │
                                         ┌────────────┐
                                         │   Canvas   │
                                         └──────┬─────┘
                                                ▼
                                              END
```

## Agent Definitions

### Orchestrator Agent
- **Role**: Routes user queries to the appropriate specialist agent
- **MCP Tools**: None (delegates to specialists)
- **Skills**: Query classification
- **Routing Logic**:
  - WiFi/wireless/connectivity/performance/latency/slow → Troubleshooting
  - Config/SSID settings/VLAN/switch port/audit → Compliance
  - Firewall/security/threat/ACL/content filtering → Security
  - Inventory/devices/networks/health/status/clients → Discovery
  - Topology/network map/device connections/LLDP/CDP → Topology
  - Run tests/instant test/connectivity validation → Testing
  - Fix/change/update/configure (write operations) → Remediation
- **System Prompt Guidelines**: Classify intent, never call MCP tools directly, return the specialist name

### Troubleshooting Agent
- **Role**: Diagnoses network issues by correlating data from Meraki and ThousandEyes
- **MCP Tools**:
  - Meraki: devices, clients, events, wireless stats, uplink status
  - ThousandEyes: test results, metrics, path visualization, anomaly detection
- **Skills**: `wireless_troubleshooting`, `wan_performance`, `application_slowness`
- **System Prompt Guidelines**: Gather data systematically, correlate across sources, provide root cause analysis

### Compliance Agent
- **Role**: Evaluates configurations against network requirements and policies
- **MCP Tools**:
  - Meraki: SSIDs, firewall rules, VPN config, switch ports, VLANs
- **Skills**: `config_audit`, `policy_compliance`
- **System Prompt Guidelines**: Check configurations methodically, flag deviations, suggest remediation

### Security Agent
- **Role**: Assesses security posture, analyzes firewall rules, detects threats
- **MCP Tools**:
  - Meraki: firewall rules, security events, content filtering, ACLs
  - ThousandEyes: alerts, outage detection
- **Skills**: `security_posture`, `firewall_review`
- **System Prompt Guidelines**: Evaluate against security best practices, identify vulnerabilities, prioritize by severity

### Discovery Agent
- **Role**: Explores network inventory, device status, client lists, overall health
- **MCP Tools**:
  - Meraki: organizations, networks, devices, clients, inventory, licensing, events
  - ThousandEyes: test inventory, account groups
- **Skills**: `network_inventory`, `organizational_summary`
- **System Prompt Guidelines**: Provide comprehensive overviews, summarize health status, organize by network/site, generate interactive tables for device/client/network listings

### Topology Agent
- **Role**: Builds network topology maps showing physical and logical device connections
- **MCP Tools**:
  - Meraki: networks, devices, LLDP/CDP (via call_meraki_api), uplink status
- **Skills**: `network_topology`
- **System Prompt Guidelines**: Never generate ASCII art or text diagrams, call LLDP/CDP for each device to discover neighbors, provide brief summary and let Canvas agent create the visual topology card
- **Special Configuration**:
  - 10 iterations (vs 6 for other agents) to handle sequential LLDP/CDP calls per device
  - 90-second timeout (vs 60s) due to multiple device queries
  - No message trimming to preserve tool call pairing

### Testing Agent
- **Role**: Runs on-demand ThousandEyes instant tests for connectivity validation
- **MCP Tools**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robbarto2/AgenticOps](https://github.com/robbarto2/AgenticOps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
