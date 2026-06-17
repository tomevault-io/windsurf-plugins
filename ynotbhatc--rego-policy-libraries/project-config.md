---
trigger: always_on
description: This repository is the **shared Rego policy library** consumed by every AAC (Ansible Automated Compliance) deployment and by the AAC Customer Portal. Every Rego policy here gets loaded into one of three OPA containers and queried by Ansible playbooks (compliance repo) or directly by the portal API.
---

# rego_policy_libraries — CLAUDE.md

## Purpose

This repository is the **shared Rego policy library** consumed by every AAC (Ansible Automated Compliance) deployment and by the AAC Customer Portal. Every Rego policy here gets loaded into one of three OPA containers and queried by Ansible playbooks (compliance repo) or directly by the portal API.

This is a **library**, not an application. It produces no binaries, runs no servers, and ships no playbooks. Its only deliverable is a tree of `.rego` files plus the unit tests that prove they evaluate correctly.

Consumers pull this repo as a git submodule at a pinned commit. Today there are two consumers in the AAC ecosystem:

- **`ynotbhatc/compliance`** → submodule at `policies/`
- **`ynotbhatc/AAC_Customer_Portal`** → reads policies via the loader playbook

## Repository structure

```
benchmarks/                    Per-platform configuration baselines
├── cis/                       CIS Benchmarks (200+ files)
│   ├── os/linux/redhat/       RHEL 8/9/10 by section
│   ├── os/linux/ubuntu/       Ubuntu 20/22/24
│   ├── os/windows/            Windows Server 2019/2022
│   ├── cloud/                 AWS / Azure / GCP foundations
│   ├── container/             Docker / Kubernetes
│   ├── saas/                  M365 (this is where the SaaS pattern lives)
│   ├── network/cisco/         Network device baselines
│   └── mobile/ios/            Mobile platforms
└── stig/                      DISA STIGs — RHEL/Win/Kubernetes/OpenShift

frameworks/                    Regulatory + management frameworks
├── federal/                   NIST 800-53, NIST 800-171, NIST CSF, NIST RMF,
│                              FISMA, FedRAMP, CMMC, NIST AI RMF
├── financial/                 PCI-DSS, SOX, DORA, NY DFS, SEC Cyber,
│                              SWIFT CSP
├── management/                ISO 27001, SOC 2, HITRUST, TISAX, CSA CCM,
│                              corporate, technical_debt
├── privacy/                   GDPR, HIPAA, ISO 27701, CCPA
├── compliance/                NCSC CAF, NIS2
├── critical_infrastructure/   NERC-CIP, AMI/NIST IR 7628, IEC 62443,
│                              NIST 800-82
├── regulatory/                CFR Part 11
└── sovereignty/               Digital Sovereignty

governance/                    Cross-cutting governance
├── ai/                        AI governance controls
├── eu_ai_act/                 EU AI Act
├── finops/                    FinOps governance
├── geisa/                     Grid Edge Interoperability + Security
├── mcp/                       MCP tool-call governance
└── oidc/                      OIDC enforcement

enforcement/                   Policy-as-Code gatekeepers (Sentinel-style)
├── ansible/                   Block bad Ansible playbooks
├── terraform/                 Block bad Terraform plans
├── dockerfile/                Block bad Dockerfiles
├── kubernetes/                Block bad K8s manifests
└── git/                       Git change approval policy (used for repo
                               protection — see Approved-By convention)

threat_detection/              Behavioral threat patterns
└── crypto_mining/             Crypto-miner indicators
```

Headline count: **476 .rego files** across the above directories.

## Skill: Rego v1 syntax (MANDATORY)

Every policy in this library **MUST** use Rego v1. The mandatory shape:

```rego
package <domain>.<section>

import rego.v1

default compliant := false

violation contains msg if {
    condition
    msg := "control_id: description — what's wrong"
}

compliant if { count(violation) == 0 }

compliance_report := {
    "section": "1.1",
    "name": "Initial Setup",
    "controls_evaluated": <int>,
    "violations": violation,
    "violation_count": count(violation),
    "compliant": compliant,
}
```

### Critical Rego v1 rules

1. `import rego.v1` at top of every file
2. Use `if` keyword on rule heads: `compliant if { ... }`
3. Use `contains` for partial sets: `violation contains msg if { ... }`
4. Use `in` for iteration: `some item in collection`
5. `default rule := false` is **required** — without it, a never-firing rule is `undefined`, and an `undefined` field in an object literal turns the entire object into `{}` at the endpoint
6. `array.concat()` takes **exactly 2 arrays** — for 3+ arrays, nest:

   ```rego
   # ✗ WRONG
   all := array.concat([v | some v in a], [v | some v in b], [v | some v in c])

   # ✓ CORRECT — nested
   ab := array.concat([v | some v in a], [v | some v in b])
   all := array.concat(ab, [v | some v in c])
   ```

7. Set → array: `[v | some v in <set>]` (sets and arrays aren't interchangeable)

## Skill: testing locally

```bash
# Test a single policy + its tests
opa test benchmarks/cis/os/linux/redhat/pam_validation.rego \
         benchmarks/cis/os/linux/redhat/tests/test_pam.rego -v

# Test an entire framework
opa test benchmarks/cis/os/linux/redhat/ -v --coverage

# Full repo (ignore the .github/ directory that lives inside the
# benchmarks tree — opa tries to parse the YAML otherwise)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ynotbhatc/rego_policy_libraries](https://github.com/ynotbhatc/rego_policy_libraries) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
