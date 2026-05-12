---
trigger: always_on
description: This file was previously `QUICK_START.md` and is now the primary startup + execution guide.
---

# AGENTS

This file was previously `QUICK_START.md` and is now the primary startup + execution guide.


### Agent workflow pointers

To keep implementation quality and session continuity high, maintain these repo-local files:

- `DEVLOG.md` - chronological engineering timeline (what changed, validation, risks). **MUST be updated for every meaningful task.**
- `AGENT_SCRATCHPAD.md` - persistent execution memory (mistakes, preferences, guardrails). **MUST be updated for every meaningful task.**
- `web-ui/design.md` - living UI consistency spec (typography hierarchy, panel rules, spacing, review checklist).
- `.cursor/skills/devlog-loop/SKILL.md` -> writes to `DEVLOG.md` (template: `.cursor/skills/devlog-loop/references/devlog-entry-template.md`).
- `.cursor/skills/learning-scratchpad-loop/SKILL.md` -> writes to `AGENT_SCRATCHPAD.md` (template: `.cursor/skills/learning-scratchpad-loop/references/scratchpad-template.md`).

Non-negotiable workflow rule:

- Never hand off or stop after implementation without appending both `DEVLOG.md` and `AGENT_SCRATCHPAD.md`.
- Treat missing updates to either file as an incomplete task (blocker), even if code is correct.

### Skills catalog (all installed) and when to use each

Use these skills proactively based on task type.

- `create-rule` (`.cursor/skills-cursor/create-rule/SKILL.md`) - creating or updating project rules under `.cursor/rules`, coding standards, or persistent guidance.
- `create-skill` (`.cursor/skills-cursor/create-skill/SKILL.md`) - authoring a new reusable Cursor skill (`SKILL.md`) with strong structure and triggers.
- `update-cursor-settings` (`.cursor/skills-cursor/update-cursor-settings/SKILL.md`) - changing editor/IDE settings, `settings.json`, formatting, autosave, or related config.
- `agent-browser` (`.cursor/skills/agent-browser/SKILL.md`) - browser automation/testing (navigate, click, fill, screenshot, extract data).
- `canvas-design` (`.cursor/skills/canvas-design/SKILL.md`) - generating static visual design artifacts (posters/artwork in `.png`/`.pdf`).
- `devlog-loop` (`.cursor/skills/devlog-loop/SKILL.md`) - meaningful work logging in `DEVLOG.md` for traceability and handoff.
- `find-skills` (`.cursor/skills/find-skills/SKILL.md`) - when user asks "is there a skill for X" or wants capability discovery/installation.
- `frontend-design` (`.cursor/skills/frontend-design/SKILL.md`) - UI building/polish for React/web interfaces where visual quality and production finish matter.
- `learning-scratchpad-loop` (`.cursor/skills/learning-scratchpad-loop/SKILL.md`) - persistent execution memory updates in `AGENT_SCRATCHPAD.md`.
- `next-best-practices` (`.cursor/skills/next-best-practices/SKILL.md`) - Next.js architecture, routing, RSC boundaries, metadata, data handling.
- `next-cache-components` (`.cursor/skills/next-cache-components/SKILL.md`) - Next.js 16 caching strategy (`use cache`, cache tags/lifetimes, PPR patterns).
- `vercel-composition-patterns` (`.cursor/skills/vercel-composition-patterns/SKILL.md`) - React component API architecture and composition refactors at scale.
- `next-upgrade` (`.cursor/skills/next-upgrade/SKILL.md`) - migrating/upgrading Next.js versions safely with codemods and migration steps.
- `vercel-next-deploy` (`.cursor/skills/vercel-next-deploy/SKILL.md`) - Vercel deployment workflows, linking, env vars, preview/prod setup, domains.
- `vercel-react-best-practices` (`.cursor/skills/vercel-react-best-practices/SKILL.md`) - React/Next performance optimization and production-grade rendering patterns.
- `vercel-react-native-skills` (`.cursor/skills/vercel-react-native-skills/SKILL.md`) - React Native/Expo performance, animations, and native module patterns.
- `web-design-guidelines` (`.cursor/skills/web-design-guidelines/SKILL.md`) - design/accessibility/UX audit pass against web interface guideline checks.

### Design skill guidance for UI fixes in this repo

- For implementing visual/layout fixes: use `frontend-design`.
- For auditing UI quality/accessibility after implementation: use `web-design-guidelines`.
- For non-UI-code static design assets only: use `canvas-design`.
- For enforcing local visual consistency standards: follow `web-ui/design.md` and update it whenever UI conventions change.

### Initial setup

Run the setup script and select what modules you want to install.

```bash
./setup.sh
```

This will install the required system packages and create a virtual environment.
It targets Python 3.12 (preferred) for broad wheel compatibility.
If 3.12 is unavailable, setup falls back to 3.11, then 3.14.
Use a single repo-local virtualenv at `.venv` (the same one used by `start.sh`).
You can force-create the venv with 3.12:

```bash
uv venv .venv --python python3.12
source ./.venv/bin/activate
uv pip install -e .[core]
```

### EtherCAT on RevPi Connect 5 (important port note)

If you're bringing up the **RTOS/EtherCAT** path on a **RevPi Connect 5**, pay attention to which RJ45
port you use for the EtherCAT drive chain.

- **Both ports are “gigabit”, but they are different NICs/drivers**:
  - **`eth0`**: Linux driver **`macb`** (SoC MAC) → **use this for EtherCAT** (stable with IgH `ec_generic` in our bring-up)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gradient-industrial-robotics/GradientOS](https://github.com/gradient-industrial-robotics/GradientOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
