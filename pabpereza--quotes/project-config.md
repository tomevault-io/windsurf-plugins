---
trigger: always_on
description: Este documento define la arquitectura, el estilo de código y las directrices técnicas para el desarrollo de la API REST de "Citas Célebres". **Como agente de IA, debes leer y seguir estas reglas estrictamente antes de generar cualquier código.**
---

# Contexto y Reglas del Proyecto: API de Citas Célebres

Este documento define la arquitectura, el estilo de código y las directrices técnicas para el desarrollo de la API REST de "Citas Célebres". **Como agente de IA, debes leer y seguir estas reglas estrictamente antes de generar cualquier código.**

## 1. Stack Tecnológico
- **Lenguaje:** Python 3.11+
- **Framework:** FastAPI
- **Servidor:** Uvicorn
- **Base de Datos:** PostgreSQL
- **ORM:** SQLAlchemy
- **Validación:** Pydantic V2
- **Gestión de Paquetes:** `pip` (standard requirements.txt)
- **Tests:** Pytest

## 2. Arquitectura del Proyecto
La estructura de carpetas es estricta. No crees archivos fuera de su lugar correspondiente.

```text
/
├── app/
│   ├── main.py            # Entry point. Configuración de FastAPI e inclusión de routers.
│   ├── controllers/       # Lógica de negocio agrupada por recursos.
│   │   ├── __init__.py
│   │   ├── auth.py        # Controlador para autenticación.
│   │   ├── quote.py       # Controlador para citas.
│   │   └── user.py        # Controlador para usuarios.
│   ├── db/                # Modelos de la base de datos (SQLAlchemy).
│   │   ├── __init__.py
│   │   ├── quote.py
│   │   └── user.py
│   ├── misc/              # Archivos misceláneos (base de datos, seguridad, hashing).
│   │   ├── __init__.py
│   │   ├── database.py    # Configuración de la conexión a la base de datos y sesión.
│   │   ├── hashing.py     # Funciones de hashing de contraseñas.
│   │   └── security.py    # Lógica de autenticación y seguridad (JWT, dependencias).
│   ├── models/            # Modelos de datos de la API (Pydantic schemas).
│   │   ├── __init__.py
│   │   ├── quote.py
│   │   ├── token.py
│   │   └── user.py
│   ├── routers/           # Lógica de endpoints (Path Operations).
│   │   ├── __init__.py
│   │   ├── auth.py        # Endpoints para autenticación.
│   │   ├── probes.py      # Endpoints para health checks.
│   │   ├── quote.py       # Endpoints para /quotes.
│   │   └── user.py        # Endpoints para /users.
│   └── test/              # Tests unitarios y de integración.
│       ├── __init__.py
│       ├── conftest.py    # Configuraciones y fixtures para Pytest.
│       ├── test_auth.py
│       ├── test_main.py
│       ├── test_quotes.py
│       └── test_users.py
├── .gemini/GEMINI.md      # Este archivo de reglas.
├── compose.yml            # Configuración de Docker Compose.
└── requirements.txt       # Dependencias del proyecto.
```

## 3. Convenciones de Código

### General
- **Idioma del Código:** Variables, funciones y clases en **Inglés** (ej: `get_quotes`, `QuoteModel`).
- **Idioma de Comentarios:** Español o Inglés (mantener consistencia).
- **Type Hinting:** **Obligatorio**. Todo argumento y retorno de función debe estar tipado.

### Separación de Modelos
- **`app/models` (Pydantic):** Define la forma de los datos en la API. Se usan para validación de requests, serialización de responses y documentación OpenAPI.
- **`app/db` (SQLAlchemy):** Define la estructura de las tablas en la base de datos. Se usan para las operaciones con el ORM.
- Esta separación es intencionada para desacoplar la capa de API de la capa de datos.

### Capa de Controladores (`app/controllers`)
- Contiene la lógica de negocio principal para cada recurso.
- Las funciones de los controladores interactúan directamente con la base de datos (SQLAlchemy) y con otras capas como la de seguridad/hashing.
- Reciben los datos de los routers (ya validados por Pydantic) y devuelven los resultados.
- Son responsables de aplicar reglas de negocio y de lanzar excepciones HTTP si es necesario.

### FastAPI & Routers (`app/routers`)
- **Modularidad:** Usar siempre `APIRouter`.
- Los routers deben ser "delgados": su única responsabilidad es definir las rutas, validar los datos de entrada con Pydantic y llamar al método apropiado del controlador.
- **Inyección de Dependencias:** Usar `Depends()` para obtener la sesión de la base de datos (`get_db`) y para la autenticación (`get_current_user`).

## 4. Ejemplos de Implementación Esperada

### Modelo Pydantic (`app/models/quote.py`)
```python
from pydantic import BaseModel, Field, ConfigDict
from typing import Optional

class QuoteBase(BaseModel):
    text: str = Field(..., title="Texto de la cita", min_length=10)
    author: str = Field(..., title="Autor", min_length=3)
    category: Optional[str] = "General"

class QuoteCreate(QuoteBase):
    pass

class Quote(QuoteBase):
    id: int

    model_config = ConfigDict(from_attributes=True)
```

### Modelo SQLAlchemy (`app/db/quote.py`)
```python
from sqlalchemy import Column, Integer, String
from ..misc.database import Base # Importar desde misc

class Quote(Base):
    __tablename__ = "quotes"

    id = Column(Integer, primary_key=True, index=True)
    text = Column(String, index=True)
    author = Column(String)
    category = Column(String, default="General")
```

### Controlador (`app/controllers/user.py`)
```python
from sqlalchemy.orm import Session
from fastapi import HTTPException
from ..models import user as user_model
from ..db import user as user_db

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pabpereza/quotes](https://github.com/pabpereza/quotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
