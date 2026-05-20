---
trigger: always_on
description: juscraper e uma biblioteca Python para raspagem de dados de tribunais brasileiros. Coleta dados de jurisprudencia (acordaos, decisoes) do TJDFT, TJPR, TJRS, TJSP e outros tribunais.
---

# CLAUDE.md

## Visao geral do projeto

juscraper e uma biblioteca Python para raspagem de dados de tribunais brasileiros. Coleta dados de jurisprudencia (acordaos, decisoes) do TJDFT, TJPR, TJRS, TJSP e outros tribunais.

## Arquitetura

- Codigo-fonte em `src/juscraper/`
- Tribunais organizados hierarquicamente: `juscraper.courts.<tribunal>.client` (ex: `juscraper.courts.tjrs.client.TJRSScraper`)
- A factory function publica e `juscraper.scraper()`
- Nomes de classes seguem PEP 8 CamelCase: `TJDFTScraper`, `TJPRScraper`, `TJRSScraper`, `TJSPScraper`
- **Regra 1 do refactor #84:** generalizar (mover para `_<familia>/`, criar mixin/base) so com **2+ ocorrencias concretas**. Duplicar com 1 caso e mais barato que abstrair errado.

## Desenvolvimento

- Python >= 3.11
- Preferir `uv` como gerenciador de pacotes (ja usado no projeto — ver `uv.lock`)
- Instalar em modo editavel: `uv pip install -e ".[dev]"`
- Nunca usar hacks de `sys.path` nos testes — confiar no install editavel
- Pre-commit hooks configurados (trailing whitespace, isort, pylint, flake8, mypy)
- Comprimento maximo de linha: 120
- Preferir trabalhar em worktree com branch específica para a mudança que desejar implementar.

## Testes

### Estrutura

- Testes ficam em `tests/` com subdiretorios por tribunal (`tests/tjdft/`, `tests/tjpr/`, etc.). Cada subdiretorio precisa ter um `__init__.py` para o pytest descobrir os testes.
- Fixtures HTML/JSON ficam em `tests/<tribunal>/samples/<endpoint>/<cenario>.html` (ex.: `tests/tjsp/samples/cjsg/results_normal.html`).
- Helper compartilhado: `tests/_helpers.py::load_sample(tribunal, relative_path)` retorna o sample como string. Use `load_sample_bytes` quando o parser precisa lidar com encoding sozinho (ex.: eSAJ em latin-1).

### Comandos

- `pytest` — roda contrato + granular (offline, ~0.5s). **Default exclui integracao.**
- `pytest -m integration` — roda so integracao (lento, hit live).
- `pytest -m ""` — roda tudo (offline + integracao).
- `pytest tests/tjsp` — escopo a um tribunal.
- `--strict-markers` esta ativo — todo marker deve ser registrado no `pyproject.toml`.

### Regras para autor de teste

- Toda mudanca em parser HTML/JSON deve incluir/atualizar sample em `tests/<tribunal>/samples/<endpoint>/`.
- Testes de contrato afirmam **schema do DataFrame** (colunas obrigatorias) e, quando relevante, **payload enviado** (matchers do `responses`).
- Testes que tocam rede ficam marcados com `@pytest.mark.integration`.
- Cassetes (`pytest-recording`) sao adotados caso a caso; medir peso agregado antes de generalizar (limite indicativo: ~20 MB no repo).
- Antes de refatorar um tribunal pela #84, ele precisa ter contratos passando.

Piramide de testes (sufixos `*_contract.py` / `*_granular.py` / `*_cassette.py` / `*_integration.py`), ferramentas (`responses`, `pytest-mock`, `pytest-recording`) e checklist de 13 itens para adicionar raspador novo: ver `CONTRIBUTING.md` > **Tests** e **Adding a new tribunal**.

## Convencao de API para raspadores

- Busca: `pesquisa` como nome padrao em todos os scrapers
- Datas: `data_julgamento_inicio/fim`, `data_publicacao_inicio/fim`
- Alias generico: `data_inicio/fim` mapeia para `data_julgamento_inicio/fim`
- **Excecao**: `DatajudScraper.listar_processos` filtra por `dataAjuizamento` (nao julgamento), entao usa `data_ajuizamento_inicio/fim` como nome canonico e **nao aceita** o alias generico `data_inicio/fim`. Quem tenta receber `TypeError` via `extra="forbid"`. Refs #49.
- Nomes antigos (`query`, `termo`, `_de/_ate`) aceitos com `DeprecationWarning`
- Paginacao: `paginas: int | list | range | None`, default `None` (todas as paginas). Sempre 1-based: `range(1, 4)` baixa paginas 1, 2 e 3; `paginas=3` e equivalente a `range(1, 4)`.
- Tamanho de pagina: `tamanho_pagina` (default 10; **TJES=20** por particularidade do backend Elasticsearch). Aliases deprecados, um por tribunal: `items_per_page` (TJBA), `quantidade_por_pagina` (TJDFT, TJMT), `per_page` (TJES), `qtde_itens_pagina` (TJGO), `linhas_por_pagina` (TJMG). Cada client conhece so o seu alias — passar alias de outro tribunal cai em `TypeError`. Refs #211.
- Normalizacao centralizada em `src/juscraper/utils/params.py`
- **Validacao da API publica via pydantic com `extra="forbid"`**. Kwargs desconhecidos levantam `ValidationError` em vez de serem silenciosamente ignorados.

Referencia completa de parametros e migracao: `docs/api-conventions.qmd`.

## Schemas pydantic (refs #93)

Todo endpoint publico (`cjsg`, `cjpg`, `cpopg`, `cposg`, `listar_processos`, `auth`, `download_documents`, ...) tem schema `Input<Endpoint><Tribunal>` em `courts/<xx>/schemas.py` ou `aggregators/<yy>/schemas.py`, **inclusive para tribunais ainda nao refatorados** — o schema vive como documentacao executavel ate o wiring. Wired hoje: TJAC/TJAL/TJAM/TJCE/TJMS + TJSP `cjsg`/`cjpg`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jtrecenti/juscraper](https://github.com/jtrecenti/juscraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
