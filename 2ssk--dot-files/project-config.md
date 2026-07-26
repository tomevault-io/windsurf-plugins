---
trigger: always_on
description: - **Quality First**: Never compromise on code quality, testing, or observability
---

# Development Standards

## Workflow Principles

### Core Values
- **Quality First**: Never compromise on code quality, testing, or observability
- **DevOps Mindset**: Infrastructure as code, automated pipelines, everything reproducible
- **Type Safety**: Full type coverage for TypeScript/Go, strict typing everywhere
- **Performance**: Profile first, optimize second. C++/systems code measured with benchmarks

### Language-Specific Standards

#### Go
```bash
go fmt ./...
golangci-lint run
go test -race -coverprofile=coverage.out ./...
go build -o binary ./cmd/...
```
- Strict `error` handling with wrapped errors using `fmt.Errorf`
- Context propagation in all API calls
- Table-driven tests with subtests
- Benchmark critical paths with `testing.B`
- Use `slog` for structured logging
- Configuration via environment variables with `viper` or `envconfig`
- Accept interfaces, return structs
- Channels for orchestration, mutexes for state
- Functional options for APIs
- Small, focused interfaces

```go
// Always wrap errors with context
if err != nil {
    return fmt.Errorf("failed to do operation: %w", err)
}

// Context in all API calls
func DoSomething(ctx context.Context, arg Arg) (Result, error) {
    req, err := http.NewRequestWithContext(ctx, ...)
    // ...
}
```

#### TypeScript/React
```bash
npm run lint
npm run type-check
npm run test
npm run build
```
- Strict TypeScript with `"strict": true`
- No `any` without explicit justification
- React functional components with hooks
- Proper error boundaries
- 90%+ test coverage on critical paths
- Use `tRPC` for type-safe API calls

#### React Native
```bash
npm run lint && npm run type-check && npx jest
cd ios && pod install && cd ..
npm run build:ios
```
- Cross-platform code > 80%
- Platform-specific code via `Platform.select()`
- Offline-first architecture with proper sync
- Performance: cold start < 1.5s, memory < 120MB
- Hermes engine, RAM bundles for production

#### C++
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
cmake -B build -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined"
ctest --output-on-failure
```
- C++20/23 features where available
- Clang-Tidy and Cppcheck clean
- Zero compiler warnings with `-Wall -Wextra`
- AddressSanitizer and UBSan clean
- RAII everywhere, no raw new/delete
- Use `std::expected` for error handling
- Profile with `perf` and `pprof`

```cpp
// Smart pointers — always
auto ptr = std::make_unique<Resource>();
auto shared = std::make_shared<Cache>();

// Concepts (C++20)
template<typename T>
concept Hashable = requires(T a) {
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};

// std::expected (C++23)
std::expected<int, Error> divide(int a, int b) {
    if (b == 0) return std::unexpected{DivisionByZero{}};
    return a / b;
}
```

#### DevOps/SRE
- Infrastructure as Code (Terraform, Pulumi, or Ansible)
- Kubernetes manifests with Helm charts
- GitOps workflows (ArgoCD or Flux)
- SLO/SLI definitions for all services
- Alerting with proper severity levels
- Postmortems for all incidents

---

## Definition of Done

A task is complete when:

### Code Quality
- [ ] All tests pass (unit, integration, E2E)
- [ ] Linting passes without warnings
- [ ] Type checking passes
- [ ] No security vulnerabilities introduced
- [ ] Code review approved

### Testing
- [ ] Unit test coverage >= 80%
- [ ] Integration tests for critical paths
- [ ] E2E tests for user-facing features
- [ ] Performance benchmarks for critical code

### Observability
- [ ] Structured logging implemented
- [ ] Metrics exposed (Prometheus format)
- [ ] Distributed tracing configured
- [ ] Health check endpoints
- [ ] Alerting rules defined

### Documentation
- [ ] API documentation (OpenAPI/Swagger)
- [ ] README updated
- [ ] Inline code comments for complex logic
- [ ] Deployment/runbook documentation

---

## TDD Workflow

Enforce test-driven development for every implementation:

1. **Write Failing Test** — smallest unit of behavior, test fails because feature doesn't exist
2. **Write Minimal Code** — only what's needed to pass, no optimization yet
3. **Refactor** — clean up while tests pass, improve structure/names/organization

### Go Testing Pattern
```go
func TestUnitName(t *testing.T) {
    t.Run("should do thing", func(t *testing.T) {
        input := ...
        expected := ...
        result := DoThing(input)
        assert.Equal(t, expected, result)
    })
}
```

### TypeScript Testing Pattern
```typescript
describe('UnitName', () => {
  it('should do thing', () => {
    const result = doThing(input);
    expect(result).toEqual(expected);
  });
});
```

### C++ Testing Pattern
```cpp
TEST_F(UnitNameTest, ShouldDoThing) {
    auto input = ...;
    auto expected = ...;
    auto result = doThing(input);
    EXPECT_EQ(expected, result);
}
```

---

## Docker Best Practices

```dockerfile
# Multi-stage build
FROM golang:1.23-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /binary ./cmd/app

FROM alpine:3.20
RUN adduser -D -u 1000 appuser
WORKDIR /app
COPY --from=builder /binary /app/app
USER appuser

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2SSK/dot-files](https://github.com/2SSK/dot-files) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
