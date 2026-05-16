---
trigger: always_on
description: > **Important**: This file contains instructions for Claude Code AI assistant working on this codebase.
---

# Project Instructions for Claude Code AI

> **Important**: This file contains instructions for Claude Code AI assistant working on this codebase.

## Documentation Structure

This project has comprehensive documentation organized in the `docs/` folder:

```
docs/
├── README.md                    # Documentation index and navigation
├── planning/                    # Project planning and roadmap
│   ├── overview.md
│   ├── quick-start.md
│   ├── roadmap.md
│   └── cost-analysis.md
├── architecture/                # System design and architecture
│   ├── system-design.md
│   ├── database-schema.md
│   └── queue-system.md
├── ai/                         # AI integration guides
│   ├── claude-setup.md
│   ├── features.md
│   ├── semantic-search.md
│   └── prompt-engineering.md
├── transcription/              # Video transcription docs
│   ├── overview.md
│   ├── cobalt-setup.md
│   ├── whisper-setup.md
│   └── queue-processing.md
├── platforms/                  # Platform adapter guides
│   ├── adapter-architecture.md
│   ├── github.md
│   └── adding-platforms.md
├── deployment/                 # Deployment guides
│   ├── vercel.md
│   ├── workers.md
│   ├── docker.md
│   └── environment.md
└── api/                        # API documentation
    └── endpoints.md
```

## Research Latest Technology Documentation

### CRITICAL: Always Research Before Implementing

Before writing any code or making architectural decisions:

1. **Research official documentation** for the latest best practices:
   - Use WebSearch or WebFetch tools to find official docs
   - Check for breaking changes and deprecations
   - Verify API versions and compatibility
   - Look for official migration guides

2. **Check for latest versions**:
   - Prisma: https://www.prisma.io/docs
   - Hono: https://hono.dev/
   - Claude API: https://docs.anthropic.com/
   - OpenAI Whisper: https://platform.openai.com/docs/guides/speech-to-text
   - Vercel: https://vercel.com/docs
   - BullMQ: https://docs.bullmq.io/

3. **Verify best practices**:
   - Security patterns (authentication, rate limiting)
   - Performance optimizations
   - Error handling standards
   - TypeScript patterns
   - Testing strategies

4. **Research before suggesting alternatives**:
   - If suggesting a library/framework, research it first
   - Compare multiple options with official docs
   - Check maintenance status, community support
   - Verify compatibility with existing stack

### Example Research Workflow

```
User: "Add Redis caching"
Claude:
1. WebSearch "Redis TypeScript best practices 2025"
2. WebFetch official Redis client docs
3. Read docs/architecture/system-design.md
4. Propose implementation following both official and local patterns
```

## How to Read Documentation

### Before Starting Any Task

1. **Always read `docs/README.md` first** - It provides the complete documentation index
2. **Research official tech docs** for the technologies you'll be using
3. **Check relevant documentation sections** based on your task:
   - Adding features? → Read architecture and relevant platform docs
   - Fixing bugs? → Read system design and specific component docs
   - Deploying? → Read deployment guides
   - Working with AI? → Read ai/ folder docs

### Understanding the Project

Start with these in order:
1. `docs/planning/overview.md` - Project goals and features
2. `docs/architecture/system-design.md` - Technical architecture
3. `docs/architecture/database-schema.md` - Data models
4. `docs/planning/roadmap.md` - Implementation timeline

### For Specific Tasks

- **AI Features**: Read `docs/ai/` folder
- **Video Transcription**: Read `docs/transcription/` folder
- **Platform Integration**: Read `docs/platforms/adapter-architecture.md` first
- **Deployment Issues**: Read `docs/deployment/` and `docs/architecture/queue-system.md`

## Keeping Documentation Up-to-Date

### When to Update Documentation

Update docs when you:
- ✅ Add new features or components
- ✅ Change architecture or system design
- ✅ Modify database schema
- ✅ Update API endpoints
- ✅ Change deployment process
- ✅ Discover better practices
- ✅ Fix documentation errors or outdated information

### How to Update Documentation

1. **Identify affected files**:
   - Use `docs/README.md` to find relevant doc files
   - Check cross-references in related docs

2. **Update multiple related files**:
   - If you change database schema, update `docs/architecture/database-schema.md`
   - If you add an API endpoint, update `docs/api/endpoints.md`
   - If you modify deployment, update relevant `docs/deployment/*.md`

3. **Maintain consistency**:
   - Use the same terminology across all docs
   - Update code examples to match actual implementation
   - Fix broken cross-references
   - Update "Last Updated" dates

4. **Add cross-references**:
   - Link related documentation files
   - Add "Related Documentation" sections at the bottom
   - Use relative links: `[Queue System](../architecture/queue-system.md)`

5. **Update the index**:
   - If adding new doc files, update `docs/README.md`
   - Add to appropriate reading paths
   - Update status table if needed

### Documentation Update Checklist

**MANDATORY**: After EVERY code change, update documentation:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duongdev/karakeep-social-ai](https://github.com/duongdev/karakeep-social-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
