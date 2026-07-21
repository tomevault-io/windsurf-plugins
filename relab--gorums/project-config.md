---
trigger: always_on
description: Gorums is a framework for building fault-tolerant distributed systems using quorum-based abstractions.
---

# Agent Instructions for Gorums

Gorums is a framework for building fault-tolerant distributed systems using quorum-based abstractions.
This document provides context and rules for AI coding assistants.

## Project Overview

Gorums provides:

- Flexible quorum call abstractions for distributed systems
- Code generation via `protoc-gen-gorums` compiler plugin
- gRPC-based RPC communication
- Supported communication styles: unicast, multicast, quorumcall, async and correctable quorum calls

**Key Technologies:**

- Language: Go 1.25+
- Build: Make
- Protocol: Protocol Buffers (protobuf)
- Testing: Go testing framework
- Code Generation: Custom protoc plugin

## Repository Structure

```text
gorums/
├── cmd/protoc-gen-gorums/     # Compiler plugin for code generation
│   ├── dev/                   # Static code + generated code examples
│   └── gengorums/             # Compiler logic + templates
├── benchmark/                 # Benchmarking code
├── examples/                  # Example implementations
├── internal/                  # Internal packages
├── doc/                       # Documentation
└── *.go                       # Core library files
```

## Development Rules

### General Guidelines

- For larger features and refactors, prepare a plan before coding
- STOP and ASK if unsure about design decisions
- ALWAYS write tests for new features and bug fixes
- Large changes must be broken into small, manageable units to be committed separately
- NEVER make unrelated changes in the same commit (e.g., code + documentation + formatting)
- Instead, if you discover a bug or issue while working, document the issue in a separate file `doc/issue-*.md`
- COMMIT messages must follow conventional commit style (at most 75 characters wide), must be human readable plain text and easily copyable, and must not contain any markdown links or formatting.

### Code Generation Workflow

**NEVER directly edit files prefixed with `zorums_*_gorums.pb.go` in `cmd/protoc-gen-gorums/dev/`**

These files are generated from templates. Instead:

1. **For Template Changes:**
   - Edit template in `cmd/protoc-gen-gorums/gengorums/template_*.go`
   - Run `make dev` to regenerate `zorums_*_gorums.pb.go` files

2. **For Static Code Changes:**
   - Edit files in `cmd/protoc-gen-gorums/dev/` that are NOT prefixed with `zorums_*`
   - Run `make dev` to bundle changes into `template_static.go`

3. **After Any Template or Static Code Changes:**
   - Run `make dev` to regenerate `zorums_*_gorums.pb.go` files
   - Or run `make genproto` to regenerate all _gorums.pb.go files

### Testing Requirements

- Use testing utilities in `testing_shared.go` for common test setup
- If the provided testing utilities are insufficient, create new ones in `testing_shared.go` and document their usage
- Always write table-driven tests when same logic needs to be tested with multiple inputs
- Organize related tests using subtests
- Test names should be capitalized, like TestFileNameFeatureName, e.g., TestQuorumCallFeatureName, for some feature in `quorumcall_test.go`
- Run relevant tests after each change
- NEVER delete failing tests - fix the underlying issue - unless the test is no longer relevant
- NEVER skip tests or ignore failures
- NEVER use another testing framework than Go's testing package
- If addressing a test failure requires significant changes, stop and ask for guidance
- Test coverage should be comprehensive
- ALL tests must pass before considering work complete

### Testing Strategy

- Follow Test Driven Development (TDD) when adding features or fixing bugs:
  1. Write failing test
  2. Confirm test fails
  3. Write minimal code to pass test
  4. Confirm test passes
  5. Refactor if needed

### Code Style and Conventions

- **Match existing code style** - consistency within files is paramount
- **Follow Go conventions** - use `gofmt`, follow effective Go practices
- **Use Go's standard library**
  - use up-to-date standard library features when relevant
  - use recent versions of packages: slices, maps, sync, rand/v2
  - use for-range iterators with yield when applicable
  - use generics when appropriate
- **Use meaningful names** - reflect domain concepts, not implementation details
- **Preserve comments** unless they are demonstrably incorrect
- **Add documentation**
  - Each exported function, type, and method must have a clear comment explaining its purpose and usage following Go doc conventions
  - Non-exported functions, types, and methods should have comments if their purpose is not immediately clear
  - Each Go package (typically in doc.go) should have a comment block describing its purpose:

  ```go
  // Package gorums provides quorum call abstractions for distributed systems.
  package gorums
  ```

- **Update user/developer documentation** - whenever public APIs or behaviors change, update relevant documentation in `doc/`

### Git Workflow

- Main branch: `master`
- Work on feature branches
- Create new branches for significant changes, unless already working on a feature branch
- Feature branches should be named: `feature/short-description` or `fix/short-description`
- If there is an associated GitHub issue, include its ID in the branch name: `feature/123/short-description`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [relab/gorums](https://github.com/relab/gorums) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
