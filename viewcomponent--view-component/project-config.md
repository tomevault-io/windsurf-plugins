---
trigger: always_on
description: - After changing dependencies in `Gemfile`, regenerate the appraisal Gemfiles with `bundle exec appraisal generate`, then update every appraisal lockfile with `bundle exec appraisal-run gemfiles/*.gemfile -- bundle lock`.
---

# Repository instructions

- After changing dependencies in `Gemfile`, regenerate the appraisal Gemfiles with `bundle exec appraisal generate`, then update every appraisal lockfile with `bundle exec appraisal-run gemfiles/*.gemfile -- bundle lock`.

---
> Source: [ViewComponent/view_component](https://github.com/ViewComponent/view_component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
