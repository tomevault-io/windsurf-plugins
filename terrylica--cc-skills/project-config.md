---
trigger: always_on
description: Claude Code skills marketplace: **37 plugins** with skills for ADR-driven development workflows.
---

# CLAUDE.md

Claude Code skills marketplace: **37 plugins** with skills for ADR-driven development workflows.

**Architecture**: Link Farm + Hub-and-Spoke with Progressive Disclosure

## Documentation Hierarchy

```
CLAUDE.md (this file)                          ◄── Hub: Navigation + Essentials
    │
    ├── plugins/CLAUDE.md                      ◄── Spoke: Plugin development (all plugins listed)
    │       └── {plugin}/CLAUDE.md             ◄── Deep: Per-plugin SSoT
    │                                                (project/stack/conventions/architecture live here,
    │                                                 NOT duplicated in root)
    │           └── skills/{skill}/CLAUDE.md   ◄── Deepest: Per-skill SSoT (emerging — opt-in per skill)
    │                                                (file table, invariants, recent-change log,
    │                                                 edit conventions; sibling to SKILL.md)
    │
    └── docs/CLAUDE.md                         ◄── Spoke: Documentation standards
            ├── HOOKS.md                       ◄── Hook development patterns
            ├── RELEASE.md                     ◄── Release workflow
            ├── PLUGIN-LIFECYCLE.md            ◄── Plugin internals
            └── LESSONS.md                     ◄── Lessons learned (dated entries)
```

**Progressive disclosure rule**: each layer must add information the next-shallower layer didn't already cover. Don't restate plugin invariants in the root; don't restate skill invariants in the plugin. When the user asks Claude something specific, Claude follows links downward — so the deepest layer's freshness matters most.

## Navigation

### Spokes & Docs

| Topic             | Document                                                                                                                     |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Installation      | [README.md](./README.md)                                                                                                     |
| Plugin Dev        | [plugins/CLAUDE.md](./plugins/CLAUDE.md)                                                                                     |
| Documentation     | [docs/CLAUDE.md](./docs/CLAUDE.md)                                                                                           |
| Hooks Dev         | [docs/HOOKS.md](./docs/HOOKS.md)                                                                                             |
| Lessons Learned   | [docs/LESSONS.md](./docs/LESSONS.md)                                                                                         |
| Cargo TTY Fix     | [docs/cargo-tty-suspension-prevention.md](./docs/cargo-tty-suspension-prevention.md)                                         |
| Claude Code Proxy | [devops-tools/skills/claude-code-proxy-patterns/SKILL.md](./plugins/devops-tools/skills/claude-code-proxy-patterns/SKILL.md) |
| Release           | [docs/RELEASE.md](./docs/RELEASE.md)                                                                                         |
| Plugin Lifecycle  | [docs/PLUGIN-LIFECYCLE.md](./docs/PLUGIN-LIFECYCLE.md)                                                                       |
| Troubleshooting   | [docs/troubleshooting/](./docs/troubleshooting/)                                                                             |
| ADRs              | [docs/adr/](./docs/adr/)                                                                                                     |
| Resume Context    | [docs/RESUME.md](./docs/RESUME.md)                                                                                           |

### Plugin CLAUDE.md Files (37/37)

All 37 plugins have their own CLAUDE.md with Hub+Sibling navigation links. Access via `plugins/{name}/CLAUDE.md` or browse the full table in [plugins/CLAUDE.md](./plugins/CLAUDE.md).

**Emerging deeper layer**: skill-level CLAUDE.mds (one per skill, sibling to `SKILL.md`) are appearing where a skill is large enough that maintainers need a separate compass from the user-invocable instructions. First adopter: [`plugins/macro-keyboard/skills/{configure-macro-keyboard,emit-fn-key-on-macos,diagnose-hid-keycodes}/CLAUDE.md`](./plugins/macro-keyboard/CLAUDE.md). Add one to your skill if SKILL.md is starting to mix "what to do when invoked" with "what to know before editing".

**Active project (SSoT):** [plugins/claude-tts-companion/CLAUDE.md](./plugins/claude-tts-companion/CLAUDE.md) — project/stack/conventions/architecture for the Swift macOS companion binary. Critical invariants (e.g., _do not replace afplay with AVAudioPlayer_) live there, not here.

Key plugin docs: [itp](./plugins/itp/CLAUDE.md) | [itp-hooks](./plugins/itp-hooks/CLAUDE.md) | [gh-tools](./plugins/gh-tools/CLAUDE.md) | [devops-tools](./plugins/devops-tools/CLAUDE.md) | [gmail-commander](./plugins/gmail-commander/CLAUDE.md) | [tts-tg-sync](./plugins/tts-tg-sync/CLAUDE.md) | [calcom-commander](./plugins/calcom-commander/CLAUDE.md) | [claude-tts-companion](./plugins/claude-tts-companion/CLAUDE.md)

## Essential Commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [terrylica/cc-skills](https://github.com/terrylica/cc-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
