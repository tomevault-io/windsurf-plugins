---
trigger: always_on
description: Validates framework + default + vendor adapter JSON files for consistency (PID references, action+domain compatibility, match_rules validity).
---

# AGENTS.md — hwhomebridge

## What This Is

A Home Assistant custom integration that bridges devices into the Huawei HiLink ecosystem. Users control devices through the Huawei Smart Life app or Huawei voice assistant.

**Architecture Evolution**: Originally designed for Xiaomi/mi devices (via the `xiaomi_miot` integration), now refactored to support **any HA device** through generic discovery mechanisms.

The architecture is **Python (business logic) + C (HiLink SDK via ctypes FFI)**, running as a HA custom component in `custom_components/hwhomebridge/`.

## Architecture Overview

```
┌──────────────────────────────────────────────────────┐
│                   HA Python Process                   │
│                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐ │
│  │ Product      │  │ Product     │  │ Virtual       │ │
│  │ Registry     │─▶│ Matcher     │─▶│ Device        │ │
│  │ (框架+适配器) │  │ (三级匹配)   │  │ (聚合单元)    │ │
│  └─────────────┘  └─────────────┘  └──────┬───────┘ │
│                                             │          │
│                 ┌─────────────┐              │          │
│                 │ Service     │              │          │
│                 │ Router      │◀─────────────┘          │
│                 │ (控制路由)   │                         │
│                 └──────┬──────┘                          │
│                        │                                 │
│  ┌─────────────────────┴───────────────────────┐       │
│  │                hwbridge.py                    │       │
│  └────────────────────┬─────────────────────────┘       │
│                       │ ctypes                           │
└───────────────────────┼─────────────────────────────┘
                        │
┌───────────────────────┼─────────────────────────────┐
│                       ▼                              │
│          libhilink_bridge.so (C)                     │
│          HiLink SDK → 华为云                          │
└─────────────────────────────────────────────────────┘
```

## Three-Layer Configuration Architecture

The configuration is split into three layers, separating stable Huawei definitions from volatile HA mappings:

```
config/
├── product_registry.json              # Layer 1: Framework (华为侧定义, 稳定)
├── adapters/
│   ├── default/                        # Layer 2: Default adapters (华为标准 HA 映射)
│   │   ├── 001.json  002.json  ...   #   从华为 profile 提取的标准映射
│   ├── xiaomi/                         # Layer 3: Vendor adapters (厂家差异覆盖)
│   │   ├── cooker.json  bulb.json  ...
│   └── midea/
│       └── cooker.json
```

### Layer 1: Framework (`product_registry.json`)

Pure Huawei-side definitions. Contains only:
- `pid`, `name`, `services`
- `services`: `service_type` + `char_name` (mirrors Huawei profile)
- `auto_match`: optional auto-match rules for standard categories (lights, switches, sensors)

**This file never changes** unless a new PID is registered with Huawei.

### Layer 2: Default Adapters (`adapters/default/*.json`)

Standard HA mappings extracted from Huawei profiles. Represents "how a Huawei device would map to HA". Contains:
- `domain`, `action`, `value_attr` (HA protocol-level mapping)
- `value_mapping` (standard enum/text mappings from profile enumList)
- `brightness_range`, `colorTemperature_range`, `colorTemperature_min` (from profile min/max)
- No `match_rules` (not matched directly, only inherited)
- No `name_keywords`, `on_command`, `stop_option` (vendor-specific)
- `exclude_keywords`, `include_keywords` (entity filtering for multi-entity devices)

Used by `auto_match` when no vendor adapter matches. Can be updated independently of framework.

### Layer 3: Vendor Adapters (`adapters/<vendor>/*.json`)

Vendor-specific overrides. Contains:
- `match_rules`: device identification (model_exact, model_keyword, name_keyword)
- `services`: only fields that differ from default (field-level override)

**Simple categories** (lights, switches): often only need `match_rules`, inheriting everything from default.

**Complex categories** (cookers, kettles): may override entire services (different domain, value_mapping, etc.).

### Three-Layer Merge

```
framework (service_type, char_name)        ← Huawei definition, never changes
  + default adapter (standard ha_mapping)    ← from Huawei profile
    + vendor adapter (override differences)  ← only fields that differ
      = final ProductDef
```

Merge rule: vendor adapter fields override default (non-null fields win). Framework provides `service_type`/`char_name` unchanged.

### Matching Flow (Three-Level)

```
Device enters
  │
  ├─ 1. Vendor adapter match_rules (by rule type priority):
  │     model_exact → model_keyword → name_keyword → entity_composition
  │     Match → merge(framework + default + vendor) → return
  │
  ├─ 2. Framework auto_match (standard categories zero-config):
  │     required_domains ⊆ device domains + optional name_keywords
  │     Match → merge(framework + default) → return
  │
  └─ 3. No match → skip device
```

## Module Responsibilities

| File | Role |
|------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wangxiaokang666-666/ha_hwhomebridge](https://github.com/Wangxiaokang666-666/ha_hwhomebridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
