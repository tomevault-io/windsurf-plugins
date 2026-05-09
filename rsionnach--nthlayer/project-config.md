---
trigger: always_on
description: **Reliability at build time, not incident time.**
---

# NthLayer

**Reliability at build time, not incident time.**

NthLayer is the Reliability Shift Left platform - bringing production readiness validation into your CI/CD pipeline. Instead of discovering reliability issues in production, catch them before deployment.

## Product Vision & Scope

**Core Value Proposition:** "Validate reliability requirements before deploy, not after incidents"

### Reliability Shift Left

| Stage | What NthLayer Does |
|-------|-------------------|
| **Generate** | Create dashboards, alerts, SLOs from service.yaml |
| **Lint** | Validate PromQL syntax before Prometheus rejects it |
| **Verify** | Confirm declared metrics exist in Prometheus |
| **Gate** | Block deploys when error budget exhausted |

**The key insight:** Generation is just the mechanism. The value is **prevention**.

### Competitive Positioning

| Competitor | Their Focus | NthLayer Difference |
|------------|-------------|---------------------|
| **PagerDuty** | Incident response | "They respond to incidents, we prevent them" |
| **Datadog** | Post-deploy monitoring | "They monitor after, we validate before" |
| **Nobl9** | SLO tracking | "They track SLOs, we enforce them as gates" |
| **Backstage** | Service catalog | "They document, we generate and enforce" |

**Strategic focus:** Compete where they won't go - build-time reliability validation.

### Core Actions

| Action | What NthLayer Does |
|--------|-------------------|
| **Generate** | Dashboards, alerts, SLOs, recording rules from service.yaml |
| **Validate** | Lint PromQL, verify metrics exist, check policies |
| **Gate** | Block deploys when error budget exhausted |

> **See also:** [Full Architecture Documentation](docs-site/architecture.md) for detailed diagrams.

### Usage Modes

| Mode | Description | Catalog Required? |
|------|-------------|-------------------|
| **Standalone** | Git + YAML, no catalog | ❌ No - Start here |
| **With Catalog** | Sync metadata from Backstage/Cortex | Optional |
| **Hybrid** | Catalog + local overrides | Optional |

**Key differentiator:** Catalogs make you adopt their platform first. NthLayer works Day 1.

### What We Generate

| Domain | Output | Status |
|--------|--------|--------|
| **Dashboards** | Grafana dashboards | ✅ Complete |
| **Alerts** | Prometheus rules | ✅ Complete |
| **Recording Rules** | Pre-aggregated metrics | ✅ Complete |
| **PagerDuty** | Teams, schedules, escalation policies | ✅ Complete |
| **SLOs** | OpenSLO definitions, error budgets | ✅ Complete |
| **Deployment Gates** | CI/CD exit codes, error budget validation | ✅ Complete |
| **Runbooks** | Auto-generated troubleshooting guides | 🔬 Exploring |

## Roadmap

### Current Focus: CI/CD Integration & Adoption

**Phase 3: SLO Portfolio** ✅ COMPLETE
- `nthlayer portfolio` command with CI/CD exit codes (0/1/2)
- Cross-service SLO aggregation with tier weighting
- Health scoring (0-100%) and insights
- Output formats: table, json, csv, markdown

**Phase 4: Deployment Gates** ✅ COMPLETE
- `nthlayer check-deploy` blocks deploys when error budget exhausted
- Tier-based thresholds (critical: 10% blocking, standard: advisory)
- Prometheus integration for live SLO data
- Exit codes: 0=approved, 1=warning, 2=blocked

**Next: CI/CD Integration Examples**
- GitHub Actions workflow templates
- ArgoCD PreSync hook examples
- GitLab CI integration

### Scope Discipline

Before adding features, ask:
1. Does it strengthen Generate → Validate → Gate?
2. Is there a dedicated tool that does this better?
3. Will this help Phase 3 & 4 adoption?

If unsure, defer to Future Considerations.

### Strategic Differentiation
**Primary narrative: Reliability Shift Left**
- Prevention over response - catch issues before production
- CI/CD integration - verification and gates as pipeline steps
- Cross-vendor SLO Portfolio (they want lock-in)

**Core differentiators:**
- `nthlayer verify` - Contract verification (metrics exist)
- `nthlayer check-deploy` - Deployment gates (error budget)
- `nthlayer apply --lint` - PromQL validation
- Generation is the mechanism, prevention is the value

**Complementary tool integrations:**
- promruval - Enhanced rule validation (40+ validators)

### Phase 1: Foundation (✅ DONE)
- service.yaml spec and parser
- Grafana dashboard generation
- Prometheus alert generation
- PagerDuty integration
- pint PromQL linting

### Phase 2: Error Budgets (✅ DONE)
- `nthlayer slo show/list` - View SLOs from service.yaml
- `nthlayer slo collect` - Real-time Prometheus queries (stateless)
- Blame deferred until CI/CD integration

### Phase 2.5: Loki Integration (✅ DONE)
**Goal:** Complete observability with logs (same Grafana ecosystem)
- `nthlayer generate-loki-alerts` - Generate LogQL alert rules from service.yaml
- Technology-specific log patterns (PostgreSQL, Redis, Kafka)
- Full test coverage in `tests/test_loki.py`

### Phase 3: SLO Portfolio (✅ DONE)
**Goal:** Stateless, cross-service SLO aggregation for CI/CD pipelines

**Completed:**
- `nthlayer portfolio` - Aggregate SLO status across all services
- Exit codes: 0=healthy, 1=warning, 2=critical
- Output formats: table, json, csv, markdown
- Health scoring by tier with org-wide score (0-100)
- Prometheus integration for live data
- Actionable insights generation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rsionnach/nthlayer](https://github.com/rsionnach/nthlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
