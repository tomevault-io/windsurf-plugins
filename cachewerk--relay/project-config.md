---
trigger: always_on
description: When bumping Relay versions, check these version-string locations first:
---

When bumping Relay versions, check these version-string locations first:

- `/home/runner/work/relay/relay/.github/workflows/docker.yml` for workflow matrix and `RELAY_VERSION`
- `/home/runner/work/relay/relay/docker/**/*.Dockerfile` for Docker build arguments

---
> Source: [cachewerk/relay](https://github.com/cachewerk/relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
