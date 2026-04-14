---
trigger: always_on
description: You are an AI coding assistant working on a Screeps bot with swarm-based architecture. This file provides instructions for autonomous and manual development.
---

# AI Agent Instructions for Screeps Bot Development

## Context

You are an AI coding assistant working on a Screeps bot with swarm-based architecture. This file provides instructions for autonomous and manual development.

**Primary Documentation**: See `AGENTS.md` for complete workflows and `ROADMAP.md` for architecture.

---

## Core Principles

1. **Follow ROADMAP.md** - Single source of truth for architecture
2. **Required Code Only** - Remove unused code completely, no dead code
3. **Verify with MCP** - Always fact-check Screeps API using MCP servers
4. **Document Decisions** - Use TODO comments for future work
5. **Measure Impact** - Base decisions on metrics and data

---

## Code Philosophy

### Required Code Only

- ❌ No config flags to disable features - remove entirely
- ❌ No commented-out code - use git history
- ❌ No "just in case" implementations
- ✅ Remove unused imports, functions, classes immediately
- ✅ Delete entire subsystems if not used
- ✅ Trust git history for removed functionality

### Code Quality

- Use TypeScript strict mode
- Write comprehensive tests (>80% coverage)
- Add JSDoc comments for public APIs
- Handle all error cases
- Validate inputs from external sources

---

## MCP Servers (MANDATORY)

**ALWAYS verify Screeps API before coding**. Use these MCP servers:

### 1. screeps-mcp (Live Game)
- `screeps_console` - Execute commands
- `screeps_memory_get/set` - Memory operations
- `screeps_room_*` - Room data
- `screeps_stats` - Performance metrics

### 2. screeps-docs-mcp (Official Docs)
- `screeps_docs_get_api` - API documentation
- `screeps_docs_search` - Search docs
- `screeps_docs_get_mechanics` - Game mechanics

### 3. screeps-typescript-mcp (Types)
- `screeps_types_get` - Type definitions
- `screeps_types_search` - Search types
- `screeps_types_related` - Type relationships

### 4. screeps-wiki-mcp (Community)
- `screeps_wiki_search` - Search strategies
- `screeps_wiki_get_article` - Get articles
- Community best practices

### 5. grafana-mcp (Monitoring)
- `query_prometheus` - Query metrics
- `query_loki_logs` - Query logs
- `get_dashboard_by_uid` - View dashboards

### Verification Protocol

```
Before coding → Verify API (docs/types)
During dev → Test with live tools
For strategy → Research wiki
For performance → Monitor Grafana
When uncertain → Search MCP servers
```

---

## TODO Comments

TODO comments are **automatically converted to GitHub issues**. Use liberally!

### When to Use

- ✅ Placeholders for future implementation
- ✅ Out of scope work
- ✅ Error documentation
- ✅ Future enhancements
- ✅ Breaking down large features

### Format

```typescript
// TODO: Brief description
// Context: Additional details
// See: ROADMAP.md Section X
```

### Examples

```typescript
// TODO: Implement advanced pathfinding with A*
// Should consider terrain costs and traffic
// See ROADMAP.md Section 20 for requirements

// TODO: TypeError - Cannot read property 'pos' of undefined
// Details: Structure destroyed before access
// Suggested Fix: Add null check
```

---

## Autonomous Development

### When to Proceed Autonomously

✅ **Safe to proceed**:
- Bug fixes with clear root cause
- Performance optimizations (proven techniques)
- Code cleanup/refactoring (well-tested)
- Implementing wiki patterns
- Incremental improvements

⚠️ **Require human review**:
- New game mechanics
- Major architectural changes
- Multi-system changes
- Experimental approaches
- Unclear impact

❌ **Never autonomous**:
- Could cause global reset
- Critical safety systems
- No proper testing
- During active combat
- Violates ROADMAP.md

### Autonomous Loop

```
1. OBSERVE - Gather metrics and game state
2. ANALYZE - Identify opportunities (priority matrix)
3. PLAN - Design solution (verify with MCP)
4. IMPLEMENT - Write code (tests + docs)
5. VALIDATE - Test thoroughly
6. DEPLOY - Create PR
7. MONITOR - Track impact (24-48h)
```

### Decision Matrix

| Impact | Effort | Priority | Action |
|--------|--------|----------|--------|
| High | Low | P0 | Implement immediately |
| High | Medium | P1 | Implement soon |
| High | High | P2 | Plan carefully |
| Medium | Low | P1 | Quick wins |
| Medium | Medium | P2 | Schedule |
| Low | Any | P3 | Defer |

### Key Metrics

Monitor these thresholds:

| Metric | Warning | Action |
|--------|---------|--------|
| CPU Usage | >80% | Optimize hot paths |
| GCL Progress | <0.01/tick | Improve upgraders |
| Error Rate | >1/tick | Fix bugs immediately |
| Creep Count | <50 or >200 | Adjust spawning |
| Energy Efficiency | <80% | Optimize harvesting |
| Bucket Level | <5000 | Reduce CPU urgently |

---

## Development Patterns

### API Verification

```typescript
// ❌ DON'T assume
creep.moveTo(target);

// ✅ DO verify first
// Verified with screeps_docs_get_api("Creep")
const result = creep.moveTo(target, {
  reusePath: 5,
  visualizePathStyle: { stroke: '#ffffff' }
});

if (result === ERR_NOT_IN_RANGE) {
  // Handle error
}
```

### Performance-Critical Code

```typescript
/**
 * Optimized pathfinding with caching
 * Verified: screeps_docs_get_api, screeps_types_get
 * Performance: ~0.1 CPU (cached), ~0.5 CPU (new)
 */
export function optimizedMoveTo(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ralphschuler) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
