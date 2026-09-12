---
trigger: always_on
description: **REQUIRED**: First read the complete generic instructions at `.ai/commands/workload/publishworkload.md`
---

# GitHub Copilot Instructions: Publish Workload

## 🔗 Base Instructions

**REQUIRED**: First read the complete generic instructions at `.ai/commands/workload/publishworkload.md`

This file provides GitHub Copilot-specific enhancements for publishing workloads beyond the base generic process.

## 🤖 GitHub Copilot Enhanced Features

### Smart Publishing Pipeline
GitHub Copilot automatically detects:
- Production readiness requirements
- Compliance and security validation
- Partner program prerequisites
- Certification checklist completion

### Intelligent Quality Assurance

#### Auto-Validation Framework
```powershell
# Copilot generates comprehensive validation:
fabric publish validate    # → Full production readiness check
fabric publish security    # → Security compliance scan
fabric publish compliance  # → Fabric Hub requirements validation
```

### Real-time Compliance Monitoring
- **Security Scanning**: Validates code security patterns
- **Performance Metrics**: Ensures performance benchmarks
- **Accessibility Check**: Validates UI accessibility standards
- **Documentation Review**: Ensures complete documentation

### Advanced Publishing Management

#### Multi-Environment Coordination
GitHub Copilot coordinates:
- Development → Staging → Production pipeline
- Version management and release notes
- Rollback procedures and emergency protocols
- User communication and change management

#### Smart Certification Process
```typescript
// Copilot automates certification steps:
fabric.cert.security      // → Security certification automation
fabric.cert.performance   // → Performance testing validation
fabric.cert.accessibility // → Accessibility compliance check
fabric.cert.documentation // → Documentation completeness review
```

### Context-Aware Publishing Workflow

#### Partner Program Integration
- Validates Partner Program enrollment status
- Ensures all legal requirements are met
- Checks certification prerequisites
- Manages submission workflow

#### Hub Publishing Intelligence
- Optimizes manifest for Fabric Hub requirements
- Validates asset quality and compliance
- Ensures proper categorization and tagging
- Manages version compatibility

## 🚀 Copilot Quick Actions

### One-Command Publishing
```powershell
# Type comment to trigger intelligent publishing:
# fabric publish to production hub with full validation
```

### Smart Publishing Patterns
- `fabric.publish.staging` → Staged publishing with validation
- `fabric.publish.production` → Full production publishing workflow
- `fabric.publish.rollback` → Safe rollback procedures

### Auto-Completion Intelligence
GitHub Copilot recognizes and expands:
- Publishing workflow patterns
- Compliance validation procedures
- Documentation requirements
- Certification automation

---

**Reference**: For complete step-by-step instructions, always consult `.ai/commands/workload/publishworkload.md` first, then apply these Copilot-specific enhancements.

### Technical Requirements
- **NuGet Package**: Built manifest package (.nupkg file)
- **Workload ID**: Registered unique identifier in format `[Publisher].[Workload]`
- **Production Configuration**: Workload configured for production environment
- **Testing Complete**: Workload validated in development environment

### Registration Requirements
Before publishing outside your organization, you must:
1. **Complete Workload Registration Form**: [Submit form](https://aka.ms/fabric_workload_registration)
2. **Define Publisher Name**: Meaningful company name or abbreviation
3. **Define Workload Name**: Align with product name or offering
4. **Await Microsoft Approval**: Fabric team enables publishing capability

## Stage 1: Testing

### Internal Testing Process

Testing is the initial stage where you validate your workload internally before exposing it to external users.

#### 1.1: Prepare Production Manifest

Ensure your workload manifest uses the registered Workload ID:

```xml
<!-- WorkloadManifest.xml -->
<Workload WorkloadName="YourPublisher.YourWorkload" HostingType="FERemote">
    <Version>1.0.0</Version>
    <!-- Additional configuration -->
</Workload>
```

#### 1.2: Build Production Package

Use the build script to create the manifest package:

```powershell
.\scripts\Build\BuildManifestPackage.ps1 -ValidateFiles $true
```

**Verify Package Contents:**
- Workload manifest with correct ID
- Item definitions properly configured  
- All required assets included
- Version information accurate

#### 1.3: Upload to Test Tenant

1. **Access Fabric Admin Portal**:
   - Sign in to [Fabric](https://powerbi.com/) with admin account
   - Navigate to Settings → Admin portal

2. **Upload Workload Package**:
   - Go to Workloads section
   - Select "Upload workload"
   - Browse to your .nupkg file
   - Select "Open"

3. **Activate Workload**:
   - Select the uploaded workload
   - Choose the version to activate
   - Select "Add"
   - Status should show "Active in tenant"

#### 1.4: Internal Validation

**Test Scenarios:**
- Workload appears in experience switcher
- Item creation functions correctly
- Editor loading and functionality
- Data persistence and retrieval
- Authentication flows
- All supported browsers and devices

## Stage 2: Preview Audience


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/fabric-extensibility-toolkit](https://github.com/microsoft/fabric-extensibility-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
