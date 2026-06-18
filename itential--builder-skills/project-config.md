---
trigger: always_on
description: This project contains skills for assisting developers on the Itential Platform. Read this first, then use the skills for detailed API references.
---

# Itential Platform - AI Agent Guide

This project contains skills for assisting developers on the Itential Platform. Read this first, then use the skills for detailed API references.

## Skill Router

Each skill owns a domain. **Invoke the skill using the Skill tool before working in that domain.** Skills contain the correct API methods, request bodies, response shapes, and patterns. Don't guess — load the skill.

| Skill | Agent | When to Use |
|-------|-------|-------------|
| `/explore` | — | Explore a platform freely — auth, discover, browse, build freestyle. |
| `/spec-agent` | **Spec Agent** | Start a delivery from a spec. Owns Requirements stage. |
| `/project-to-spec` | — | Read an existing project → produce customer-spec.md + solution-design.md. |
| `/documentation` | — | Survey global platform assets → discover relationships → group by use case → produce HLD+LLD per use case → optionally create projects and move assets in. For a specific named project, redirect to `/project-to-spec`. |
| `/flowagent-to-spec` | — | Read a FlowAgent → produce customer-spec.md as a deterministic workflow spec. |
| `/solution-arch-agent` | **Solution Architecture Agent** | Feasibility assessment + solution design. Runs after Requirements. |
| `/builder-agent` | **Builder Agent** | Build all assets, run tests, produce as-built record. |
| `/iag` | — | Automation Gateway: IAG services (Python, Ansible, OpenTofu). |
| `/flowagent` | — | AI Agents: configure LLM providers, tools, missions. |
| `/itential-mop` | — | Command templates with validation rules. |
| `/itential-devices` | — | Devices, backups, diffs, device groups. |
| `/itential-golden-config` | — | Golden config, compliance, grading, remediation. |
| `/itential-inventory` | — | Device inventories, nodes, actions, tags. |
| `/itential-lcm` | — | Resource models, instances, lifecycle actions. |

### Delivery Lifecycle

Spec-based delivery follows five stages. Each stage has a named agent, a clear input, and a deliverable.

```
Requirements  →  Feasibility  →  Design  →  Build  →  As-Built
      │                │              │          │           │
  Spec Agent   Solution Architecture  Solution   Builder     Builder
                     Agent           Architecture Agent       Agent
                                      Agent
      │                │              │          │           │
  customer-        feasibility.md  solution-    assets/    as-built.md
  spec.md          (assessment     design.md    configs    (delivered state,
  (approved)       + decision)     (approved)  (delivered)  deviations,
                                                            learnings)
                                                           ↳ design updates
                                                           ↳ spec amendments
```

**Deliverables:**

| Deliverable | Artifact | Produced by | Audience |
|-------------|----------|-------------|----------|
| HLD | `customer-spec.md` | Spec Agent | Customer / stakeholder |
| Feasibility Assessment | `feasibility.md` | Solution Architecture Agent | Customer / architect |
| Solution Design / LLD | `solution-design.md` | Solution Architecture Agent | Engineer / delivery team |
| As-Built | `as-built.md` | Builder Agent | Customer / delivery / support / system of record |

**Explore path** (no spec, no delivery lifecycle):
```
/explore → auth → pull platform data → summarize → use skills directly
```

**IMPORTANT: Invoke skills using the Skill tool** — don't just reference them in text. When you need to build workflows/templates, invoke `/builder-agent`. The skills contain the API details you need. Without loading them, you're guessing.

### Auth Reuse — Authenticate Once, Reuse Everywhere

**Auth happens when first needed** — in `/explore` (explore path) or in `/solution-arch-agent` during Feasibility. The token is saved to `{use-case}/.auth.json`. Every subsequent skill should:
1. Read `{use-case}/.auth.json` for `platform_url`, `auth_method`, and `token`
2. Use the token for all API calls (Bearer header for OAuth, query param for local)
3. On auth error (401/403): re-authenticate silently — see procedure below
4. **Never ask the user for credentials if `.env` exists**

This means the user authenticates once and every subsequent skill just works.

**Token expiry — silent re-auth procedure:**

When any API call returns 401 or 403, do not stop and do not ask the user. Re-authenticate silently:

1. Read credentials from `{use-case}/.env` (or `${CLAUDE_PLUGIN_ROOT}/environments/*.env` if no use-case `.env`)
2. Call the appropriate auth endpoint:
   - **OAuth:** `POST {PLATFORM_URL}/oauth/token` with `Content-Type: application/x-www-form-urlencoded` and body `grant_type=client_credentials&client_id={CLIENT_ID}&client_secret={CLIENT_SECRET}`
   - **Local/password:** `POST {PLATFORM_URL}/login` with `{"username": "...", "password": "..."}`
3. Write the new token back to `{use-case}/.auth.json`
4. Retry the failed request with the new token

If `.env` does not exist and re-auth is needed, then and only then ask the user for credentials.

### Key Rule: Look Up Before You Act — Don't Guess


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itential/builder-skills](https://github.com/itential/builder-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
