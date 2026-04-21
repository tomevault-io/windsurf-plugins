---
trigger: always_on
description: **skillkit** is a Python library that implements Anthropic's Agent Skills functionality, enabling LLM-powered agents to autonomously discover and utilize packaged expertise. The library provides:
---

# skillkit

**skillkit** is a Python library that implements Anthropic's Agent Skills functionality, enabling LLM-powered agents to autonomously discover and utilize packaged expertise. The library provides:

- Multi-source skill discovery from personal directories, project directories, and plugins
- SKILL.md parsing with YAML frontmatter validation
- Progressive disclosure pattern (metadata loading → on-demand content)
- Framework integrations (LangChain, LlamaIndex, CrewAI, Haystack, Google ADK)
- Security features (path traversal prevention, permission checks)
- Model-agnostic design supporting Claude, GPT, Gemini, and open-source LLMs

## Development Approach

This project follows a **Vertical Slice MVP strategy** to deliver working functionality quickly:

- **v0.1 (Released)**: Core functionality + LangChain integration (sync only)
- **v0.2 (Released)**: Async support + multi-source discovery + plugin integration
- **v0.3 (Released)**: Script execution with security controls
- **v1.0 (Planned)**: Additional framework integrations + production polish + comprehensive documentation + 90% test coverage

### Current Focus (v0.4)

The v0.4 release implements advanced progressive disclosure with intelligent caching:

1. **LRU Content Cache**: In-memory cache with configurable size (default: 100 entries)
2. **Mtime-based Invalidation**: Automatic cache invalidation when SKILL.md files are modified
3. **Argument Normalization**: Whitespace variations map to same cache entry for maximum efficiency
4. **Thread-Safe Concurrency**: Per-skill asyncio locks enable safe parallel invocations
5. **Cache Management API**: get_cache_stats(), clear_cache() for monitoring and control
6. **Performance**: <1ms cache hits vs 10-25ms first invocation (up to 25x faster)
7. **Memory Efficient**: ~2.1KB per cached entry, ~5MB cache overhead
8. **High Hit Rate**: 80%+ cache hit rate achievable with typical usage patterns
9. **Script Integration**: Script detection integrates with Level 2 caching lifecycle
10. **Backward Compatible**: 100% compatible with v0.1/v0.2/v0.3 APIs

**What's deferred to v1.0+**: Additional framework integrations (LlamaIndex, CrewAI, Haystack), advanced argument schemas, CI/CD pipeline, 90% test coverage.

## Key Architectural Decisions

### Core Architecture (v0.1)
The foundation is built on 8 critical decisions (see `specs/001-mvp-langchain-core/research.md` for rationale):

1. **Progressive Disclosure Pattern**: Two-tier architecture (SkillMetadata + Skill) with lazy content loading achieves 80% memory reduction
2. **Framework-Agnostic Core**: Zero dependencies in core modules; optional framework integrations via extras
3. **$ARGUMENTS Substitution**: `string.Template` for security + standard escaping (`$$ARGUMENTS`), 1MB size limit, suspicious pattern detection
4. **Error Handling**: Graceful degradation during discovery, strict exceptions during invocation, 11-exception hierarchy
5. **YAML Parsing**: `yaml.safe_load()` with cross-platform support, detailed error messages, typo detection
6. **LangChain Integration**: StructuredTool with closure capture pattern
7. **Testing**: 70% coverage with pytest, parametrized tests, fixtures in conftest.py
8. **Performance-First Design**: Optimized for LLM-bound workflows

### v0.2 Enhancements
Additional architectural patterns added in v0.2:

1. **Async-First I/O**: Full async/await support with `aiofiles` for non-blocking file operations, enabling concurrent skill discovery and invocation
2. **Multi-Source Resolution**: Priority-based discovery (project:100, config:50, plugins:10, custom:5) with fully qualified names (`plugin:skill-name`)
3. **Plugin Architecture**: MCPB manifest parsing with namespace isolation and conflict resolution
4. **Secure Path Resolution**: Traversal prevention, symlink validation, and file reference security
5. **Backward Compatibility**: All v0.1 sync APIs preserved; async methods are additive (`discover()` + `adiscover()`)

**Security Validated**: All decisions reviewed against 2024-2025 Python library best practices (scores 8-9.5/10).

## Documentation

Main project documentation is located in the `.docs/` directory:

### `.docs/SKILL format specification`
- Full specification for skills and SKILL.md

### `.specify/` folder
This project was developed using speckit method. all development phases have been documented thoroughly inside `.specify/` folder

## Project Status

**Current Phase**: ✅ v0.4.0 RELEASED

**v0.1 Completed**:
- ✅ Core functionality (discovery, parsing, models, manager, processors)
- ✅ LangChain integration with StructuredTool (sync)
- ✅ Progressive disclosure pattern with lazy loading
- ✅ YAML frontmatter parsing and validation
- ✅ Argument substitution with security features
- ✅ 70%+ test coverage with comprehensive test suite
- ✅ Published to PyPI

**v0.2 Completed**:
- ✅ Full async/await support (`adiscover()`, `ainvoke_skill()`)
- ✅ Multi-source skill discovery (project, Anthropic config, plugins, custom paths)
- ✅ Plugin ecosystem with MCPB manifest support
- ✅ Priority-based conflict resolution
- ✅ Fully qualified skill names (`plugin:skill-name`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxvaega/skillkit](https://github.com/maxvaega/skillkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
