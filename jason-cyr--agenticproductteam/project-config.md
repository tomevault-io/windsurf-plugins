---
trigger: always_on
description: AgenticPod is an agentic product lifecycle accelerator. You (Claude Code) are the UI. Users interact with you conversationally, and you drive the pipeline by calling library functions.
---

# AgenticPod — Claude Code Instructions

AgenticPod is an agentic product lifecycle accelerator. You (Claude Code) are the UI. Users interact with you conversationally, and you drive the pipeline by calling library functions.

## How It Works

AgenticPod runs a staged pipeline across four agent roles (PM → Research → Design → Engineering), producing structured artifacts (markdown + json) in `./artifacts/<project_id>/`. Each stage calls an LLM to generate content based on prior artifacts.

## Calling AgenticPod Functions

Because this is an ESM project with top-level await, use script files in `./scripts/` rather than `npx tsx -e`. Write a short `.ts` script, run it with `npx tsx scripts/your-script.ts`, then delete it after.

Example pattern:
```ts
// scripts/run-stage.ts
import { runStage } from '../src/index.js';
const r = await runStage('snackquest', 'pm');
console.log(JSON.stringify(r, null, 2));
```
```bash
npx tsx scripts/run-stage.ts
```

## Commands (what users will ask you to do)

### Start a new project
When the user describes a product idea, init a project:
```ts
import { initProject } from '../src/index.js';
const r = await initProject('THE IDEA TEXT', 'optional-project-id');
console.log(JSON.stringify(r, null, 2));
```
Then show the user the generated `idea.json` and ask if they want to refine it before proceeding.

### Run the next stage
Run one stage at a time. After each stage, present:
1. What artifacts were created/updated
2. A brief summary of the content
3. Any quality issues
4. Ask: "Ready to proceed to [next stage]?"

```ts
import { runStage } from '../src/index.js';
const r = await runStage('PROJECT_ID', 'STAGE');
console.log(JSON.stringify(r, null, 2));
```

Stages in order: `pm` → `research` → `design` → `engineering`

### Check project status
```ts
import { getProjectStatus } from '../src/index.js';
console.log(JSON.stringify(getProjectStatus('PROJECT_ID'), null, 2));
```

### Run quality checks
```ts
import { runQualityChecks } from '../src/index.js';
console.log(JSON.stringify(runQualityChecks('PROJECT_ID'), null, 2));
```

### Pull Figma snapshot
After a designer has made changes in Figma, save a snapshot:
```ts
import { saveFigmaSnapshot } from '../src/index.js';
saveFigmaSnapshot('PROJECT_ID', SNAPSHOT_DATA);
```

### Run iteration
After pulling a new Figma snapshot, update specs from the changes:
```ts
import { runIteration } from '../src/index.js';
const r = await runIteration('PROJECT_ID');
console.log(JSON.stringify(r, null, 2));
```

## Stage Gate Flow (Mode A — default)

This is the expected conversational pattern:

1. **User provides idea** → You run `initProject`, show `idea.json`, ask to refine or proceed
2. **PM stage** → You run `runStage(id, 'pm')`, read and summarize `PRD.md`, show quality report, ask to proceed
3. **Research stage** → You run `runStage(id, 'research')`, summarize `Research.md`, ask to proceed
4. **Design stage** → You run `runStage(id, 'design')`, summarize `PrototypeSpec.md`, then **build coded HTML prototype screens** in `./artifacts/<project_id>/screens/`. Present screens for review. Ask if user wants to push to Figma or proceed.
5. **Engineering stage** → You run `runStage(id, 'engineering')`, summarize `TechSpec.md` and `Backlog.md`

Between each stage, **always ask for confirmation** before advancing. Show what files were written and a brief content summary.

**After every stage**, update `./artifacts/<project_id>/CONTEXT.md` with the current project state, completed stages, key decisions, next steps, and any setup notes (API keys, MCP config, etc.) so a new Claude Code session can pick up seamlessly.

## Prototype Workflow (Code-First)

After the design stage completes, **always build coded HTML prototype screens** before involving Figma:

### Step 1: Build HTML Screens
1. Read `FigmaLink.json` and `PrototypeSpec.md` for the frame structure, screen specs, and microcopy
2. Create HTML files in `./artifacts/<project_id>/screens/` — one per frame, plus a shared `styles.css`
3. Style screens to match the target platform (e.g., Slack dark theme for Slack apps, Material for Android, etc.)
4. Include all UI elements, states, copy, and interactions described in `PrototypeSpec.md`
5. Name files with numbered prefixes matching `FigmaLink.json` frame order (e.g., `01-slash-command.html`)
6. Present the screens to the user for review. They can open any HTML file in a browser to preview.

### Step 2: Send to Figma (Optional)
If the user wants to refine designs in Figma and `config/figma.mcp.json` has a file key:

1. Use the **Figma MCP** (`figma-remote`) to push the coded screens to Figma as editable frames ("Code to Canvas")
   - The MCP server is at `https://mcp.figma.com/mcp` — authenticate via `/mcp` if needed
   - Note: The Figma REST API is **read-only** for design content. Frame creation requires the MCP tools or Figma desktop app's local MCP server (`127.0.0.1:3845`)
2. Save the frame IDs/metadata using `saveFigmaSnapshot`

### Step 3: Figma Round-Trip (if Figma is used)
1. User says "I've updated the Figma designs"
2. Use Figma MCP to read the current frames and text layers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jason-Cyr/AgenticProductTeam](https://github.com/Jason-Cyr/AgenticProductTeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
