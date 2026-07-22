---
trigger: always_on
description: kgateway is a control plane implementing the Kubernetes Gateway API for Envoy. It's built on KRT (Kubernetes Declarative Controller Runtime from Istio) and uses a plugin-based architecture for extensibility.
---

# kgateway AI Agent Instructions

## Project Overview
kgateway is a control plane implementing the Kubernetes Gateway API for Envoy. It's built on KRT (Kubernetes Declarative Controller Runtime from Istio) and uses a plugin-based architecture for extensibility.

## Architecture (Read This First!)

### Translation Pipeline (3 phases)
1. **Policy → IR**: Plugins translate CRDs to PolicyIR (close to Envoy protos). Done once per policy CRD change.
2. **HTTPRoute/Gateway → IR with Policies Attached**: Core kgateway aggregates routes/gateways and performs policy attachment via `targetRefs`.
3. **IR → xDS**: Translates to Envoy config. Plugins provide `NewGatewayTranslationPass` functions called during route/listener translation.

See `/devel/architecture/overview.md` and the translation diagram at `/devel/architecture/translation.svg`.

### Key Components
- **cmd/**: 3 binaries: `kgateway` (controller), `envoyinit` (does some envoy bootstrap config manipulation), `sds` (secret server)
- **api/v1alpha1/kgateway/**: kgateway CRD definitions. Use `+kubebuilder` markers for validation/generation
- **pkg/pluginsdk/**: Plugin interfaces (`Plugin`, `PolicyPlugin`, `BackendPlugin`)
- **pkg/kgateway/extensions2/plugins/**: Plugin implementations (trafficpolicy, httplistenerpolicy, etc.)
- **pkg/kgateway/krtcollections/**: KRT collections for core resources
- **test/e2e/**: End-to-end tests using custom framework (see test/e2e/README.md)

### Plugin System
At the core kgateway translates kubernetes Gateway API resources to Envoy configuration. To add features
like policies, or backends, we use a plugin system. Each plugin *contributes* to the translation, usually by
adding a new type of CRD (most commonly a Policy CRD) that users can create to express their desired configuration.

Policy CRDs are attached to Gateway API resources via `targetRefs` or `targetSelectors`. kgateway manages the attachment
of policies to the appropriate resources during translation.

The plugin is then called in the translation process to affect the dataplane configuration.
To do this efficiently, the plugin should convert the CRD to an intermediate representation (IR) that is as close to Envoy protos as possible. This minimizes the amount of logic needed in the final translation, and allows for better status reflected back to the user if there are errors.

Plugins are **stateless across translations** but maintain state during a single gateway translation via `ProxyTranslationPass`. Each plugin:
- Provides a KRT collection of `ir.PolicyWrapper` (contains `PolicyIR` + `TargetRefs`)
- Implements `NewGatewayTranslationPass(tctx ir.GwTranslationCtx, reporter reporter.Reporter) ir.ProxyTranslationPass`
- Can process backends via `ProcessBackend`, `PerClientProcessBackend`, or `PerClientProcessEndpoints`

Example: `/pkg/kgateway/extensions2/plugins/trafficpolicy/traffic_policy_plugin.go`

## Development

## Critical Developer Patterns

### go build tag e2e

If you intend to include all source code, run 'go' commands that accept '-tags' with '-tags e2e'.

### IR Equals() Methods (STRICTLY ENFORCED)
IRs output by KRT collections **must** implement `Equals(other T) bool`:
- **Compare ALL fields** or mark with `// +noKrtEquals` (last line of comment)
- **Never use `reflect.DeepEqual`** (flagged by custom analyzer in `/hack/krtequals/`)
- Use proto equality helpers: `proto.Equal()`, not `==`

### Code Generation Workflow
Common targets:
- `make generate-code`: Ignores stamp files, generates all (takes around 30 seconds)
- `make generate-all`: Uses stamp files, only regenerates changed code (fast)
- `make verify`: CI target - always regenerates everything, checks git diff
- `make go-generate-apis`: Only API changes (~1m)
- `make fmt` or `make fmt-changed`: Format code (always run before commit)

After API changes: Run `make go-generate-apis` then `make fmt-changed`. The Makefile uses dependency tracking in `_output/stamps/`.
If not sure, just run `make generate-all`.

### Testing Conventions
- **Unit tests**: For new code, avoid Ginkgo. You may use Gomega matchers if appropriate.
- **E2E tests**: Use framework in `/test/e2e/` - DO NOT directly kubectl apply in tests
- **Custom matchers**: `/test/gomega/matchers/` (e.g., `HaveHttpResponse`)
- **Transforms**: Compose matchers with `WithTransform()` (see `/devel/testing/writing-tests.md`)
- Prefer explicit error checking: `Expect(err).To(MatchError("msg"))` over `HaveOccurred()`
- Add descriptions: `Expect(x).To(BeEmpty(), "list should be empty on init")`

Run tests:
```bash
make test TEST_PKG=./path/to/package  # Unit tests
make e2e-test TEST_PKG=./test/e2e/tests/...  # E2E tests
make unit  # All unit tests (excludes e2e)
```

### API/CRD Development

#### Adding New CRDs
1. Create `*_types.go` in `api/v1alpha1/` with `+kubebuilder` markers. You can use `+kubebuilder:validation:AtLeastOneOf` or `+kubebuilder:validation:ExactlyOneOf` for field groups.
2. **Required fields**: Use `+required`, NO `omitempty` tag
3. **Optional fields**: Use `+optional`, pointer types (except slices/maps), `omitempty` tag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kgateway-dev/kgateway](https://github.com/kgateway-dev/kgateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
