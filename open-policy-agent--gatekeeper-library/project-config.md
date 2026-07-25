---
trigger: always_on
description: **OPA Gatekeeper Library** is a community-owned library of policies for the [OPA Gatekeeper project](https://open-policy-agent.github.io/gatekeeper/website/docs/). This repository contains **validation** and **mutation** policies implemented as Kubernetes Custom Resource Definitions (ConstraintTemplates and Constraints) that integrate with the OPA Gatekeeper admission controller.
---

# Agent Instructions for OPA Gatekeeper Library

## Project Overview
**OPA Gatekeeper Library** is a community-owned library of policies for the [OPA Gatekeeper project](https://open-policy-agent.github.io/gatekeeper/website/docs/). This repository contains **validation** and **mutation** policies implemented as Kubernetes Custom Resource Definitions (ConstraintTemplates and Constraints) that integrate with the OPA Gatekeeper admission controller.

**Repository Size & Structure:**
- **Large repository** (~50+ validation policies, ~6 mutation policies)
- **Languages:** Rego (policy logic), Go (tooling/scripts), YAML (Kubernetes manifests), Gomplate (templating), **CEL (Common Expression Language for K8sNativeValidation)**
- **Key Technologies:** Open Policy Agent (OPA), Kubernetes CRDs, Gatekeeper, Docker, Helm, **CEL (K8sNativeValidation engine)**
- **Architecture:** Template-driven code generation from `src/` to `library/` using gomplate
- **Policy Engines:** **Dual-engine support** - policies can have both Rego (OPA) and CEL (K8sNativeValidation) implementations

## Critical Build & Validation Commands

### Environment Prerequisites
**Always ensure these tools are available before development:**
- **Docker**: Required for all generation and testing (containerized tools)
- **Go 1.20+**: Required for script validation and tooling
- **gator CLI**: Required for policy testing (install from [Gatekeeper releases](https://github.com/open-policy-agent/gatekeeper/releases))
- **OPA CLI**: Required for Rego unit tests (install from [OPA releases](https://github.com/open-policy-agent/opa/releases))

### Essential Build Commands (Must Work Before PR)
**CRITICAL: Always run these commands in this exact order before submitting any changes:**

1. **Generate library manifests** (REQUIRED for every src/ change):
   ```bash
   make generate
   ```
   - Generates `library/*/template.yaml` from `src/*/constraint.tmpl`
   - Uses dockerized gomplate - will build container if needed
   - **Always run this after editing any file in src/**

2. **Validate repository structure** (REQUIRED):
   ```bash
   make validate
   ```
   - Validates website docs directory structure
   - Runs in ~1 second

3. **Run Go unit tests for scripts** (REQUIRED):
   ```bash
   make unit-test
   ```
   - Tests all Go tooling in scripts/artifacthub, scripts/validate, scripts/require-sync
   - Runs in ~1-2 seconds

4. **Validate sync requirements** (REQUIRED):
   ```bash
   make require-sync
   ```
   - Validates that policies using data.inventory have proper sync.yaml files
   - Validates metadata.gatekeeper.sh/requires-sync-data annotations
   - Runs in ~2-3 seconds

5. **Validate suite.yaml files exist** (REQUIRED):
   ```bash
   make require-suites
   ```
   - Ensures every template.yaml has a corresponding suite.yaml test file
   - Runs in ~1 second

6. **Run OPA unit tests** (REQUIRED - needs OPA CLI):
   ```bash
   ./test.sh
   ```
   - Runs `opa test` on all Rego files in strict mode
   - **Will fail with "opa: command not found" if OPA CLI not installed**
   - Install OPA: `curl -L -o opa https://github.com/open-policy-agent/opa/releases/download/v0.57.1/opa_linux_amd64 && chmod +x opa && sudo mv opa /usr/local/bin/`

7. **Run gator integration tests** (REQUIRED - needs gator CLI):
   ```bash
   make verify-gator-dockerized POLICY_ENGINE=rego
   ```
   - Tests all suite.yaml files using dockerized gator
   - **Will fail if gator CLI not available**
   - For CEL policies: `make verify-gator-dockerized POLICY_ENGINE=cel`

### Complete CI Validation Sequence
**To replicate exactly what CI does:**
```bash
# Generate and validate everything is up-to-date
make generate generate-website-docs generate-artifacthub-artifacts
git diff --exit-code || (echo "Files need regeneration" && exit 1)

# Run all validation checks
make validate
make unit-test  
make require-suites
make require-sync

# Run policy tests (requires OPA and gator CLI)
./test.sh
make verify-gator-dockerized POLICY_ENGINE=rego
make verify-gator-dockerized POLICY_ENGINE=cel
```

## Project Structure & Architecture

### Repository Layout
```
├── src/                                     # Source templates and Rego/CEL policies (EDIT HERE)
│   ├── general/                             # General validation policies  
│   │   ├── httpsonly/                       # Example: HTTPS-only ingress policy (Rego only)
│   │   │   ├── constraint.tmpl              # Gomplate template → generates library/.../template.yaml
│   │   │   ├── src.rego                     # Rego policy logic
│   │   │   └── src_test.rego                # OPA unit tests
│   │   ├── requiredlabels/                  # Example: Required labels policy (Rego + CEL)
│   │   │   ├── constraint.tmpl              # Dual-engine template (both Rego and CEL)
│   │   │   ├── src.rego                     # Rego policy logic
│   │   │   ├── src.cel                      # CEL policy logic (K8sNativeValidation engine)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-policy-agent/gatekeeper-library](https://github.com/open-policy-agent/gatekeeper-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
