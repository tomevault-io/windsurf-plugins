---
trigger: always_on
description: Complete reference for Wazuh agent management and monitoring tools. These tools provide comprehensive visibility into agent status, health, configuration, and running processes across your infrastructure.
---

# Agent Management API

Complete reference for Wazuh agent management and monitoring tools. These tools provide comprehensive visibility into agent status, health, configuration, and running processes across your infrastructure.

## Overview

The agent management tools offer six main capabilities:
- **Agent Discovery**: List and filter agents by various criteria
- **Health Monitoring**: Check agent connectivity and operational status
- **Configuration Management**: View agent configurations and settings
- **Process Monitoring**: Monitor running processes on agents
- **Network Monitoring**: Track open ports and network connections
- **Operational Intelligence**: Real-time agent status and performance

---

## 🖥️ get_wazuh_agents

Retrieve comprehensive information about Wazuh agents with flexible filtering options.

### Parameters

| Parameter | Type | Default | Required | Description |
|-----------|------|---------|----------|-------------|
| `agent_id` | string | `null` | No | Specific agent ID to query (3-8 alphanumeric characters) |
| `status` | string | `null` | No | Filter by agent status |
| `limit` | integer | `100` | No | Maximum number of agents to retrieve (1-1000) |

### Agent Status Values

| Status | Description | Typical Use Case |
|--------|-------------|------------------|
| `active` | Agent is connected and sending data | Normal operations |
| `disconnected` | Agent is not currently connected | Troubleshooting connectivity |
| `never_connected` | Agent registered but never connected | Initial setup verification |
| `pending` | Agent registration pending approval | New agent deployment |

### Usage Examples

#### List All Active Agents
```
Ask Claude: "Show me all active Wazuh agents"
```

This queries:
- `status`: "active"
- `limit`: 100 (default)

#### Get Specific Agent Details
```
Ask Claude: "Get details for agent 001"
```

This queries:
- `agent_id`: "001"

#### Find Disconnected Agents
```
Ask Claude: "Show me all disconnected agents"
```

This queries:
- `status`: "disconnected"

#### List First 50 Agents
```
Ask Claude: "List the first 50 agents in the system"
```

This queries:
- `limit`: 50

### Response Format

```json
{
  "agents": [
    {
      "id": "001",
      "name": "web-server-01",
      "ip": "192.168.1.100",
      "status": "active",
      "last_keep_alive": "2024-01-01T14:58:30Z",
      "os": {
        "platform": "ubuntu",
        "version": "20.04",
        "arch": "x86_64"
      },
      "version": "4.8.0",
      "manager": "wazuh-manager-01",
      "group": ["default", "web-servers"],
      "node_name": "worker-01",
      "register_date": "2024-01-01T10:00:00Z",
      "configuration_hash": "ab12cd34ef56",
      "merged_sum": "98765432",
      "config_sum": "12345678"
    }
  ],
  "total_agents": 156,
  "summary": {
    "active": 142,
    "disconnected": 12,
    "never_connected": 2,
    "pending": 0
  },
  "metadata": {
    "query_time": "2024-01-01T15:00:00Z",
    "api_source": "wazuh_server"
  }
}
```

### Agent Information Fields

| Field | Description | Example |
|-------|-------------|---------|
| `id` | Unique agent identifier | "001", "web-01" |
| `name` | Agent hostname | "web-server-01" |
| `ip` | Agent IP address | "192.168.1.100" |
| `status` | Current connection status | "active", "disconnected" |
| `last_keep_alive` | Last communication timestamp | "2024-01-01T14:58:30Z" |
| `os.platform` | Operating system | "ubuntu", "windows", "centos" |
| `version` | Wazuh agent version | "4.8.0" |
| `group` | Agent groups | ["default", "web-servers"] |

---

## ✅ get_wazuh_running_agents

Get a quick list of currently active and running Wazuh agents.

### Parameters

None - returns all active agents.

### Usage Examples

#### Quick Status Check
```
Ask Claude: "Show me all running agents"
```

#### Operational Overview
```
Ask Claude: "Which agents are currently online?"
```

### Response Format

```json
{
  "running_agents": [
    {
      "id": "001",
      "name": "web-server-01",
      "ip": "192.168.1.100",
      "last_keep_alive": "2024-01-01T14:58:30Z",
      "uptime": "72h 15m",
      "status": "active"
    },
    {
      "id": "003",
      "name": "db-server-01",
      "ip": "192.168.1.103",
      "last_keep_alive": "2024-01-01T14:58:45Z",
      "uptime": "168h 22m",
      "status": "active"
    }
  ],
  "summary": {
    "total_running": 142,
    "average_uptime": "96h 30m",
    "oldest_uptime": "720h 15m",
    "newest_connection": "2h 15m"
  }
}
```

---

## 🏥 check_agent_health

Perform comprehensive health check on a specific Wazuh agent.

### Parameters

| Parameter | Type | Default | Required | Description |
|-----------|------|---------|----------|-------------|
| `agent_id` | string | - | **Yes** | ID of the agent to check (3-8 alphanumeric characters) |

### Usage Examples

#### Basic Health Check
```
Ask Claude: "Check the health of agent 001"
```

#### Troubleshooting
```
Ask Claude: "Is agent web-01 healthy?"
```

### Response Format

```json
{
  "agent_health": {
    "agent_id": "001",
    "agent_name": "web-server-01",
    "overall_status": "healthy",
    "health_score": 95,
    "checks": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gensecaihq/Wazuh-MCP-Server](https://github.com/gensecaihq/Wazuh-MCP-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
