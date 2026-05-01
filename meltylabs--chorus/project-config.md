---
trigger: always_on
description: Chorus is a native Mac AI chat app that lets you chat with all the AIs.
---

# Claude's Onboarding Doc

## What is Chorus?

Chorus is a native Mac AI chat app that lets you chat with all the AIs.

It lets you send one prompt and see responses from Claude, o3-pro, Gemini, etc. all at once.

It's built with Tauri, React, TypeScript, TanStack Query, and a local sqlite database.

Key features:

-   MCP support
-   Ambient chats (start a chat from anywhere)
-   Projects
-   Bring your own API keys

Most of the functionality lives in this repo. There's also a backend that handles accounts, billing, and proxying the models' requests; that lives at app.chorus.sh and is written in Elixir.

## Your role

Your role is to write code. You do NOT have access to the running app, so you cannot test the code. You MUST rely on me, the user, to test the code.

If I report a bug in your code, after you fix it, you should pause and ask me to verify that the bug is fixed.

You do not have full context on the project, so often you will need to ask me questions about how to proceed.

Don't be shy to ask questions -- I'm here to help you!

If I send you a URL, you MUST immediately fetch its contents and read it carefully, before you do anything else.

## Workflow

We use GitHub issues to track work we need to do, and PRs to review code. Whenever you create an issue or a PR, tag it with "by-claude". Use the `gh` bash command to interact with GitHub.

To start working on a feature, you should:

1. Setup

-   Identify the relevant GitHub issue (or create one if needed)
-   Checkout main and pull the latest changes
-   Create a new branch like `claude/feature-name`. NEVER commit to main. NEVER push to origin/main.

2. Development

-   Commit often as you write code, so that we can revert if needed.
-   When you have a draft of what you're working on, ask me to test it in the app to confirm that it works as you expect. Do this early and often.

3. Review

-   When the work is done, verify that the diff looks good with `git diff main`
-   While you should attempt to write code that adheres to our coding style, don't worry about manually linting or formatting your changes. There are Husky pre-commit Git hooks that will do this for you.
-   Push the branch to GitHub
-   Open a PR.
    -   The PR title should not include the issue number
    -   The PR description should start with the issue number and a brief description of the changes.
    -   Next, you should write a test plan. I (not you) will execute the test plan before merging the PR. If I can't check off any of the items, I will let you know. Make sure the test plan covers both new functionality and any EXISTING functionality that might be impacted by your changes

4. Fixing issues

-   To reconcile different branches, always rebase or cherry-pick. Do not merge.

Sometimes, after you've been working on one feature, I will ask you to start work on an unrelated feature. If I do, you should probably repeat this process from the beginning (checkout main, pull changes, create a new branch). When in doubt, just ask.

We use pnpm to manage dependencies.

Don't combine git commands -- e.g., instead of `git add -A && git commit`, run `git add -A` and `git commit` separately. This will save me time because I won't have to grant you permission to run the combined command.

## Project Structure

-   **UI:** React components in `src/ui/components/`
-   **Core:** Business logic in `src/core/chorus/`
-   **Tauri:** Rust backend in `src-tauri/src/`

Important files and directories to be aware of:

-   `src/core/chorus/db/` - Queries against the sqlite database, which are split up by entity type (e.g. message, chat, project)
-   `src/core/chorus/api/` - TanStack Query queries and mutations, which are also split up by entity type
-   `src/ui/components/MultiChat.tsx` - Main interface
-   `src/ui/components/ChatInput.tsx` - The input box where the user types chat messages
-   `src/ui/components/AppSidebar.tsx` - The sidebar on the left
-   `src/ui/App.tsx` - The root component

You can see an up-to-date schema of all database tables in SQL_SCHEMA.md. Use this file as a reference to understand the current
database schema.

Other features:

-   Model picker, which lets the user select which models are available in the chat -- implemented in`ManageModelsBox.tsx`
-   Quick chats (aka Ambient Chats), a lightweight chat window -- implemented, alongside regular chats, in `MultiChat.tsx`
-   Projects, which are folders of related chats -- start with `AppSidebar.tsx`
-   Tools and "connections" (aka toolsets) -- start with `Toolsets.ts`
-   react-router-dom for navigation -- see `App.tsx`

## Screenshots

I've put some screenshots of the app in the `screenshots` directory. If you're working on the UI at all, take a look at them. Keep in mind, though, that they may not be up to date with the latest code changes.

## Data model changes

Changes to the data model will typically require most of the following steps:

-   Making a new migration in `src-tauri/src/migrations.rs` (if changes to the sqlite database scheme are needed)
-   Modifying fetch and read functions in `src/core/chorus/DB.ts`
-   Modifying data types (stored in a variety of places)
-   Adding or modifying TanStack Query queries in `src/core/chorus/API.ts`

## Coding style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meltylabs/chorus](https://github.com/meltylabs/chorus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
