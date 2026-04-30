---
trigger: always_on
description: This workspace helps GitHub employees write performance reflections and peer feedback through a guided, interactive process. Copilot pulls real-time data from GitHub and references official guidance from Copilot Spaces.
---

# Reflections & Feedback Workspace - Copilot Instructions

## Purpose

This workspace helps GitHub employees write performance reflections and peer feedback through a guided, interactive process. Copilot pulls real-time data from GitHub and references official guidance from Copilot Spaces.

## First-Time Setup Check

**Before starting any workflow**, check if the ADX integration is set up by verifying the Python virtual environment exists at `tools/.venv/`. If it does NOT exist, set it up for the user:

> "Before we begin, let me set up the ADX integration so I can automatically pull your support metrics (CSAT, tickets, IR Met, escalations, etc.)."

**Step 1: Install dependencies** — Run this directly in the terminal for the user (do NOT ask them to run it themselves):
```bash
cd tools && python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
```

**Step 2: Azure authentication** — First check whether Azure CLI is installed by running `which az`. Then tell the user:
> "The MCP server needs your **@githubazure.com** credentials to access the ADX clusters. You have two options: (a) Azure CLI (`az login`) or (b) the VS Code Azure extension. If you've already signed in via either with that account, you're all set."

- **If `az` is installed and the user needs to sign in:** offer to run `az login` for them in the terminal (sign in with **@githubazure.com**, not @github.com).
- **If `az` is NOT installed:** offer two paths and let the user choose:
  1. Install Azure CLI (`brew install azure-cli`) and then run `az login`.
  2. Use the VS Code Azure extension instead — install the [Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account) extension and sign in via the Accounts icon in the bottom-left of VS Code with their **@githubazure.com** email.
  3. Skip auth setup and rely on the interactive browser fallback (the Kusto MCP server will pop a browser on the first query).

**Step 3: Verify MCP connections** — After setup (or if `tools/.venv/` already exists), verify **both** MCP servers are connected before proceeding:

1. **ADX MCP** — Try listing ADX clusters (e.g., call `list_clusters`). If it returns `gh-analytics` and `dotcomro`, ADX is working.
2. **GitHub MCP** — Try a simple GitHub MCP call (e.g., `get_me` to fetch the authenticated user). If it returns a valid user, GitHub MCP is working.

**If both succeed:** Confirm to the user and proceed to the requested workflow.

**If one or both fail:** Tell the user which connection(s) failed and offer two options:
> "It looks like **[ADX/GitHub/both]** MCP isn't connected yet. I can help troubleshoot this, or you can skip it and we'll work with whatever data you can provide manually. What would you prefer?"

- **Troubleshoot ADX:** Check Azure auth (`az login` if Azure CLI is installed, or VS Code Azure extension sign-in), verify the MCP server config in `.vscode/mcp.json`, refer to README troubleshooting section.
- **Troubleshoot GitHub:** Verify the GitHub MCP server is enabled in VS Code's MCP settings, check that the `github` server entry in `.vscode/mcp.json` is correct.
- **Skip:** Continue without the failed integration — note which data will need to be provided manually (ADX metrics, GitHub contributions, or both) and adjust the workflow accordingly.

If `tools/.venv/` already exists, skip Steps 1–2 and go directly to Step 3 (verify connections).

---

## Required Copilot Spaces

**ALWAYS query these Copilot Spaces for the latest official questions and guidance:**

| Space | URL | Use For |
|-------|-----|---------|
| **IC Reflections FY26** | https://github.com/copilot/spaces/github/998 | Official reflection questions, performance philosophy, GitHub values |
| **Peer & Manager Feedback** | https://github.com/copilot/spaces/github/50 | Official 3-question feedback template, Manager Fundamentals |
| **Support Repository Reference** | https://github.com/copilot/spaces/github/1106 | Support career ladder, level expectations, promotion criteria. Backed by the entire [github/support](https://github.com/github/support) repository — query it directly (via GitHub MCP or `mcp_github_get_copilot_space`) for ladder docs, role definitions, processes, and team guidance. |

**CRITICAL:** Always pull the latest from these Spaces before starting any workflow.

---

## Spaces Cache Management

Official Copilot Spaces content is cached locally in `spaces_cache/` at the workspace root. Each Space has its own subdirectory; one Space may contribute multiple documents.

| Source Space | Cache Directory | Primary File(s) |
|--------------|-----------------|-----------------|
| IC Reflections FY26 (github/998) | `spaces_cache/ic-reflections-fy26/` | `reflection-questions.md` |
| Peer & Manager Feedback (github/50) | `spaces_cache/peer-manager-feedback/` | `peer-feedback-questions.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamrr724/reflections-feedback-helper](https://github.com/adamrr724/reflections-feedback-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
