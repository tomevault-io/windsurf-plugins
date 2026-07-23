---
trigger: always_on
description: This file provides project-specific guidance for the `terraform-provider-zia` Terraform provider. Follow these conventions when creating, modifying, or reviewing code.
---

# Terraform Provider ZIA — Claude Code Guidelines

This file provides project-specific guidance for the `terraform-provider-zia` Terraform provider. Follow these conventions when creating, modifying, or reviewing code.

## Project Overview

This is a Terraform provider for Zscaler Internet Access (ZIA). It wraps the ZIA REST API via the Go SDK (`zscaler-sdk-go/v3`). The provider implements resources and data sources following the `terraform-plugin-sdk/v2` patterns.

## Project Structure

```
zia/                           # All Go source files (resources, data sources, tests, helpers)
  provider.go                  # Provider registration
  resource_zia_<name>.go       # Resource implementations
  data_source_zia_<name>.go    # Data source implementations
  resource_zia_<name>_test.go  # Acceptance tests
  common.go                    # Shared schema builders & expand/flatten helpers
  utils.go                     # Shared utility functions
  common/version.go            # Provider version string
docs/resources/                # Resource documentation (published to Terraform Registry)
docs/data-sources/             # Data source documentation
examples/                      # Example .tf files
```

## SDK Client Pattern

```go
zClient := meta.(*Client)
service := zClient.Service
```

## Resource ID Convention (string `id` + typed `<resource>_id`)

The ZIA API identifies objects with an integer ID, but Terraform requires the
resource's internal `id` to be a string. Every resource therefore exposes two
attributes:

- `id` — `schema.TypeString`, `Computed: true`. This is Terraform's internal
  resource ID. It is set with `d.SetId(strconv.Itoa(resp.ID))` and is what users
  reference and import against.
- `<resource>_id` (e.g. `rule_label_id`, `header_profile_id`) —
  `schema.TypeInt`, `Computed: true`. This holds the API's native integer ID and
  is what the Create/Read/Update/Delete paths pass to the SDK.

Read the integer ID back out with the shared helper
`getIntFromResourceData(d, "<resource>_id")` (in `utils.go`) — never parse
`d.Id()` ad hoc in the CRUD body. The string↔int conversion lives only at the
boundaries: `d.SetId(strconv.Itoa(resp.ID))` writes the string form, and the
importer parses `d.Id()` with `strconv.ParseInt` (falling back to a name lookup
when the import key is not numeric).

```go
"id": {
    Type:     schema.TypeString,
    Computed: true,
},
"rule_label_id": {
    Type:     schema.TypeInt,
    Computed: true,
},
```

```go
// CRUD bodies read the typed integer ID, not d.Id():
id, ok := getIntFromResourceData(d, "rule_label_id")

// Create/Read set both forms from the API response:
d.SetId(strconv.Itoa(resp.ID))
_ = d.Set("rule_label_id", resp.ID)
```

`resource_zia_rule_labels.go` is the canonical reference implementation. Do not
reinvent the conversion or expose the integer ID as the Terraform `id`.

Avoid `Computed: true` on input attributes that the user fully controls. Use it
only where the API genuinely assigns or defaults the value (the two ID fields
above, API-defaulted fields, and API-assigned nested `id`s) — see "Schema
Conventions".

## Rule-Based Resources — Critical Conventions

Rule-based resources include: `zia_ssl_inspection_rules`, `zia_firewall_filtering_rule`, `zia_firewall_dns_rules`, `zia_firewall_ips_rules`, `zia_cloud_app_control_rules`, `zia_url_filtering_rules`, `zia_dlp_web_rules`, `zia_forwarding_control_rule`, `zia_nat_control_rules`, `zia_sandbox_rules`, `zia_bandwidth_control_rules`, `zia_traffic_capture_rules`, `zia_file_type_control_rules`, `zia_casb_dlp_rules`, `zia_casb_malware_rules`.

### Order Field Validation

The `order` field MUST include `ValidateFunc: validation.IntAtLeast(1)`:

```go
"order": {
    Type:         schema.TypeInt,
    Required:     true,
    ValidateFunc: validation.IntAtLeast(1),
},
```

Negative or zero order values are not supported. Rules with negative orders are internal default/predefined rules managed by the API.

### Stripping Read-Only Fields in updateOrder Callbacks

When the provider reorders rules, the `updateOrder` callback fetches each rule via GET and sends it back via PUT with the new order. For predefined rules, the API returns read-only fields that MUST NOT be included in the PUT body:

- `Predefined` (bool)
- `DefaultRule` (bool)
- `AccessControl` (string)

The `updateOrder` callback MUST strip these before the update:

```go
func(id int, order OrderRule) error {
    rule, err := <sdk_package>.Get(ctx, service, id)
    if err != nil {
        return err
    }
    rule.LastModifiedTime = 0
    rule.LastModifiedBy = nil
    rule.Predefined = false
    rule.DefaultRule = false
    rule.AccessControl = ""
    rule.Order = order.Order
    rule.Rank = order.Rank
    _, err = <sdk_package>.Update(ctx, service, id, rule)
    return err
},
```

Only strip fields that exist in the SDK struct for each rule type:

| Rule Resource | Fields to Strip |
|---|---|
| ssl_inspection, firewall_filtering, firewall_dns, firewall_ips, nat_control, traffic_capture | `Predefined`, `DefaultRule`, `AccessControl` |
| cloud_app_control | `Predefined`, `AccessControl` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zscaler/terraform-provider-zia](https://github.com/zscaler/terraform-provider-zia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
