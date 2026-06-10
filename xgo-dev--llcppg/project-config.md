---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# llcppg - LLGo C/C++ Library Binding Generator

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap Dependencies and Build
llcppg requires LLGo (https://github.com/goplus/llgo) and specific LLVM version. Follow these exact steps:

- Install system dependencies (Ubuntu/Linux):
  ```bash
  sudo apt-get update
  sudo apt-get install -y llvm-19-dev clang-19 libclang-19-dev lld-19 libunwind-19-dev libc++-19-dev pkg-config libgc-dev libssl-dev zlib1g-dev libffi-dev libuv1-dev libcjson-dev
  echo "/usr/lib/llvm-19/bin" | sudo tee -a /etc/environment
  export PATH="/usr/lib/llvm-19/bin:$PATH"
  ```

- Install system dependencies (macOS):
  ```bash
  brew install llvm@19 bdw-gc openssl libffi libuv lld@19 zlib cjson
  brew link --force zlib libffi
  echo "$(brew --prefix llvm@19)/bin" >> $GITHUB_PATH
  echo "$(brew --prefix lld@19)/bin" >> $GITHUB_PATH
  ```

- Clone and install LLGo (CRITICAL dependency):
  ```bash
  git clone https://github.com/goplus/llgo.git .llgo
  cd .llgo
  git checkout 298669b02c41190b2be623fcacace56a01ced3d7
  go install -v ./cmd/llgo/...
  export LLGO_ROOT=$(pwd)
  cd ..
  ```

- Build llcppg components:
  ```bash
  go build -v ./...
  ```
  **Timing**: 15 seconds. Build always succeeds without LLGo dependencies.

### Install llcppg Tools
Execute install script for complete setup (**REQUIRED FOR ALL TESTING**):
```bash
bash ./install.sh
```
**NEVER CANCEL**: Installation takes 2-3 minutes including LLGo compilation. Set timeout to 10+ minutes.

This installs:
- `llcppcfg` - Configuration file generator (Go)
- `gogensig` - Go signature generator (Go) 
- `llcppg` - Main binding generator (Go)
- `llcppsymg` - Symbol generator (LLGo, CRITICAL)
- `llcppsigfetch` - Signature fetcher (LLGo, CRITICAL)

### Test the Installation
- Standard Go tests (partial functionality):
  ```bash
  go test -v ./config ./internal/name ./internal/arg ./internal/unmarshal
  ```
  **Timing**: 2 seconds. Always run these to verify basic functionality.

- LLGo-dependent tests (full functionality):
  ```bash
  llgo test ./_xtool/internal/...
  llgo test ./_xtool/llcppsigfetch/internal/...
  llgo test ./_xtool/llcppsymg/internal/...
  llgo test ./config ./internal/arg ./internal/name
  ```
  **NEVER CANCEL**: LLGo tests take 3-5 minutes. Set timeout to 10+ minutes.

- Full test suite:
  ```bash
  go test -v ./...
  ```
  **Timing**: 8-12 minutes. Some tests require LLGo tools to pass.

### Demo Validation
Test real functionality with demos:
```bash
bash .github/workflows/test_demo.sh
```
**NEVER CANCEL**: Demo tests take 5-10 minutes. Set timeout to 15+ minutes.

**VALIDATION SCENARIOS**: After making changes, always test a complete workflow:
1. Generate config: `llcppcfg cjson` 
2. Edit the generated `llcppg.cfg` to add proper includes: `["cJSON.h", "cJSON_Utils.h"]`
3. Run llcppg: `llcppg llcppg.cfg`
4. Verify generated Go bindings work: check that `.go` files are created
5. Test with demo: run any demo from `_demo/` or `_llcppgtest/` directories

## Usage

### Generate Configuration Files
Use llcppcfg to bootstrap configuration:
```bash
llcppcfg [options] libname
```
**Timing**: Instant (under 1 second).

Examples:
- `llcppcfg cjson` - generates basic cjson config
- `llcppcfg -cpp libname` - for C++ libraries
- `llcppcfg -deps "c/os,github.com/author/llpkg" libname` - with dependencies

### Generate Bindings
Process configuration file to generate LLGo bindings:
```bash
llcppg [config-file]
```
If config-file is not specified, uses `llcppg.cfg` in current directory.

**NEVER CANCEL**: Binding generation takes 30 seconds to 5 minutes depending on library size. Set timeout to 10+ minutes.

### Key Validation Steps
Always run before committing changes:
```bash
go fmt ./...
go vet ./...
go test -timeout=10m ./...
```
**NEVER CANCEL**: Full validation takes 8-12 minutes. Set timeout to 20+ minutes.

## Validation

### Manual Testing Requirements
**CRITICAL**: After code changes, always perform these validation steps:
1. Build all components: `go build -v ./...`
2. Install tools: `bash ./install.sh` (**ESSENTIAL FOR TESTING**) 
3. Generate a test config: `llcppcfg sqlite`
4. Edit config to add proper headers (check examples in `_llcppgtest/` directories)
5. Run binding generation: `llcppg llcppg.cfg`
6. Verify output Go files are generated correctly
7. Test with a known working example from `_demo/` or `_llcppgtest/`

### Build Requirements Checklist
- [ ] Go 1.23+ installed
- [ ] LLGo installed with exact commit: `298669b02c41190b2be623fcacace56a01ced3d7`
- [ ] LLVM 19 development libraries installed  
- [ ] System dependencies (libcjson-dev, libgc-dev, etc.) installed
- [ ] PATH includes LLVM 19 binary directory
- [ ] LLGO_ROOT environment variable set

### Continuous Integration
The `.github/workflows/` directory contains CI configurations:
- `go.yml` - Main test suite (8-15 minutes)
- `end2end.yml` - End-to-end validation (15-30 minutes)  
- `test_demo.sh` - Demo validation script

## Navigation

### Key Directories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xgo-dev/llcppg](https://github.com/xgo-dev/llcppg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
