---
trigger: always_on
description: This document provides guidelines and best practices for AI agents contributing to the Antrea
---

# AI Agents Development Guidelines

This document provides guidelines and best practices for AI agents contributing to the Antrea
project. Antrea is a Kubernetes networking solution that leverages Open vSwitch as the networking
data plane to provide networking and security services for Kubernetes clusters.

## Table of Contents

- [Code Quality and Style](#code-quality-and-style)
- [Go Coding Conventions](#go-coding-conventions)
- [Bash Coding Conventions](#bash-coding-conventions)
- [Logging Guidelines](#logging-guidelines)
- [Git and Commit Guidelines](#git-and-commit-guidelines)
- [File Management](#file-management)
- [Copyright and Licensing](#copyright-and-licensing)
- [Testing](#testing)
- [Documentation](#documentation)

## Code Quality and Style

### General Principles

- Write clear, maintainable, and idiomatic Go code
- Follow established patterns and conventions within the codebase
- Prioritize code readability and simplicity
- Ensure all code is properly tested
- Document complex logic and design decisions

### Code Formatting

- **No trailing whitespaces**: Never introduce trailing whitespaces in any files, whether it is code or documentation
- **File endings**: Always end files with a newline character
- **Consistent indentation**: Match the existing code style in the file/package
- **Line length**: Keep lines reasonably short (typically under 120 characters for code and under
  100 characters for documentation)

## Go Coding Conventions

Follow the Go coding conventions as outlined in the [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)
and additional project-specific guidelines:

### Key Conventions

1. **Use `gofmt`**: Run `gofmt` on all Go code to ensure consistent formatting
2. **Import organization**: Use `goimports` to automatically manage imports
3. **Variable naming**:
   - Use short names for local variables with limited scope
   - Use descriptive names for variables with broader scope
   - Follow Go naming conventions (camelCase for unexported, PascalCase for exported)
4. **Error handling**: Always handle errors appropriately, never ignore them with `_`
5. **Comments**:
   - Write complete sentences for documentation comments
   - Start comments with the name of the item being described
   - End comments with a period
6. **Interfaces**: Define interfaces at the point of use, not implementation
7. **Context**: Pass `context.Context` as the first parameter to functions that need it

### Specific Guidelines

- **Receiver names**: Use consistent, short receiver names (typically 1-2 letters)
- **Package names**: Use short, lowercase names without underscores
- **Error strings**: Don't capitalize error strings or end them with punctuation
- **Empty slices**: Prefer `var t []string` over `t := []string{}`

## Bash Coding Conventions

### Bash Script Guidelines

- **Shebang**: For all Bash scripts, prefer `#!/usr/bin/env bash` over alternatives like `#!/bin/bash` for better portability
- **Error handling**: Use `set -o errexit` and `set -o pipefail` to catch errors early
- **Variable quoting**: Always quote variables to prevent word splitting and pathname expansion
- **Function naming**: Use lowercase with underscores for function names
- **Comments**: Include clear comments explaining complex logic or non-obvious behavior

## Logging Guidelines

Antrea uses structured logging with the `k8s.io/klog/v2` module. Follow these guidelines:

### Logging Levels

- **Use `klog.InfoS()` or `klog.Info()`**: For general informational messages
- **Use `klog.ErrorS()` or `klog.Error()`**: For error conditions
- **DO NOT use `klog.Warning()` or `klog.Warningf()`**: These are deprecated

### Structured Logging

- Prefer structured logging with key-value pairs
- Use consistent key names across the codebase
- Follow Kubernetes logging guidelines from the [community documentation](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-instrumentation/logging.md)

### Examples

```go
// Good: Structured logging
klog.InfoS("Processing network policy", "policy", policyName, "namespace", namespace)

// Good: Simple info logging
klog.Info("Starting Antrea agent")

// Good: Error with context
klog.ErrorS(err, "Failed to create OVS bridge", "bridge", bridgeName)

// Bad: Using deprecated Warning
klog.Warning("This is deprecated") // Don't do this
```

## Git and Commit Guidelines

Follow the guidelines from [How to Write a Git Commit Message](https://cbea.ms/git-commit/) for all commits:

### Commit Message Format

1. **Subject line**:
   - Limit to 50 characters
   - Use imperative mood ("Add feature" not "Added feature")
   - Don't end with a period
   - Capitalize the first letter

2. **Body** (if needed):
   - Wrap at 72 characters
   - Explain what and why, not how
   - Separate from subject with a blank line

3. **Examples**:

   ```text
   Fix memory leak in flow exporter
   
   The connection tracking module was not properly releasing
   memory when flows expired, causing gradual memory growth
   in long-running agents.
   ```

### Commit Requirements

- **All commits must be signed**: Use `git commit -s` to sign your commits
- **Clear descriptions**: Write clear, concise commit messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antrea-io/antrea](https://github.com/antrea-io/antrea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
