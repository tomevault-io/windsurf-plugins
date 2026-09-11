---
trigger: always_on
description: Terraform provider for Jamf Platform APIs, built on the [Terraform Plugin Framework](https://github.com/hashicorp/terraform-plugin-framework) v1.19.0 with Protocol v6. Go module path: `github.com/jamf/terraform-provider-jamfplatform`.
---

# Repository Guidelines

## Overview

Terraform provider for Jamf Platform APIs, built on the [Terraform Plugin Framework](https://github.com/hashicorp/terraform-plugin-framework) v1.19.0 with Protocol v6. Go module path: `github.com/jamf/terraform-provider-jamfplatform`.

Five construct types: **resources** (CRUD), **data sources** (read-only lookups), **list resources** (RSQL-filtered streaming), **actions** (fire-and-forget device commands), and **functions** (offline provider-defined functions under the `jamfplatform::` namespace — no API client, no provider config).

The Jamf Platform API client is the external Go SDK `github.com/Jamf-Concepts/jamfplatform-go-sdk` (package `jamfplatform`). Not vendored.

## Companion Docs (authoritative on conflict)

- `STYLE_GUIDE.md` — Go style, file conventions, schema rules, Jamf Pro naming/version/endpoint policies, scope helper, profile payload normalisation, ID/import conventions.
- `TESTING.md` — test categories, commands, build tags, CI.
- `CONTRIBUTING.md` — contribution workflow, Pro-resource workflow (incl. ProClassic SDK payload audit), release-versioning policy.
- `README.md` — user-facing provider usage.
- `spike/JAMF_PRO_INVENTORY.md` — gitignored Pro SDK namespace adoption tracker.
- `spike/PRO_ROLLOUT_PLAN.md` — gitignored Pro rollout planning doc.

## Project Structure

```
internal/
├── provider/          # Provider config, registration, logging
├── providerdata/      # Shared Data{} carrying SDK client + cached Pro version
├── resources/
│   ├── account/       # Jamf Account — organization-level, flat single tier   (Jamf Account)
│   │                  #   sso_domain/
│   │                  #   (folder name = Terraform slug minus `jamfplatform_account_`)
│   ├── ai_governance/ # policy/, tool/                                      (Jamf AI Governance)
│   │                  #   (folder name = Terraform slug minus `jamfplatform_ai_governance_`)
│   ├── blueprints/    # blueprint/, blueprints/, component/, components/    (Platform Services)
│   ├── cbengine/      # benchmark/, benchmarks/, baselines/, rules/         (Platform Services)
│   ├── device/        # Single device data source                            (Platform Services)
│   ├── device_group/  # Resource, data source, list resource                 (Platform Services)
│   ├── device_groups/ # Plural data source                                   (Platform Services)
│   ├── devices/       # Plural data source                                   (Platform Services)
│   ├── security_cloud/ # Jamf Security Cloud — flat single tier                (Security Cloud)
│   │                  #   device_group/, dns_zone/, uem_connect/, ztna_gateway/,
│   │                  #   ztna_grouped_gateway/, ztna_shared_gateways/
│   │                  #   (folder name = Terraform slug minus `jamfplatform_security_cloud_`)
│   └── pro/           # Jamf Pro resources — flat single tier: every leaf package sits directly under pro/
│                      #   (folder name = Terraform slug minus `jamfplatform_pro_`, snake_case). No domain tier.
│                      #   ~115 packages, fully flat (e.g. the five PKI constructs are pki_adcs/, pki_venafi/, pki_digicert/, … — no pki/ grouping dir).
├── actions/
│   ├── account/       # sso_domain/ (verify)                                 (Jamf Account)
│   ├── device/        # erase, restart, shutdown, unmanage                       (Platform Device Actions API)
│   └── pro/           # managed_software_updates (plan + abandon), maintenance/ (flush_policy_logs, redeploy_management_framework), mdm/ (send_blank_push, renew_mdm_profile, flush_mdm_commands), patch/ (retry_patch_policy_logs)   (Jamf Pro)
├── functions/         # Provider-defined functions (offline; no SDK client, no provider config)
│   ├── mobileconfig/       # mobileconfig(profile) — build a full .mobileconfig from HCL payloads; also holds the shared Assemble core
│   └── mcx_forced_payload/ # mcx_forced_payload(domain, prefs) — MCX "Custom Settings" envelope; thin wrapper over mobileconfig.Assemble
├── common/
│   ├── aischemas/     # Vendor JSON Schema fetch/cache/validate for AI Governance policy settings
│   ├── appledeclarations/ # Apple declarative device management schemas (generated table + plan-time declaration validation; see §Apple declaration schemas)
│   ├── appleprofiles/ # Apple configuration profile schemas (generated table + plan-time payload validation; see §Apple profile schemas)
│   ├── availabletitles/ # Shared patch available-titles lookup (patch_external_source, patch_internal_source)
│   ├── criteria/      # Shared smart-group / advanced-search criteria operator vocabulary (device_group, user_group, future searches)
│   ├── enumguard/     # Recurrence guard: no enum value or error code restated as a literal the SDK generates
│   ├── files/         # Shared upload-source plumbing for resources that upload file content
│   ├── filters/       # RSQL + classic filter schema/expression builder
│   ├── helpers/       # Type conversions, polling, timeout, state reconciliation, dynamic JSON, IDs, Pro version

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamf/terraform-provider-jamfplatform](https://github.com/jamf/terraform-provider-jamfplatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
