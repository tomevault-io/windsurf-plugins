---
trigger: always_on
description: Instructions for Codex-style agents working in this repository.
---

# SigMap — AGENTS.md

Instructions for Codex-style agents working in this repository.

## Append Strategy (Required)

When writing generated signature content, never overwrite human-written notes above the marker.

Use this marker block for all appendable context files:

```
## Tools

<!-- sigmap-tools -->

```json
[
  {
    "name": "sigmap_ask",
    "description": "Rank source files by relevance to a natural-language query. Run before exploring the codebase.",
    "command": "sigmap ask \"$QUERY\""
  },
  {
    "name": "sigmap_validate",
    "description": "Validate SigMap config and measure context coverage. Run after changing config or source dirs.",
    "command": "sigmap validate"
  },
  {
    "name": "sigmap_judge",
    "description": "Score an LLM response for groundedness against source context. Use to verify answer quality.",
    "command": "sigmap judge --response \"$RESPONSE\" --context \"$CONTEXT\""
  },
  {
    "name": "sigmap_query",
    "description": "Rank all files by relevance using TF-IDF and write a focused mini-context.",
    "command": "sigmap --query \"$QUERY\" --context"
  },
  {
    "name": "sigmap_weights",
    "description": "Show learned file-ranking multipliers accumulated from past sessions.",
    "command": "sigmap weights"
  }
]
```

## Auto-generated signatures
<!-- Updated by gen-context.js -->
# Code signatures

## changes (last 5 commits — 0 seconds ago)
```
src/learning/weights.js                       +exportWeights  +importWeights  ~resetWeights
packages/adapters/codex.js                    ~write  ~format
packages/adapters/claude.js                   ~format  ~write
packages/adapters/gemini.js                   ~format
packages/adapters/copilot.js                  ~format
packages/adapters/cursor.js                   ~format
packages/adapters/openai.js                   ~format
packages/adapters/windsurf.js                 ~format
```

## packages

### packages/cli/index.js
```
module.exports = { CLI_ENTRY, run }
function run(argv, cwd) → void
```

### packages/adapters/index.js
```
module.exports = { getAdapter, listAdapters, adapt, outputsToAdapters }
function getAdapter(name) → { name: string, format: F
function listAdapters() → string[]
function adapt(context, adapterName, opts = {}) → string
function outputsToAdapters(outputs) → string[]
```

### packages/adapters/llm-full.js
```
module.exports = { name: 'llm-full', format, outputPath, write }
function outputPath(cwd)
function format(context, opts)
function write(context, cwd, opts)
```

### packages/core/README.md
```
h1 sigmap-core
h2 Installation
h2 Quick start
h2 API reference
h3 `extract(src, language)` → `string[]`
h3 `rank(query, sigIndex, opts?)` → `Result[]`
h3 `buildSigIndex(cwd)` → `Map<string, string[]>`
h3 `scan(sigs, filePath)` → `{ safe: string[], redacted: boolean }`
h3 `score(cwd)` → `HealthResult`
h2 Migration from v2.3 and earlier
h2 v3.0 — Multi-Adapter Architecture (released)
h2 Zero dependencies
code-fence bash
code-fence plain
code-fence js
code-fence ---
```

### packages/core/index.js
```
module.exports = { extract, rank, buildSigIndex, scan, score, adapt }
function _resolveExtractor(language)
function extract(src, language) → string[]
function rank(query, sigIndex, opts) → { file: string, score: nu
function buildSigIndex(cwd) → Map<string, string[]>
function scan(sigs, filePath) → { safe: string[], redacte
function score(cwd) → { * score: number, * grad
function adapt(context, adapterName, opts = {}) → string
```

### packages/adapters/codex.js
```
module.exports = { name, format, outputPath, write }
function format(context, opts = {}) → string
function outputPath(cwd) → string
function write(context, cwd, opts = {})
```

### packages/adapters/claude.js
```
module.exports = { name, format, outputPath, write }
function format(context, opts = {}) → string
function _confidenceMeta(opts)
function outputPath(cwd) → string
function write(context, cwd, opts = {})
```

### packages/adapters/gemini.js
```
module.exports = { name, format, outputPath, write }
function format(context, opts = {}) → string
function outputPath(cwd) → string
function write(context, cwd, opts = {})
function _confidenceMeta(opts)
```

### packages/adapters/copilot.js
```
module.exports = { name, format, outputPath, write }
function format(context, opts = {}) → string
function _confidenceMeta(opts)
function outputPath(cwd) → string
function write(context, cwd, opts = {})
```

### packages/adapters/cursor.js
```
module.exports = { name, format, outputPath }
function format(context, opts = {}) → string
function _confidenceMeta(opts)
function outputPath(cwd) → string
```

### packages/adapters/openai.js
```
module.exports = { name, format, outputPath }
function format(context, opts = {}) → string
function outputPath(cwd) → string
function _confidenceMeta(opts)
```

### packages/adapters/windsurf.js
```
module.exports = { name, format, outputPath }
function format(context, opts = {}) → string
function _confidenceMeta(opts)
function outputPath(cwd) → string
```

## src

### src/security/patterns.js
```
module.exports = { PATTERNS }
```

### src/security/scanner.js
```
module.exports = { scan }
function scan(signatures, filePath) → { safe: string[], redacte
```

### src/extractors/cpp.js
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manojmallick/sigmap](https://github.com/manojmallick/sigmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
