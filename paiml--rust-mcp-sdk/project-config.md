---
trigger: always_on
description: We have ZERO tolerance for defects. Your "clippy warnings won't..." is a P0 problem.
---

# PMCP SDK Development Standards

## Toyota Way Quality System - ZERO TOLERANCE FOR DEFECTS

We have ZERO tolerance for defects. Your "clippy warnings won't..." is a P0 problem.

## Quality Gate Enforcement

### Pre-Commit Quality Gates (MANDATORY)
**ALL commits are blocked until quality gates pass:**
- Pre-commit hook automatically runs Toyota Way quality checks
- Format checking: `cargo fmt --all -- --check`  
- Clippy analysis: Zero warnings allowed
- Build verification: Must compile successfully
- Doctest validation: All doctests must pass

**To commit code:**
```bash
make quality-gate  # Run before any commit
git add -A
git commit -m "message"  # Will be blocked if quality fails
```

### CI Quality Gates (PR-blocking, added Phase 75 Wave 5)

**PRs are blocked from merging if PMAT detects new cognitive-complexity violations.**

The check runs in `.github/workflows/ci.yml` `quality-gate` job:

```bash
pmat quality-gate --fail-on-violation --checks complexity
```

PMAT version is pinned to `3.15.0` (matches `.github/workflows/quality-badges.yml`; see Phase 75 Wave 0 Task 3 for rationale). The `gate` aggregate job lists `quality-gate` in its `needs:` array, so a PMAT failure propagates to the org-required `gate` status check and blocks merge.

**If your PR fails this check:**

1. Run locally to see which functions exceed cog 25:
   ```bash
   pmat analyze complexity --format json --max-cognitive 25 \
     | jq '.violations[] | select(.path | startswith("src/"))'
   ```
2. Apply one of the 6 refactor techniques (P1–P6) documented in `.planning/phases/75-fix-pmat-issues/75-RESEARCH.md` Architecture Patterns.
3. If the function is irreducibly complex (parser, AST walker, protocol dispatch), apply a `// Why:` annotated `#[allow(clippy::cognitive_complexity)]` per the template in `.planning/phases/75-fix-pmat-issues/75-00-PLAN.md`. Hard cap is cog 50 (D-03).
4. Re-push and the gate re-runs.

**DO NOT** disable, weaken, or remove this gate without explicit Phase-level approval — it is the mechanism that keeps the README "Quality Gate: passing" badge accurate.

Pre-commit `make quality-gate` covers fmt/clippy/build/test/audit but does **not** run PMAT (per Phase 75 D-07: PMAT runs only in CI to keep the dev loop fast).

### PMAT Quality-Gate Proxy Mode (REQUIRED DURING DEVELOPMENT)

**MANDATORY: Use pmat quality-gate proxy via MCP during development**

All code changes MUST go through pmat quality-gate proxy before writing:

```bash
# Start pmat MCP server with quality-gate proxy
pmat mcp-server --enable-quality-proxy

# In Claude Code, use quality_proxy MCP tool for all file operations:
# - write operations
# - edit operations  
# - append operations
```

**Quality Proxy Enforcement Modes:**
- **Strict Mode** (default): Reject code that doesn't meet quality standards
- **Advisory Mode**: Warn about quality issues but allow changes
- **Auto-Fix Mode**: Automatically refactor code to meet standards

**Quality Checks Applied:**
- Cognitive complexity limits (≤25 per function)
- Zero SATD (Self-Admitted Technical Debt) comments
- Comprehensive documentation requirements
- Lint violation prevention
- Automatic refactoring suggestions

## Task Management - PDMT Style

**MANDATORY: Use PDMT (Pragmatic Deterministic MCP Templating) for all todos**

### PDMT Todo Generation
Use the `pdmt_deterministic_todos` MCP tool for creating quality-enforced todo lists:

```bash
# Generate PDMT todos with quality enforcement
pdmt_deterministic_todos --requirement "implement feature X" --mode strict --coverage-target 80
```

**PDMT Todo Features:**
- **Quality Gates Built-in**: Each todo includes validation commands
- **Success Criteria**: Clear, measurable completion requirements  
- **Test Coverage**: Enforce 80%+ coverage targets
- **Zero SATD**: No technical debt tolerance
- **Complexity Limits**: Automatic complexity validation
- **Documentation**: Comprehensive docs required

### PDMT Todo Structure
```
## Todo: [ID] Implementation Task
**Quality Gate**: `cargo test --coverage && cargo clippy`
**Success Criteria**: 
- [ ] Feature implemented with 80%+ test coverage
- [ ] Zero clippy warnings
- [ ] Comprehensive documentation with examples
- [ ] Property tests included
- [ ] Integration tests passing
**Validation Command**: `make quality-gate && make test-coverage`
```

## Development Workflow (MANDATORY)

### 1. Planning Phase
- Use `pdmt_deterministic_todos` for task breakdown
- Set quality targets: 80%+ coverage, zero SATD, complexity ≤25

### 2. Development Phase  
- **ALL code changes via pmat quality-gate proxy**
- Use MCP `quality_proxy` tool for file operations
- Continuous quality validation during development

### 3. Pre-Commit Phase
- Pre-commit hook enforces Toyota Way quality gates
- **Cannot commit** without passing all quality checks
- Zero tolerance: formatting, clippy, build, tests

### 4. CI/CD Phase
- Tests run with `--test-threads=1` (race condition prevention)
- Full quality gate validation
- Documentation coverage verification

## ALWAYS Requirements for New Features (MANDATORY)

**Every new feature MUST include ALL of the following - NO EXCEPTIONS:**

### 1. FUZZ Testing (ALWAYS REQUIRED)
```bash
# Property-based fuzzing for robustness
cargo fuzz run fuzz_target_name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paiml/rust-mcp-sdk](https://github.com/paiml/rust-mcp-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
