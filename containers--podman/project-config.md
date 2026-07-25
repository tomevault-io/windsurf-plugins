---
trigger: always_on
description: ![PODMAN logo](https://raw.githubusercontent.com/containers/common/main/logos/podman-logo-full-vert.png)
---

# AI Agent Guide for Podman Development

![PODMAN logo](https://raw.githubusercontent.com/containers/common/main/logos/podman-logo-full-vert.png)

## Persona

This guide is for AI coding assistants (for example Claude, ChatGPT, Copilot). Use it for context on codebase layout, development patterns, testing, pitfalls, and upstream expectations when helping **contributors to [containers/podman](https://github.com/containers/podman)**—people writing patches, tests, and in-tree docs, triaging or fixing issues, and preparing pull requests.

When assisting them, align with how upstream describes the project and how contributors are expected to work.

- **Audience**: Assume the user is an **upstream contributor** (or aspiring one), not an end user or downstream packager. Optimize for implementing and reviewing changes in this repository: correct layer (`cmd/` vs `libpod/` vs `pkg/domain/`), tests that match existing frameworks, and merge-ready hygiene. Be direct and technical; skip tutorial and brochure tone unless they are editing tutorials or man pages in-tree.
- **Product mental model (for patch context)**: Podman is **daemonless**; lifecycle logic lives in **libpod**. When touching behavior, remember **Docker-compatible CLI/API** paths versus **Podman-specific** surfaces (pods, Quadlet, advanced REST, `podman machine`). Many fixes must consider **rootless vs root** and **local vs remote** (`pkg/domain/infra/abi` vs `tunnel`) so both paths stay consistent.
- **Vendored dependencies**: Most external code Podman depends on (containers/image, containers/storage, containers/buildah, containers/common) is checked into `vendor/`. **Never edit vendored files directly**—use `go get` then `make vendor`. When diagnosing behavior that originates in a vendored library, trace the call but propose fixes in the upstream library repo, not in `vendor/`.
- **Quality bar**: Backend/libpod development expects **Linux**; macOS/Windows instructions apply to **clients** and `podman machine`, not the Linux engine. Use the **Makefile** (`make help`, `make binaries`, `make validatepr`); match the **Go** version in `go.mod`. **Security** issues use the private process linked from CONTRIBUTING, not public GitHub. AI-assisted contributions must follow **[LLM_POLICY.md](LLM_POLICY.md)**. For issues they file upstream, insist on reproducers and full `podman info`; discourage noise ("+1" without new data).

## Project Overview

**Podman** is a daemonless container engine with Docker-compatible CLI, rootless support, native pod management, and systemd integration via Quadlet.

## Quick Start

```bash
# Build and test
make binaries           # Build all binaries
make validatepr         # Format, lint, and validate (required for PRs)
make localintegration   # Run integration tests
make localsystem        # Run system tests

# Development tools
make install.tools      # Install linters and dev tools
```

## Codebase Structure

```text
podman/
├── cmd/podman/               # CLI commands (Cobra framework)
├── cmd/quadlet/              # Quadlet systemd unit generator
├── libpod/                   # Core container/pod management (Linux only)
├── pkg/
│   ├── api/                  # REST API server
│   ├── bindings/             # HTTP client (stable API)
│   ├── domain/               # Business logic layer
│   │   ├── entities/         # Interfaces and data structures
│   │   ├── infra/abi/        # Local implementation
│   │   └── infra/tunnel/     # Remote implementation
│   └── specgen/              # Container/pod specifications
├── test/e2e/                 # Integration tests (Ginkgo)
├── test/system/              # System tests (BATS)
├── docs/source/markdown/     # Man pages
└── vendor/                   # Vendored dependencies (DO NOT EDIT)
```

## Development Patterns

### CLI Command Pattern

```go
// cmd/podman/command.go
var commandCmd = &cobra.Command{
    Use:   "command [options] args",
    RunE:  commandRun,
}

func commandRun(cmd *cobra.Command, args []string) error {
    return registry.ContainerEngine().Command(registry.GetContext(), options)
}
```

### Domain Layer Pattern

```go
// pkg/domain/infra/abi/command.go (local)
func (ic *ContainerEngine) Command(ctx context.Context, options entities.CommandOptions) error {
    return ic.Libpod.Command(options)  // Direct libpod call
}

// pkg/domain/infra/tunnel/command.go (remote)
func (ic *ContainerEngine) Command(ctx context.Context, options entities.CommandOptions) error {
    return bindings.Command(ic.ClientCtx, options)  // HTTP API call
}
```

## Testing

### Integration Tests ([Ginkgo](https://github.com/onsi/ginkgo))

**Integration Tests** (`test/e2e/`): Test Podman CLI commands end-to-end, using actual binaries and real containers. Use for testing user-facing functionality and CLI behavior.

```go
It("should work correctly", func() {
    session := podmanTest.Podman([]string{"command", "args"})
    session.WaitWithDefaultTimeout()
    Expect(session).Should(Exit(0))
})
```

### System Tests ([BATS](https://github.com/bats-core/bats-core))

**System Tests** (`test/system/`): Test Podman in realistic environments with shell scripts. Use for testing complex scenarios, multi-command workflows, and system integration.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [containers/podman](https://github.com/containers/podman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
