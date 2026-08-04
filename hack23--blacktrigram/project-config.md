---
trigger: always_on
description: PRIO 1: Follow existing React + Three.js patterns for 3D rendering and UI overlays
---

# GitHub Copilot Instructions for Black Trigram (흑괘)

PRIO 1: Follow existing React + Three.js patterns for 3D rendering and UI overlays
PRIO 2: Use established component structure and Korean martial arts theming
PRIO 3: Maintain type safety and proper error handling throughout

## 🎯 Core Operating Principles

**YOU MUST BE DECISIVE AND AUTONOMOUS:**

1. **DO NOT ask questions when the rules are clear** - Apply the rules and implement solutions
2. **DO NOT create new .md documentation files** unless explicitly requested by the user
3. **DO run comprehensive checks** before committing any code changes:
   - Type checking: `npm run check`
   - Linting: `npm run lint`
   - Unit tests: `npm test`
   - Build verification: `npm run build`

### Enforcement Rules

```
Rule 1: Korean Theming
IF (creating UI component OR modifying styles)
THEN (use KOREAN_COLORS constants AND bilingual text format)
ELSE (reject component - refer to korean-theming-standards skill)

Rule 2: Test Coverage
IF (adding new feature OR modifying logic)
THEN (add/update tests to maintain >90% coverage)
ELSE (implementation incomplete - refer to testing-strategy-enforcement skill)

Rule 3: Security Changes
IF (modifying authentication OR handling user data OR external APIs)
THEN (update SECURITY_ARCHITECTURE.md AND add security tests)
ELSE (reject change - refer to security-architecture-validation skill)

Rule 4: Architecture Changes
IF (modifying component structure OR data models OR system design)
THEN (update relevant C4 architecture documents)
ELSE (documentation incomplete - refer to c4-architecture-documentation skill)

Rule 5: Performance Requirements
IF (adding Three.js rendering OR heavy computations)
THEN (ensure 60fps target AND bundle size <500KB initial)
ELSE (optimize - refer to performance-optimization skill)

Rule 6: Game Development
IF (implementing game loop OR combat system OR animations)
THEN (use clamped delta AND fixed timestep for physics AND state machine)
ELSE (refer to game-development-patterns skill)

Rule 7: Korean Martial Arts Authenticity
IF (implementing Eight Trigrams OR vital points OR combat techniques)
THEN (verify anatomical accuracy AND cultural respect AND proper terminology)
ELSE (refer to korean-martial-arts-authenticity skill)

Rule 8: 3D Combat Systems
IF (implementing combat physics OR collision detection OR damage calculation)
THEN (use Rapier physics AND anatomical hitboxes AND deterministic formulas)
ELSE (refer to 3d-combat-systems skill)

Rule 9: Audio Integration
IF (adding audio effects OR spatial audio OR combat sounds)
THEN (use Howler.js AND PositionalAudio for 3D AND Korean-themed audio)
ELSE (refer to audio-game-integration skill)
```

## 📚 Agent and Skills Catalog

**Skills** = automatic enforcement of quality standards.
**Agents** = on-demand implementation assistance.

### Available Skills (Automatically Loaded)

1. **[security-architecture-validation](./skills/security-architecture-validation/SKILL.md)** - ISMS security-by-design enforcement
2. **[c4-architecture-documentation](./skills/c4-architecture-documentation/SKILL.md)** - C4 Model architecture standards
3. **[korean-theming-standards](./skills/korean-theming-standards/SKILL.md)** - Korean cyberpunk aesthetic rules
4. **[testing-strategy-enforcement](./skills/testing-strategy-enforcement/SKILL.md)** - >90% test coverage requirements
5. **[performance-optimization](./skills/performance-optimization/SKILL.md)** - 60fps and bundle size enforcement
6. **[isms-compliance-checking](./skills/isms-compliance-checking/SKILL.md)** - ISO 27001, NIST CSF, CIS Controls
7. **[threejs-best-practices](./skills/threejs-best-practices/SKILL.md)** - Three.js/React optimization patterns
8. **[game-development-patterns](./skills/game-development-patterns/SKILL.md)** - Game loop, state machines, deterministic physics
9. **[korean-martial-arts-authenticity](./skills/korean-martial-arts-authenticity/SKILL.md)** - Eight Trigrams, vital points, cultural accuracy
10. **[3d-combat-systems](./skills/3d-combat-systems/SKILL.md)** - Physics-based combat, hitboxes, damage calculations
11. **[audio-game-integration](./skills/audio-game-integration/SKILL.md)** - Spatial audio, combat feedback, Korean soundscapes

**📖 [Complete Skills Documentation](./skills/README.md)**

### Available Custom Agents (On-Demand)

1. **[@task-agent](./agents/task-agent.md)** - Product quality orchestrator, creates issues
2. **[@coding-agent](./agents/coding-agent.md)** - TypeScript/React/Three.js implementation
3. **[@frontend-specialist](./agents/frontend-specialist.md)** - React 19 and strict TypeScript expert
4. **[@game-developer](./agents/game-developer.md)** - Three.js game systems and 60fps optimization
5. **[@korean-martial-arts-expert](./agents/korean-martial-arts-expert.md)** - Martial arts authenticity and vital point systems
6. **[@testing-agent](./agents/testing-agent.md)** - Vitest and Cypress test implementation
7. **[@test-engineer](./agents/test-engineer.md)** - Test strategy and CI integration
8. **[@documentation-writer](./agents/documentation-writer.md)** - Technical docs and bilingual content
9. **[@code-review-agent](./agents/code-review-agent.md)** - Code quality and standards review

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hack23/blacktrigram](https://github.com/Hack23/blacktrigram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
