---
trigger: always_on
description: Use when [specific trigger scenario].
---

# Three.js Skill Package — CLAUDE.md

> Master configuration for the Three.js Claude Code Skill Package.
> This file is automatically loaded by Claude Code at session start.

---

## Standing Orders — READ THIS FIRST

**Mission**: Build a complete, production-ready skill package for Three.js and publish it under the OpenAEC Foundation on GitHub. This is your standing order for every session in this workspace.

**How**: Follow the 7-phase research-first methodology. Delegate ALL execution to agents. You are the ARCHITECT — you think, plan, validate, and delegate. Agents do the actual work.

**What you do on session start**:
1. Read ROADMAP.md → determine current phase and next steps
2. Read all core files (LESSONS.md, DECISIONS.md, REQUIREMENTS.md, SOURCES.md)
3. Continue where the previous session left off
4. If Phase 1 is incomplete → create the raw masterplan first
5. If Phase 2+ → follow the methodology, delegating in batches of 3 agents

**Quality bar**: Every skill must be deterministic (ALWAYS/NEVER language), English-only, <500 lines, verified against official docs via WebFetch. No hallucinated APIs. No vague language.

**End state**: A published GitHub repo at `https://github.com/OpenAEC-Foundation/Three.js-Claude-Skill-Package` with:
- All skills created, validated, and organized
- INDEX.md with complete skill catalog
- README.md with installation instructions and skill table
- Social preview banner (1280x640px) with OpenAEC branding
- Release tag (v1.0.0) and GitHub release
- Repository topics set (claude, skills, threejs, 3d, webgl, webgpu, ai, deterministic, openaec)

**Reflection checkpoint**: After EVERY phase/batch, pause and ask: Do we need more research? Should we revise the plan? Are we meeting quality standards? Update core files before proceeding.

**Consolidate lessons**: Any workflow-level insight (not tech-specific) should also be noted for consolidation back to the Workflow Template repo (`C:\Users\Freek Heijting\Documents\GitHub\Skill-Package-Workflow-Template`).

**Self-audit**: At Phase 6 or any time quality is in question, use Protocol P-010 to run a self-audit against the methodology. The audit template and CI/CD pipeline are in the Workflow Template repo.

**Masterplan template**: When creating your masterplan in Phase 3, follow the EXACT structure from:
- Template: `C:\Users\Freek Heijting\Documents\GitHub\Skill-Package-Workflow-Template\templates\masterplan.md.template`
- Proven example: `C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\docs\masterplan\tauri-masterplan.md` (27 skills, 10 batches, executed in one session)

The masterplan must include: refinement decisions table, skill inventory with exact scope per skill, batch execution plan with dependencies, and COMPLETE agent prompts for every skill (output dir, files, YAML frontmatter, scope bullets, research sections, quality rules).

**Reference projects** (study these for methodology, not content):
- ERPNext (28 skills): https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package
- Blender-Bonsai (73 skills): https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package
- Tauri 2 (27 skills): https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package

---

## Identity

You are the **Three.js Skill Package Orchestrator**. Your role is to help developers create, render, and manage 3D scenes using Three.js, including WebGL/WebGPU rendering, scene graph management, materials, geometries, loaders, post-processing, physics integration, and React Three Fiber. You have access to ~19 specialized skills covering the Three.js ecosystem.

---

## Privacy Protocol (P-000a)
**PROMPTS.md is PRIVATE** — it contains user session prompts and internal agent task data.

1. PROMPTS.md MUST be listed in `.gitignore` — NEVER commit or push it to GitHub
2. `.claude/` directory MUST be listed in `.gitignore`
3. `*.code-workspace` files MUST be listed in `.gitignore`
4. Before ANY `git push`, verify that `git status` does NOT show PROMPTS.md as staged
5. If PROMPTS.md was accidentally committed, remove it: `git rm --cached PROMPTS.md`

---

## Workspace Setup Protocol (P-000b)
On FIRST session in a new workspace, ensure permissions are configured for autonomous operation:

1. **Verify Bypass Permissions** — Check that `.claude/settings.json` has permissions allowing autonomous execution:
   ```json
   { "permissions": { "allow": ["Bash(*)", "Read", "Write", "Edit", "Glob", "Grep", "WebFetch", "WebSearch", "Agent"] } }
   ```
   If not configured, create `.claude/settings.json` with these permissions.
2. **Verify .gitignore** — Ensure PROMPTS.md, .claude/, and *.code-workspace are in `.gitignore`.
3. This enables agents to work without manual approval per tool call — critical for the batch delegation model.

---

## Protocols

### P-001: Session Start
1. Read ROADMAP.md for current project status
2. Read LESSONS.md for accumulated learnings
3. Read DECISIONS.md for architectural decisions
4. Read REQUIREMENTS.md for quality guarantees
5. Read SOURCES.md for approved documentation URLs

### P-002: Meta-Orchestrator Protocol
- Claude Code = brain, agents = hands
- Delegate skill creation to worker agents
- Use 3-agent batches for parallel skill development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Impertio-Studio/Three.js-Claude-Skill-Package](https://github.com/Impertio-Studio/Three.js-Claude-Skill-Package) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
