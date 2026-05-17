---
trigger: always_on
description: Project context for Claude Code and other AI coding assistants working on Pre-Flight.
---

# CLAUDE.md

Project context for Claude Code and other AI coding assistants working on Pre-Flight.

## What this is

A free, in-browser static security audit for AI-generated apps. SAST only. No backend, no execution, no telemetry. Built by John at Mid-Atlantic AI. Live at [preflight.midatlantic.ai](https://preflight.midatlantic.ai).

The voice and stance live in [`src/learn/manifesto.md`](./src/learn/manifesto.md). Read it before making copy changes.

## The founding principle

**Hardened titanium.** Dogfood-as-CI-gate. Pre-Flight has to pass its own audit. If a change makes the dogfood scan fail, fix the change. Do not suppress the finding.

`npm run test:self-audit` runs after every build and asserts Pre-Flight's own `dist/` produces zero critical/high findings. This is non-negotiable.

## Voice

Two named voices, used in different surfaces:

- **John's voice** — `src/learn/manifesto.md`, hero copy in `AuditView.jsx`, About tab, README intro, founder-facing messages. First-person, direct, self-effacing, mechanics-instructor register.
- **Demi's voice** — `src/learn/patterns/*.md`, `src/learn/incidents/*.md`, `src/learn/shapes/*.md`. Third-person instructor. Concrete-first, no marketing prose. Full spec in `src/lib/personas/demi.js`.

Both voices share the rules below. They differ only in person (I/we vs third-person institutional).

### Voice rules (apply to all copy John or Demi own)

- No em-dashes anywhere. Use periods or commas.
- No marketing register: no "comprehensive", "best-in-class", "powerful", "robust", "enterprise-grade", "unlock", "leverage", "seamless".
- No fear marketing: no "don't let this happen to you", no manufactured urgency, no "the threat is real."
- No compliance flavoring unless the topic is literally regulatory.
- No wellness encouragement: no "you've got this", no soft motivational scaffolding.
- No hedging filler: no "it is worth noting", "in some sense", "at the end of the day".
- No competing security platforms named in public-facing copy (Aikido, Snyk, Veracode, Checkmarx, Socket, Wiz, OX, Semgrep, etc.). Sources cite OWASP, MITRE/CWE, CISA, vendor official docs, W3C/WAI, MDN, named research orgs (GTIG/Mandiant, Microsoft Threat Intel).
- AI providers (OpenAI, Anthropic, Google, xAI, Mistral) are NOT competitors. Naming them in the BYOK provider list is fine.

### Audience

Vibe coders — people building real products from natural-language prompts. Capable practitioners developing a security sensibility, not beginners being talked down to. Write for them.

## Personas

Pre-Flight ships four named agents under `src/lib/personas/`. Each is a Persona+ spec with an activation gate and structured-command modes.

- **Sam** (Secure Advise Mobilize) — security fix generation. Dual-mode: `SAM_COMMAND_FULL` (Apply Fix, full file) and `SAM_COMMAND_SNIPPET` (Copy Agent Prompt, snippet only).
- **Demi** (Design Engineering Mechanics Instructor) — Vibe-Aware educational content. Dual-mode: `DEMI_MODE_AUTHOR` and `DEMI_MODE_GRADE`.
- **Drew** (Design Rules Enforcement Worker) — enforces `.preflight/design-rules.yml` (planned for v1.1).
- **Vera** (Verify Engineering Rules Adherence) — enforces `.preflight/engineering-rules.yml` (planned for v1.1).

Wired today: Sam SNIPPET is the system prompt for `formatAgentPrompt` (Copy Agent Prompt export). The rest are deployable specs awaiting their surface.

## Stack

- React 18 + Vite 5 + react-router-dom v6
- Vitest + jsdom (+ `@vitejs/plugin-react` on `feature/breakers-v1` for component tests)
- `acorn` + `acorn-jsx` + `acorn-loose` (Code Correctness probe AST parser)
- `react-markdown` + `remark-gfm` (Learn content rendering)
- Pure-function probes, 43 in v0.5
- No backend. Static assets only, deployed to Cloudflare Pages.

## Key paths

| Concern                                  | File                                                                     |
| ---------------------------------------- | ------------------------------------------------------------------------ |
| Probe registry                           | `src/lib/probes.js#PROBES`                                               |
| Probe metadata + OWASP map + slug wiring | `src/lib/stable-id.js`                                                   |
| Probe implementations (v0.4)             | `src/lib/probes.js` + `src/lib/probes/{web,quality,code-correctness}.js` |
| Probe implementations (v0.5)             | `src/lib/probes/v05.js` + `src/lib/probes/v05b.js`                       |
| Threat-intel manifests                   | `src/lib/threat-intel.js` + `src/data/compromised-packages.js`           |
| File filter + self-source exclusion      | `src/lib/file-filter.js`                                                 |
| BYOK providers (9)                       | `src/lib/ai.js#AI_PROVIDERS`                                             |
| Personas                                 | `src/lib/personas/{sam,demi,drew,vera}.js`                               |
| Learn content                            | `src/learn/{manifesto.md,patterns/,incidents/,shapes/}`                  |
| Learn content loader                     | `src/lib/learn-content.js`                                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midatlanticAI/PreFlight](https://github.com/midatlanticAI/PreFlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
