---
trigger: always_on
description: <!-- This AGENTS.md file is generated. Look for an agents.md package.json script to see what files to update instead. -->
---

<!-- This AGENTS.md file is generated. Look for an agents.md package.json script to see what files to update instead. -->

this is the playwriter codebase

the extension uses chrome.debugger to manage the user browser

read ./README.md for an overview of how this extension and mcp work
read playwriter/src/skill.md to understand the MCP docs (source of truth)

## backward compatibility

breaking changes to the WS protocol MUST never be made. publishing the extension code will never be instant, which means the extension must keep working with newer versions of the MCP and WS relay server.

## architecture

- user installs the extension in chrome. we assume there is only one chrome window for now, the first opened.
- extension connects to a websocket server on port 19988. if this server is not yet open, it retries connecting in a loop
- the MCP spawns the ws server if not already listening on 19988, in background. the mcp then connects to this same server with a playwright client
- the server exposes /cdp/client-id which is used by playwright clients to communicate with the extension
- the extension instead connects to /extension which is used to receive cdp commands and send responses and cdp events.
- some events are treated specially for example because
  - we need to send attachedToTarget to let playwright know which pages are available
  - we need to send detachedFromTarget when we disable the extension in a tab
  - a few more events need custom handling
- tabs are identified by sessionId or targetId (CDP concepts) or tabId (chrome debugger concept only)

mcp.ts MUST never use console.log. only console.error

write code that will run on all platforms: mac, linux, windows. especially around paths handling and command execution

## development

### running MCP locally

to test the MCP server with local changes, add it to your MCP client config with tsx:

```json
{
  "mcpServers": {
    "playwriter": {
      "command": "tsx",
      "args": ["/path/to/playwriter/playwriter/src/mcp.ts"]
    }
  }
}
```

make sure you have tsx installed globally: `pnpm i -g tsx`

### running CLI locally

to test CLI changes without publishing:

```bash
 # mac/linux: kill any existing relay on 19988
 PIDS=($(lsof -ti :19988))
 if [ ${#PIDS[@]} -gt 0 ]; then kill "${PIDS[@]}"; fi
 # verify port is free (must print nothing)
 lsof -ti :19988

 # windows (powershell): kill any existing relay on 19988
 Get-NetTCPConnection -LocalPort 19988 | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
 # verify port is free (must print nothing)
 Get-NetTCPConnection -LocalPort 19988 -ErrorAction SilentlyContinue

 tsx playwriter/src/cli.ts -s 1 -e "await page.goto('https://example.com')"
 tsx playwriter/src/cli.ts -s 1 -e "console.log(await snapshot({ page }))"
 tsx playwriter/src/cli.ts session new
 tsx playwriter/src/cli.ts -s 1 -e "await page.click('button')"
```

### reloading extension during development

after making changes to extension code:

```bash
pnpm --filter mcp-extension reload  # builds and opens chrome://extensions page
```

then click the reload button on the extension card in Chrome. the extension has a stable dev ID (`pebbngnfojnignonigcnkdilknapkgid`) so you don't need to reconfigure anything.

## extension version

after EVERY change made inside extension/ folder you MUST bump the manifest.json version and update the CHANGELOG.md file. then create a git tag with extension@version after committing.

### testing

```bash
pnpm test              # run all tests (takes ~90 seconds)
pnpm test -t "screenshot"  # run specific test by name
pnpm test:watch        # watch mode
```

tests run against a real Chrome instance with the extension loaded.

the test script passes `-u` to update inline snapshots automatically.

#### test setup

tests use these utilities from `test-utils.ts`:

```ts
// setup browser with extension loaded + relay server
const testCtx = await setupTestContext({
  port: 19987,
  tempDirPrefix: 'pw-test-',
  toggleExtension: true, // creates initial page with extension enabled
})

// get extension service worker to call extension functions
const serviceWorker = await getExtensionServiceWorker(testCtx.browserContext)

// toggle extension on current tab
await serviceWorker.evaluate(async () => {
  await globalThis.toggleExtensionForActiveTab()
})

// cleanup after tests
await cleanupTestContext(testCtx, cleanup)
```

to test MCP tools, create an MCP client:

```ts
import { createMCPClient } from './mcp-client.js'

const { client, cleanup } = await createMCPClient({ port: 19987 })
const result = await client.callTool({
  name: 'execute',
  arguments: { code: 'await page.goto("https://example.com")' },
})
```

#### adding tests

tests live in `playwriter/src/*.test.ts`. add new tests to existing describe blocks or create new test files.

each test should reset the extension connection. NEVER call `browser.close()` in tests.

remember: toggling extension on a tab adds it to available pages. if you toggle then call `context.newPage()`, you'll have 2 pages.

IMPORTANT: set bash timeout to at least 300000ms (5 minutes) when running `pnpm test`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remorses/playwriter](https://github.com/remorses/playwriter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
