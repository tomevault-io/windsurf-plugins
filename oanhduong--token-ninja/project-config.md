---
trigger: always_on
description: Notes for Claude Code (and other AI coding assistants) working inside this
---

# CLAUDE.md

Notes for Claude Code (and other AI coding assistants) working inside this
repository.

## What this project is

`token-ninja` is a CLI + MCP server that intercepts commands meant for an AI
coding assistant and runs the deterministic ones locally — zero LLM calls.
Anything it doesn't confidently recognize falls back to the user's AI tool
(Claude, Codex, Cursor, Aider, Gemini, Continue).

- 765 built-in rules across 46 tool domains
- Classifier hot path: ~19 µs/call; safety validator: ~10 µs/call (warm JIT)
- 312 tests across 21 test files; 90.67% lines / 84.61% branches /
  95.45% functions (v8 coverage over `src/router/**`, `src/safety/**`,
  `src/rules/**`; thresholds enforced at 88 / 82 / 88 / 95)

## Source layout

```
src/
  cli.ts                     # commander entry; subcommands: mcp, route, setup, uninstall, init, stats, doctor, shim, rules
  doctor/
    index.ts                 # `ninja doctor` health check (config, rules, shim, MCP, hook, stats)
  router/
    index.ts                 # runRouter(): safety → classify → safety (again) → exec-or-fallback
    route-once.ts            # one-shot classify+exec used by the Claude Code UserPromptSubmit hook (no AI fallback)
    classifier.ts            # exact → prefix → regex → nl matching; template expansion
    executor.ts              # execa wrapper (shell:true, captures + streams)
    fallback.ts              # hand off to user's AI tool
  rules/
    loader.ts                # loads builtin/*.yaml + ~/.config/token-ninja/rules/*.yaml
    types.ts                 # Rule / MatchSpec / ActionSpec
    builtin/*.yaml           # 765 rules, grouped by domain (46 files)
  safety/
    denylist.ts              # DENY_PATTERNS regex list (rm -rf, sudo, curl|sh, etc.)
    validator.ts             # pipeline split + NFKC + homoglyph strip, then match
  adapters/                  # per-AI-tool metadata + shim generator (claude-code, codex, cursor, aider, gemini, continue, generic)
  setup/
    index.ts                 # orchestrates `ninja setup`: shell shim + MCP register + hook install
    shell-install.ts         # writes managed block into ~/.bashrc / ~/.zshrc
    mcp-install.ts           # registers ninja mcp with Claude Code, Cursor, Claude Desktop, Gemini CLI
    hook-install.ts          # installs Claude Code UserPromptSubmit hook
  mcp/server.ts              # stdio MCP server exposing tool: maybe_execute_locally
  telemetry/stats.ts         # hit/miss counters, tokens-saved estimate
  config/user-config.ts      # ~/.config/token-ninja/config.yaml
  utils/
    shell-parse.ts           # splitPipelineSegments, tokenize, normalizeNl
    repo-detect.ts           # detect markers (package.json, Cargo.toml, …) and PMs
    logger.ts                # ANSI-colored stderr
hooks/
  claude-code-user-prompt.cjs # shipped hook: reads a prompt, calls `ninja route`, short-circuits the model on hit
tests/                       # 20 test files, 268 tests, vitest + v8 coverage
  fixtures/real-commands.txt # ≥85% of these must classify (rules-coverage.test.ts)
```

## Commands

```bash
npm run build            # tsc + copy YAML rules to dist/rules/builtin/
npm run dev              # tsc -w
npm test                 # vitest run (all 268 tests)
npm run test:watch
npm run test:coverage    # v8, thresholds: 85% lines, 80% branches, 95% functions
npm run lint             # eslint flat config
npm run typecheck        # tsc --noEmit
```

## Invariants the code relies on

- **Safety is checked twice.** Once on the raw input (`runRouter` start) and
  once on the resolved command (after template expansion). Both use the same
  `validate()`; both must pass.
- **Match order is strict**: exact → prefix → regex → nl. Longest-prefix
  wins inside the prefix phase. First confident match wins overall.
- **`args_passthrough: true`** means: if the rule's base command is a prefix
  of the user's input, run the full input (preserving flags like `-D`);
  otherwise normalize (`git st` → `git status`) and append captured args.
  Do not regress this — see `tests/classifier-templates.test.ts`.
- **User rules override builtins** by id. Loader warns on duplicates; the
  later-defined rule wins.
- **The rule cache is memoized.** Tests that load rules multiple times
  should call `invalidateRulesCache()` in `beforeEach` if they mutate state.

## Adding a rule

1. Find the right domain file in `src/rules/builtin/<domain>.yaml`.
2. Choose the narrowest match type: `exact` > `prefix` > `regex` > `nl`.
3. Choose the right safety tier: `read-only` < `write-confined` <
   `write-network` < `blocked`.
4. Add at least one fixture line in `tests/fixtures/real-commands.txt`.
5. `npm test` — the coverage suite enforces ≥85% hit rate.

## Adding a deny pattern

1. Add a regex to `DENY_PATTERNS` in `src/safety/denylist.ts`. Case-insensitive
   flag is enforced by a test.
2. Add a positive test to `tests/denylist.test.ts` or `tests/safety.test.ts`.
3. Add a negative test confirming a similar-looking safe command still passes.

## Things NOT to do

- Don't add runtime dependencies casually. Current deps: `commander`, `execa`,
  `yaml`, `@modelcontextprotocol/sdk`. Every new one is a supply-chain risk
  for a tool that runs shell commands.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oanhduong/token-ninja](https://github.com/oanhduong/token-ninja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
