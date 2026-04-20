---
trigger: always_on
description: **⚠️ CRITICAL: Always use the `azure-naming-research` skill to lookup CAF abbreviations and Azure naming constraints before suggesting or validating resource names.**
---

# Azure Deployment Standards

## Naming Conventions

**⚠️ CRITICAL: Always use the `azure-naming-research` skill to lookup CAF abbreviations and Azure naming constraints before suggesting or validating resource names.**

Use consistent CAF-compliant naming patterns across all Azure resources:

**Standard Format:**
```
{resource-type-abbreviation}-{project}-{environment}-{region}[-{instance}]
```

**Resource Groups:**
- CAF abbreviation: `rg`
- Format: `rg-{project}-{environment}-{region}`
- Example: `rg-webapp-prod-eastus`
- Constraints: 1-90 chars, alphanumeric + hyphens + underscores + periods

**Function Apps:**
- CAF abbreviation: `func`
- Format: `func-{project}-{environment}-{region}`
- Example: `func-api-dev-westus2`
- Constraints: 2-60 chars, alphanumeric + hyphens, globally unique

**Storage Accounts:**
- CAF abbreviation: `st`
- Format: `st{project}{env}{random}` (lowercase, no hyphens, max 24 chars)
- Example: `stwebappdev8k3m`
- Constraints: 3-24 chars, lowercase alphanumeric only, globally unique
- Special: Use `uniqueString(resourceGroup().id)` for uniqueness

**App Service Plans:**
- CAF abbreviation: `asp`
- Format: `asp-{project}-{environment}-{region}`
- Example: `asp-webapp-prod-eastus`

**Web Apps:**
- CAF abbreviation: `app`
- Format: `app-{project}-{environment}-{region}`
- Example: `app-webapp-prod-eastus`
- Constraints: 2-60 chars, alphanumeric + hyphens, globally unique

**SQL Servers:**
- CAF abbreviation: `sql`
- Format: `sql-{project}-{environment}-{region}`
- Example: `sql-webapp-prod-eastus`
- Constraints: 1-63 chars, lowercase alphanumeric + hyphens, globally unique

**SQL Databases:**
- CAF abbreviation: `sqldb`
- Format: `sqldb-{project}-{environment}`
- Example: `sqldb-webapp-prod`

**Cosmos DB:**
- CAF abbreviation: `cosmos`
- Format: `cosmos-{project}-{environment}-{region}`
- Example: `cosmos-webapp-prod-eastus`
- Constraints: 3-44 chars, lowercase alphanumeric + hyphens, globally unique

**Application Insights:**
- CAF abbreviation: `appi`
- Format: `appi-{project}-{environment}-{region}`
- Example: `appi-webapp-prod-eastus`

**Key Vaults:**
- CAF abbreviation: `kv`
- Format: `kv-{project}-{env}-{region}`
- Example: `kv-webapp-prod-eus` (shortened region for length)
- Constraints: 3-24 chars, alphanumeric + hyphens, globally unique

**Log Analytics Workspaces:**
- CAF abbreviation: `log`
- Format: `log-{project}-{environment}-{region}`
- Example: `log-webapp-prod-eastus`
- Constraints: 4-63 chars, alphanumeric + hyphens, must start/end with alphanumeric

**Container Apps Environments:**
- CAF abbreviation: `cae`
- Format: `cae-{project}-{environment}-{region}`
- Example: `cae-api-prod-eastus`
- Constraints: 1-60 chars, alphanumeric + hyphens
- Note: Requires a Log Analytics workspace for app logs

**Container Apps:**
- CAF abbreviation: `ca`
- Format: `ca-{project}-{environment}-{region}`
- Example: `ca-api-prod-eastus`
- Constraints: 2-32 chars, lowercase alphanumeric + hyphens, must start with letter

**Naming Workflow:**

1. **Before naming ANY resource:**
   ```bash
   # Use the azure-naming-research skill
   /azure-naming-research {resource-type}
   
   # Example:
   /azure-naming-research "Azure Functions"
   
   # Returns: CAF abbreviation, length constraints, valid characters, scope
   ```

2. **Construct name using pattern:**
   ```
   {caf-abbrev}-{project}-{env}-{region}
   ```

3. **Validate against constraints:**
   - Length (min-max)
   - Character set (alphanumeric, hyphens, etc.)
   - Uniqueness scope (global, resource group, subscription)
   - Start/end character requirements

4. **For globally unique resources (Storage, Functions, SQL, Cosmos, Key Vault):**
   - Add random suffix if needed: `{name}-${random}`
   - Or use ARM's `uniqueString()` function in templates

**Reference:**
- CAF Abbreviations: https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations
- Naming Rules: https://learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules

## Default Regions

Prefer these regions for deployments unless specified otherwise:
- **Primary:** East US
- **Secondary:** West US 2
- **Europe:** West Europe

## Environment Tags

Always include these tags on all resources:
```json
{
  "Environment": "dev|staging|prod",
  "Project": "project-name",
  "ManagedBy": "git-ape-agent",
  "CreatedDate": "YYYY-MM-DD"
}
```

## ARM Template Standards

- Include `parameters` section for configurable values
- Use `outputs` section to return resource IDs and endpoints
- Apply Azure best practices via `bestpractices` service validation
- Include resource health monitoring configurations
- **Always use `/azure-rest-api-reference` to look up resource properties and API versions before writing or modifying ARM template resources** — never rely on memorized property names or schemas
- **Subscription-level templates** (`subscriptionDeploymentTemplate.json`) must use nested deployments with `"expressionEvaluationOptions": { "scope": "inner" }` — without this, `reference()` and `resourceId()` resolve at subscription scope, causing `ResourceNotFound` errors at deploy time

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/git-ape](https://github.com/Azure/git-ape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
