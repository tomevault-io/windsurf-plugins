---
trigger: always_on
description: The Windows Machine Config Operator (WMCO) configures Windows Server instances as worker nodes in OpenShift/OKD clusters, enabling Windows container workloads.
---

# AGENTS.md

## Project Overview

The Windows Machine Config Operator (WMCO) configures Windows Server instances as worker nodes in OpenShift/OKD clusters, enabling Windows container workloads.

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                    OpenShift Cluster                        │
├────────────────────────┬────────────────────────────────────┤
│   Linux Control Plane  │        Windows Worker Nodes        │
│  ┌──────────────────┐  │  ┌──────────────────────────────┐  │
│  │  WMCO Operator   │──┼──│  WICD (daemon)               │  │
│  │  - Controllers   │SSH│  │  - Service reconciliation    │  │
│  │  - CSR Approval  │  │  │  - Certificate rotation      │  │
│  └──────────────────┘  │  │  - Environment management    │  │
│                        │  └──────────────────────────────┘  │
│                        │  ┌──────────────────────────────┐  │
│                        │  │  Windows Components          │  │
│                        │  │  kubelet, containerd,        │  │
│                        │  │  kube-proxy, hybrid-overlay  │  │
│                        │  └──────────────────────────────┘  │
└────────────────────────┴────────────────────────────────────┘
```

**Provisioning Modes:**
- **Machine API**: Create Windows VMs via MachineSets, auto-configured by WMCO
- **BYOH**: Define existing instances in `windows-instances` ConfigMap

**Tech Stack:** Go 1.24+, Kubernetes Operator (controller-runtime), OpenShift APIs, Windows Server 2019/2022

---

## Product Overview

Red Hat OpenShift support for Windows Containers is a layered component of OpenShift that allows the integration of Windows Nodes for running Windows Containers on an OpenShift 4 Cluster.

This is achieved by installing the Windows Machine Config Operator (WMCO), which runs on Linux based control-plane nodes. The WMCO bootstraps Windows nodes to join the cluster as Windows worker nodes.

### Subscription Requirements

Windows Containers requires a specific subscription, in addition to an OpenShift Container Platform subscription:
- **Worker nodes only** - Control plane and infrastructure nodes don't need a paid subscription for Windows Containers
- When working cases, verify entitlement mapping to ensure compliance

### Entitlement Validation

Look for the **"for Windows"** phrase in the entitlement name.

**Example Entitlements:**
- `MW01465`: Red Hat OpenShift Container Platform, Standard (2 Cores or 4 vCPUs, for Windows)
- `MW01615`: Red Hat OpenShift Container Platform, Premium (2 Cores or 4 vCPUs, for Windows)

---

## Requesting Engineering Assistance

### Jira Projects

| Project | Purpose | Link |
|---------|---------|------|
| OpenShift Bugs (OCPBUGS) | Bug reports | [Red Hat Issue Router](https://access.redhat.com/labs/rhir/) → "Windows Containers" component |
| Windows Containers (WINC) | Engineering tracking | [issues.redhat.com/projects/WINC](https://issues.redhat.com/projects/WINC/issues/) |
| RFE | Feature requests | [issues.redhat.com/project/RFE](https://issues.redhat.com/project/RFE) → "Windows Containers" component |
| Portfolio Backlog | Roadmap/planning | [Portfolio Plan View](https://issues.redhat.com/secure/PortfolioPlanView.jspa?id=1226&sid=1226&vid=4843#plan/backlog) |

### Creating a Bug Report

Search for the **"Windows Containers"** component on the [Red Hat Issue Router](https://access.redhat.com/labs/rhir/), then create a bug with:

1. **Detailed explanation** of the issue including:
   - Troubleshooting steps already taken
   - Any recent changes to the cluster
   - Relevant contextual information
2. **Link to customer case** (if one exists)
3. **Must-gather archive** recently generated on the cluster
4. **Command outputs** (not present in older must-gather archives):

```bash
oc get network.operator cluster -o yaml
oc logs -f deployment/windows-machine-config-operator -n openshift-windows-machine-config-operator
```

5. **MachineSet object** describing Windows instances (if using Machine API/IPI)
6. **windows-instances ConfigMap** (if using BYOH method)

### Bug Report Template

Use this template when filing bugs in **OpenShift Bugs (OCPBUGS)** with **Component: Windows Containers**:

```
Description of problem:
{code:none}
    
{code}

Version-Release number of selected component (if applicable):
{code:none}
    
{code}

How reproducible:
{code:none}
    
{code}

Steps to Reproduce:
{code:none}
    1.
    2.
    3.
{code}

Actual results:
{code:none}
    
{code}

Expected results:
{code:none}
    
{code}

Additional info:
{code:none}
    
{code}
```

### Reaching Out on Slack

Slack is a **supplement** to bug reports in Jira. Only reach out if:
- A bug is already filed
- The issue hasn't received a prompt response

**Channel:** `#forum-ocp-winc` (CoreOS Slack)
**Tag:** Use only `@winc-watcher` to get the team's attention
Do NOT use `@here` or `@everyone`

---

## Setup Commands

### Build
- `make build` - Build operator binary
- `GOOS=windows make build-daemon` - Build Windows daemon (WICD)
- `make build-all` - Build everything

### Test
- `make unit` - Run all unit tests
- `go test -v ./pkg/nodeconfig/...` - Test specific package

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/windows-machine-config-operator](https://github.com/openshift/windows-machine-config-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
