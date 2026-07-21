---
trigger: always_on
description: make e2e                                    # Run all tests
---

# E2E Test Guide

## Quick Reference

```bash
make e2e                                    # Run all tests
make e2e-select select=resource=httproute   # By resource
make e2e-select select=layer=layer7         # By layer
make e2e-select select=suite=conversion     # Conversion only
make e2e-select select=test=basic           # Single test name
chainsaw test --test-file test/e2e/tests/layer7/httproute/basic/chainsaw-test.yaml
```

## Clusters

| Cluster | Role | Nodes | kubelb_namespace |
|---------|------|-------|-----------------|
| `kubelb` | Manager cluster | 1 | N/A |
| `tenant1` | Default CCM tenant | 3 workers | `tenant-primary` |
| `tenant2` | Single-node CCM tenant | 1 | `tenant-secondary` |
| `standalone` | Conversion CCM | 1 | N/A |

## Test Structure

Tests live in `test/e2e/tests/{layer4,layer7}/{resource}/{test-name}/chainsaw-test.yaml`.
Templates live in `test/e2e/step-templates/{common,gateway,layer4,layer7,ingress,conversion,isolated}/`.

Standard L7 test pattern:

1. Deploy backend(s) via `deploy-echo-backend` template
2. Create Gateway via `create-http-gateway` template
3. Create route (HTTPRoute/GRPCRoute/Ingress) — usually inline script (needs Gateway IP for nip.io hostnames)
4. Verify Route CRD via `verify-route-crd` template
5. Verify HTTP connectivity — inline script
6. Cleanup in `finally` block — inline script (always inline, `catch` blocks don't support `use`)
7. Verify cleanup via `verify-route-cleanup` template

## Chainsaw Pitfalls

### `$namespace` is reserved

Chainsaw sets `$namespace` to auto-generated test namespace. Never use `namespace` as a binding name. Hardcode `default` directly.

### `use.with.bindings` must be hardcoded

JMESPath refs (`($var)`) silently resolve to empty. Always pass literal strings:

```yaml
# WRONG — silently empty
- name: gateway_name
  value: ($gateway_name)

# CORRECT
- name: gateway_name
  value: basic-gw
```

### StepTemplates use `try`, never `finally`

`finally` is invalid in StepTemplate spec. Tests that call templates handle cleanup themselves.

### StepTemplate `spec.bindings` — hardcoded defaults only

Cannot reference other bindings. Use bash defaults for optional params:

```yaml
content: |
  SERVICE_NAME="${SERVICE_NAME:-$INGRESS_NAME}"
```

### `catch` blocks don't support `use`

Only inline operations work in `catch`. Cleanup templates must be called as regular steps.

### `assert` is strict

- Ports must be int, not string
- Don't assert partial arrays (must match full length)

### JMESPath limitations

No `??` operator. Use bash defaults instead.

## Writing a New L7 Test

```yaml
apiVersion: chainsaw.kyverno.io/v1alpha1
kind: Test
metadata:
  name: my-test
  labels:
    all:
    test: my-test
    resource: httproute     # or: gateway, grpcroute, ingress
    layer: layer7
spec:
  steps:
    # 1. Deploy backend
    - name: deploy-backend
      use:
        template: ../../../../step-templates/common/deploy-echo-backend.yaml
        with:
          bindings:
            - name: backend_name
              value: my-echo
            - name: backend_message
              value: hello-from-my-test

    # 2. Create Gateway + wait for IP
    - name: create-gateway
      use:
        template: ../../../../step-templates/gateway/create-http-gateway.yaml
        with:
          bindings:
            - name: gateway_name
              value: my-gw

    # 3. Create HTTPRoute (needs Gateway IP for nip.io)
    - name: create-httproute
      cluster: tenant1
      try:
        - script:
            timeout: 60s
            content: |
              set -e
              IP=$(kubectl get gateway my-gw -n default \
                -o jsonpath='{.status.addresses[0].value}')
              HOST="my-test.${IP}.nip.io"
              cat <<EOF | kubectl apply -f -
              apiVersion: gateway.networking.k8s.io/v1
              kind: HTTPRoute
              metadata:
                name: my-route
                namespace: default
              spec:
                parentRefs:
                  - name: my-gw
                hostnames:
                  - "${HOST}"
                rules:
                  - backendRefs:
                      - name: my-echo
                        port: 80
              EOF
            check:
              ($error == null): true

    # 4. Verify Route CRD
    - name: verify-route-crd
      use:
        template: ../../../../step-templates/common/verify-route-crd.yaml
        with:
          bindings:
            - name: resource_name
              value: my-route
            - name: expected_kind
              value: HTTPRoute.gateway.networking.k8s.io
            - name: kubelb_namespace
              value: tenant-primary

    # 5. Verify HTTP
    - name: verify-http
      cluster: tenant1
      try:
        - script:
            timeout: 60s
            content: |
              set -e
              IP=$(kubectl get gateway my-gw -n default \
                -o jsonpath='{.status.addresses[0].value}')
              HOST="my-test.${IP}.nip.io"
              for i in $(seq 1 10); do
                RESPONSE=$(curl -s --max-time 5 -H "Host: ${HOST}" "http://${IP}/" 2>/dev/null || true)
                if echo "$RESPONSE" | grep -q "hello-from-my-test"; then
                  echo "OK"
                  exit 0

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubermatic/kubelb](https://github.com/kubermatic/kubelb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
