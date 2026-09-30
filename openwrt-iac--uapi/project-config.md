---
trigger: always_on
description: Native, lightweight, production-grade HTTP REST API for OpenWrt. Translates standard REST verbs into ubus calls so modern edge routers become first-class targets for Infrastructure-as-Code workflows. Primary design validation: serving as the backend for the openwrt-iac Terraform provider.
---

# uapi

Native, lightweight, production-grade HTTP REST API for OpenWrt. Translates standard REST verbs into ubus calls so modern edge routers become first-class targets for Infrastructure-as-Code workflows. Primary design validation: serving as the backend for the openwrt-iac Terraform provider.

This file is the every-turn meta document: project identity, non-negotiable principles, code-style and workflow rules, plus a pointer table to topic-specific docs under `docs/`. Topical normative content (process model, transaction recipe, auth, error envelope, observability, testing, packaging, versioning) lives in the corresponding `docs/<topic>.md` file. When in doubt, the table at the bottom of this file is the index.

---

## Architectural principles (non-negotiable)

1. **Native integration (direct-to-bus).** Communicate with OpenWrt via ubus through the ucode runtime. No intermediate proxy daemons. No direct `/etc/config/` file manipulation; all writes go through uci's API.
2. **Zero-bloat footprint.** Target is resource-constrained embedded hardware. Runtime overhead, memory, and storage stay negligible. Reject dependencies that don't earn their keep.
3. **Atomic transactions.** A single HTTP write request stages → validates → commits → reloads in one transaction. No partial-failure states, no config drift.
4. **Report state, never correct it.** A read answers what uci holds *and* what the daemon is actually doing, and says so when they disagree (`network/interfaces.effective_proto` is the shape). A write reports whether the effect landed, not merely that the intent was stored (`X-Kernel-Status`). uapi never repairs drift on its own: there is no loop to host one, and the IaC tool above it already is that loop, so a second corrector would fight the first. Persistence is uci's job, boot-time convergence is the daemon's, and being trustworthy about both is uapi's.

Before adopting any library, daemon, persistence layer, or abstraction, check it against these four. Prefer ucode-native solutions; flag anything requiring a long-running auxiliary process, direct `/etc/config/` writes, splitting a logical state change across multiple HTTP requests, or a timer or background pass that would make uapi correct drift rather than report it.

**Aim.** Every change should move uapi closer to state-of-the-art for an embedded HTTP control plane: correctness, observability, security posture, test discipline, lock-and-state hygiene, drift detection. The roadmap (`docs/roadmap.md`) is not aspirational backlog; it is the gap between today's posture and that target. Prefer hardening that closes a real gap over a feature that adds wire surface for its own sake.

**Design reference: LuCI.** When a design choice is non-obvious (should this field be required? what should happen on a proto switch? how is this option meant to interact with that one?), read LuCI's source for the same surface before deciding. The OpenWrt SDK feeds carry it at `build/sdk/feeds/luci/`; the form/view code under `modules/luci-mod-*/htdocs/luci-static/resources/view/` and the platform abstractions under `modules/luci-base/htdocs/luci-static/resources/` are the two main entry points. LuCI is the long-baked baseline every OpenWrt operator already lives with; matching its behavior is the safe default. *Deliberately* diverging from it is fine when the divergence is a documented improvement; *accidentally* diverging because we didn't check is the failure mode to avoid.

---

## Code and documentation style

- **Priorities, in order:** simplicity, maintainability, modularity, readability.
- **No em-dashes.** Applies to code, comments, docs, commit messages, and design notes.
- **Comments are rare.** Default to writing none. Naming and structure should carry the meaning.
- **When a comment is necessary, explain why, not what.** A reader can see what the code does; what they cannot see is the non-obvious constraint, invariant, or workaround that motivated the choice.

### Avoid AI slop (HIGH IMPORTANCE)

Slop is plausible-looking ceremony that adds no signal. It is the single most common failure mode for AI-generated patches and the most expensive to remove in review. Treat every line you write or accept as carrying a justification cost. Apply ruthlessly:

- **No narration headers.** No "What this file does" preambles, no `// ---- section ----` banner comments, no multi-paragraph docstrings explaining the obvious. The filename and the first function are the header.
- **No what-comments.** Anything a competent reader can read directly off the code (`// Loop over keys`, `// Check if X is null`, `// Cleanup`) is slop. Delete it.
- **One-call-site helpers are suspect.** A helper that wraps a single-line operation in a function is slop unless naming it adds real meaning. Inline.
- **Defensive code for cases that cannot happen** (given the rest of the code, not the universe) is slop. Either prove the case is reachable and handle it, or delete the guard.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openwrt-iac/uapi](https://github.com/openwrt-iac/uapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
