---
trigger: always_on
description: CAPL is a Kubernetes Cluster API infrastructure provider for Linode/Akamai cloud services. It enables declarative management of Kubernetes clusters on Linode infrastructure using native Kubernetes APIs and follows the Cluster API v1beta1 specification.
---

# Cluster API Provider Linode (CAPL) - AI Agent Instructions

## Project Overview
CAPL is a Kubernetes Cluster API infrastructure provider for Linode/Akamai cloud services. It enables declarative management of Kubernetes clusters on Linode infrastructure using native Kubernetes APIs and follows the Cluster API v1beta1 specification.

## Architecture Patterns

### Controller-Scope-Service Architecture
All infrastructure resources follow a three-layer pattern:
1. **Controllers** (`internal/controller/`) - Handle Kubernetes reconciliation events
2. **Scopes** (`cloud/scope/`) - Encapsulate reconciliation context with both K8s and Linode clients
3. **Services** (`cloud/services/`) - Abstract Linode API interactions (loadbalancers, domains, object storage)

### Standard Controller Structure
```go
func (r *LinodeResourceReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. Fetch the resource
    // 2. Create scope with clients: scope.NewResourceScope(ctx, r.LinodeClientConfig, params)
    // 3. Check pause conditions
    // 4. Call reconcile helper with proper defer for status updates
    // 5. Handle errors and events
}
```

### Scope Pattern Usage
Every resource controller creates a scope that manages:
- Kubernetes client for CRD operations
- Linode client for API calls
- Resource references and credentials
- Patch helpers for status updates

Example: `scope.NewClusterScope()` combines `LinodeCluster` + CAPI `Cluster` resources.

## Key Resource Types

### Core Infrastructure
- **LinodeCluster**: Cluster networking, load balancers (NodeBalancer/DNS), VPC references, firewalls
- **LinodeMachine**: Compute instances with placement groups, disk configuration, networking
- **LinodeVPC**: Virtual Private Cloud with IPv4/IPv6 subnets
- **LinodeFirewall**: Cloud firewall rules with AddressSet references
- **LinodePlacementGroup**: Anti-affinity constraints for high availability

### API Design Conventions
- **Dual Reference Pattern**: Support both direct IDs (`vpcID: 123`) and K8s object refs (`vpcRef: {name: "vpc-1"}`)
- **Credential References**: All resources support `credentialsRef` for multi-tenancy
- **Immutable Fields**: Use `+kubebuilder:validation:XValidation:rule="self == oldSelf"` for region, type, etc.
- **Status Structure**: Always include `ready`, `failureReason`, `failureMessage`, `conditions`

## Development Workflows

### Build & Test Commands
- `mise run generate` - Regenerate CRDs and mocks after API changes
- `mise run test` - Run unit tests with mocked clients
- `mise run e2e E2E_SELECTOR=quick` - Run specific E2E tests using Chainsaw
- `mise run lint` - Run golangci-lint with project-specific rules
- `mise run build` - Build the controller manager binary

### Adding New Resources
1. Define API types in `api/v1alpha2/` with proper validation markers
2. Implement controller in `internal/controller/` following the standard pattern
3. Add scope in `cloud/scope/` for client management
4. Add validation webhook in `internal/webhook/v1alpha2/`
5. Add cloud services in `cloud/services/` if needed
6. Run `mise run generate` to update CRDs and mocks
7. Add E2E tests in `e2e/<resource>-controller/`

### Testing Patterns
- Unit tests use GoMock with `mock.MockLinodeClient` and `mock.MockK8sClient`
- Mock expectations pattern: `mockLinodeClient.EXPECT().Method().Return(result, error)`
- E2E tests use Chainsaw YAML manifests in `e2e/` directories organized by controller and flavors
- Service tests mock both success and error scenarios from Linode API
- Test naming uses dynamic identifiers: `(join('-', ['e2e', 'feature', env('GIT_REF')]))`
- Table-driven tests with `name`, `objects`, `expectedError`, `expectedResult`, `expectations` structure

## Linode Platform Integration

### Load Balancer Types
- **NodeBalancer**: Linode's managed load balancer for cluster API endpoints
- **DNS**: Uses Linode or Akamai DNS for API endpoint resolution
- **External**: For existing external load balancers

### Networking Features
- **VPC**: Private networking with configurable IPv4/IPv6 subnets
- **Firewalls**: Cloud firewalls with inbound/outbound rules and AddressSet reuse
- **Placement Groups**: Anti-affinity for spreading instances across failure domains

### Bootstrap Integration
- Supports kubeadm, k3s, and rke2 bootstrap providers
- Uses cloud-init with Linode's metadata service
- Object storage integration for large bootstrap payloads via pre-signed URLs

## Common Patterns

### Standard Reconciliation Structure
```go
func (r *Controller) reconcile(ctx context.Context, scope *ScopeType) (res ctrl.Result, err error) {
    scope.Resource.Status.Ready = false
    scope.Resource.Status.FailureReason = nil
    
    defer func() {
        if err != nil {
            scope.Resource.Status.FailureReason = util.Pointer("ReconcileError")
            scope.Resource.Status.FailureMessage = util.Pointer(err.Error())
        }
        if patchErr := scope.Close(ctx); patchErr != nil {
            err = errors.Join(err, patchErr)
        }
    }()
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linode/cluster-api-provider-linode](https://github.com/linode/cluster-api-provider-linode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
