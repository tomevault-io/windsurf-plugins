---
trigger: always_on
description: Flask app que coleta anúncios da OLX (Dell OptiPlex / Lenovo ThinkCentre), extrai specs de hardware via LLM (DeepSeek) e expõe UI + API REST. Banco: SQLite + SQLAlchemy/Alembic. Deps: `uv` (Python 3.13).
---

# AGENTS.md

Flask app que coleta anúncios da OLX (Dell OptiPlex / Lenovo ThinkCentre), extrai specs de hardware via LLM (DeepSeek) e expõe UI + API REST. Banco: SQLite + SQLAlchemy/Alembic. Deps: `uv` (Python 3.13).

**Este diretório NÃO é um repositório git** — não use comandos git.

## Comandos (sempre via `uv run`, nunca pip/venv manual)

- Deps: `uv sync` · adicionar: `uv add <pkg>` / `uv add --group dev <pkg>`
- Migrações (após alterar `app/models/*`): `uv run flask db migrate -m "..."` → `uv run flask db upgrade`
- Coletar: `uv run flask scrape "dell optiplex" --region estado-sp [--max-pages 5] [--no-details]`
- Cadastrar anúncio por link: `uv run flask add "https://.../anuncio-12345" [--no-process]`
- Preencher descrição/imagens de ads sem descrição: `uv run flask enrich`
- Extrair specs: `uv run flask process [--limit N] [--ad <id> --force]`
- Servidor (UI + API em :5000): `uv run flask run`
- Testes (offline): `uv run pytest`

## Config (.env)

- `DEEPSEEK_KEY` é obrigatória para `flask process`.
- `config.py` resolve `DATABASE_URL` sqlite relativo para **absoluto** — o Flask-SQLAlchemy resolve relativos contra o *instance path* (senão vira `instance/instance/olx.db`). Não "simplifique" isso.
- `LOG_LEVEL`/`LOG_FILE` (rotativo) via `app/logging_setup.py`; `SQLALCHEMY_ENGINE_OPTIONS` com `busy_timeout=30` para concorrência no SQLite.

## Arquitetura

- App factory em `app/__init__.py`; registra blueprints `api` (REST, prefixo `/api`) e `main` (UI, templates em `app/blueprints/main/templates/`).
- Pipeline de specs: `extractors/regex.py` (determinístico, **prevalece** sobre a LLM) → `extractors/llm.py` (DeepSeek) → `pipeline.py` (merge). `extract_specs()` retorna tupla de 6: `(spec, usage, method, cpu_family, cpu_model, cpu_generation)`.
- `services/ad_service.py` centraliza consultas (filtros/sort/serializers). Novos filtros entram em `AdFilters` + `_apply_filters`, e nos parsers de `blueprints/api/routes.py` e `blueprints/main/routes.py`. `upsert_raw(ad, refresh=False)` ganhou o flag `refresh=True` para o **recadastro manual** (atualiza preço/título/descrição de existentes; sem o flag preserva o comportamento legado de só preencher lacunas).
- **Cadastro manual por link (D-026)** — `import_single_ad(app, url, process=True)` em `services/runner.py` (valida link OLX + `olx_id`, **normaliza a URL** removendo query de tracking/fragmento — evita 403 do Cloudflare e duplicatas por parâmetro —, busca o detalhe com `OlxClient`, parse JSON-LD e `upsert_raw(..., refresh=True)`; extrai specs na hora se `process` e `DEEPSEEK_KEY`, best-effort). Superfícies: painel em `/run` (`#add-ad-form`, JS `initAddAd` em `app/static/js/app.js`), API `POST /api/ads/import` (resposta `{status, created, processed, ad}`) e CLI `flask add`. É síncrono e fora do `RunManager` (não compete com a trava uma-run-por-vez; não entra no `run_history`). Spec: `docs/specs/14-cadastro-manual.md`.
- **Ocultar anúncio manualmente (D-028)** — coluna `Ad.user_disabled` (bool, default False) controlada por um toggle "Disponível" em `/ads/<id>` (`POST /api/ads/<id>/disabled`). O scrape (`upsert_raw`) **não** toca essa flag — anúncio oculto continua oculto mesmo reaparecendo na OLX. Filtros "ativo" = `is_active=True AND user_disabled=False` (`_apply_filters` em `ad_service.py` cobre lista/gráfico/ofertas/review; `stats`, `_price_by_family_gen`, filas de trabalho `run_check`/`run_process`/`list_pending_extraction`/`list_missing_description` também pulam ocultos). `include_inactive` revela removidos **e** ocultos; `stats` expõe `ocultos`; `ad_to_dict` expõe `user_disabled`. Spec: `docs/specs/16-ocultar-anuncio-manualmente.md`.
- **Filtro de CPU por família (D-029)** — `cpu_generation` usa escalas incomparáveis (Intel `modelo//1000` 1–14, Ryzen série 1–9), então `gen_min`/`gen_max` **só se aplicam dentro de `cpu_family`** (`_apply_filters`); sem família, são ignorados (API permissiva). Constantes/helpers em `ad_service.py`: `CPU_GROUPS`, `GEN_RANGE`, `gen_range_for` (exposto ao Jinja2 via `@bp.app_template_global()`), `cpu_group`. O select de família no `_filters.html` é `<optgroup>` Intel/AMD e o seletor de geração fica `disabled` sem família (faixa dinâmica server-side; `initCpuFamilyFilter` no `app.js` dá auto-submit ao trocar família). `/chart` **exige família** (aviso sem ela); `/offers` compara preços por **(família, geração)** (`_price_by_family_gen`). Spec: `docs/specs/17-filtro-cpu-por-familia.md`.
- **Fabricante de CPU (D-030)** — valores especiais `cpu_family=intel|amd` (`VENDOR_ALIASES`) filtram pelo grupo de famílias do fabricante (`IN(CPU_GROUPS[...])`) e valem como **domínio de escala** da geração (`VENDOR_RANGE` intel 1–14, amd 1–9). UI: opções `Intel (qualquer)`/`AMD (qualquer)` hardcoded no topo do select de família; parsers normalizam `cpu_family` com `.lower()`; `_cpu_family_groups` não insere aliases nos optgroups. `/chart` aceita fabricante. Spec: `docs/specs/18-filtro-por-fabricante.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leourl/olx-scraper](https://github.com/leourl/olx-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
