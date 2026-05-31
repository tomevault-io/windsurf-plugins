---
trigger: always_on
description: Before reporting any work as done, run agent-ci to validate changes locally:
---

## CI

Before reporting any work as done, run agent-ci to validate changes locally:

```bash
AI_AGENT=1 npx @redwoodjs/agent-ci run --all
```

If it fails, fix the issue and re-run. Do not report work as done until it passes.

---
> Source: [redwoodjs/sdk](https://github.com/redwoodjs/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
