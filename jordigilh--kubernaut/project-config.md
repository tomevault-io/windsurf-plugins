---
trigger: always_on
description: Testing strategy and patterns for kubernaut's defense-in-depth approach
---


# Testing Strategy for Kubernaut

## 🔑 **CRITICAL: KUBERNETES CLIENT MANDATE**

**AUTHORITATIVE RULE**: All services that interact with Kubernetes MUST use the approved K8s client interface:

| Test Tier | MANDATORY Interface | Package |
|-----------|---------------------|---------|
| **Unit Tests** | **Fake K8s Client** | `sigs.k8s.io/controller-runtime/pkg/client/fake` |
| **Integration Tests** | Real K8s API (envtest/KIND) | `sigs.k8s.io/controller-runtime/pkg/client` |
| **E2E Tests** | Real K8s API (OCP/KIND) | `sigs.k8s.io/controller-runtime/pkg/client` |

**❌ FORBIDDEN**: Custom `MockK8sClient` implementations - use fake client instead
**✅ APPROVED**: `fake.NewClientBuilder()` for all unit tests requiring K8s interaction

**See**: [ADR-004: Fake Kubernetes Client for Unit Testing](mdc:docs/architecture/decisions/ADR-004-fake-kubernetes-client.md)

---

## 📊 **Defense-in-Depth Testing Pyramid**

### Coverage Targets: Per-Tier Testable Code Coverage (>=80%)

**TDD Mandate**: Every business requirement MUST have a corresponding test. If a feature has no test, it risks not being implemented. Coverage is measured **per tier against the tier-specific code subset**.

**Per-Tier Code Coverage** (AUTHORITATIVE):
- **Unit**: >=80% of **unit-testable** code (pure logic: config, validators, scoring, builders, types)
- **Integration**: >=80% of **integration-testable** code (I/O: reconciler, K8s clients, HTTP handlers, DB adapters)
- **E2E**: >=80% of full service code (full stack execution)
- **All Tiers**: >=80% merged (line-by-line dedup across all tiers)

**Code Partitioning**: Each service defines `unit_exclude`/`int_include` patterns in `scripts/coverage/coverage_report.py` that partition code into unit-testable (pure logic) and integration-testable (I/O-dependent) subsets. See TESTING_GUIDELINES.md v2.7.0 "Code Partitioning" section.

**Measurement**: `make coverage-report` (runs `scripts/coverage/coverage_report.py`)

**Business Requirement (BR) Coverage** - OVERLAPPING:
- **Unit**: 70%+ of ALL BRs (maximum coverage foundation)
- **Integration**: >50% of ALL BRs (critical interactions)
- **E2E**: <10% BR coverage (essential user journeys)

**Key**: Same BRs tested at multiple tiers (e.g., retry logic in unit, integration, AND E2E)

**See**: [TESTING_GUIDELINES.md](mdc:docs/development/business-requirements/TESTING_GUIDELINES.md) for full coverage model

---

## 🧪 **Test Tier Specifications**

### Unit Tests (70%+ BRs, >=80% Unit-Testable Code Coverage)

**Location**: `test/unit/`
**Purpose**: Extensive business logic validation covering ALL unit-testable business requirements
**Execution**: `make test`
**Framework**: **Ginkgo/Gomega BDD** (MANDATORY - no standard Go testing)
**Confidence**: 85-90%

**Mock Strategy**:
```go
// ✅ CORRECT: Mock ONLY external dependencies
var (
    mockLLMProvider   *mocks.MockLLMProvider    // External: AI service
    mockK8sClient     client.Client              // External: Use fake.NewClientBuilder()
    mockVectorDB      *mocks.MockVectorDatabase  // External: Database

    // Use REAL business logic components
    workflowBuilder   *engine.IntelligentWorkflowBuilder
    safetyFramework   *platform.SafetyFramework
    analyticsEngine   *insights.AnalyticsEngine
)
```

**What to Mock**:
- ✅ External APIs (LLM, HolmesGPT, OpenAI)
- ✅ Databases (PostgreSQL, Vector DB, Redis)
- ✅ Kubernetes API (use `fake.NewClientBuilder()`)
- ✅ Network services (external HTTP/gRPC)

**What to Keep Real**:
- ✅ **ALL** business logic (`pkg/` code)
- ✅ **ALL** internal algorithms
- ✅ **ALL** business validators/analyzers/optimizers
- ✅ **ALL** cross-package business interactions

**See**: [Unit Test Patterns](mdc:docs/testing/TESTING_PATTERNS_QUICK_REFERENCE.md)

---

### Integration Tests (<20% BRs, 50% Code Coverage)

**Location**: `test/integration/`
**Purpose**: Critical component interactions requiring real infrastructure
**Execution**: `make test-integration-[service]`
**Framework**: Ginkgo/Gomega BDD with envtest/KIND
**Confidence**: 75-85%

**When to Write**:
- ✅ CRD lifecycle with Kubernetes API
- ✅ Database transaction coordination
- ✅ Multi-service coordination flows
- ✅ Complex state management requiring real infrastructure

**Mock Strategy**:
```go
// ✅ ZERO MOCKS for business logic
// ✅ Real K8s API via envtest
// ✅ Real databases via testcontainers
// ✅ Mock ONLY external services (LLM, external APIs)
```

**See**: [Integration Test Infrastructure](mdc:docs/testing/INTEGRATION_E2E_NO_MOCKS_POLICY.md)

---

### E2E Tests (<10% BRs, 50% Code Coverage)

**Location**: `test/e2e/`
**Purpose**: Essential customer-facing workflows in production-like environments
**Execution**: `make test-e2e-[service]`
**Framework**: Ginkgo/Gomega BDD with KIND/OCP clusters
**Confidence**: 90-95%

**When to Write**:
- ✅ Critical user journeys (onboarding, alert → remediation)
- ✅ End-to-end security flows (auth, RBAC)
- ✅ Production deployment verification
- ✅ Cross-service full-stack scenarios

**Mock Strategy**:
```go
// ✅ ZERO MOCKS for internal components
// ✅ Real Kubernetes clusters (KIND/OCP)
// ✅ Real databases and infrastructure
// ✅ Mock LLM MAY be acceptable for test speed
```

**See**: [E2E Testing Strategy](mdc:docs/testing/DEFENSE_IN_DEPTH_CI_CD_STRATEGY.md)

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
