---
trigger: always_on
description: Expert guidance for generating security visualizations with the USecVisLib library ecosystem
---


# USecVisLib Security Visualization Skill

Expert guidance for generating security visualizations with the USecVisLib library ecosystem.

---

## 1. Visualization Selection Guide

Match the security question to the right visualization type and tools.

| Security Question | Visualization Type | Primary Tool | Analysis Tools |
|---|---|---|---|
| What are the attack paths to a target? | Attack Tree | `generate_attack_tree` | `validate_attack_tree`, `get_attack_tree_stats`, `build_attack_tree_from_spec` |
| How do attackers traverse the network? | Attack Graph | `generate_attack_graph` | `validate_attack_graph`, `get_attack_graph_stats`, `find_attack_paths`, `analyze_critical_nodes`, `find_chokepoints`, `analyze_centrality` |
| What threats exist in data flows? | Threat Model (DFD) | `generate_threat_model` | `validate_threat_model`, `analyze_stride_threats`, `get_threat_model_stats` |
| What does the binary look like inside? | Binary Analysis | `analyze_binary_entropy` | `analyze_binary_distribution`, `analyze_binary_heatmap`, `analyze_binary_all`, `get_binary_stats` |
| What's the cloud architecture? | Cloud Diagram | `generate_cloud_diagram` | `validate_cloud_diagram`, `get_cloud_diagram_stats`, `search_cloud_icons`, `list_cloud_providers`, `list_cloud_icons` |
| Where are trust boundary violations? | Privilege Gradient | `generate_privilege_gradient` | `validate_privilege_gradient`, `get_privilege_gradient_stats`, `detect_privilege_inversions`, `analyze_zone_influence` |
| What's the system architecture? | Component Diagram | `generate_component_diagram` | `validate_component_diagram`, `get_component_diagram_stats` |
| How do modules depend on each other? | Dependency Graph | `generate_dependency_graph` | `validate_dependency_graph`, `get_dependency_graph_stats` |
| Custom topology or flowchart? | Custom Diagram | `generate_custom_diagram` | `validate_custom_diagram`, `get_custom_diagram_stats`, `list_shapes` |

**Additional tools**: `render_mermaid` (render Mermaid syntax to image), `convert_to_mermaid`, `calculate_cvss_score`, `detect_config_type`, `export_data`, `generate_report`, `batch_process`, `compare_visualizations`. See [REFERENCE.md](REFERENCE.md) for the complete 49-tool catalog with parameters and return types.

---

## 2. Configuration Best Practices

### Attack Trees

Structure: hierarchical goal decomposition with AND/OR gates.

- **Root**: Single top-level attacker goal
- **Sub-goals**: 3-5 levels of decomposition (depth 3-5 optimal)
- **Gates**: Use AND for multi-step attacks, OR for alternatives
- **Leaves**: Attach CVSS scores for quantitative risk
- **Config keys**: `tree` (name, root, params), `nodes` (styling), `edges` (connections with labels)
- **Optional fields**: `params.rankdir` (layout direction), node `fontcolor`, `style`, `penwidth`

```json
{
  "tree": {"name": "...", "root": "Root Goal", "params": {"rankdir": "TB"}},
  "nodes": {"Root Goal": {"shape": "oval", "fillcolor": "#e74c3c"}},
  "edges": {"Root Goal": [{"to": "Sub Goal", "label": "OR"}]}
}
```

See [examples/attack_tree_web_app.json](examples/attack_tree_web_app.json) for a full web application attack tree.

### Attack Graphs

Structure: network topology with hosts, vulnerabilities, exploits.

- **Hosts**: Include IP addresses and zone assignments (external/dmz/internal)
- **Vulnerabilities**: Link to specific hosts, include CVE IDs and CVSS scores
- **Exploits**: Define preconditions (vulnerabilities) and postconditions (privileges)
- **Services**: Map to hosts with port/protocol
- **Config keys**: `graph`, `hosts`, `vulnerabilities`, `exploits`
- **Optional fields**: `privileges` (privilege definitions), `services` (port/protocol mappings per host), `network_edges` (explicit network topology links between hosts)

*Note: IP addresses in examples are fictional (RFC 1918 private ranges) and for illustration only.*

```json
{
  "graph": {"name": "..."},
  "hosts": [
    {"id": "web", "label": "Web Server", "ip": "10.0.1.10", "zone": "dmz"},
    {"id": "app", "label": "App Server", "ip": "10.0.2.20", "zone": "internal"},
    {"id": "db", "label": "Database", "ip": "10.0.2.30", "zone": "internal"}
  ],
  "vulnerabilities": [
    {"id": "v1", "label": "RCE - CVE-2024-1234", "cvss": 9.8, "affected_host": "web"},
    {"id": "v2", "label": "SQLi - CVE-2024-5678", "cvss": 8.6, "affected_host": "app"}
  ],
  "services": [
    {"id": "s1", "host": "web", "port": 443, "protocol": "https"},
    {"id": "s2", "host": "app", "port": 8080, "protocol": "http"}
  ],
  "exploits": [
    {"id": "e1", "label": "Exploit RCE", "preconditions": ["v1"], "postconditions": ["priv_web_shell"]},
    {"id": "e2", "label": "Exploit SQLi", "preconditions": ["v2", "priv_web_shell"], "postconditions": ["priv_db_read"]}
  ],
  "network_edges": [
    {"from": "web", "to": "app", "label": "HTTP 8080"},
    {"from": "app", "to": "db", "label": "SQL 3306"}
  ]
}
```

See [examples/attack_graph_network.json](examples/attack_graph_network.json) for a full network topology example.

### Threat Models

Structure: Data Flow Diagram with STRIDE analysis.

- **Format**: Dict-of-dicts (not lists) for processes, datastores, externals, dataflows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vulnex/usecvislib-skill](https://github.com/vulnex/usecvislib-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
