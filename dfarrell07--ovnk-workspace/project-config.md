---
trigger: always_on
description: Multi-repo workspace for OVN-Kubernetes (ovnk) CI, AI, and docs automation.
---

# OVN-Kubernetes Workspace

Multi-repo workspace for OVN-Kubernetes (ovnk) CI, AI, and docs automation.
This directory is a git repo tracking workspace-level files only -- the 36
cloned sub-repos live under org/ subdirectories and are managed independently.
For repo-specific agent guidance, check for AGENTS.md inside each repo.

Repos with AGENTS.md: openshift-eng/ai-helpers, containers/kubernetes-mcp-server,
openshift-eng/openshift-ci-mcp, openshift-eng/rebasebot, openshift/sippy.

Repos with .claude/ commands or skills: openshift/ci-tools, openshift/enhancements,
openshift-eng/openshift-ci-mcp, openshift/sippy.

## Upstream vs Downstream

ovn-org/ovn-kubernetes/ is upstream (ovn-org/ovn-kubernetes) -- community project,
GitHub Actions CI.

openshift/ovn-kubernetes/ is downstream (openshift/ovn-kubernetes) -- Prow CI via
openshift/release.

Downstream adds: Dockerfile, Dockerfile.base, Dockerfile.microshift, Dockerfile.utest,
and an openshift/ directory with OTE (ovn-kubernetes-tests-ext) code.

rebasebot syncs upstream -> downstream via automated rebase PRs.

Note: upstream ovnk has no CLAUDE.md yet. PR #5597 (by abhat) is stalled -- maintainer
trozet requested a vendor-neutral name. Convention in openshift-eng: AGENTS.md symlinked
to CLAUDE.md.

## Repo Map (by GitHub org)

### containers (1 repo)

<!-- markdownlint-disable MD013 -->

| Path | Description | Category |
|------|-------------|----------|
| containers/kubernetes-mcp-server | General K8s/OpenShift MCP server (1.5k stars) | AI |

### kubernetes (1 repo)

| Path | Description | Category |
|------|-------------|----------|
| kubernetes/enhancements | Upstream Kubernetes KEPs | Docs |

### kubernetes-sigs (2 repos)

| Path | Description | Category |
|------|-------------|----------|
| kubernetes-sigs/kube-agentic-networking | Network APIs for AI agent communication | AI |
| kubernetes-sigs/network-policy-api | NetworkPolicy v2 API (AdminNetworkPolicy) | Networking |

### metallb (1 repo)

| Path | Description | Category |
|------|-------------|----------|
| metallb/frr-k8s | FRR routing for BGP/EVPN integration | Networking |

### openshift (14 repos)

| Path | Description | Category |
|------|-------------|----------|
| openshift/api | OpenShift API types and feature gates | Core |
| openshift/ci-docs | OpenShift CI system documentation | CI |
| openshift/ci-tools | ci-operator engine and tooling | CI |
| openshift/cloud-network-config-controller | Cloud EgressIP management (AWS/Azure/GCP) | Core |
| openshift/cluster-network-operator | Deploys and configures ovnk on OpenShift | Core |
| openshift/enhancements | OpenShift enhancement proposals | Docs |
| openshift/multus-cni | Meta-CNI for multiple network interfaces | Core |
| openshift/network-tools | Debug tools + Claude Code JIRA labeling (PR #168) | Core |
| openshift/openshift-docs | OCP product documentation (AsciiDoc) | Docs |
| openshift/origin | openshift-tests binary and networking e2e suites | CI |
| openshift/ovn-kubernetes | Downstream fork with OpenShift integration | Core |
| openshift/release | Prow job definitions and step registry | CI |
| openshift/runbooks | Alert runbooks for OCP operators | Docs |
| openshift/sippy | CI analytics dashboard (job/test pass rates) | CI |

### openshift-eng (8 repos)

| Path | Description | Category |
|------|-------------|----------|
| openshift-eng/ai-helpers | Claude Code plugin marketplace (35+ plugins) | AI |
| openshift-eng/ci-test-mapping | Maps tests to components for readiness tracking | CI |
| openshift-eng/gangway-cli | CLI for triggering Prow jobs via API | CI |
| openshift-eng/ocp-build-data | ART image/RPM build configs per OCP version | Release |
| openshift-eng/ocp-performance-analyzer-mcp | MCP for OCP performance analysis | AI |
| openshift-eng/openshift-ci-mcp | MCP server for Sippy/Search.CI/Release Controller | CI |
| openshift-eng/openshift-tests-extension | Framework for decentralized OCP test contributions | CI |
| openshift-eng/rebasebot | Automated upstream -> downstream sync (Python) | CI |

### openvswitch (1 repo)

| Path | Description | Category |
|------|-------------|----------|
| openvswitch/ovs | Open vSwitch datapath (C) | Networking |

### ovn-kubernetes (3 repos)

| Path | Description | Category |
|------|-------------|----------|
| ovn-kubernetes/kubernetes-traffic-flow-tests | Network connectivity test suite (Python) | CI |
| ovn-kubernetes/libovsdb | Go OVSDB client library (236 files import it) | Networking |
| ovn-kubernetes/ovn-kubernetes-mcp | MCP server for ovnk troubleshooting (OKEP-5494) | AI |

### ovn-org (5 repos)

| Path | Description | Category |
|------|-------------|----------|
| ovn-org/ovn | OVN virtual network control plane (C) | Networking |
| ovn-org/ovn-fake-multinode | Simulated multi-chassis OVN for testing | CI |
| ovn-org/ovn-heater | OVN scale/performance test framework | CI |
| ovn-org/ovn-kubernetes | Upstream ovnk controller + CNI plugin (Go) | Core |
| ovn-org/ovn-website | OVN project website (Hugo) | Docs |

<!-- markdownlint-enable MD013 -->

## Cross-Repo Dependencies (A -> B means B builds on A)

openvswitch/ovs -> ovn-org/ovn -> ovn-kubernetes/libovsdb ->
ovn-org/ovn-kubernetes -> openshift/ovn-kubernetes ->
openshift/cluster-network-operator


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dfarrell07/ovnk-workspace](https://github.com/dfarrell07/ovnk-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
