---
trigger: always_on
description: - **Name**: kubectl-mcp-server
---

# kubectl-mcp-server Project Memory

## Project Overview
- **Name**: kubectl-mcp-server
- **Version**: 1.22.0
- **Description**: A Model Context Protocol (MCP) server for Kubernetes with 270+ tools, 8 resources, and 8 prompts
- **Framework**: FastMCP 3.0.0b1 (Python)
- **Repository**: https://github.com/rohitg00/kubectl-mcp-server

## Current State (as of 2026-02-02)

### Latest Release: v1.22.0
- **Changes**: Added kubectl-mcp-app - 8 interactive UI dashboards for Kubernetes management
- **Tool Count**: 270 core tools
- **Skills**: 26 Agent Skills covering all tools
- **Optional**: 26 browser tools (with MCP_BROWSER_ENABLED=true)
- **NEW**: kubectl-mcp-app npm package with 8 interactive UIs

### Release v1.22.0 Changes

#### kubectl-mcp-app (New npm Package)
Added standalone npm package for interactive Kubernetes UI dashboards using MCP ext-apps SDK.

**Installation:**
```bash
npm install -g kubectl-mcp-app
# or
npx kubectl-mcp-app
```

**8 Interactive UI Tools:**
| Tool | Description |
|------|-------------|
| `k8s-pods` | Interactive pod viewer with filtering, sorting, status indicators |
| `k8s-logs` | Real-time log viewer with syntax highlighting and search |
| `k8s-deploy` | Deployment dashboard with rollout status, scaling, rollback |
| `k8s-helm` | Helm release manager with upgrade/rollback actions |
| `k8s-cluster` | Cluster overview with node health and resource metrics |
| `k8s-cost` | Cost analyzer with waste detection and recommendations |
| `k8s-events` | Events timeline with type filtering and grouping |
| `k8s-network` | Network topology graph showing Services/Pods/Ingress |

**Features:**
- TypeScript + React 19 + Vite
- Single-file HTML bundles (~220KB each)
- Dark/light theme support
- 27 tests with full coverage

**Claude Desktop Configuration:**
```json
{
  "mcpServers": {
    "kubectl-app": {
      "command": "npx",
      "args": ["kubectl-mcp-app"]
    }
  }
}
```

### Release v1.21.0 Changes

#### Comprehensive kind (Kubernetes IN Docker) Support
Expanded kind toolset from 15 to 32 tools covering all kind CLI capabilities:

**New kind Tools (17 additional tools):**

**Configuration Management:**
| Tool | Description |
|------|-------------|
| `kind_config_validate_tool` | Validate kind config YAML before cluster creation |
| `kind_config_generate_tool` | Generate sample config for multi-node/HA clusters |
| `kind_config_show_tool` | Show effective config for a running cluster |
| `kind_available_images_tool` | List available kindest/node images (K8s versions) |

**Registry Integration:**
| Tool | Description |
|------|-------------|
| `kind_registry_create_tool` | Create local Docker registry for kind |
| `kind_registry_connect_tool` | Connect cluster to local registry |
| `kind_registry_status_tool` | Check local registry status and config |

**Node Management:**
| Tool | Description |
|------|-------------|
| `kind_node_exec_tool` | Execute command on kind node (docker exec) |
| `kind_node_logs_tool` | Get logs from specific node |
| `kind_node_inspect_tool` | Inspect node container details |
| `kind_node_restart_tool` | Restart a kind node container |

**Networking & Ports:**
| Tool | Description |
|------|-------------|
| `kind_network_inspect_tool` | Inspect kind Docker network |
| `kind_port_mappings_tool` | List all port mappings for cluster |
| `kind_ingress_setup_tool` | Setup NGINX/Contour ingress controller |

**Advanced Diagnostics:**
| Tool | Description |
|------|-------------|
| `kind_cluster_status_tool` | Detailed cluster health/status |
| `kind_images_list_tool` | List images loaded on cluster nodes |
| `kind_provider_info_tool` | Get container runtime provider info |

**Usage Examples:**
```python
# Config generation & validation
kind_config_generate_tool(workers=2, ingress=True, registry=True)
kind_config_validate_tool(config_path="/tmp/kind.yaml")

# Registry setup
kind_registry_create_tool()
kind_create_cluster_tool(name="dev", config="/tmp/kind-with-registry.yaml")
kind_registry_connect_tool(cluster_name="dev")
kind_registry_status_tool()

# Node operations
kind_node_exec_tool(node="dev-control-plane", command="crictl images")
kind_node_inspect_tool(node="dev-control-plane")
kind_cluster_status_tool(name="dev")

# Networking
kind_network_inspect_tool()
kind_port_mappings_tool(cluster="dev")
kind_ingress_setup_tool(cluster="dev")
```

### Release v1.20.0 Changes

#### kind (Kubernetes IN Docker) Support
Added initial 15 tools for managing local Kubernetes clusters using kind CLI:

**kind Tools (15 tools):**
| Tool | Description |
|------|-------------|
| `kind_detect_tool` | Detect if kind CLI is installed |
| `kind_version_tool` | Get kind CLI version |
| `kind_list_clusters_tool` | List all kind clusters |
| `kind_get_nodes_tool` | List nodes in a cluster |
| `kind_get_kubeconfig_tool` | Get kubeconfig for a cluster |
| `kind_export_logs_tool` | Export cluster logs for debugging |
| `kind_cluster_info_tool` | Get cluster information |
| `kind_node_labels_tool` | Get node labels |
| `kind_create_cluster_tool` | Create a new kind cluster |
| `kind_delete_cluster_tool` | Delete a cluster |
| `kind_delete_all_clusters_tool` | Delete all kind clusters |
| `kind_load_image_tool` | Load Docker images into cluster |
| `kind_load_image_archive_tool` | Load images from tar archive |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rohitg00/kubectl-mcp-server](https://github.com/rohitg00/kubectl-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
