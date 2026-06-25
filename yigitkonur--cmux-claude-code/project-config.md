---
trigger: always_on
description: All builds run on the Mac mini. Never build locally.
---

# cmux-claude-pro

## Build (remote — Mac mini)

All builds run on the Mac mini. Never build locally.

```
make up        # sync + build on mini (~7s incremental)
make test      # run tests on mini
make dev       # start dev server on mini
make info      # show detected config
```

---
> Source: [yigitkonur/cmux-claude-code](https://github.com/yigitkonur/cmux-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
