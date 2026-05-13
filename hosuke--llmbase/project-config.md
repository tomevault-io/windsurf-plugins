---
trigger: always_on
description: - Domain-agnostic: no hardcoded domains, all structure emerges from content via LLM
---

# LLMBase Project Guidelines

## Architecture
- Domain-agnostic: no hardcoded domains, all structure emerges from content via LLM
- Three-layer: raw/ → wiki/concepts/ → config.yaml (Karpathy pattern)
- Trilingual by default: EN / 中文 / 日本語
- All wiki-links use [[target]] syntax; resolved via alias map (aliases.json)

## Code Patterns
- LLM calls go through llmwiki/llm.py:chat() — never call OpenAI directly
- Alias resolution via llmwiki/resolve.py — always use resolve_link() for wiki-link targets
- Taxonomy is LLM-generated (not hardcoded) — llmwiki/taxonomy.py
- Article slugs are pinyin/kebab-case; titles are bilingual "English / 中文"
- Never expose specific LLM provider names in public code or commits

## Customization Contract (for downstream projects)
Downstream projects can override module-level constants at import time to
customize behavior without forking functions. This is a **stable contract**.

| Module               | Constant                  | Purpose                                  |
|----------------------|---------------------------|------------------------------------------|
| llmwiki/compile.py     | SYSTEM_PROMPT             | LLM system message for compilation       |
| llmwiki/compile.py     | COMPILE_USER_PROMPT       | User prompt template ({title}, {content}, {existing}, {article_format}) |
| llmwiki/compile.py     | COMPILE_ARTICLE_FORMAT    | Example article format in user prompt    |
| llmwiki/compile.py     | SECTION_HEADERS           | Language sections for split/merge        |
| llmwiki/taxonomy.py    | TAXONOMY_SYSTEM_PROMPT    | LLM system message for taxonomy          |
| llmwiki/taxonomy.py    | TAXONOMY_LABEL_KEYS       | Language keys in label dicts             |
| llmwiki/taxonomy.py    | TAXONOMY_GENERATOR        | Callable to replace LLM taxonomy (or None) |
| llmwiki/lint/checks.py | ALLOW_CJK_SLUGS           | Accept CJK slugs as valid (bool)         |
| llmwiki/lint/checks.py | SYSTEM_PROMPT             | LLM system for deep lint                 |
| llmwiki/lint/fixes.py  | STUB_SYSTEM_PROMPT        | LLM system for stub generation           |
| llmwiki/search.py      | SEARCH_TOKENIZER          | Callable(text)->list[str] to replace tokenizer (or None) |
| llmwiki/search.py      | STOPWORDS / CJK_STOPWORDS | Stopword sets used by default tokenizer  |
| llmwiki/query.py       | SYSTEM_PROMPT             | LLM system message for Q&A               |
| llmwiki/query.py       | TONE_INSTRUCTIONS         | Dict of tone_id → instruction string     |
| llmwiki/query.py       | PROMOTE_SYSTEM_PROMPT     | LLM system for Q&A→concept promotion judge |
| llmwiki/query.py       | PROMOTE_CONTENT_EXAMPLE   | Content-schema hint for promote judge (None = auto-derive from SECTION_HEADERS) |
| llmwiki/query.py       | PROMOTE_TITLE_EXAMPLE     | Title-schema hint for promote judge (None = auto-derive from SECTION_HEADERS) |
| config.yaml          | query.prefilter_threshold | Above this many articles, TF-IDF prefilter the index before LLM selector (default 500) |
| config.yaml          | query.prefilter_top_k     | Number of candidates to keep after prefilter (default 200) |
| llmwiki/xici.py        | XICI_SYSTEM_PROMPT        | LLM system for guided introduction       |
| llmwiki/xici.py        | LANG_STYLES               | Dict of lang → style instruction         |
| llmwiki/entities.py    | ENTITY_SYSTEM_PROMPT      | LLM system for entity extraction         |
| llmwiki/entities.py    | ENTITY_PROMPT             | User prompt template for entities        |
| llmwiki/entities.py    | ENTITY_ARTICLE_FORMATTER  | Callable to format article list for LLM  |
| llmwiki/export.py      | (uses SECTION_HEADERS)    | Language sections from compile module    |
| llmwiki/normalize.py   | SENTENCE_TERMINATORS      | Line terminators for paragraph-merge (default CJK + ASCII) |
| llmwiki/normalize.py   | CLOSING_WRAPPERS          | Brackets/quotes that may follow a terminator before merge-check |
| llmwiki/chunk_cache.py | ChunkCache(base, subdir=) | Content-hash-validated (cid, content_hash)→output cache for pipelines |
| llmwiki/split.py       | split_by_heading(body, level) | Flat section parse — `list[Section]` at target ATX depth; no heuristics |
| llmwiki/pipeline/      | run_stage(base, stage, key, *, ttl, meta_init) | Contextmanager — guarantees `ok`/`failed`/`partial` terminal event on every exit; SIGKILL recovery via stale-lock `interrupted` |
| llmwiki/pipeline/      | rebuild_state(base, stage, key) → StageState | Derive status/attempts/started/finished/last_err/artifacts/meta from log (append-only JSONL is source of truth) |
| llmwiki/pipeline/      | StagePartialExit / ctx.mark_partial(reason) | Handler marks run `partial` (not `ok`/`failed`) — e.g. LLM quota at chunk 50/62; next run resumes from cache |
| llmwiki/pipeline/      | RESERVED_EVENTS | Event names refused by `ctx.log()` (start/ok/failed/partial/interrupted/artifact/meta_update) — prefix custom events with `chunk_` / `cache_` |
| llmwiki/llm.py         | chat_with_meta(prompt, ...) → (str, LLMMeta) | Rich-return chat with finish_reason / usage (incl. reasoning_tokens) / attempts. `meta.truncated` == length-cut; primitive does NOT raise — caller decides policy |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hosuke/llmbase](https://github.com/Hosuke/llmbase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
