---
trigger: always_on
description: **Read this first. Follow these 10 rules strictly.**
---

# AGENT RULES FOR CERBER V2.0 MVP

**Read this first. Follow these 10 rules strictly.**

**Last Updated:** January 12, 2026  
**Phase:** MVP Release (2.0.0-rc1)  
**Primary Source:** [ONE_TRUTH_MVP.md](../ONE_TRUTH_MVP.md)

---

## 1. SCOPE FREEZE (MANDATORY)

**❌ DO NOT:**
- Add features beyond MVP definition
- Refactor existing code
- Reorganize folder structures
- "Clean up" imports or naming

**✅ DO:**
- Wire existing components together
- Fix CLI speed issues
- Implement missing CLI commands
- Write tests for new code only

---

## 2. MVP DEFINITION (THE 5 ROUTES)

Cerber v2.0.0-rc1 must have these 5 components working:


```yaml
# ✅ CORRECT STRUCTURE
contractVersion: 1

# Tool execution config
tools:
  actionlint:
    enabled: true
    version: '1.6.27'  # Pin version
    args: ['-ignore', 'SC2086']  # Tool-specific args
  
  zizmor:
    enabled: true
    version: 'latest'

# Cerber rules (gating, severity mapping)
rules:
  github-actions:
    severity: error
    gate: true  # Fail CI if violated
  
  secrets:
    severity: warning
    gate: false  # Don't fail, just warn

# Profiles
profiles:
  solo:
    tools: [actionlint, gitleaks]
    failOn: [error]
  
  team:
    tools: [actionlint, zizmor, gitleaks]
    failOn: [error, warning]
```

### Violation Source

Every violation **MUST** have `source`:

```json
{
  "id": "action-syntax-error",
  "source": "actionlint",  // Tool name OR "cerber-core"
  "message": "...",
  "severity": "error"
}
```

---

## 4) Output Schema Rules

### Determinism Requirements

**CRITICAL:** Core output must be deterministic.

```json
{
  "contractVersion": 1,
  "deterministic": true,
  
  "summary": {
    "total": 5,
    "errors": 2,
    "warnings": 3
  },
  
  "violations": [
    // Sorted by: path, line, column, id, source
  ],
  
  "metadata": {
    "tools": {
      "actionlint": { "version": "1.6.27", "exitCode": 1 }
      // Sorted by key
    }
    // ❌ NO TIMESTAMP HERE (breaks determinism)
  },
  
  "runMetadata": {
    // ✅ Optional metadata (not part of deterministic core)
    "generatedAt": "2026-01-09T12:00:00Z",
    "executionTime": 1234,
    "cwd": "/path/to/repo"
  }
}
```

### Timestamp Rule

- **Deterministic core:** NO required timestamp
- **Optional metadata:** `runMetadata.generatedAt` is OK (but optional)
- **Test:** `deepEqual(run1.summary, run2.summary)` must pass

### Sorting Rule

All arrays in deterministic output **MUST** be sorted:

```typescript
violations.sort((a, b) => {
  if (a.path !== b.path) return a.path.localeCompare(b.path);
  if (a.line !== b.line) return a.line - b.line;
  if (a.column !== b.column) return a.column - b.column;
  if (a.id !== b.id) return a.id.localeCompare(b.id);
  return a.source.localeCompare(b.source);
});
```

---

## 5) Profile Rules

Profiles decide:

1. **Which tools run** (pre-commit → fast tools only)
2. **Which severities fail CI** (team → fail on warnings, solo → only errors)
3. **Time budget** (pre-commit < 5s, full scan unlimited)

### Built-in Profiles

```yaml
profiles:
  solo:
    description: Fast checks for solo dev
    tools: [actionlint]
    failOn: [error]
    timeout: 5000  # 5s
  
  dev:
    description: Pre-commit checks
    tools: [actionlint, gitleaks-staged]
    failOn: [error]
    timeout: 10000  # 10s
  
  team:
    description: Full CI checks
    tools: [actionlint, zizmor, gitleaks, hadolint]
    failOn: [error, warning]
    timeout: 60000  # 60s
```

### Profile Selection

```bash
# Explicit
cerber validate --profile=team

# Auto-detect from contract
cerber validate  # Uses contract.activeProfile

# Pre-commit (always fast)
cerber guard  # Uses dev profile
```

---

## 6) Exit Code Rules

Cerber **MUST** use consistent exit codes:

| Code | Meaning | When |
|------|---------|------|
| 0 | Success | No violations OR all below threshold |
| 1 | Validation failed | Violations above threshold |
| 2 | Configuration error | Invalid contract, missing required field |
| 3 | Tool error | Tool not found, tool crashed, timeout |

**Example:**
```typescript
if (hasConfigError) process.exit(2);
if (hasToolError && !options.continueOnError) process.exit(3);
if (hasBlockingViolations) process.exit(1);
process.exit(0);
```

---

## 7) Definition of Done (per PR)

Before merging, verify:

- [ ] Tests updated/added (unit OR fixture OR e2e)
- [ ] Docs updated if behavior changes (`CERBER.md`)
- [ ] Output schema unchanged OR versioned
- [ ] Deterministic ordering verified by snapshot test
- [ ] Windows compatibility preserved (`execa`, path handling with `/`)
- [ ] No drive-by refactors (one PR = one atomic change)
- [ ] CI green on all platforms (ubuntu, windows, macos)

---

## 8) Format Support Rules

### Required Formats

- `--format=text` (human-readable, default)
- `--format=json` (machine-readable, deterministic)

### Future Formats

- `--format=sarif` (GitHub Code Scanning integration)
- `--format=github` (GitHub Actions annotations)

### Implementation

```typescript
interface Reporter {
  format: 'text' | 'json' | 'sarif' | 'github';
  report(result: ValidationResult): string;
}
```

**Rule:** Tool adapters that support SARIF (e.g., zizmor) should expose it.

---

## 9) Orchestrator Rules

### Timeout per Tool

```typescript
const result = await adapter.run({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Agaslez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
