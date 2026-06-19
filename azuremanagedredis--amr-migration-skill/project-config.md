---
trigger: always_on
description: |
---


# Azure Managed Redis Migration Skill

This skill assists users in migrating from Azure Cache for Redis (ACR) Basic/Standard/Premium tiers to Azure Managed Redis (AMR), including automated migration via ARM REST APIs.

## 📝 Terminology Note

Users may refer to Azure Cache for Redis by several names: **OSS**, **ACR**, or by tier name (**Basic**, **Standard**, **Premium**). These all refer to the same product. Treat these terms interchangeably when users ask about migration.

## ⚠️ Scope Limitation: Enterprise Tier NOT Supported

This skill does **not** cover Azure Cache for Redis Enterprise (ACRE) migrations. If users ask about migrating from Enterprise or Enterprise Flash tiers, explain that those have different migration paths and suggest contacting Microsoft support or consulting the official documentation.

**Supported source tiers**: Basic (C0-C6), Standard (C0-C6), Premium (P1-P5)

## ⚠️ AMR Terminology: No "Shards"

Avoid using the term "shards" when describing AMR. In AMR, sharding is managed internally and not exposed to customers, so the concept doesn't apply and would be confusing. The term only applies to ACR Premium clustered caches. When discussing AMR, refer to the performance tier (e.g., Balanced, Memory Optimized) and size (e.g., B10, M20) instead.

---

## Agent Guidance

### ⛔ Destructive Operation Confirmation (Mandatory)
Before executing **Migrate** or **Cancel** actions via the migration scripts, the agent **must**:
1. Display the **full source and target resource IDs** to the user
2. Clearly state the action being performed (e.g., "This will initiate a DNS switch migration from ACR to AMR")
3. **Ask for explicit user confirmation** using the `ask_user` tool before running the script
4. Never auto-confirm or assume consent for these operations

This applies regardless of how the user phrased their request. Even if the user says "go ahead and migrate", the agent must confirm the specific resource IDs before executing.

For the bash script, always pass `--yes` when the agent runs it (since the agent cannot interact with the terminal prompt), but only after obtaining user confirmation through the `ask_user` tool first.

### 🔍 Validate-Before-Migrate (Mandatory)
Before executing **Migrate**, the agent **must** first run **Validate** with the same source and target resource IDs and present the results to the user. Do not proceed to Migrate if validation returns errors. If validation returns warnings, explain them and ask the user whether to proceed with `-ForceMigrate $true` (PowerShell) or `--force-migrate` (bash).

### Version Check (manual only — triggered by user request)
Do **not** check for updates automatically. Only perform a version check when the user explicitly asks (e.g., "check for updates for the amr skill", "is there a newer version of amr-migration-skill?").

When requested:
1. Read the local `VERSION` file in this skill's root directory.
2. Fetch the remote version from: `https://raw.githubusercontent.com/AzureManagedRedis/amr-migration-skill/main/VERSION`
3. If the remote version is newer, tell the user: _"A newer version of the AMR Migration Skill is available (local: X, latest: Y). Update from: https://github.com/AzureManagedRedis/amr-migration-skill"_
4. If versions match, tell the user: _"You're on the latest version (X)."_
5. If the fetch fails, tell the user the check failed and suggest trying again later.

### Detecting Platform for Script Selection
Check the user's OS to choose the right migration script variant:
- **Windows / PowerShell**: Use `.ps1` scripts (requires Azure CLI)
- **Linux / macOS / WSL / Bash**: Use `.sh` scripts (requires Azure CLI + jq)

If the OS is unclear, prefer the bash scripts — they work cross-platform. Both script variants use Azure CLI (`az rest`) for ARM API calls.

### Constructing ARM Resource IDs
Users will typically provide a cache name, resource group, and subscription. Construct the full ARM resource IDs as follows:

- **ACR source**: `/subscriptions/<subId>/resourceGroups/<rg>/providers/Microsoft.Cache/Redis/<cacheName>`
- **AMR target**: `/subscriptions/<subId>/resourceGroups/<rg>/providers/Microsoft.Cache/redisEnterprise/<cacheName>`

If the user only provides a cache name, use `az redis show -n <name> -g <rg> --query id -o tsv` to retrieve the full resource ID. If the subscription or RG is also unknown, use `az redis list --query "[?name=='<name>'].{id:id, rg:resourceGroup}" -o table` to find it.

### Validating SKU Recommendations
Before recommending an AMR SKU to the user, cross-check it against the valid SKU list in [AMR SKU Specs](references/amr-sku-specs.md). Never recommend a SKU that doesn't appear in that file. If the ideal capacity falls between two SKU sizes, recommend the next size up.

### Connection Changes Reminder
Always mention these when discussing migration — they require application changes:
- **TLS port**: ACR uses **6380** → AMR uses **10000**
- **Non-TLS**: ACR 6379 → AMR uses **10000** (Plaintext mode, set at creation; only one mode active at a time)
- **DNS suffix**: `.redis.cache.windows.net` → `<region>.redis.azure.net`
- **Redis version**: 6 → 7.4

If the user is using the automated migration with DNS switching, the old hostname continues to work, but the port change still applies.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzureManagedRedis/amr-migration-skill](https://github.com/AzureManagedRedis/amr-migration-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
