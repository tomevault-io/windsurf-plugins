---
trigger: always_on
description: Deploy it right. Share it easily.
---

# CLAUDE.md

## Vision

Deploy it right. Share it easily.

A strong protocol means nothing if the deployment leaks — an open port, a TLS mismatch, a fingerprint that gives the server away. Meridian deploys the strongest available protocol AND configures every layer correctly: firewall, certificates, SNI routing, fingerprinting. Your server is indistinguishable from any other website. One command. Done right.

**The north star:** always offer the strongest available protocol with an airtight deployment. Today that's VLESS+Reality. Tomorrow the protocol may change, but the promise stays: undetectable, zero-mistakes deploy, effortless client handoff.

**Audience:** deployers (self-hosters who want correctness out of the box) + their users (people who scan a QR code and connect)

**Design principles:**
- **Strongest protocol, always** — research and adopt whatever is most resistant to detection. Don't chase breadth (20 protocols); chase depth (the best one, perfectly deployed). No open ports, no TLS leaks, no fingerprinting mistakes.
- **It just works** — Meridian makes no stupid mistakes. Every deployment is hardened by default. The deployer doesn't need to understand firewall rules, TLS configuration, or SNI routing — Meridian handles it correctly every time.
- **Two-sided UX** — deployer experience (CLI wizard, smart defaults, rebuild-fast) AND end-user experience (PWA connection pages, QR codes, subscription auto-update, deep links) both matter equally.
- **Aesthetics are trust** — every surface is UI. CLI output, terminal colors, connection pages, URLs, error messages. In censored regions, a polished page says "this is safe to use" more than any documentation.

## Architecture

Python CLI (`meridian-vpn` on PyPI). nginx (port 443 stream SNI routing + TLS termination + web serving + reverse proxy) + acme.sh (certificates) + Xray (VLESS+Reality). Domain mode adds WSS through Cloudflare CDN. Relay nodes are L4 TCP forwarders (Realm) for domestic entry points. Website at `getmeridian.org` built with Astro.

## Per-folder CLAUDE.md — the knowledge system

Every meaningful folder has a `CLAUDE.md`. Together they form a **self-sustaining, self-healing knowledge system**.

**How it works:** AI assistants automatically load CLAUDE.md files into context before writing code. A design decision documented here is a decision that gets followed. A pitfall documented here is a bug that never recurs — even across different developers, different sessions, different months.

**The self-healing loop:**
1. A subtle bug is found and fixed
2. The pitfall is added to the relevant folder's CLAUDE.md
3. Next session, the AI reads it before touching that code
4. The same bug is never introduced again

**The self-evolving property:** as the codebase grows, new folders get CLAUDE.md files. As bugs are found, pitfalls accumulate. The system gets smarter over time without any maintenance burden — updating CLAUDE.md is part of the fix, not a separate task.

**Format** — each file has three sections:
- **Design decisions** — WHY, not what. Rationale survives code rewrites.
- **What's done well** — preserve these properties. Don't "improve" them away.
- **Pitfalls** — learned the hard way. The most valuable section.

**Rules:**
- Create one for any folder with 3+ files and distinct architectural concerns
- Update it as part of the PR, not as a separate task
- Keep it under 40 lines. Less is more — a long CLAUDE.md won't be read
- No line numbers, function signatures, or anything that drifts with code
- No duplication across files. Root has the big picture; folders have the details

**Current coverage:**
```
CLAUDE.md                          — this file (vision, system, workflow)
src/meridian/                      — protocol registry, credentials, SSH, API quirks
  commands/                        — resolution cascade, three-step pattern
  provision/                       — step pipeline, idempotency, lockout prevention
  templates/pwa/                   — vanilla JS rationale, security model, CSS lessons
tests/                             — testing philosophy, MockConnection, fixtures
  provision/                       — mock boundary, idempotency dual-path testing
  e2e/                             — Docker lifecycle, shell-script E2E, PID namespace
  systemlab/                       — multi-node system lab, Docker topology, CI boundary
website/                           — Astro rationale, i18n strategy, CLI relationship
  src/components/                  — composition pattern, Accordion CSS lesson
  src/content/docs/                — locale parity, frontmatter schema, machine-readable
  src/i18n/                        — asymmetric i18n, detection cascade, translation keys
  src/layouts/                     — Base/Docs/BlogPost shells, locale bridge, RTL
  src/styles/                      — token system, warm light-first palette
  src/pages/                       — dynamic routing, machine-readable endpoints
  src/pages/blog/                  — blog index, post route, English-only linear reads
.github/workflows/                 — two-stage pipeline, VERSION-driven releases
```

## Development

```bash
make install     # uv sync --extra dev --reinstall-package meridian-vpn
uv run meridian  # always use uv run (not system-wide meridian)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uburuntu/meridian](https://github.com/uburuntu/meridian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
