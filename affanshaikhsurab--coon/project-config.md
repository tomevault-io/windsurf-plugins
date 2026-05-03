---
trigger: always_on
description: COON (Code-Oriented Object Notation) is a **token-efficient compression format** for Dart/Flutter code designed to reduce LLM context window usage by 30-70%. It's a monorepo with Python and JavaScript SDKs, CLI tools, benchmarking suite, and Next.js documentation site.
---

# COON Project - AI Agent Instructions

## Project Overview

COON (Code-Oriented Object Notation) is a **token-efficient compression format** for Dart/Flutter code designed to reduce LLM context window usage by 30-70%. It's a monorepo with Python and JavaScript SDKs, CLI tools, benchmarking suite, and Next.js documentation site.

**Core Value Proposition**: Compress `class LoginScreen extends StatelessWidget {...}` (150 tokens) to `c:LoginScreen<StatelessWidget>;m:b S{a:B{t:T"Login"},...}` (45 tokens, 70% reduction) while maintaining lossless reversibility.

## Architecture

### Monorepo Structure
```
packages/
  ├── python/          # Python SDK (pip install coon)
  ├── javascript/      # JS/TS SDK (npm install coon-format)
  └── cli/             # Language-agnostic CLI (@coon/cli)
benchmarks/            # LLM comprehension testing
docs/                  # Next.js documentation site
spec/                  # Format specification & shared fixtures
```

**Critical Files**:
- [spec/SPEC.md](spec/SPEC.md) - Single source of truth for compression rules
- [spec/data/](spec/data/) - Canonical abbreviation mappings (widgets.json, properties.json, keywords.json)
- [spec/fixtures/conformance/](spec/fixtures/conformance/) - Cross-SDK validation tests

### Compression Pipeline (Strategy Pattern)

Both Python and JavaScript implementations follow identical architecture:

```
Code → Analyzer → Strategy Selector → Strategy → Compressed Output
                     ↓
              [Basic, Aggressive, AST-Based, Component-Ref]
```

**Strategies** ([packages/javascript/src/strategies/](packages/javascript/src/strategies/), [packages/python/src/coon/strategies/](packages/python/src/coon/strategies/)):
- `Basic`: Simple keyword/widget abbreviations (40-50% reduction)
- `Aggressive`: Maximum compression with whitespace removal (60-70%)
- `AST-Based`: Parse-aware optimization for complex code (300+ chars)
- `Component-Ref`: Registry-based for repeated patterns

**Key Pattern**: Strategies inherit from `CompressionStrategy` base class and implement `compress()` method. The `StrategySelector` auto-selects based on code size and complexity.

## Development Workflows

### Testing

**Python SDK**:
```bash
cd packages/python
pytest tests/                    # All tests
pytest tests/test_compressor.py  # Specific test
```

**JavaScript SDK**:
```bash
cd packages/javascript
npm test                         # Jest tests
npm run test:conformance         # Cross-SDK validation
```

**Cross-SDK Conformance** (CRITICAL for consistency):
```bash
python scripts/run_conformance.py  # Validates both SDKs against spec/fixtures/conformance/
```

### Benchmarks

**LLM Comprehension Testing** ([benchmarks/](benchmarks/)):
```bash
cd benchmarks
npm run benchmark:comprehension       # Test Gemini, Groq, OpenRouter models
npm run benchmark:compression         # Token efficiency metrics
npm run benchmark:full                # Complete suite
```

**Important**: Benchmarks require API keys in `.env` (GOOGLE_GENERATIVE_AI_API_KEY, GROQ_API_KEY, OPENROUTER_API_KEY). Use `DRY_RUN=true` for testing without API calls.

### Documentation Site

```bash
cd docs
npm run dev      # Local development (http://localhost:3000)
npm run build    # Static generation
```

Content lives in [docs/guide/](docs/guide/), [docs/reference/](docs/reference/), [docs/ecosystem/](docs/ecosystem/) as Markdown files.

## Project-Specific Conventions

### Abbreviation System

**NEVER hardcode abbreviations** - Always load from [spec/data/](spec/data/):

```typescript
// ✅ Correct
import { loadWidgets } from './data';
const widgets = loadWidgets();  // Loads from spec/data/widgets.json

// ❌ Wrong
const widgets = { "Scaffold": "S", "Column": "C" };  // Will drift from spec
```

**Adding New Abbreviations**:
1. Update [spec/data/widgets.json](spec/data/widgets.json) or [spec/data/properties.json](spec/data/properties.json)
2. Run conformance tests: `python scripts/run_conformance.py`
3. Both SDKs automatically pick up changes (data module caches are cleared)

### Reversibility is Sacred

Every compression **MUST** be reversible:

```typescript
const original = "class MyWidget extends StatelessWidget {...}";
const compressed = compressor.compress(original);
const decompressed = decompressor.decompress(compressed);
assert(normalize(original) === normalize(decompressed));  // Must pass
```

Use `CompressionValidator` ([packages/javascript/src/utils/validator.ts](packages/javascript/src/utils/validator.ts)) to verify:
```typescript
const validator = new CompressionValidator();
const result = validator.validate(original, compressed);
console.log(result.reversible);  // Must be true
```

### Token Counting

**Consistent across codebase**: 4 characters ≈ 1 token (LLM tokenizer approximation)

```python
# Python
def count_tokens(text: str) -> int:
    return len(text) // 4
```

```typescript
// JavaScript
function countTokens(text: string): number {
  return Math.floor(text.length / 4);
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AffanShaikhsurab/COON](https://github.com/AffanShaikhsurab/COON) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
