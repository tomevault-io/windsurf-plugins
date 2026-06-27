---
trigger: always_on
description: \"CI pipeline setup with pre-built templates and local validation. Generates GitHub Actions workflows, validates YAML syntax and permissions, supports dry-run via act/gh. The CI equivalent of wire-observability.\
---



# Wire CI

> **HARD GATE** — Do not ship a project without CI. Run this skill before first merge to main or when adding CI to an existing project.
>
> **HARD GATE** — CI that is untestable locally will break every cycle. Always run `--validate` after generating workflows and `--dry-run` before pushing.

Generate, validate, and test CI workflows. Detects your project type, produces platform-appropriate GitHub Actions configurations, and provides local verification to catch auth, permissions, and syntax issues before they reach CI.

## What this sets up

1. **CI workflow** — `.github/workflows/ci.yaml` with test, lint, typecheck, build steps
2. **Release workflow** — `.github/workflows/release.yaml` with semantic-release (if applicable)
3. **`--validate` mode** — checks YAML syntax, workflow permissions, required secrets, and common pitfalls
4. **`--dry-run` mode** — runs workflows locally via `act` or `gh workflow run` to prove correctness before push
5. **Failure pattern documentation** — common CI failure categories and their fixes

## Process

### 1. Detect project type

Read the project root for manifest files to determine which template to use:

| Manifest | Type | Template |
|----------|------|----------|
| `Cargo.toml` | Rust | Rust CI: test, clippy, fmt, build |
| `package.json` | Node | Node CI: test, lint, typecheck, build |
| `setup.py` / `pyproject.toml` | Python | Python CI: pytest, ruff/mypy/flake8, build |
| `go.mod` | Go | Go CI: test, vet, staticcheck, build |
| `CMakeLists.txt` | C/C++ | C/C++ CI: cmake build, ctest |
| Multiple detected | Polyglot | Combined workflows or error if ambiguous |

If no manifest is found, prompt the user to specify the type or pass `--type <rust|node|python|go|cpp>`.

### 2. Generate CI workflow

Create `.github/workflows/ci.yaml` with standard steps derived from the project type and its manifest:

**Rust template (`Cargo.toml`):**
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions-rust/toolchain@v1
        with:
          toolchain: stable
          components: clippy, rustfmt
      - run: cargo fmt --all -- --check
      - run: cargo clippy -- -D warnings
      - run: cargo test
      - run: cargo build --release
```

**Node template (`package.json`):**
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm test
      - run: npm run lint 2>/dev/null || true
      - run: npm run typecheck 2>/dev/null || true
      - run: npm run build 2>/dev/null || true
```

**Python template (`setup.py` / `pyproject.toml`):**
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -e ".[dev]" || pip install -e .
      - run: pip install pytest ruff mypy
      - run: ruff check .
      - run: mypy . 2>/dev/null || true
      - run: pytest
```

**Go template (`go.mod`):**
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: stable
          cache: true
      - run: go vet ./...
      - run: go test ./...
      - run: go build ./...
```

**C/C++ template (`CMakeLists.txt`):**
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cmake -B build
      - run: cmake --build build
      - run: ctest --test-dir build
```

### 3. Generate release workflow (if semantic-release detected)

If the project has semantic-release configured (in `package.json`, `.releaserc`, or `release.config.js`), also generate `.github/workflows/release.yaml`:

```yaml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build 2>/dev/null || true
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: \${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: \${{ secrets.NPM_TOKEN }}
```

> **NPM_TOKEN is required** for publishing to npm. Without it, semantic-release will fail at the publish step. See `--validate` to check this.

### 4. Validate workflows (`--validate`)

Run `wire-ci --validate` to check all generated workflow files:

```bash
# Validate YAML syntax
for f in .github/workflows/*.yaml; do
  python3 -c "import yaml; yaml.safe_load(open('$f'))" || echo "FAIL: $f has YAML syntax errors"
done

# Check permissions block presence

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielvm-git/bigpowers](https://github.com/danielvm-git/bigpowers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
