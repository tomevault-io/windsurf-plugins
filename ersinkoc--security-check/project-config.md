---
trigger: always_on
description: When the user says any of the following, begin a full security scan:
---

# Security Check - Master Orchestration for Claude Code

## Scan Triggers

When the user says any of the following, begin a full security scan:
- "run security check"
- "scan for vulnerabilities"
- "security audit"
- "security scan"
- "check security"
- "find vulnerabilities"
- "pentest this codebase"
- "threat analysis"
- "run all security skills"

When the user says any of the following, begin a diff-mode scan:
- "scan diff"
- "scan changes"
- "PR scan"
- "scan my PR"
- "check changes for security"
- "security review PR"
- "scan staged changes"
- "diff security check"

---

## Full Scan Pipeline (4 Phases)

### Step 0: Pre-Check

Before starting any scan:

1. Check if a `security-report/` folder already exists in the project root.
2. If it exists, ask the user:
   - "A previous security report exists. Should I archive it (rename to security-report-YYYY-MM-DD/) and start fresh, or overwrite it?"
3. If it does not exist, create `security-report/` and proceed.
4. Create `security-report/.scan-state.json` to track progress:
   ```json
   {
     "scan_id": "<uuid>",
     "started_at": "<ISO-8601>",
     "status": "in-progress",
     "current_phase": 0,
     "completed_skills": [],
     "failed_skills": [],
     "detected_languages": [],
     "findings_count": 0
   }
   ```

### Step 1: Reconnaissance

Run these skills sequentially, as later skills depend on recon output:

1. **sc-recon** (`.claude/skills/sc-recon/SKILL.md`)
   - Discovers technology stack, architecture, entry points, data flows
   - Produces: `security-report/architecture.md`
   - Extracts detected languages list for Step 2

2. **sc-dependency-audit** (`.claude/skills/sc-dependency-audit/SKILL.md`)
   - Analyzes lock files, supply chain risks, known CVE patterns
   - Produces: `security-report/dependency-audit.md`

After Step 1 completes, update `.scan-state.json` with detected languages and completed skills.

### Step 2: Vulnerability Hunting

Run vulnerability scanning skills **in parallel** based on detected languages and frameworks from Step 1.

#### Language-Specific Skills (run if language detected):

| Language | Skill | File |
|----------|-------|------|
| JavaScript/TypeScript | sc-lang-typescript | `.claude/skills/sc-lang-typescript/SKILL.md` |
| Python | sc-lang-python | `.claude/skills/sc-lang-python/SKILL.md` |
| Go | sc-lang-go | `.claude/skills/sc-lang-go/SKILL.md` |
| Rust | sc-lang-rust | `.claude/skills/sc-lang-rust/SKILL.md` |
| Java/Kotlin | sc-lang-java | `.claude/skills/sc-lang-java/SKILL.md` |
| PHP | sc-lang-php | `.claude/skills/sc-lang-php/SKILL.md` |
| C#/.NET | sc-lang-csharp | `.claude/skills/sc-lang-csharp/SKILL.md` |

#### Category Skills — Injection (always run):

| Category | Skill | File |
|----------|-------|------|
| SQL Injection | sc-sqli | `.claude/skills/sc-sqli/SKILL.md` |
| NoSQL Injection | sc-nosqli | `.claude/skills/sc-nosqli/SKILL.md` |
| GraphQL Injection | sc-graphql | `.claude/skills/sc-graphql/SKILL.md` |
| XSS | sc-xss | `.claude/skills/sc-xss/SKILL.md` |
| SSTI | sc-ssti | `.claude/skills/sc-ssti/SKILL.md` |
| XXE | sc-xxe | `.claude/skills/sc-xxe/SKILL.md` |
| LDAP Injection | sc-ldap | `.claude/skills/sc-ldap/SKILL.md` |
| Command Injection | sc-cmdi | `.claude/skills/sc-cmdi/SKILL.md` |
| Header Injection | sc-header-injection | `.claude/skills/sc-header-injection/SKILL.md` |

#### Category Skills — Code Execution:

| Category | Skill | File |
|----------|-------|------|
| Remote Code Execution | sc-rce | `.claude/skills/sc-rce/SKILL.md` |
| Deserialization | sc-deserialization | `.claude/skills/sc-deserialization/SKILL.md` |

#### Category Skills — Access Control:

| Category | Skill | File |
|----------|-------|------|
| Authentication | sc-auth | `.claude/skills/sc-auth/SKILL.md` |
| Authorization / IDOR | sc-authz | `.claude/skills/sc-authz/SKILL.md` |
| Privilege Escalation | sc-privilege-escalation | `.claude/skills/sc-privilege-escalation/SKILL.md` |
| Session Security | sc-session | `.claude/skills/sc-session/SKILL.md` |

#### Category Skills — Data Exposure:

| Category | Skill | File |
|----------|-------|------|
| Hardcoded Secrets | sc-secrets | `.claude/skills/sc-secrets/SKILL.md` |
| Data Exposure | sc-data-exposure | `.claude/skills/sc-data-exposure/SKILL.md` |
| Weak Cryptography | sc-crypto | `.claude/skills/sc-crypto/SKILL.md` |

#### Category Skills — Server-Side:

| Category | Skill | File |
|----------|-------|------|
| SSRF | sc-ssrf | `.claude/skills/sc-ssrf/SKILL.md` |
| Path Traversal | sc-path-traversal | `.claude/skills/sc-path-traversal/SKILL.md` |
| File Upload | sc-file-upload | `.claude/skills/sc-file-upload/SKILL.md` |
| Open Redirect | sc-open-redirect | `.claude/skills/sc-open-redirect/SKILL.md` |

#### Category Skills — Client-Side:

| Category | Skill | File |
|----------|-------|------|
| CSRF | sc-csrf | `.claude/skills/sc-csrf/SKILL.md` |
| CORS Misconfiguration | sc-cors | `.claude/skills/sc-cors/SKILL.md` |
| Clickjacking | sc-clickjacking | `.claude/skills/sc-clickjacking/SKILL.md` |
| WebSocket Security | sc-websocket | `.claude/skills/sc-websocket/SKILL.md` |

#### Category Skills — Logic & Design:

| Category | Skill | File |
|----------|-------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ersinkoc/security-check](https://github.com/ersinkoc/security-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
