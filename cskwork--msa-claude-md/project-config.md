---
trigger: always_on
description: Generate minimal (<100 line) root CLAUDE.md + per-service CLAUDE.md files for MSA projects. Auto-detects tech stacks, uses See (not @) for zero-cost context loading, domain grouping for 10+ services. Use when setting up or auditing CLAUDE.md in microservice architectures.
---


# MSA CLAUDE.md Generator

Generate a **minimal root CLAUDE.md** (< 100 lines) as navigation hub, plus **per-service CLAUDE.md** files. All details are doc-referenced, never inlined.

## When to Activate

- User asks to "set up CLAUDE.md" or "generate CLAUDE.md" for a project with multiple services
- User has a monorepo or MSA project and wants context-optimized Claude Code instructions
- User asks to "audit" or "fix" existing CLAUDE.md in a microservice architecture
- User mentions "context optimization" or "too many services" in relation to CLAUDE.md
- Project contains 3+ services detected by build files (pom.xml, package.json, go.mod, etc.)

## Core Philosophy

1. **Root = index, not encyclopedia** -- service registry + shared conventions only
2. **Per-service = autonomous** -- each service CLAUDE.md is self-contained
3. **Conditional loading via `See`** -- Claude reads on-demand, zero startup cost
4. **< 100 lines root** -- every line must pass: "would removing this cause Claude to make mistakes?"
5. **Context isolation** -- backend services don't need frontend guides, and vice versa

## CRITICAL: `@` vs `See` References

| Syntax | Behavior | Context Cost |
|--------|----------|-------------|
| `@docs/guide.md` | Loads into memory at startup | **Full file size** |
| `See docs/guide.md` | Claude reads on-demand when needed | **Zero until accessed** |

**ALWAYS use `See` (without `@`)** for doc references in CLAUDE.md files.

Source: [Monorepo CLAUDE.md Organization](https://dev.to/anvodev/how-i-organized-my-claudemd-in-a-monorepo-with-too-many-contexts-37k7)

## Workflow

### Phase 1: Discover MSA Structure

Scan the project to identify services and their tech stacks:

```bash
find . -maxdepth 3 \( \
  -name "pom.xml" -o -name "build.gradle*" \
  -o -name "package.json" -o -name "requirements.txt" -o -name "pyproject.toml" \
  -o -name "go.mod" -o -name "Cargo.toml" \
  -o -name "Dockerfile" -o -name "docker-compose*.yml" \
\) 2>/dev/null | head -60
```

**Stack detection rules:**

| Indicator | Backend Stack |
|-----------|--------------|
| `pom.xml` or `build.gradle` + Spring deps | Java / Spring Boot |
| `package.json` + express/nestjs/fastify | Node.js / Express or NestJS |
| `requirements.txt` or `pyproject.toml` + fastapi/django/flask | Python / FastAPI or Django |
| `go.mod` | Go |
| `Cargo.toml` | Rust |

| Indicator | Frontend Stack |
|-----------|---------------|
| `package.json` + vue | Vue.js |
| `package.json` + react or next | React / Next.js |
| `package.json` + svelte or @sveltejs | Svelte / SvelteKit |
| `package.json` + angular | Angular |
| `package.json` + nuxt | Nuxt.js |

Identify:
- Service directories and their detected tech stacks
- Shared libraries / common modules
- Infrastructure configs (docker-compose, k8s, terraform)
- API contract files (OpenAPI specs, protobuf)
- Existing documentation (docs/, README files)

### Phase 2: Generate Root CLAUDE.md (< 100 lines)

The root file is a **navigation hub only**. For 10+ services, use domain grouping.

```markdown
# <Project Name>

<One-line description of the system>

## Service Domains

### Core
| Service | Path | Stack | Port |
|---------|------|-------|------|
| `auth` | `services/auth/` | <detected stack> | <port> |
| `user` | `services/user/` | <detected stack> | <port> |

### Business
| Service | Path | Stack | Port |
|---------|------|-------|------|
| `order` | `services/order/` | <detected stack> | <port> |
| `payment` | `services/payment/` | <detected stack> | <port> |

### Frontend
| Service | Path | Stack | Port |
|---------|------|-------|------|
| `admin` | `frontends/admin/` | <detected stack> | <port> |
| `customer` | `frontends/customer/` | <detected stack> | <port> |

Full service catalog: See `docs/services.md`

## Quick Commands

| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start all services |
| `<stack-specific run command>` | Start single backend service |
| `<stack-specific dev command>` | Start single frontend |

## Shared Conventions

- API specs: See `docs/api/`
- Shared libs: See `libs/common/`
- Env config: `.env.example` per service
- New service setup: See `docs/service-template.md`

## Cross-Service Patterns

- Sync: REST / gRPC -- See `docs/api-contracts.md`
- Async: Message queue -- See `docs/events.md`
- Auth: JWT / session -- See `docs/auth-flow.md`
- Observability -- See `docs/observability.md`

## Gotchas

- <critical cross-service startup order>
- <shared env vars across services>

Each service has its own `CLAUDE.md` -- auto-loaded when working in that directory.
```

**Rules:**
- **Domain grouping**: Group services by domain (Core / Business / Frontend / Infra) when > 6 services
- Service table: 4 columns only (`Service | Path | Stack | Port`)
- If > 15 services: show top 8-10, point to `docs/services.md` for full catalog
- Quick commands: max 5 rows, use detected stack commands
- All doc references use **`See` not `@`**
- **Total must be < 100 lines -- count with `wc -l`**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cskwork/msa-claude-md](https://github.com/cskwork/msa-claude-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
