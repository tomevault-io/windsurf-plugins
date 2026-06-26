---
trigger: always_on
description: This project uses a team of specialised sub-agents to enforce security throughout the entire
---

# Secure SDLC — Multi-Agent Orchestration

This project uses a team of specialised sub-agents to enforce security throughout the entire
Software Development Lifecycle (SDLC). Each agent has a defined role, phase, and set of
responsibilities. The orchestrator (you, the main Claude Code session) coordinates them.

---

## Agent Roster

| Agent file | Role | Primary phases |
|---|---|---|
| `product-manager` | Secure requirements via ASVS | Plan |
| `grc-analyst` | Compliance, risk register, audit evidence | Plan → Release |
| `appsec-engineer` | Threat modelling, SAST/DAST, vuln triage | Design → Test |
| `cloud-platform-engineer` | IaC security, CSPM, secrets, hardening | Design → Release |
| `dev-lead` | Secure coding patterns, PR review, dependency review | Build → Test |
| `release-manager` | Security sign-off, go/no-go gate | Release |
| `security-champion` | First-line security Q&A and lightweight review | All phases |
| `ai-security-engineer` | AI/LLM feature security, prompt injection, agentic risks | Design → Test |

---

## Lifecycle Phases & Handoffs

### 1. PLAN
- Invoke `product-manager` to elicit and document security requirements mapped to ASVS levels.
- Invoke `grc-analyst` to produce the initial risk register and identify applicable compliance
  frameworks (SOC 2, ISO 27001, NIST CSF, PCI-DSS, etc.).
- Output: `docs/security-requirements.md`, `docs/risk-register.md`

### 2. DESIGN
- Invoke `appsec-engineer` to run a structured threat model (STRIDE or LINDDUN) against the
  proposed architecture.
- Invoke `cloud-platform-engineer` to review infrastructure design for misconfigurations,
  privilege escalation paths, and secrets handling.
- Invoke `grc-analyst` to map architecture decisions to compliance controls.
- Output: `docs/threat-model.md`, `docs/infra-security-review.md`

### 3. BUILD
- Invoke `dev-lead` on every pull request or significant code change to enforce secure coding
  standards and review dependencies (SCA).
- Invoke `appsec-engineer` to triage any SAST findings and provide remediation guidance.
- Invoke `cloud-platform-engineer` to validate IaC changes (Terraform, Helm, etc.) and
  check for exposed secrets.
- Output: inline PR comments, `docs/sast-findings.md`

### 4. TEST
- Invoke `appsec-engineer` to coordinate DAST, fuzz testing, and interpret penetration test
  findings.
- Invoke `dev-lead` to implement fixes for confirmed vulnerabilities and run security
  regression tests.
- Invoke `grc-analyst` to collect test evidence for audit artefacts.
- Output: `docs/test-security-report.md`, `docs/audit-evidence/`

### 5. RELEASE
- Invoke `release-manager` to execute the pre-release security checklist and issue a
  go/no-go decision.
- Invoke `grc-analyst` for final compliance attestation.
- Invoke `cloud-platform-engineer` to confirm production hardening (WAF, SIEM alerts,
  runtime protection) is in place.
- Output: `docs/release-security-sign-off.md`

---

## Orchestration Rules

1. **Never skip a phase gate.** Each phase produces artefacts that the next phase depends on.
 If a required artefact is missing, halt and request it before proceeding.

2. **Severity thresholds block progression:**
 - CRITICAL or HIGH unmitigated findings block the Build → Test and Test → Release gates.
 - MEDIUM findings must have an accepted risk or remediation plan before release.
 - LOW findings are tracked in the risk register.

3. **All findings are traceable.** Every vulnerability or risk identified by any agent must
 be recorded in `docs/risk-register.md` with an owner, severity, and status.

4. **ASVS is the requirements anchor.** The product-manager agent maps every security
 requirement to an ASVS control reference. All other agents reference these when providing
 guidance.

5. **Agents collaborate, not compete.** If two agents produce conflicting guidance (e.g.
 appsec-engineer and cloud-platform-engineer disagree on an approach), escalate to the
 orchestrator for resolution and document the decision.

6. **AI features require the ai-security-engineer.** Any feature that calls an LLM API,
 processes user input sent to a model, or uses agentic patterns MUST be reviewed by
 `ai-security-engineer` in addition to the standard AppSec review. Prompt injection,
 indirect prompt injection, and excessive agency are SDLC risks, not afterthoughts.

7. **Check `secure-sdlc.yaml` for project configuration.** If `secure-sdlc.yaml` exists
 in the project root, use it to determine the ASVS level, applicable compliance frameworks,
 and which CI gates are configured. If it doesn't exist, prompt the user to run
 `secure-sdlc init` or create it manually.

8. **Phase detection.** Before starting work, check which SDLC artefacts exist in `docs/`:
 - No artefacts → start with PLAN phase
 - Requirements + risk register exist → proceed to DESIGN
 - Threat model exists → proceed to BUILD
 - SAST findings documented → proceed to TEST
 - Test report exists → ready for RELEASE gate
 The command `secure-sdlc status` provides a visual summary.

---

## Quick-start Commands

```bash
# ── Zero-friction setup ────────────────────────────────────────────────────
# Install Secure SDLC in your project (docs, hooks, CI, config)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaademos/secure-sdlc-agents](https://github.com/Kaademos/secure-sdlc-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
