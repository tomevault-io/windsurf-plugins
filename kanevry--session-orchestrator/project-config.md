---
trigger: always_on
description: When the user types /discovery or requests quality analysis, code audit, or issue detection
---


# Discovery — Quality Analysis & Issue Detection

Systematic quality discovery that runs modular probes adapted to the project's tech stack, presents findings interactively, and creates VCS issues for confirmed problems.

## Invocation

- **Standalone** (`/discovery [scope]`): Full flow with interactive triage (Phases 0-5 + Phase 6 stats)
- **Embedded** (from session-end when `discovery-on-close: true`): Phases 0-3 only, returns structured JSON to caller

Scope accepts: `all` (default), `code`, `infra`, `ui`, `arch`, `session`, or comma-separated like `code,session`.

## Phase 0: Read Session Config

Read Session Config from CLAUDE.md. Parse discovery-specific fields:
- `discovery-probes`, `discovery-exclude-paths`, `discovery-severity-threshold`, `discovery-confidence-threshold`, `discovery-on-close`

## Phase 1: Stack Detection

Detect the project's tech stack via marker file checks (Glob):

| Marker File(s) | Activates |
|----------------|-----------|
| `package.json` | JS/TS probes |
| `tsconfig.json` | TypeScript probes |
| `requirements.txt` / `pyproject.toml` | Python probes |
| `Dockerfile` / `docker-compose.yml` | Container probes |
| `.github/workflows/` | GitHub CI probes |
| `.gitlab-ci.yml` | GitLab CI probes |
| `next.config.*` / `nuxt.config.*` | SSR probes |
| `tailwind.config.*` | Tailwind probes |
| `.orchestrator/` exists | Session probes |

Build activation set: start with marker-matched probes → intersect `discovery-probes` config (if set) → restrict to scope argument (if passed).

Default exclude paths (always apply): `node_modules/`, `.git/`, `dist/`, `build/`, `.next/`, `.nuxt/`, `coverage/`. Add paths from `discovery-exclude-paths`.

Report: "Discovery: [N] probes active across [categories]. Stack: [detected]. Threshold: [severity]."

## Phase 2: Probe Execution

Run probes **sequentially** — Cursor has no parallel agents. One category at a time. Complete each category's analysis before moving to the next.

### 6 Probe Categories

**Code probes** (any project with source files):
- `hardcoded-values`, `orphaned-annotations`, `dead-code`, `ai-slop`, `type-safety-gaps`, `test-coverage-gaps`, `test-anti-patterns`, `security-basics`

**Infra probes** (when CI/Docker markers found):
- CI configuration issues, Dockerfile anti-patterns

**UI probes** (when frontend frameworks detected):
- Accessibility gaps, component anti-patterns

**Arch probes** (any project):
- Circular dependencies, complexity hotspots, deep nesting

**Session probes** (when `.orchestrator/` exists):
- Session metric anomalies, recurring failures, stale learnings

For each probe match, record a finding in this exact format:
```
FINDING:
  probe: <probe_name>
  category: <category>
  severity: <critical|high|medium|low>
  file_path: <absolute path>
  line_number: <number>
  matched_text: <exact text from tool output>
  title: <short title>
  description: <1-2 sentence description>
  recommended_fix: <concrete fix suggestion>
```

If a probe's activation condition is not met, skip with note. If a probe command fails, skip gracefully and continue.

**CRITICAL**: Do NOT fabricate findings. Only report what tool output confirms.

### Key Detection Patterns

| Probe | Severity | Pattern |
|-------|----------|---------|
| Hardcoded secrets | Critical | `(password\|api_key\|secret\|token)\s*[:=]\s*["'][^"']+["']` (exclude test/env/fixtures) |
| eval usage | High | `eval\s*\(` |
| XSS (React) | High | `dangerouslySetInnerHTML` |
| SQL injection | High | `` `[^`]*SELECT[^`]*\$\{`` |
| any type | Medium | `:\s*any\b` or `as\s+any\b` |
| TS suppression | Medium | `@ts-ignore` |
| AI filler | Medium | `(as you can see\|it's worth noting\|needless to say)` |

## Phase 3: Verification & Scoring

### 3.1 Verify Each Finding

For EACH finding: read the file at `file_path:line_number`. Confirm `matched_text` appears at or near that line (+/-3 lines tolerance). If NOT confirmed, discard as false positive.

Report: "Verification: N confirmed, M discarded as false positives"

### 3.2 Confidence Scoring

Start at 40 baseline, add factor scores, clamp to 0-100. Critical severity findings always get minimum 70.

| Factor | Low (+0) | Medium (+10) | High (+20) |
|--------|----------|-------------|------------|
| Pattern specificity | Generic (URL, TODO) | Moderate (orphaned annotation) | Specific (API key regex, eval()) |
| File context | Test/example/docs | Utility/config/scripts | Production source/API handler |
| Historical signal | Previously dismissed | No prior data | Recurring issue |

Read `discovery-confidence-threshold` from config (default: 60).

### 3.3 Deduplication

Same `file_path` + overlapping line range (+/-5 lines) + different probes = duplicate. Keep higher severity finding.

### 3.4 Apply Thresholds

Remove findings below `discovery-severity-threshold` and below `discovery-confidence-threshold`. Log auto-dismissed count.

### 3.5 Embedded Mode Exit

If in embedded mode (called from session-end): STOP HERE. Return this JSON schema:

```json
{
  "findings": [
    {"probe": "string", "category": "string", "severity": "critical|high|medium|low",
     "confidence": 0, "file": "string", "line": 0, "description": "string", "recommendation": "string"}
  ],
  "stats": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kanevry/session-orchestrator](https://github.com/Kanevry/session-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
