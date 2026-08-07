---
trigger: always_on
description: OpenAI-compatible AI proxy for DeepSeek, Claude & ChatGPT — no API keys, real browser sessions
---

# ZeroKey

## PROJECT
 OpenAI-compatible AI proxy for DeepSeek, Claude & ChatGPT — no API keys, real browser sessions
 Node.js >= 18, Express 5, pnpm, SSE streaming

## DIRECTORY
 config/
  constants.js # CONFIG, MODEL_HASH, MODELS — single source of truth for models/ports
 core/
  chat-router.js # buildRouter → per-provider route builder dispatch
  session-selector.js # SessionSelector — TUI wizard for provider/user/session, live-credential validation, rate-limit awareness
  claude/
   api.js # ClaudeAPI — browser-session client, org-id extraction, stream completion, file upload
   stream-handler.js # claudeStreamHandler — SSE parsing, limit detection, summary fallback
   set-instructions.js # setClaudeInstructions — project+system-instructions upsert
  deepseek/
   api.js # DeepSeekAPI — PoW challenge solver, session CRUD, file upload with polling
   stream-handler.js # streamHandler — SSE parsing, auto-retry on stream close
   pow.js # DeepSeekPOW — WASM-based proof-of-work solver
  chatgpt/
   api.js # ChatGPTAPI — sentinel refresh, conduit token, prepare flow, file upload (Azure blob)
   stream-handler.js # chatgptStreamHandler — SSE parsing, session-id tracking
   pow.js # ChatGPTProofOfWork — sentinel proof token decode/generate/solve
  session-selector.js # session-selector.js
 engine/
  compiler.js # ToolCompiler — singleton per IDE×provider: uploadAndGetMessages, uploadAndFormatPrompt, uploadAndFormatPromptForRaw, buildPrompt, compile/parse/emit, matchSkill
  pipeline.js # StreamPipeline — SSE stream head: scanning, emitting, MCP injection, skill handling, error formatting
  instructions.js # Instructions — lazy-loads instructions.md + skills-extra.md, hash for change detection
  instructions.md # Base system prompt (agent rules, BPI syntax, execution model, output contract)
  skills-extra.md # Extra prompt appends (tool grammar, dynamic-tools listing)
  triggers.js # Skills: $cwd, $save, $test, $browser, $mcp, $mcp-dump; MCP auto-registration, passthrough, restore
  tool-defs.js # TOOLS — generic tool grammar + per-IDE mappings (vscode, terax, opencode), output shorteners
  mcp/
   browser.js # BROWSER_MCP — built-in browser MCP alias map
   inject.js # injectMcpAliases — registers MCP tools into compiler.tools
   auto.js # buildAutoAliasMaps, hashTools — auto-registration from mcp_<server>_<tool> naming
   playwright.js # playwrightMCP — Playwright MCP alias map
 routes/
  info.js # GET / — API info
  health.js # GET /health — uptime, user, provider, model
  models.js # GET /v1/models, GET /v1/models/:model — OpenAI-compatible model listing
  claude.js # POST /v1/chat/completions — Claude router: instructions, tools, limit handling
  deepseek.js # POST /v1/chat/completions — DeepSeek router: PoW, session creation, retry
  chatgpt.js # POST /v1/chat/completions — ChatGPT router: sentinel, prepare, instructions
 utils/
  cookie-jar.js # CookieJar — shared cookie store, seed/capture/serialize
  errors.js # classifyError, toOpenAIError — provider error → OpenAI-compatible error
  extract-files.js # decodeContentParts — base64 data-URI → Buffer[] for file upload
  find-port.js # findPort, isPortActive — port scanning
  har-to-capture.js # harToCapture — HAR JSON → network-capture format
  capture-request.js # captureRequest — dumps req.body to temp/captures/*.json ($req skill + unconditional on DeepSeek requests)
  ephemeral-session.js # ephemeralSession — clones session with chatSessionId/parentMessageId nulled, for ephemeral/utility calls
  sequential-queue.js # sequentialQueue — Express middleware serializing all requests through one app instance, one in flight at a time
  session-classifier.js # isRealChatSession — per-IDE fingerprinted system-prompt prefix match; default-deny classifies non-matching system-first calls as ephemeral
  logger.js # console color wrappers (debug, info, success, warn, error)
  rate-limiter.js # acquireSlot — per-provider rate limiting (5 req / 15s window)
  route-helpers.js # validateMessages — shared route middleware
  sse-reader.js # readSSE — generic SSE stream reader for both Web and Node streams
  sync-ide-config.js # syncIdeConfig — writes ZeroKey model entry into VS Code chatLanguageModels.json
 scripts/
  check-modules.js # Dependency integrity check
 server.js # Express app entry: session selection, IDE config sync, route mounting, graceful shutdown

## BUILD
 pnpm 10.13.1
 start: node server.js

## ENTRY-POINTS
 server.js # main entry: node server.js / pnpm start
 start.bat # Windows launcher
 zerokey.bat # npm runner for Windows
 zerokey.sh # npm runner for Unix

## MODULES
 server.js
  → express
  → routes/info, routes/health, routes/models, core/chat-router
  → core/session-selector
  → utils/find-port, utils/sync-ide-config, utils/logger, utils/errors, utils/sequential-queue
 chat-router.js
  → routes/claude, routes/chatgpt, routes/deepseek
 session-selector.js
  → prompts (TUI)
  → core/claude/api, core/deepseek/api, core/chatgpt/api
  → config/constants
 claude.js
  → engine/pipeline (StreamPipeline, passes messages → pipeline.session/rawMode)
  → core/claude/api, core/claude/stream-handler, core/claude/set-instructions
  → utils/rate-limiter, utils/route-helpers
 deepseek.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [downloaddoctor/zerokey](https://github.com/downloaddoctor/zerokey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
