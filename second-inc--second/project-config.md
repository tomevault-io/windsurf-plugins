---
trigger: always_on
description: Second is a generative workspace, which allows teams to build internal tools.
---

# Second
Second is a generative workspace, which allows teams to build internal tools.

This is a monorepo containing everything.

# Layout
- apps/web: nextjs + shadcn

# General instructions (not necessarlly task dependent)

For big tasks (or whenever you feel like it), please first of all read the docs/ to understand about this project. Read them also to orient yourself in this project, find relevant files, understand the overall architecture and the code base. Obviously they might be outdated so the code is the real truth eventually.

In this repo, identify key files, the architecture, what this project is about, and how everything works (in they way which is relevant for this task) using the docs/.

Remember that we have currently 2 deployment models:
1. Local, using `npx @second-inc/cli ...`
2. On prem - on GCP (AWS is also not currently supported but it will and should be sometime). On prem can be either on the customer's cloud OR a managed instance - where I deploy it on Second's cloud and help them register the Auth provider, etc.

We are now in the open source repo. You should probably only work on the open source repo (this was just context for you).

Also important: Whatever you do, it's very important that this will be enforced securely and beautifully and will actually work and will not have any security issues. Also keep in mind tenant isolation (critical). I mean I know that right now we haven't released our SAAS solution yet, but still it's very important. Keep all of the best practices from the docs and the current structure.

It's also very important to keep everything very secure and go with the security best practices and everything that we currently have set up and fixed to not regress and to not introduce any security issues.

# About app UI changes
- Before changing app UI, inspect strong local references such as `apps/web/src/components/ai-elements/tool-card.tsx`, `apps/web/src/components/ai-elements/agents-card.tsx`, `apps/web/src/app/w/[workspaceId]/apps/[appId]/agents/page.tsx`, and `apps/web/src/app/w/[workspaceId]/settings/integrations/page.tsx`.
- Match the existing restrained shadcn/Radix theme: compact rows, muted borders, mono metadata, semantic badges, and polished tool-call / agent-card patterns.

# About local browser testing
- Do not assume the app is on `localhost:3000`. `npm run dev` generates `.second-dev.txt` in the repo root; read its `url=` value before opening the app or telling a browser/agent what to visit. The file is local-only and gitignored.
- `npm run dev` is worktree-aware: it automatically chooses a per-worktree dev ID, Compose project name, and non-conflicting ports. Multiple worktrees can run at once, and each active worktree has its own MongoDB and Redis containers.
- If the human requests browser QA, that request grants permission to run `npm run dev` for the current worktree when no matching dev server is already running. Start it only as needed for QA, then use `.second-dev.txt` as the source of truth for the URL.
- When using the in-app browser for QA, show the browser to the human unless they explicitly ask to keep it hidden.
- If onboarding or sign-in needs QA identity details and the user did not specify them, use email `john@doe.com`, display name `John Doe`, role `Founder`, and workspace name `Second`.
- When testing app-building flows, keep prompts intentionally tiny so runs finish quickly: ask for a tiny to-do list with minimal UI and no agents.
- Unless explicitly requested, send the message and verify the response starts successfully, but do not approve or complete the build.

# About QA guides
- For broad manual QA, use the `QA/` folder. Keep a reusable date-prefixed E2E guide such as `QA/YYYY-MM-DD-E2E.md`, and create a separate date-prefixed task guide such as `QA/YYYY-MM-DD-<feature-or-merge>-qa.md` for the current feature, branch, or merge.
- When a user asks for both a general E2E pass and a feature-specific QA pass, run the general E2E guide first, then the feature-specific guide, and record results in the feature-specific QA file.
- Only run QA when the user explicitly asks you to run QA, test, verify, or manually check the product. Creating or updating QA documents is not permission to execute them.
- If the user asks to create the QA docs before running tests, stop after writing the docs so they can review them.
- QA result files should mark each area as pass/fail/blocked, include concrete evidence such as URLs, app IDs, run IDs, model used, and audit event names, and list bugs separately with repro, expected behavior, observed behavior, impact, and status.
- Respect the local testing constraints above: use the in-app browser, do not start the dev server unless the user explicitly allows it, and do not run containers or infrastructure commands.

# No human in the loop mode
- When the human says something like "just do it e2e and open a pr", "do it without me in the loop", or "do it e2e", treat that as permission to complete the whole loop without stopping for routine review.
- In this mode, perform the task, create a plan first if the work is large or risky, run the relevant automated checks, run in-app browser QA, write a QA document, and iterate until the implementation and QA are clean.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Second-Inc/second](https://github.com/Second-Inc/second) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
