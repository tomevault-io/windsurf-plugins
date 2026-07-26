---
trigger: always_on
description: - You are an agent: continue working until the user's request is fully resolved.
---


# Power Apps Code Apps Development Guide

# General Coding Guidance

## General Behavior

- You are an agent: continue working until the user's request is fully resolved. 
  Only end your turn when you're confident the problem is solved and no further 
  action is required.

- Your thinking should be thorough—it's absolutely fine (and encouraged) if your 
  reasoning is long. Think step by step before and after each action you take.

- Plan extensively before making any function calls. Reflect critically after 
  each one. Avoid chaining function calls without introspection between them, as 
  that can impair insight and decision-making.

- If you're unsure about file contents or the codebase structure, use tools to 
  inspect and read relevant files. Never guess or make assumptions.

- Only make necessary, intentional changes that are either directly requested or 
  clearly required for task completion. Avoid editing unrelated or unclear areas.

## Code Quality and Style

- Prefer simple solutions that are easy to understand and maintain.

- Avoid code duplication: before writing new logic, check if similar 
  functionality already exists in the codebase.

- Only introduce a new pattern or technology if all options for improving the 
  current implementation have been exhausted. If you do introduce something new, 
  make sure to fully remove the old implementation to avoid duplication or 
  confusion.

- Keep the codebase clean and organized. Use consistent patterns and naming 
  conventions where applicable.

- Avoid writing one-off scripts in the main codebase—especially if they are 
  only intended to run once.

- Refactor files when they exceed 200–300 lines of code to preserve modularity 
  and clarity.

- Never overwrite the .env file without asking for and receiving explicit 
  confirmation.

- Follow best practices around formatting and consistency. Use linters, 
  formatters, and style guides where appropriate.

## Coding Workflow

- Stay focused on the parts of the code directly relevant to the current task.

- Do not touch unrelated code, even if it could be improved, unless explicitly 
  instructed to do so.

- Avoid major architectural changes or large refactors unless they are 
  structured, justified, and approved.

- Before making a change, always consider its impact on other parts of the 
  system—downstream dependencies, shared services, and global logic should be 
  reviewed.

- Document or summarize your reasoning and decision-making if a change affects 
  multiple components.

---

# Using this template repository to build Power Apps Code App.

Follow these steps in order to create a complete Power Apps Code App:

1. **Install dependencies and build the app** - Set up the foundation
2. **Configure for Power Apps Code App** - Add Power Platform SDK integration
3. **Build app to work locally with mocked data** - Develop and test locally
4. **Check requirements for connecting with live data** - Ensure you have connectors setup to connect.
5. **Connect to Office 365** - Connect to live data sources
6. **Connect to SQL** - Connect to live data sources
7. **Connect to custom API** - Connect to custom connector
7. **Test and deploy** - Final testing and deployment

---

## Step 1: Install depedencies

Use only the files within FluentSample folder.

Open the VS Code terminal (Terminal → New Terminal) and navigate to the FluentSample folder:

```bash
cd samples/FluentSample
npm install
npm run build
```

This ensures the project compiles successfully before running.

---

## Step 2: Configure for Power Apps Code App

Be sure the user has PAC CLI installed and it is connected to the right environment. To know what is your user - 'pac auth who'. To know what is current environment 'pac org who'.

Initialize the app

```bash
pac code init
```

If any error occurs, it is likely due to the wrong environment. Inform the user and don't proceed.

---

## Step 3: Build app to work locally with mocked data

```bash
npm run dev
```

This will start both Vite dev server and PAC Code Run automatically for local development and testing.
The server will start in a new terminal window, don't look for terminal output. You can assume the app is running in the port 3000. The file power.config.json has the necessary configuration for the Power Apps environment.

---
## Step 4: Check requirements for connecting with live data

Check if the user has the needed connections. Use:

```bash
pac connection list
```

This will show all available connections with their IDs and names. It is important to have the Office 365 Users, SQL and Custom Connector connections available.

---

## Step 5: Connect to Office 365

For Office 365 Users:

```bash
pac code add-data-source -a "shared_office365users" -c <connectionId>
```

Replace static data with real data.

The Office365Page.tsx file is specifically designed for easy conversion from mock to live data:

#### Step 1: Update Imports
**Current (Mock Data):**
```typescript
// TODO: Replace with live Office365UsersService when connecting to real data
// import { Office365UsersService } from '../Services/Office365UsersService';
import * as mockData from '../mockData/office365Data';
```

**Replace with (Live Data):**
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/PowerAppsCodeApps](https://github.com/microsoft/PowerAppsCodeApps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
