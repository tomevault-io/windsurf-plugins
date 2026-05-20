---
trigger: always_on
description: Makefile best practices for developer workflows: Apply when working with Makefiles or build automation
---

# Makefile Best Practices for Developer Workflows

## Overview

When working with Makefiles, follow these best practices to create maintainable, reliable, and developer-friendly build automation. Makefiles should be structured for clarity, correctness, and ease of use.

## Core Principles

1. **Embrace the Filesystem**: `make` is fundamentally a tool for building files. Rules should correspond to actual files on the filesystem to leverage dependency tracking.
2. **Be Explicit and Predictable**: Configure Makefiles to have consistent behavior and avoid built-in rules that can cause surprises.
3. **Prioritize Clarity**: Use namespacing, includes, and self-documenting targets.
4. **Write Small, Focused Recipes**: Delegate complex logic to external scripts.
5. **Ensure Compatibility**: Write Makefiles that work across different Make versions and platforms.
6. **Test Thoroughly**: Always test your Makefile targets on the actual target platforms.

## Compatibility Considerations

### Make Version Compatibility

**CRITICAL**: Always ensure your Makefile works with commonly available Make versions:

- **macOS**: Ships with GNU Make 3.81 (from 2006) by default
- **Linux**: Usually has GNU Make 4.0+ 
- **Windows**: Varies widely depending on installation

### Checking Make Version

Before using advanced features, check the Make version:

```bash
make --version
```

For GNU Make 4.0+ features, consider providing fallbacks or clear error messages.

### Common Compatibility Issues

1. **`.RECIPEPREFIX` Support**: Only available in GNU Make 4.0+
2. **Some built-in functions**: Newer versions have more features
3. **Shell behavior**: Different platforms may have different default shells

## Required Preamble

Every Makefile MUST start with this compatibility-focused preamble:

```makefile
# Use bash as the shell with strict mode
SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c

# Run all recipe lines in a single shell instance
.ONESHELL:

# If a rule fails, delete its target file
.DELETE_ON_ERROR:

# Disable make's built-in rules and suffix rules
MAKEFLAGS += --no-builtin-rules
.SUFFIXES:
```

### Advanced Features (Optional)

For teams using modern Make versions, you can optionally add:

```makefile
# Optional: Set custom recipe prefix (requires GNU Make 4.0+)
# Only use if your team standardizes on modern Make versions
ifeq ($(shell make --version 2>/dev/null | head -1 | cut -d' ' -f3 | cut -d'.' -f1),4)
  .RECIPEPREFIX = >
endif
```

**Recommendation**: Use traditional tabs for maximum compatibility unless you have specific requirements for alternative prefixes.

## Recipe Formatting Guidelines

### Use Traditional Tabs (Recommended)

For maximum compatibility, use traditional tab characters for recipe indentation:

```makefile
clean:
	@echo "🧹 Cleaning up..."
	rm -rf build/ tmp/
.PHONY: clean

test:
	@echo "🧪 Running tests..."
	pytest
.PHONY: test
```

### Alternative: Custom Recipe Prefix (Advanced)

Only use custom recipe prefixes if you're certain all team members use GNU Make 4.0+:

```makefile
# Only if you've verified Make 4.0+ usage across your team
.RECIPEPREFIX = >

clean:
> @echo "🧹 Cleaning up..."
> rm -rf build/ tmp/
.PHONY: clean
```

## Structure and Organization

### Target Namespacing

- Use `/` as namespace delimiter for clarity
- Organize related targets under common namespaces

```makefile
# Good: Namespaced structure
lint/python:
	@echo "Linting Python code..."
	ruff check .

format/python:
	@echo "Formatting Python code..."
	ruff format .

docker/build:
	docker build -t myapp .

docker/run:
	docker run -p 8080:8080 myapp

test/unit:
	pytest tests/unit/

test/integration:
	pytest tests/integration/
```

### File Organization

For larger projects, split Makefiles using includes:

```makefile
# In root Makefile
-include make/*.mk
```

Create separate files like:
- `make/lint.mk` - Linting and formatting targets
- `make/docker.mk` - Container-related targets
- `make/test.mk` - Testing targets

### Standard Target Names

Always provide these conventional targets:
- `all`: Default target that builds main artifacts
- `install`: Install the application
- `test`: Run all tests
- `clean`: Remove build artifacts and temporary files
- `help`: Display available targets and descriptions

## Rule Writing Guidelines

### Use .PHONY Correctly

- Mark targets as `.PHONY` if they don't create a file with the target name
- Place `.PHONY` declarations immediately after the target definition

```makefile
clean:
	rm -rf build/ tmp/
.PHONY: clean

test:
	pytest
.PHONY: test
```

### File Targets Must Create Exact Files

When a rule creates a file, the target name must be the exact file path:

```makefile
build/app: main.c utils.c
	mkdir -p $(@D)
	gcc -o $@ $^
```

### Use Sentinel Files for Abstract Outputs

For tasks that don't produce a single obvious file, use sentinel files:

```makefile
# Track when tests last passed
SRC_FILES := $(shell find src -type f -name "*.py")

tmp/.tests-passed: $(SRC_FILES)
	mkdir -p $(@D)
	pytest
	touch $@

test: tmp/.tests-passed
.PHONY: test
```

### Delegate Complex Logic to Scripts

Keep Makefile recipes simple. Move complex logic to external scripts:

```makefile
# Bad: Complex inline script
deploy/staging:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [biokraft/my-cursor-framework](https://github.com/biokraft/my-cursor-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
