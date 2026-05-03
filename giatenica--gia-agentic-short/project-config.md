---
trigger: always_on
description: - **Claude 4.5 Family** via Anthropic API with task-based model selection
---



## Architecture

- **Claude 4.5 Family** via Anthropic API with task-based model selection
- **Cloudflare** Workers, KV, R2, D1 available for edge compute and storage
- **Microsoft Azure** available for additional cloud services

## Claude API Best Practices
1. **No hard token limits**: Let Claude use full context for best results
2. **Prompt caching**: Use cache control for stable system prompts when possible
3. **Efficient prompts**: Be specific, provide examples, use structured output formats
4. **Batch processing**: Use for non-urgent bulk tasks
5. **Extended thinking**: Enable for complex reasoning tasks (budget 16k+ tokens)
6. **Date awareness**: All agents include current date in system prompt
7. **Web search awareness**: Agents flag when they need current information

## Agent Framework (REQUIRED for new agents)
All new agents MUST:
1. Import from `src.agents.best_practices`
2. Use `build_enhanced_system_prompt()` or inherit from `BaseAgent`
3. Include current date context (automatic via BaseAgent)
4. Include web search awareness (automatic via BaseAgent)
5. Use `cache_ttl="ephemeral"` when caching system prompts
6. Follow model selection guidelines below

## Agent Registry
- Always add new agents to `src/agents/registry.py` (`AGENT_REGISTRY`)
- The registry currently includes `A01` through `A25`:
  - `A01`-`A04`: Intake and initial analysis
  - `A05`-`A09`: Literature and planning
  - `A10`-`A11`: Gap resolution
  - `A12`-`A15`: Quality and tracking
  - `A16`: Evidence extraction
  - `A17`-`A23`: Section writers
  - `A24`-`A25`: Data analysis execution and feasibility validation

## Agent Model Configuration
| Task Type | Model | Use Case |
|-----------|-------|----------|
| Complex Reasoning | `claude-opus-4-5-20251101` | Research, scientific analysis, academic writing |
| Coding/Agents | `claude-sonnet-4-5-20250929` | Default for most tasks, agents, data analysis |
| High-Volume | `claude-haiku-4-5-20251001` | Classification, summarization, extraction |
| Fallback | `openai/gpt-4.1-mini` | GitHub Models backup |

## Caching Configuration
| Cache Type | TTL | Use Case |
|------------|-----|----------|
| `ephemeral` | 5 min | Dynamic content, frequent updates |

Note: This repo currently uses `cache_ttl="ephemeral"`.

## Cloud Infrastructure Available
| Service | Provider | Use Case |
|---------|----------|----------|
| Workers | Cloudflare | Edge compute, API endpoints, scheduled tasks |
| KV | Cloudflare | Key-value storage, caching, session data |
| R2 | Cloudflare | Object storage (S3-compatible), large files |
| D1 | Cloudflare | SQLite database at edge |
| Azure | Microsoft | Additional compute, storage, AI services |

## Critical Rules for All Agents
1. **NEVER make up data, statistics, numbers, or facts**
2. **NEVER use emojis**
3. **NEVER use em dashes** (use semicolons, colons, or periods)
4. **ALWAYS include current date** in system prompts (automatic via BaseAgent)
5. **ALWAYS flag outdated knowledge** that needs web search
6. **ALWAYS cite sources** for quantitative claims

## Banned Words (NEVER USE)
delve, realm, harness, unlock, tapestry, paradigm, cutting-edge, revolutionize,
landscape, potential, findings, intricate, showcasing, crucial, pivotal, surpass,
meticulously, vibrant, unparalleled, underscore, leverage, synergy, innovative,
game-changer, testament, commendable, meticulous, highlight, emphasize, boast,
groundbreaking, align, foster, showcase, enhance, holistic, garner, accentuate,
pioneering, trailblazing, unleash, versatile, transformative, redefine, seamless,
optimize, scalable, robust (non-statistical), breakthrough, empower, streamline,
intelligent, smart, next-gen, frictionless, elevate, adaptive, effortless,
data-driven, insightful, proactive, mission-critical, visionary, disruptive,
reimagine, agile, customizable, personalized, unprecedented, intuitive,
leading-edge, synergize, democratize, automate, accelerate, state-of-the-art,
dynamic (non-technical), reliable, efficient, cloud-native, immersive, predictive,
transparent, proprietary, integrated, plug-and-play, turnkey, future-proof,
open-ended, AI-powered, next-generation, always-on, hyper-personalized,
results-driven, machine-first, paradigm-shifting, novel, unique, utilize, impactful

## Development Guidelines
- Use async/await patterns for all agent operations
- Implement proper error handling in workflows
- Add tracing for debugging multi-agent interactions
- Test agents independently before integration
- Write chapters sequentially (phased approach) for better memory management
- Do not mutate `sys.path` inside `src/` modules; if a script needs repo-root imports, do it in `scripts/` or run via `python -m`.
- Validate workflow inputs using `src.utils.validation.validate_project_folder()` and handle JSON/file read errors without crashing.

## Environment Loading
- Do not auto-load `.env` at import time inside `src/` modules.
- CLI entrypoints in `scripts/` may call `load_env_file_lenient()` to support local runs.
- When instantiating `ClaudeClient` directly, prefer explicit env configuration; `GIA_LOAD_ENV_FILE=1` is available as an opt-in convenience.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giatenica/gia-agentic-short](https://github.com/giatenica/gia-agentic-short) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
