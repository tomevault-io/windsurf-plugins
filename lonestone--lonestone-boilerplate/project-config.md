---
trigger: always_on
description: Working or creating database migrations
---

Migrations should be created using the `mikro-orm` CLI, after modifying any entity file.

```bash
cd apps/api
pnpm db:migrate:create
```

---
> Source: [lonestone/lonestone-boilerplate](https://github.com/lonestone/lonestone-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
