---
trigger: always_on
description: Passo a passo para criar um novo package seguindo a estrutura do projeto
---


# Creating a New Package

## Passos para Criar um Novo Package

### 1. Estrutura de Diretórios

Seguir o modelo do package `billing`:

```
packages/{package_name}/src/{package_name}/
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── events/
│   ├── exceptions/
│   └── services/
├── application/
│   ├── usecases/
│   └── interfaces/
├── infrastructure/
│   ├── persistence/
│   ├── http/
│   ├── messaging/
│   ├── tasks/
│   ├── di/
│   └── tools/
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/
```

### 2. Adicionar ao Workspace

Editar `pyproject.toml` raiz:

```toml
[tool.uv.workspace]
members = [
    "packages/shared",
    "packages/billing",
    "packages/{package_name}",  # Adicionar aqui
]

[tool.uv.sources]
shared = { workspace = true }
billing = { workspace = true }
{package_name} = { workspace = true }  # Adicionar aqui
```

### 3. Adicionar Dependência

Editar `pyproject.toml` raiz em `dependencies`:

```toml
dependencies = [
    "fastapi[standard]>=0.127.0",
    "shared",
    "billing",
    "{package_name}",  # Adicionar aqui
    # ...
]
```

### 4. Criar pyproject.toml do Package

Criar `packages/{package_name}/pyproject.toml`:

```toml
[project]
name = "{package_name}"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "shared",
    # outras dependências específicas do package
]
```

### 5. Criar Plugin (para Monólito)

Criar `infrastructure/http/plugin.py`:

Veja `@architecture/monolith-plugins.mdc` para detalhes.

### 6. Configurar Logging e Observabilidade

#### No App Factory (Standalone API)

O logging deve ser configurado no `app_factory.py` **antes** de importar controllers:

```python
"""Factory para criar aplicação FastAPI"""
from __future__ import annotations

import logging

from fastapi import FastAPI

from shared.config.settings import Settings
from shared.migrations.runner import run_package_migrations
from shared.observability import setup_observability
from shared.observability.elk.logging import setup_structured_logging


def create_{package_name}_app() -> FastAPI:
    """Cria e configura a aplicação FastAPI"""
    settings = Settings()

    # Configurar logging ANTES de importar controllers
    # para garantir que os loggers já tenham os handlers configurados
    if settings.observability_enabled and settings.apm_logs_enabled:
        setup_structured_logging(settings)

    # Executar migrations se AUTO_MIGRATE=true
    if settings.auto_migrate:
        run_package_migrations("{package_name}")
        # Reconfigurar logging após migrations (Alembic pode resetar handlers)
        if settings.observability_enabled and settings.apm_logs_enabled:
            setup_structured_logging(settings, force_reconfigure=True)

    # Configurar APM client
    apm_client = None
    if settings.observability_enabled:
        try:
            apm_client = setup_observability(settings)
        except Exception as e:
            logger = logging.getLogger(__name__)
            logger.warning(f"Failed to setup observability: {e}", exc_info=True)

    # Criar app FastAPI
    app = FastAPI(title="{package_name}-api")

    # Adicionar middleware ElasticAPM se cliente estiver disponível
    if apm_client and settings.apm_traces_enabled:
        try:
            from elasticapm.contrib.starlette import ElasticAPM
            app.add_middleware(ElasticAPM, client=apm_client)
        except Exception as e:
            logger = logging.getLogger(__name__)
            logger.warning(f"Failed to add APM middleware: {e}", exc_info=True)

    # Registrar rotas (importar DEPOIS de configurar logging)
    from {package_name}.infrastructure.http.controllers.{entity}_controller import router
    app.include_router(router, tags=["{package_name}"])

    return app
```

**⚠️ IMPORTANTE - Arquitetura e Logging:**

- **Application Layer (Use Cases)**: Use apenas `logging.getLogger(__name__)` padrão do Python. **NÃO** importe `get_logger` de `shared.observability` (violaria a dependency rule).
- **Infrastructure Layer (Controllers)**: Use apenas `logging.getLogger(__name__)` padrão do Python.
- O logging é configurado uma vez na infraestrutura (via `setup_structured_logging()`), e todos os loggers criados com `logging.getLogger()` herdam automaticamente essa configuração.

**Exemplo em Use Case:**
```python
import logging

logger = logging.getLogger(__name__)

async def execute(self, input: UseCaseInput) -> UseCaseOutput:
    logger.info("Executing use case", extra={"input_id": input.id})
    # Lógica do use case
    return result
```

**Exemplo em Controller:**
```python
import logging

logger = logging.getLogger(__name__)

@router.post("/entities")
async def create_entity(request: CreateEntityRequest):
    logger.info("Creating entity", extra={"entity_id": request.id})
    # Lógica do controller
    return result
```

### 7. Criar App Factory (para Standalone API)

Veja `@setting-up-standalone-api.mdc` para detalhes completos.

### 8. Criar Worker (se necessário)

Veja `@setting-up-worker-app.mdc` para detalhes.

### 9. Sincronizar Dependências

```bash
uv sync
```

### 10. Configurar Testes com PostgreSQL

Os testes de integração usam PostgreSQL com helpers do `shared.testing`. Configure o `conftest.py`:

**Criar `tests/integration/conftest.py`:**

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rafaelcostaf4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
