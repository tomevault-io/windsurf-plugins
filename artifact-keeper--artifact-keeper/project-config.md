---
trigger: always_on
description: Auto-generated from all feature plans. Last updated: 2026-01-14
---

# Artifact Keeper Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-01-14

## Active Technologies
- Rust 1.75+ (backend) + wasmtime 21.0+, wasmtime-wasi, wit-bindgen, git2, axum
- PostgreSQL (existing), filesystem for WASM binaries
- Rust 1.75+ + axum, sqlx, tokio, reqwest
- Rust 1.75+ + axum, serde, serde_json

## Project Structure

```text
src/
tests/
```

## Commands

### Fast CI (Tier 1) - Every Push/PR
```bash
# Backend lint and unit tests
cargo fmt --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace --lib
```

### Integration Tests (Tier 2) - Main & Release Branches Only
```bash
# Backend integration tests (requires PostgreSQL)
cargo test --workspace
```

### Full E2E Tests (Tier 3) - Release/Manual Only
```bash
# Run all E2E tests with default (smoke) profile
./scripts/run-e2e-tests.sh

# Run with specific profile
./scripts/run-e2e-tests.sh --profile all      # All native clients
./scripts/run-e2e-tests.sh --profile pypi     # PyPI only
./scripts/run-e2e-tests.sh --profile smoke    # Quick smoke tests (default)

# Include stress and failure tests
./scripts/run-e2e-tests.sh --stress --failure

# Run with test tag filter
./scripts/run-e2e-tests.sh --tag @smoke       # Only smoke-tagged tests
./scripts/run-e2e-tests.sh --tag @full        # Full test suite

# Cleanup after tests
./scripts/run-e2e-tests.sh --clean
```

### Native Client Tests
```bash
# Run individual native client tests
./scripts/native-tests/run-all.sh smoke   # PyPI, NPM, Cargo
./scripts/native-tests/run-all.sh all     # All 10 package formats
./scripts/native-tests/test-pypi.sh       # Individual test
```

### gRPC SBOM Tests
```bash
# Run SBOM JSON structure validation unit tests (no database required)
cargo test sbom_service::tests --lib

# Run gRPC integration tests (requires PostgreSQL at localhost:30432)
DATABASE_URL="postgresql://registry:registry@localhost:30432/artifact_registry" \
  cargo test --test grpc_sbom_tests -- --ignored

# Run gRPC E2E tests with grpcurl (requires backend running with gRPC on port 9090)
./scripts/native-tests/test-grpc-sbom.sh
```

### Dependency-Track Integration Tests
```bash
# Run Dependency-Track integration tests (requires docker compose up)
./scripts/native-tests/test-dependency-track.sh

# With API key for full tests
DEPENDENCY_TRACK_API_KEY=your-key ./scripts/native-tests/test-dependency-track.sh
```

### WASM Plugin E2E Tests
```bash
# Run all WASM plugin tests (requires backend running on port 8080)
./scripts/native-tests/test-wasm-plugins.sh

# Run individual test suites
./scripts/native-tests/test-wasm-plugins.sh git        # Git installation tests
./scripts/native-tests/test-wasm-plugins.sh lifecycle  # Enable/disable/uninstall
./scripts/native-tests/test-wasm-plugins.sh reload     # Hot-reload tests

# Run with custom API URL
API_URL=http://localhost:8080 ./scripts/native-tests/test-wasm-plugins.sh
```

### Stress and Failure Tests
```bash
# Stress tests (100 concurrent uploads)
./scripts/stress/run-concurrent-uploads.sh
./scripts/stress/validate-results.sh

# Failure injection tests
./scripts/failure/run-all.sh
./scripts/failure/test-server-crash.sh
./scripts/failure/test-db-disconnect.sh
./scripts/failure/test-storage-failure.sh
```

### GitHub Actions
```bash
# Manually trigger E2E workflow
gh workflow run e2e.yml -f profile=all -f include_stress=true
```

## Code Style

Rust 1.75+: Follow standard conventions

## Git & GitHub

### Branch Protection — NEVER push directly to main

All changes must go through pull requests:

1. **Create a feature branch** from main:
   ```bash
   git checkout main && git pull
   git checkout -b feat/short-description   # or fix/, chore/, docs/
   ```

2. **Make changes and commit** to the feature branch

3. **Push and create PR**:
   ```bash
   git push -u origin feat/short-description
   gh pr create --fill   # or with --title and --body
   ```

4. **Merge via GitHub** after CI passes (squash merge preferred)

Branch naming conventions:
- `feat/` — new features
- `fix/` — bug fixes
- `chore/` — maintenance, dependencies, CI
- `docs/` — documentation only

### Parallel Agent Work (shallow clones)

When dispatching multiple agents to work on separate features or fixes in parallel, use **shallow clones in `/tmp/`** instead of git worktrees. Worktrees share the `.git` directory and agents end up switching branches in the main worktree, corrupting each other's state.

**Pattern for each agent:**
```bash
WORK_DIR="/tmp/$(uuidgen)-artifact-keeper"
git clone --depth 50 --branch main git@github.com:artifact-keeper/artifact-keeper.git "$WORK_DIR"
cd "$WORK_DIR"
git checkout -b feat/issue-description
# ... make changes, run checks, commit, push, create PR ...
rm -rf "$WORK_DIR"
```

Each agent gets a fully isolated repo copy. No shared state, no branch conflicts, no rust-analyzer cross-contamination. The agent must do ALL work inside `$WORK_DIR` and never touch the primary working directory at `/Users/khan/ak/artifact-keeper`.

### Pre-push Quality Checklist

Every commit must pass these checks locally before pushing. Do NOT use "push and see if CI passes" as a strategy.

```bash
cargo fmt --check                                          # formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artifact-keeper/artifact-keeper](https://github.com/artifact-keeper/artifact-keeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
