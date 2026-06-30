---
trigger: always_on
description: **Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# g - Enhanced ls Alternative

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

g is a feature-rich, customizable, and cross-platform `ls` alternative written in Go. It provides enhanced file listing with icons, Git integration, multiple layout options, and advanced sorting capabilities.

## Working Effectively

### Prerequisites and Setup
- Requires Go version >= 1.26.1 (project uses Go 1.26.1)
- Works on Linux, Windows, and macOS
- Repository uses `go mod` for dependency management

### Bootstrap and Build Commands
**ALWAYS run these commands in sequence for a fresh setup:**

```bash
# 1. Verify Go version (must be >= 1.26.1)
go version

# 2. Download dependencies and build (NEVER CANCEL: first build takes ~20 seconds)
time go build -v .
```

**Build timing expectations:**
- **NEVER CANCEL**: Initial build from scratch: ~12 seconds. Set timeout to 60+ seconds.
- **NEVER CANCEL**: Subsequent builds: ~1.5 seconds. Set timeout to 30+ seconds.
- **NEVER CANCEL**: Tests: ~11 seconds. Set timeout to 30+ seconds.
- **NEVER CANCEL**: Linting: ~60 seconds. Set timeout to 180+ seconds.

### Build Variants
The project supports multiple build configurations using Go build tags:

```bash
# Lite build (minimal dependencies, 7.4MB binary)
CGO_ENABLED=0 go build -ldflags="-s -w" -o g-lite .

# Full build (all features, 8.1MB binary)  
CGO_ENABLED=0 go build -ldflags="-s -w" -tags="fuzzy mounts" -o g-full .

# Fuzzy search only
CGO_ENABLED=0 go build -ldflags="-s -w" -tags="fuzzy" -o g-fuzzy .

# Mounts support only
CGO_ENABLED=0 go build -ldflags="-s -w" -tags="mounts" -o g-mounts .
```

**Build tags:**
- `fuzzy`: Enables fuzzy search and path indexing (~500KB size impact)
- `mounts`: Enables mount point detection (~200KB size impact)

### Testing
```bash
# Run all tests (NEVER CANCEL: takes ~11 seconds)
time go test -v ./...
```

### Code Quality and CI Validation
**Always run these before committing changes:**

```bash
# Install formatting tool
go install mvdan.cc/gofumpt@latest

# Install linter (NEVER CANCEL: takes ~30 seconds to install)
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Format code (required by CI)
export PATH=$PATH:~/go/bin
gofumpt -l -extra .  # Check formatting
gofumpt -w -extra .  # Fix formatting

# Lint code (NEVER CANCEL: takes ~60 seconds)
golangci-lint run ./... --timeout=3m

# Run single test file/package
go test -v ./internal/cli/                    # Test specific package
go test -v ./internal/cli/ -run TestDive      # Test specific function
go test -bench=. ./internal/cli/              # Run benchmarks
go test -bench=BenchmarkDive ./internal/cli/  # Run specific benchmark
```

**CI Pipeline:** The project has three GitHub workflows that must pass:
- `go.yml`: Multi-platform builds and tests (Linux, macOS, Windows)
- `gofumpt.yml`: Code formatting verification with gofumpt --extra
- `lint.yml`: Static code analysis with golangci-lint

**Performance Testing:** Use hyperfine for rigorous performance comparisons:
```bash
# Compare optimized vs baseline performance
hyperfine --warmup 10 --min-runs 200 \
  './g-baseline /large-test-dir' \
  './g-optimized /large-test-dir'
```

## Application Usage and Validation

### Basic Functionality Testing
**Always test these scenarios after making changes:**

```bash
# 1. Test basic listing
./g .

# 2. Test with icons and formatting
./g --icon --long .

# 3. Test tree view
./g --tree --icon .

# 4. Test Git integration (if in git repo)
./g --git --icon .

# 5. Test table format
./g --table --size --time .

# 6. Test JSON output
./g --json . | head -10
```

### Comprehensive Validation Scenario
**Always run this complete end-to-end test after making significant changes:**

```bash
# 1. Create test directory and files
mkdir -p /tmp/g-validation-test && cd /tmp/g-validation-test
mkdir -p subdir
echo "test content" > file1.txt  
echo "another test" > subdir/file2.txt

# 2. Test core functionality with actual files
/path/to/g --tree --icon --size .  # Should show tree structure with sizes
/path/to/g --table --time .         # Should show tabular output with timestamps
/path/to/g --json . | jq '.'        # Should produce valid JSON
/path/to/g --recurse .              # Should list all files recursively

# 3. Verify output contains expected elements
# - Icons should be displayed if terminal supports them
# - File sizes should be human-readable (B, KiB, etc.)
# - Tree structure should use box-drawing characters
# - JSON should be valid and parseable

# 4. Clean up
cd /tmp && rm -rf g-validation-test
```

### Advanced Feature Validation
```bash
# Test fuzzy search (requires fuzzy build tag)
./g-full --fuzzy pattern

# Test mount information (requires mounts build tag)  
./g-full --mounts .

# Test recursive listing
./g --recurse directory/

# Test shell integration generation
./g --init bash
./g --init zsh
./g --init fish
```

### Performance Expectations
- Basic listing: ~0.015 seconds
- Tree view: < 1 second for typical directories
- Recursive operations: varies by directory size

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Equationzhao/g](https://github.com/Equationzhao/g) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
