---
trigger: always_on
description: Full ASPM + LLMSecOps Claude Code plugin. Delivers SAST, SCA (OSV + CISA KEV + function-level reachability), secrets, IaC, prompt-injection, MCP/agent-tool audit, auth/authZ deep analysis, attack chains, PoC generation, SBOM/PBOM/AI-BOM, SARIF ingest, and compliance attestation (NIST AI 600-1, OWASP ASVS, OWASP LLM Top 10, EU AI Act).
---

# agentic-security

Full ASPM + LLMSecOps Claude Code plugin. Delivers SAST, SCA (OSV + CISA KEV + function-level reachability), secrets, IaC, prompt-injection, MCP/agent-tool audit, auth/authZ deep analysis, attack chains, PoC generation, SBOM/PBOM/AI-BOM, SARIF ingest, and compliance attestation (NIST AI 600-1, OWASP ASVS, OWASP LLM Top 10, EU AI Act).

**Version:** 0.125.0  
**License:** PolyForm Internal Use 1.0.0  
**Author:** Ross Young <ross@clearcapabilities.com> / Clear Capabilities Inc.

**ICP focus:** vibecoder-first; pro is follow-on. See `docs/POSITIONING.md` for the in/out call.

---

## Repository layout

| Path | Purpose | Local CLAUDE.md? |
|------|---------|------------------|
| `scanner/` | Node.js scan engine (ESM, Node ≥ 24). Bundle at `dist/agentic-security.mjs`. | `scanner/CLAUDE.md` |
| `scanner/src/sast/` | SAST detector modules. 60+ files. Adding a rule? Read here. | `scanner/src/sast/CLAUDE.md` |
| `scanner/src/posture/` | Annotation pipeline + state stores. 90+ modules. | `scanner/src/posture/CLAUDE.md` |
| `scanner/src/dataflow/` | Layer-2 taint engine (k=1 monovariant return-taint; see local file for what is and isn't modelled). | `scanner/src/dataflow/CLAUDE.md` |
| `scanner/src/mcp/` | MCP server. Six tools; OWASP MCP Top 10 hardened. | `scanner/src/mcp/CLAUDE.md` |
| `scanner/src/ir/` | Layer-1 IR: Babel-based JS/TS; Python via stdlib `ast` subprocess (default, when python3 available) with regex fallback; `java-parser`-based Java. | `scanner/src/ir/CLAUDE.md` |
| `scanner/src/lsp/` | LSP server wrapping `runScan`. Ships with the JetBrains + Neovim plugins. |  |
| `scanner/src/llm-validator/` | Layer-3 LLM validator (opt-in via `AGENTIC_SECURITY_LLM_VALIDATE=1`). |  |
| `scanner/test/` | Node test runner suite. Scoped via `npm run test:{smoke,sast,posture,dataflow,mcp,report,lifecycle}` — see `scanner/CLAUDE.md`. |  |
| `bench/cve-replay/` | Real-CVE replay corpus + runner. 185 entries (3 regression + 182 capability), all `pre:TP post:TN`; target 500. Baseline-gated via `npm run bench:cve-replay:check` (`bench/cve-replay/CONTRIBUTING.md`). |  |
| `bench/owasp-benchmark-v1.2/`, `bench/sard-juliet-java/`, `bench/polyglot/` | External benches (gitignored, regenerated). |  |
| `commands/` | Slash-command markdown files. Primary dispatchers (`secure`, `find-and-fix-everything`, `scan`, `triage`, `fix`, `posture`, `compliance`, `supply`, `setup`, `labs`) plus standalone `ci` and `three-agent-review`. Every capability is a mode of a dispatcher; the legacy single-purpose aliases have been removed. |  |
| `agents/` | Sub-agent system prompts. Edit-capable agents follow `agents/_CONFINEMENT.md`. |  |
| `hooks/` | Claude Code hook scripts + `hooks.json`. |  |
| `scripts/` | Compliance + helper scripts + CI templates (`scripts/ci-templates/`). |  |
| `docs/POSITIONING.md` | ICP statement: vibecoder-first; pro follow-on. |  |
| `docs/HARNESS_ASSESSMENT_SPEC.md` | Six-domain rubric for scoring an AI agent harness (PRD-derived, versioned). |  |
| `docs/HARNESS_ASSESSMENT_EVIDENCE.md` + `docs/schemas/harness-evidence.schema.json` | Wire format a conforming harness must emit so it can be scored. |  |
| `ide/{jetbrains,nvim,vscode}/` | IDE distributions. |  |
| `.claude-plugin/` | Plugin manifest (`plugin.json`, `marketplace.json`). |  |
| `.claude/settings.json` | Team-committed Claude Code settings (read-deny rules for bundles + cached artifacts). |  |
| `.agentic-security/` | Runtime state (last scan, streak, rules override, hook throttle). |  |

---

## Build & test

```bash
cd scanner/
npm install
npm run build          # bundles dist/agentic-security.mjs via @vercel/ncc; emits a SHA-256 sidecar
npm test               # full CI gate (chains the scoped scripts below)
npm run test:smoke     # one-file fixture, fast
npm run test:sast      # SAST detector tests
npm run test:dataflow  # IR + taint engine + calibration + held-out eval
npm run test:mcp       # MCP server + audit log
npm run smoke          # bundle smoke: CLI vs vulnerable-js fixture
```

All scoped scripts are defined in `scanner/package.json`. Pick the one closest to what you touched; `scanner/CLAUDE.md` documents which test files are in which scope.

After any change to `scanner/src/` or `scanner/bin/`, run `npm run build` before relying on the bundle. Unit tests run against `src/` directly and do not require a rebuild.

---

## Verification discipline (read before you claim anything works)

Several releases (v0.106.0–v0.107.1) shipped broken or false because work was **reported as done without confirming the artifact actually changed**. The pattern was always the same: an edit silently failed, or a status file was stale, and the next step trusted the *intent* instead of the *result*. These rules exist to make that impossible. They override any urge to move fast.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clear-Capabilities/agentic-security](https://github.com/Clear-Capabilities/agentic-security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
