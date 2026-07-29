---
trigger: always_on
description: Krkn (Kraken) is a chaos engineering tool for Kubernetes/OpenShift clusters. It injects deliberate failures to validate cluster resilience. Plugin-based architecture with multi-cloud support (AWS, Azure, GCP, IBM Cloud, VMware, Alibaba, OpenStack).
---

# CLAUDE.md - Krkn Chaos Engineering Framework

## Project Overview

Krkn (Kraken) is a chaos engineering tool for Kubernetes/OpenShift clusters. It injects deliberate failures to validate cluster resilience. Plugin-based architecture with multi-cloud support (AWS, Azure, GCP, IBM Cloud, VMware, Alibaba, OpenStack).

## Repository Structure

```
krkn/
├── krkn/
│   ├── scenario_plugins/        # Chaos scenario plugins (pod, node, network, hogs, etc.)
│   ├── utils/                   # Utility functions
│   ├── rollback/                # Rollback management
│   ├── prometheus/              # Prometheus integration
│   └── cerberus/                # Health monitoring
├── tests/                       # Unit tests (unittest framework)
├── scenarios/                   # Example scenario configs (openshift/, kube/, kind/)
├── config/                      # Configuration files
└── CI/                          # CI/CD test scripts
```

## Quick Start

```bash
# Setup (ALWAYS use virtual environment)
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run Krkn
python run_kraken.py --config config/config.yaml

# Note: Scenarios are specified in config.yaml under kraken.chaos_scenarios
# There is no --scenario flag; edit config/config.yaml to select scenarios

# Run tests
python -m unittest discover -s tests -v
python -m coverage run -a -m unittest discover -s tests -v
```

## Critical Requirements

### Python Environment
- **Python 3.11+** required
- **NEVER install packages globally** - always use virtual environment
- **CRITICAL**: `docker` must be >=7.0.0 and `requests` must be >=2.32.4 — match the pins in `requirements.txt` (docker 7+ handles Unix sockets natively; requests bumped for security fixes)

### Key Dependencies
- **krkn-lib** (6.1.2): Core library for Kubernetes/OpenShift operations
- **kubernetes** (>=35.0.0,<36.0.0): Kubernetes Python client
- **docker** (>=7.0.0), **requests** (>=2.32.4): match `requirements.txt`; do not change without verifying Unix-socket compatibility
- Cloud SDKs: boto3 (AWS), azure-mgmt-* (Azure), google-cloud-compute (GCP), ibm_vpc (IBM), pyVmomi (VMware)

## Plugin Architecture (CRITICAL)

**Strictly enforced naming conventions:**

### Naming Rules
- **Module files**: Must end with `_scenario_plugin.py` and use snake_case
  - Example: `pod_disruption_scenario_plugin.py`
- **Class names**: Must be CamelCase and end with `ScenarioPlugin`
  - Example: `PodDisruptionScenarioPlugin`
  - Must match module filename (snake_case ↔ CamelCase)
- **Directory structure**: Plugin dirs CANNOT contain "scenario" or "plugin"
  - Location: `krkn/scenario_plugins/<plugin_name>/`

### Plugin Implementation
Every plugin MUST:
1. Extend `AbstractScenarioPlugin`
2. Implement `run()` method
3. Implement `get_scenario_types()` method

```python
from krkn.scenario_plugins import AbstractScenarioPlugin

class PodDisruptionScenarioPlugin(AbstractScenarioPlugin):
    def run(self, config, scenarios_list, kubeconfig_path, wait_duration):
        pass
    
    def get_scenario_types(self):
        return ["pod_scenarios", "pod_outage"]
```

### Creating a New Plugin
1. Create directory: `krkn/scenario_plugins/<plugin_name>/`
2. Create module: `<plugin_name>_scenario_plugin.py`
3. Create class: `<PluginName>ScenarioPlugin` extending `AbstractScenarioPlugin`
4. Implement `run()` and `get_scenario_types()`
5. Create unit test: `tests/test_<plugin_name>_scenario_plugin.py`
6. Add example scenario: `scenarios/<platform>/<scenario>.yaml`

**DO NOT**: Violate naming conventions (factory will reject), include "scenario"/"plugin" in directory names, create plugins without tests.

## Testing

### Unit Tests
```bash
# Run all tests
python -m unittest discover -s tests -v

# Specific test
python -m unittest tests.test_pod_disruption_scenario_plugin

# With coverage
python -m coverage run -a -m unittest discover -s tests -v
python -m coverage html
```

**Test requirements:**
- Naming: `test_<module>_scenario_plugin.py`
- Mock external dependencies (Kubernetes API, cloud providers)
- Test success, failure, and edge cases
- Keep tests isolated and independent

### Functional Tests
Located in `CI/tests/`. Can be run locally on a kind cluster with Prometheus and Elasticsearch set up.

**Setup for local testing:**
1. Deploy Prometheus and Elasticsearch on your kind cluster:
   - Prometheus setup: https://krkn-chaos.dev/docs/developers-guide/testing-changes/#prometheus
   - Elasticsearch setup: https://krkn-chaos.dev/docs/developers-guide/testing-changes/#elasticsearch

2. Or disable monitoring features in `config/config.yaml`:
   ```yaml
   performance_monitoring:
       enable_alerts: False
       enable_metrics: False
       check_critical_alerts: False
   ```

**Note:** Functional tests run automatically in CI with full monitoring enabled.

## Cloud Provider Implementations

Node chaos scenarios are cloud-specific. Each in `krkn/scenario_plugins/node_actions/<provider>_node_scenarios.py`:
- AWS, Azure, GCP, IBM Cloud, VMware, Alibaba, OpenStack, Bare Metal


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krkn-chaos/krkn](https://github.com/krkn-chaos/krkn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
