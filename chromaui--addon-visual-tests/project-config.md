---
trigger: always_on
description: Before creating, updating, or pushing a pull request, run:
---

# Agent instructions

Before creating, updating, or pushing a pull request, run:

```sh
yarn gate
```

The command must complete successfully. Fix any failures and rerun the full gate before pushing.

---
> Source: [chromaui/addon-visual-tests](https://github.com/chromaui/addon-visual-tests) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
