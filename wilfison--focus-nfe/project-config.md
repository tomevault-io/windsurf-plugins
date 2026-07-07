---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

Unofficial Ruby gem: a client for the [Focus NFe](https://focusnfe.com.br) API (Brazilian electronic
fiscal documents — NFe, NFCe, NFSe, CTe, MDFe, NFCom, DCe, …). Early stage: most of the gem is still
to be built; the substantive existing code is the field-scraping tooling (*Field schemas* below).

## Coding conventions

### Identifier language (hybrid: English plumbing, Portuguese domain)

Dividing question: **does the identifier name something in the Focus NFe / SEFAZ fiscal domain?**
Yes → **Portuguese**. The gem's own machinery → **English**.

- **English (gem plumbing, not in the API):** classes/modules (`Client`, `Configuration`, `Connection`,
  `Response`, `Adapter`, `Authentication`, `Error`, `Errors::*`); structural methods (`configure`,
  `connection`, `call`, `get`/`post`/`put`/`delete`, `validate!`, `success?`, `from_response`,
  `class_for`); infra config & HTTP terms (`environment`, `timeout`, `logger`, `http_adapter`, `headers`,
  `path`, `params`, `body`, `status`, `url`); all internal vars, private methods, constants.
- **Portuguese (the API's own vocabulary):** resource accessors / fiscal operations mapping 1:1 to API
  actions (`nfe`, `nfce`, `nfse`, `cte`, `mdfe`, `emitir`, `consultar`, `cancelar`, `inutilizar`,
  `justificativa`, `referencia`); **payload field names verbatim from the schemas — never translated**
  (`natureza_operacao`, `cnpj_emitente`, `valor_total`, …).
- **Domain values stay Portuguese even behind an English key:** `config.environment = :homologacao`
  (`:producao`/`:homologacao`); fiscal statuses like `"autorizado"`/`"cancelado"`.
- Debated boundaries: `environment` is English (resolves the base URL; `ambiente` is not an API field);
  `referencia` is Portuguese (the API's `ref`).

General Ruby/gem best practices: `frozen_string_literal`, double-quoted strings, Ruby 3.3 target (`.rubocop.yml`).

### Comments

- **No unnecessary comments** — names carry intent; no narration, restatement, or banner/section comments.
- **Only YARD docs allowed**, on public classes/modules/methods, with type tags (`@param`, `@return`,
  `@raise`). **Prose in Portuguese**; type references use the real English identifier (`@return [FocusNfe::Configuration]`).
- A genuinely non-obvious *why* (workaround, external constraint) goes in the method's YARD doc, not inline.

## TDD (mandatory — hard rule)

Red → Green → Refactor for every behavior. **No production code without a failing spec first** (new
classes, methods, branches arrive test-first; each bug fix starts with a reproducing spec).

- **RSpec** under `spec/`, mirroring `lib/` (`spec/focus_nfe/recursos/nfe_spec.rb`, …).
- HTTP never hit for real — stub with **WebMock** (VCR for recorded interactions if useful). Cover both
  `:homologacao` and `:producao` base URLs.
- `bundle exec rake` (RSpec + RuboCop) must stay green before any commit.

## Commands

- `bin/setup` — install deps; then `bundle exec overcommit --install` (git hooks, see below).
- `bin/console` — IRB with the gem loaded.
- `bin/rspec` / `bin/rubocop` — binstubs (faster than `bundle exec`). `bin/rubocop -a` auto-corrects.
- `bundle exec rake` — default; runs **RSpec + RuboCop**. Keep green before committing.
- `bundle exec rake ci` — roda **localmente todas as verificações do CI** (`.github/workflows/ci.yml`):
  specs, RuboCop, Steep, YARD (`--fail-on-warning`), cobertura de docs e schemas atualizados. Continua
  mesmo se um passo falhar e imprime um resumo no fim — pegue o CI quebrado antes de enviar ao GitHub.
- `bundle exec rake steep` — **Steep** type-check (`sig/` vs `lib/`). Standalone, *not* in default rake;
  CI gates it in a `typecheck` job. See *Type signatures*.
- `bundle exec rake pull_fields` — regenerate field schemas into `data/schemas/`.
- `bundle exec rake coverage:open` — open SimpleCov HTML report (generate first via `bin/rspec`/`rake spec`;
  configured atop `spec/spec_helper.rb`, line+branch; `coverage/` git-ignored).
- `bundle exec rake install` / `release` — build/publish (gemspec metadata still has TODOs).

CI (`.github/workflows/ci.yml`) pins Ruby `4.0.2`; gemspec requires `>= 3.3.0`.

## Git hooks / commits (overcommit + Conventional Commits)

Managed by [overcommit](https://github.com/sds/overcommit) (`.overcommit.yml`); activate once with
`bundle exec overcommit --install`. After editing `.overcommit.yml`, re-sign: `bundle exec overcommit --sign`.

- **pre-commit** — `rubocop` on changed files + whitespace/tab/merge-conflict/YAML checks.
- **pre-push** — full `rspec` suite.
- **commit-msg** — [Conventional Commits](https://www.conventionalcommits.org): `<tipo>(escopo)!: <descrição>`.
  Types: `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`.
  Subject ≤ 72 cols, no trailing period (`Merge`/`Revert`/`fixup!`/`squash!` exempt).

## Field schemas (source of truth for API fields)

The request/field layer is derived from `campos.focusnfe.com.br`, not hand-transcribed.

- `tools/pull_fields.rb` fetches each page, extracts the embedded `__NEXT_DATA__` JSON
  (`props.pageProps.json.object_attributes`), and writes `data/schemas/schema_<name>.json` per document
  type (URL map at the top of the script).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilfison/focus_nfe](https://github.com/wilfison/focus_nfe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
