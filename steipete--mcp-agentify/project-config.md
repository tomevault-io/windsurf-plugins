---
trigger: always_on
description: - **Core Principle:** When UI behavior doesn't match backend changes, systematically isolate the issue: frontend (caching, request payload), server (CWD, request handling, service initialization), or E2E interaction tools (Playwright state).
---

- **Core Principle:** When UI behavior doesn't match backend changes, systematically isolate the issue: frontend (caching, request payload), server (CWD, request handling, service initialization), or E2E interaction tools (Playwright state).

- **Frontend Caching & Stale Bundles:**
  - **Symptom:** Backend code is updated, but UI behaves as if using old code. Server logs show correct processing, but UI shows old errors or data.
  - **High Probability Cause:** Browser or Playwright is serving/using a stale frontend JavaScript bundle.
  - **Robust Fix Workflow:**
    1.  **Stop Dev Server(s):** Halt the `mcp-agentify` server and any separate frontend dev server (e.g., Parcel watch mode).
    2.  **Clear Frontend Caches (Claude Code can do this):**
        -   `rm -rf .parcel-cache` (if Parcel is used)
        -   `rm -rf frontend/.parcel-cache` (if Parcel cache is nested)
    3.  **Delete Old Build Output (Claude Code can do this):**
        -   `rm -rf frontend/public` (or your configured Parcel `--dist-dir`)
        -   Alternatively, `rm -rf dist` if that's your main output for frontend assets.
    4.  **Force Full Rebuild (Claude Code can do this):**
        -   Execute the clean frontend build script from `package.json` (e.g., `npm run build:ui` which might be `cd frontend && parcel build src/index.html --dist-dir public --public-url ./ --no-cache`).
    5.  **Verify Build Output:** Check timestamps in the output directory (`frontend/public`) to ensure new files were generated.
    6.  **Restart Server.**
    7.  **Test with Playwright:** **CRITICAL:** Use `playwright_close` before `playwright_navigate` to guarantee a completely fresh browser context, devoid of any prior session's cache or state.
    8.  **Test with Manual Browser:** Use an incognito window or manually clear browser cache for `localhost` to ensure a fresh load.

- **Server Current Working Directory (CWD) Issues:**
  - **Symptom:** Server fails to start with `ERR_MODULE_NOT_FOUND` for relative script paths (e.g., `./src/cli.ts`), or `FrontendServer` fails with `ENOENT` trying to serve static files from an incorrect base path (e.g., user home `~` instead of project root).
  - **Cause:** The Node.js process CWD is not the project root. This is common when scripts are launched via tools like `npx tsx` from a shell where the CWD wasn't explicitly set for the final command execution environment.
  - **Fixes & Best Practices:**
    1.  **Launch Command (`run_terminal_cmd`, iTerm tools):** *Always* prepend `cd /path/to/your/project_root &&` to your `npx tsx ./src/your_script.ts` command. This sets the CWD for the shell that invokes `npx`.
    2.  **`src/cli.ts` (Entry Point):** Capture `process.cwd()` at the very start. This should reflect the project root due to the `cd` above. Pass this `projectRoot` string via `initialCliOptions` to `startAgentifyServer`.
        ```typescript
        // src/cli.ts (in main())
        const projectRoot = process.cwd();
        logger.info({ projectRoot }, 'Captured project root in cli.ts');
        const initialCliOptions = { projectRoot, /* ...other options */ };
        await startAgentifyServer(initialCliOptions);
        ```
    3.  **`src/server.ts` (`startAgentifyServer`):** Receive `projectRoot` from `initialCliOptions`. Pass it to `FrontendServer`'s constructor via its `initialConfig` parameter. Also store it in `internalGatewayOptions`.
    4.  **`src/frontendServer.ts` (Constructor):**
        -   Receive `projectRoot` via `initialConfig.projectRoot`.
        -   **Force CWD (Defensive):** `if (this.projectRoot) { process.chdir(this.projectRoot); }`. Log success/failure of `chdir`. This ensures subsequent relative path operations within `FrontendServer` (like `express.static`) use the correct base.
        -   **Static Path Resolution:** `const staticPath = path.resolve(this.projectRoot || process.cwd(), 'frontend/public');`. Log all components (`this.projectRoot`, `process.cwd()`, `staticPath`) to confirm.
    5.  **Verify with Logs:** Add `console.error(\`[SUPER EARLY CLI] CWD: \${process.cwd()}\`);` at the top of `cli.ts` and check server logs. In `FrontendServer` constructor and `setupExpressMiddleware`, log `this.projectRoot` and `process.cwd()` to confirm they align with the intended project root.

- **Debugging API Request Payloads (UI -> Server):**
  - **Symptom:** Server returns HTTP 400 Bad Request, or handler complains about missing parameters, even though UI *seems* to send correct data.
  - **Fix/Debug Workflow:**
    1.  **Client-Side (`ChatTab.tsx` or similar):**
        -   `console.log('Request body being sent:', JSON.stringify(bodyObject, null, 2));` right before the `fetch` call.
    2.  **Server-Side API Handler (`FrontendServer.ts`):**
        -   At the very start of the handler: `this.logger.info({ rawBody: req.body, headers: req.headers }, 'Raw request received');`
        -   After attempting to parse/access expected fields: `this.logger.info({ parsedField1: body?.field1, parsedField2: body?.field2 }, 'Parsed fields');`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steipete/mcp-agentify](https://github.com/steipete/mcp-agentify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
