---
trigger: always_on
description: This document provides comprehensive guidance for developing and maintaining azure-nuke. It documents the current design patterns, SDK choices, and architectural decisions.
---

# CLAUDE.md - Azure-Nuke Development Guide

This document provides comprehensive guidance for developing and maintaining azure-nuke. It documents the current design patterns, SDK choices, and architectural decisions.

## 1. Project Overview

**What azure-nuke does:** Azure-nuke is a CLI tool for automatically deleting Azure resources. It scans Azure tenants, subscriptions, and resource groups, then removes resources based on configurable filters. Built on the libnuke framework, it provides consistent resource cleanup across Azure services.

### Core Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| libnuke | v0.24.5 | Core nuke framework (registry, queue, filters) |
| urfave/cli/v2 | v2.27.7 | CLI framework |
| logrus | v1.9.3 | Structured logging |

### Directory Structure

```
azure-nuke/
├── main.go                 # Entry point
├── pkg/
│   ├── azure/              # Azure-specific utilities
│   │   ├── auth.go         # Authentication configuration
│   │   ├── types.go        # Authorizers struct definition
│   │   └── resource.go     # Scope definitions
│   ├── commands/           # CLI command implementations
│   ├── config/             # Configuration handling
│   └── common/             # Common utilities, version info
├── resources/              # Resource implementations (40+ resources)
│   ├── base-resource.go    # Base struct all resources embed
│   ├── disk.go             # Example: Track 1 SDK
│   ├── resource-group.go   # Example: HashiCorp SDK
│   ├── application.go      # Example: msgraph/hamilton
│   └── ...
├── docs/                   # Documentation
└── tools/                  # Utility tools
```

---

## 2. SDK Strategy & Decision Tree

### Why Multiple SDKs Exist

Azure-nuke intentionally uses multiple Azure SDKs. This is not technical debt—it's a deliberate design choice because:

1. **Different Azure services have different SDK maturity levels** - Not all services are available in all SDK generations
2. **Microsoft Graph resources require hamilton** - Azure AD/Entra ID resources use a separate API
3. **Track 1 is deprecated but still necessary** - Some services only exist in Track 1
4. **Consistency within service families** - Resources of the same Azure service should use the same SDK

### SDK Decision Tree

```
Adding or modifying a resource?
│
├── Is it an Azure AD/Entra ID resource (users, groups, apps, service principals)?
│   └── Use: manicminer/hamilton (msgraph)
│
├── Is there an existing resource of the same Azure service type?
│   └── Use: Same SDK as existing resource for consistency
│       Examples:
│       - New compute resource? Use Track 1 (like Disk, VM, Snapshot)
│       - New network resource? Check existing - some use Track 1, some HashiCorp
│
├── Does hashicorp/go-azure-sdk support this resource?
│   └── Use: HashiCorp SDK (preferred for new resources)
│       Import: github.com/hashicorp/go-azure-sdk/resource-manager/...
│
├── Does Track 2 arm* SDK support this resource?
│   └── Use: Track 2 SDK
│       Import: github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/...
│
└── Otherwise
    └── Use: Track 1 SDK (with //nolint:staticcheck)
        Import: github.com/Azure/azure-sdk-for-go/services/...
```

---

## 3. Per-Resource SDK Mapping Table

### Microsoft Graph Resources (manicminer/hamilton)

| Resource | Scope | Import |
|----------|-------|--------|
| AADGroup | Tenant | `hamilton/msgraph` |
| AADUser | Tenant | `hamilton/msgraph` |
| Application | Tenant | `hamilton/msgraph` |
| ApplicationCertificate | Tenant | `hamilton/msgraph` |
| ApplicationFederatedCredential | Tenant | `hamilton/msgraph` |
| ApplicationSecret | Tenant | `hamilton/msgraph` |
| ServicePrincipal | Tenant | `hamilton/msgraph` |

### HashiCorp SDK Resources (go-azure-sdk)

| Resource | Scope | Import Path |
|----------|-------|-------------|
| ApplicationGateway | ResourceGroup | `resource-manager/network/2023-09-01/applicationgateways` |
| Budget | Subscription | `resource-manager/consumption/2021-10-01/budgets` |
| ManagementLock | Subscription | `resource-manager/resources/2020-05-01/managementlocks` |
| MonitorDiagnosticSetting | Subscription | `resource-manager/insights/2021-05-01-preview/diagnosticsettings` |
| NetworkInterface | ResourceGroup | `resource-manager/network/2023-09-01/networkinterfaces` |
| RecoveryServicesBackupPolicy | ResourceGroup | `resource-manager/recoveryservicesbackup/2023-02-01/...` |
| RecoveryServicesVault | ResourceGroup | `resource-manager/recoveryservices/2023-02-01/vaults` |
| ResourceGroup | Subscription | `resource-manager/resources/2022-09-01/resourcegroups` |

### Track 2 SDK Resources (arm* packages)

| Resource | Scope | Import Path |
|----------|-------|-------------|
| RecoveryServicesBackupProtectedItem | ResourceGroup | `sdk/resourcemanager/recoveryservices/armrecoveryservicesbackup` |
| RecoveryServicesBackupProtectionContainer | ResourceGroup | `sdk/resourcemanager/recoveryservices/armrecoveryservicesbackup` |
| RecoveryServicesBackupProtectionIntent | ResourceGroup | `sdk/resourcemanager/recoveryservices/armrecoveryservicesbackup` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekristen/azure-nuke](https://github.com/ekristen/azure-nuke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
