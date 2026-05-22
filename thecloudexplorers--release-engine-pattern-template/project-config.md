---
trigger: always_on
description: This repository serves as the **Abstraction Layer** in the Release Engine three-tier solution. It contains reusable workload patterns that define "how" infrastructure and applications are deployed through Infrastructure as Code (Bicep) templates and pipeline configurations.
---

# AI Agent Instructions for Release Engine Workload Patterns

## 🏗️ Repository Overview

This repository serves as the **Abstraction Layer** in the Release Engine three-tier solution. It contains reusable workload patterns that define "how" infrastructure and applications are deployed through Infrastructure as Code (Bicep) templates and pipeline configurations.

## Repository Role in Release Engine Solution

### Three-Tier Architecture
```text
Configuration Layer (Simple) → Abstraction Layer (THIS REPO) → Core Layer (Pipelines)
```

### This Repository's Purpose
- **Template Repository**: Organizations clone this repo to create their own workload pattern repositories
- **Pattern Library**: Contains reusable deployment patterns for common workloads
- **Infrastructure as Code**: Bicep templates with Azure Verified Modules integration
- **Pipeline Orchestration**: Workload-specific pipeline configurations

## 📁 Repository Structure

### Pattern Organization
```text
patterns/
├── multi_stage_pattern/          # Complex multi-stage deployments
│   ├── workload.yml             # Pipeline configuration
│   ├── multi_stage_pattern.prerequisite.bicep
│   └── multi_stage_pattern.dependent.bicep
├── resource_group_scope_pattern/ # Simple single-resource deployments within resource group
│   ├── workload.yml             # Pipeline configuration
│   └── resource_group_scope_pattern.bicep
└── subscription_scope_pattern/   # Subscription-level deployments
    ├── workload.yml             # Pipeline configuration
    └── subscription_scope_pattern.bicep
```

### Required Files Per Pattern
Each pattern directory must contain:
- **`workload.yml`**: Pipeline configuration and deployment orchestration
- **`workload.bicep`** (or multiple .bicep files): Infrastructure as Code definitions
- **`README.md`** (recommended): Pattern documentation and usage instructions

## 🔧 Development Guidelines

### Creating New Patterns

1. **Pattern Naming Convention**
   - Use descriptive names: `webapp_with_database`, `function_app_premium`, `aks_cluster_basic`
   - Use underscores for separation
   - Keep names concise but clear

2. **Bicep Template Standards**
   ```bicep
   metadata resources = {
     version: '0.1.0'
     author: '<Author Name>'
     company: '<Organization Name>'
     description: '<Pattern Description>'
   }

   targetScope = 'subscription' // or 'resourceGroup'

   @allowed([
     'westeurope'
     'uksouth'
     'eastus'
   ])
   @description('Region in which resources should be deployed')
   param resourceLocation string

   param tags object = {}
   ```

3. **Azure Verified Modules (AVM) Priority**
   - **Always check AVM availability first**: Use `br/public:avm/res/` registry
   - **Document AVM unavailability**: If AVM doesn't exist, document why direct resources are used
   - **Version pinning**: Always pin to specific AVM versions for consistency
   - **Parameter mapping**: Map pattern parameters to AVM module parameters

4. **Parameter Management**
   - Use consistent parameter naming across patterns
   - Provide default values where appropriate
   - Document all parameters with descriptions
   - Use validation rules (@allowed, @minLength, etc.)

### Pipeline Configuration (`workload.yml`)

#### Template Structure
```yaml
parameters:
  - name: deploymentSettings
    type: object

variables:
  - name: serviceConnection
    value: <default-service-connection> # Override per environment

stages:
  - template: /pipelines/01-orchestrators/pattern.orchestrator.yml@release-engine-core
    parameters:
      patternSettings:
        name: <pattern_name>
        configurationFilePath: ${{ parameters.deploymentSettings.configurationFilePath }}
        environments: ${{ parameters.deploymentSettings.environments }}
        patternArtifactsPath: /patterns/<pattern_name>
        stages:
          - infrastructure:
              iac:
                name: <deployment_stage_name>
                displayName: <human_readable_name>
                deploymentScope: <Subscription|ResourceGroup|Tenant>
                serviceConnection: $(serviceConnection)
                iacMainFileName: <bicep_file_name>.bicep
                iacParameterFileName: ${{ parameters.deploymentSettings.iacParameterFileName }}
                dependsOn: <optional_dependency_stage>
                lastInStage: <true|false>
```

#### Multi-Stage Dependencies
For complex patterns with multiple deployment stages:
```yaml
stages:
  # Prerequisites stage (no dependencies)
  - infrastructure:
      iac:
        name: prerequisite_stage
        displayName: Prerequisites
        deploymentScope: Subscription
        iacMainFileName: prerequisite.bicep
        
  # Dependent stage 1 (depends on prerequisites)
  - infrastructure:
      iac:
        name: dependent_stage1
        displayName: Main Infrastructure
        deploymentScope: ResourceGroup
        iacMainFileName: main.bicep
        dependsOn: prerequisite_stage
        
  # Final stage (depends on stage 1, marks end of environment)
  - infrastructure:
      iac:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecloudexplorers/release-engine-pattern-template](https://github.com/thecloudexplorers/release-engine-pattern-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
