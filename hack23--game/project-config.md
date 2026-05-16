---
trigger: always_on
description: **Target Shooter** — a 3D target-clicking game built with React 19, TypeScript 6, Three.js (`@react-three/fiber` + `@react-three/drei`), Vite 8, and Howler.js. **Node ≥ 25** required.
---

# Copilot Instructions

## Project Overview

**Target Shooter** — a 3D target-clicking game built with React 19, TypeScript 6, Three.js (`@react-three/fiber` + `@react-three/drei`), Vite 8, and Howler.js. **Node ≥ 25** required.

**Security & Compliance:** Aligned with [Hack23 AB's ISMS](https://github.com/Hack23/ISMS-PUBLIC). Every change MUST respect the ISMS. See [ISMS Policy Mapping](../docs/ISMS_POLICY_MAPPING.md) for the full feature-to-policy traceability matrix.

## Required Context

Before starting any non-trivial task, read:

1. `.github/workflows/copilot-setup-steps.yml` — Copilot environment & permissions
2. `.github/copilot-mcp.json` — MCP server wiring (GitHub Insiders, filesystem, memory, sequential-thinking, playwright)
3. `README.md` — project overview, features, security posture
4. `.github/agents/README.md` & `.github/skills/README.md` — available specialists and patterns
5. The specific agent and skill files relevant to your task
6. [Hack23 ISMS-PUBLIC](https://github.com/Hack23/ISMS-PUBLIC) — authoritative security policies

## Quick Reference

```bash
npm install           # Install dependencies
npm run dev           # Dev server at http://localhost:5173
npm run build         # TypeScript check + Vite build
npm run lint          # ESLint
npm run test          # Vitest unit tests
npm run coverage      # Tests with coverage (≥80% lines, ≥70% branches)
npm run test:e2e      # Cypress E2E tests
npm run test:licenses # License compliance check (Open Source Policy)
npm audit             # Dependency vulnerability scan
```

## Project Structure

```text
src/
├── App.tsx                    # Root: game loop, audio, overlays
├── components/
│   ├── GameScene.tsx          # 3D scene (targets, particles, lights)
│   ├── TargetSphere.tsx       # Clickable 3D target
│   ├── BackgroundParticles.tsx # Particle effects
│   ├── ParticleExplosion.tsx   # Hit explosion effect
│   ├── HUD.tsx                # Score/time/combo display
│   └── GameOverlay.tsx        # Controls, instructions, overlays
├── hooks/
│   ├── useGameState.ts        # Game state management
│   └── useAudioManager.ts     # Howler.js audio integration
├── utils/
│   ├── gameConfig.ts          # Game constants and level progression
│   └── targetPhysics.ts       # Target creation and movement
└── test/setup.ts              # Vitest test setup
```

## ISMS Policy Quick Map

| Applies When… | Policy | Link |
|---|---|---|
| Any security-relevant code, governance, incident | Information Security Policy | [ISP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Information_Security_Policy.md) |
| SDLC, CI/CD, testing, deployment, threat modeling | Secure Development Policy | [SDP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md) |
| Adding / updating / removing dependencies, licenses, SBOM | Open Source Policy | [OSP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Open_Source_Policy.md) |
| Copilot agents, MCP, LLM-assisted changes | AI Policy | [AI](https://github.com/Hack23/ISMS-PUBLIC/blob/main/AI_Policy.md) |
| Auth, identity, permissions, least privilege | Access Control Policy | [ACP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Access_Control_Policy.md) |
| Encryption, hashing, key material, TLS | Cryptography Policy | [CP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Cryptography_Policy.md) |
| Data handling, classification, storage, logs | Data Classification Policy | [DCP](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Data_Classification_Policy.md) |
| Personal data, analytics, telemetry | Privacy Policy | [Privacy](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Privacy_Policy.md) |
| Changes to agents, MCP, workflows | Change Management | [CM](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Change_Management.md) |
| STRIDE, abuse cases, attack surface | Threat Modeling Policy | [TM](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Threat_Modeling.md) |
| Reporting / patching vulnerabilities | Vulnerability Management | [VM](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Vulnerability_Management.md) |

Cite the applicable policy in code comments, commit messages, and PR descriptions for security-sensitive changes.

## Secure SDLC Phases (per SDP)

1. **Plan & Design** — classify impact (CIA triad), threat model (STRIDE), link to ISMS policies
2. **Develop** — OWASP-aligned coding, no hardcoded secrets, typed everywhere, `no-any`
3. **Test** — SAST (CodeQL + ESLint), SCA (`npm audit`), DAST (ZAP), unit + E2E, coverage ≥80%/70%
4. **Deploy** — CI/CD gates, SHA-pinned Actions, SLSA ≥ L3 attestations, SBOM + SBOMQS ≥ 7.0
5. **Operate** — monitor CodeQL alerts, Dependabot, Scorecard ≥ 8.0, patch per Vulnerability Management SLAs

## AI-Augmented Development Controls (ISMS SDP §AI-Augmented Controls)

1. **AI outputs are proposals, never authoritative** — human review required before merge
2. **No autonomous deployment** — AI may not bypass CI gates, branch protections, or approvals
3. **Change attribution** — document AI assistance in PR descriptions (e.g., "assisted by Copilot agent `game-developer`")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hack23/game](https://github.com/Hack23/game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
