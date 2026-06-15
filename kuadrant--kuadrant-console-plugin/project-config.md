---
trigger: always_on
description: This is an OpenShift Console dynamic plugin for managing Kuadrant resources. It's built on top of the [console-plugin-template](https://github.com/openshift/console-plugin-template) but significantly extended with domain-specific functionality for gateway and policy management.
---

# Kuadrant Console Plugin Codebase Guide

## Overview

This is an OpenShift Console dynamic plugin for managing Kuadrant resources. It's built on top of the [console-plugin-template](https://github.com/openshift/console-plugin-template) but significantly extended with domain-specific functionality for gateway and policy management.

## Project Structure

```
kuadrant-console-plugin/
├── src/
│   ├── components/         # React components
│   │   ├── apikey/        # API Key request and approval components
│   │   ├── apiproduct/    # API Product catalog components
│   │   ├── dnspolicy/     # DNSPolicy components and types
│   │   ├── gateway/       # Gateway components
│   │   ├── httproute/     # HTTPRoute components
│   │   ├── issuer/        # Certificate issuer components
│   │   ├── ratelimitpolicy/ # RateLimitPolicy components
│   │   ├── tlspolicy/     # TLSPolicy components
│   │   └── topology/      # Topology visualization components
│   ├── hooks/             # Custom React hooks
│   │   └── topology/      # Topology-specific hooks
│   ├── utils/             # Utility functions
│   │   └── topology/      # Topology utilities and parsers
│   └── constants/         # Configuration constants
├── charts/                # Helm chart for deployment
├── locales/               # i18n translation files
├── config/
│   └── rbac/             # RBAC role definitions
├── e2e/                   # End-to-end Playwright tests
├── docs/                  # Documentation and images
└── scripts/               # Build and deployment scripts
```

## Key Technologies

- **React + TypeScript**: Core UI framework
- **PatternFly 6**: Red Hat's design system
- **OpenShift Console SDK**: Dynamic plugin integration
- **Kubernetes Client**: For resource management
- **webpack**: Build tooling
- **i18next**: Internationalisation

## Kuadrant Resources

The plugin manages these Custom Resource Definitions (CRDs):

### Gateway & Policy Resources
1. **AuthPolicy** (`kuadrant.io/v1`) - Authentication and authorisation policies
2. **RateLimitPolicy** (`kuadrant.io/v1`) - Rate limiting configurations
3. **DNSPolicy** (`kuadrant.io/v1`) - DNS management for gateways
4. **TLSPolicy** (`kuadrant.io/v1`) - TLS certificate configurations
5. **Gateway** (`gateway.networking.k8s.io/v1`) - Kubernetes Gateway API
6. **HTTPRoute** (`gateway.networking.k8s.io/v1`) - HTTP routing rules

### API Management Resources
7. **PlanPolicy** (`extensions.kuadrant.io/v1alpha1`) - Rate limiting plans with tiers
8. **APIProduct** (`devportal.kuadrant.io/v1alpha1`) - Published API catalog entries
9. **APIKeyRequest** (`devportal.kuadrant.io/v1alpha1`) - Consumer requests for API access
10. **APIKey** (`devportal.kuadrant.io/v1alpha1`) - API key credentials for consumers
11. **APIKeyApproval** (`devportal.kuadrant.io/v1alpha1`) - Approval records for API key requests

## Common Patterns

### 1. Resource Watching
```typescript
// use this pattern for real-time updates
const resource = {
  groupVersionKind: { group: 'kuadrant.io', version: 'v1', kind: 'AuthPolicy' },
  isList: true,
  namespace: activeNamespace
};
const [data, loaded, error] = useK8sWatchResource(resource);
```

### 2. Form Creation Pattern
All policy creation forms follow a similar structure:
- Toggle between Form and YAML views
- Form validation before submission
- Use `KuadrantCreateUpdate` component for save operations
- Redirect to list view after success

### 3. Error Handling
```typescript
const [errorAlertMsg, setErrorAlertMsg] = React.useState('');
try {
  await k8sCreate({ model, data: resource });
  history.push(redirectUrl);
} catch (error) {
  setErrorAlertMsg(error.message);
}
```

### 4. RBAC Checks
```typescript
const accessReviews = useAccessReviews(resourceAttributes);
const canRead = accessReviews[0];
```

### 5. Configuration

The plugin supports configurable Topology and Prometheus metrics for gateway traffic monitoring. This allows the console to work with different Gateway API implementations (OpenShift 4.19+, OSSM, etc.).

**Configuration is managed through:**
- `src/utils/configLoader.ts` - Configuration schema and defaults
- `src/utils/metricsQueries.ts` - Query utilities
- Environment variables in deployment manifests

**Example ENV Configuration:**
```yaml
TOPOLOGY_CONFIGMAP_NAME: "topology"
TOPOLOGY_CONFIGMAP_NAMESPACE: "kuadrant-system"
METRICS_WORKLOAD_SUFFIX: "-openshift-default"
```

## Key Components

- **KuadrantOverviewPage**: Main dashboard with gateway health status
- **PolicyTopologyPage**: Visual representation of gateway/route/policy relationships (refactored into modular components)
  - `CustomNode`: Renders topology nodes with icons and context menus
  - `ResourceFilterToolbar`: Manages resource type filtering
  - `TopologyControls`: Zoom/pan control buttons
  - `useVisualizationController`: Hook managing visualisation lifecycle
  - `useTopologyData`: Hook processing ConfigMap data into nodes/edges
  - `graphParser`: DOT string parsing and transitive edge preservation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kuadrant/kuadrant-console-plugin](https://github.com/Kuadrant/kuadrant-console-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
