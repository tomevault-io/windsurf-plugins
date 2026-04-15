---
trigger: always_on
description: Этот файл содержит правила и соглашения для работы с проектом Telegram Shift Bot в Cursor AI.
---

# Правила проекта Telegram Shift Bot для Cursor AI

Этот файл содержит правила и соглашения для работы с проектом Telegram Shift Bot в Cursor AI.

## 🏗️ Архитектура

### Слоистая архитектура
Проект использует четкое разделение на слои:
- **Handlers** (`src/handlers/`) — только обработка событий Telegram, минимум логики
- **Services** (`src/services/`) — вся бизнес-логика, валидация, обработка данных
- **Repositories** (`src/repositories/`) — только доступ к данным, без бизнес-логики
- **Middlewares** (`src/middlewares/`) — инъекция зависимостей, проверка прав, общая обработка

**Правило:** Handlers НЕ должны содержать бизнес-логику. Вся логика должна быть в Services.

### Dependency Injection
Зависимости передаются через middleware и параметры функций:
```python
# ✅ Правильно
async def handler(
    message: Message,
    group_service: GroupService,  # Инъекция через middleware
    state: FSMContext,
) -> None:
    groups = await group_service.get_all_groups()

# ❌ Неправильно
async def handler(message: Message) -> None:
    pool = await get_db_pool()  # НЕ ДЕЛАТЬ ТАК
    service = GroupService(pool)
```

### Асинхронность
Все операции с БД и внешними API должны быть асинхронными:
```python
# ✅ Правильно
async def get_groups(self) -> List[Dict[str, Any]]:
    async with self.pool.acquire() as conn:
        rows = await conn.fetch("SELECT * FROM groups")
```

## 📁 Структура проекта

### Именование файлов
- **Handlers:** `admin_<section>.py`, `courier_ai.py`, `user_handlers.py`
- **Services:** `<entity>_service.py` (например, `group_service.py`)
- **Repositories:** `<entity>_repository.py` (например, `group_repository.py`)
- **Middlewares:** `<purpose>_middleware.py` (например, `auth_middleware.py`)
- **States:** `<purpose>_states.py` (например, `admin_panel_states.py`)
- **Utils:** `<purpose>.py` (например, `admin_keyboards.py`)

### Организация handlers
- `admin_panel_navigation.py` — навигация по админ-панели (главное меню, переходы)
- `admin_groups.py` — управление группами
- `admin_settings.py` — настройки (расписание, слоты)
- `admin_polls.py` — управление опросами
- `admin_curator.py` — управление AI-куратором (FAQ, сообщения, замечания)
- `admin_broadcast.py` — рассылки
- `admin_monitoring.py` — мониторинг и верификация
- `courier_ai.py` — обработка вопросов от курьеров
- `user_handlers.py` — обработчики для обычных пользователей

## 💻 Стандарты кода

### Python Style Guide
Следуем **PEP 8** с обязательной типизацией и docstrings:

```python
# ✅ Правильно
from typing import Optional, List, Dict, Any

async def get_group(
    group_id: int,
    active_only: bool = False,
) -> Optional[Dict[str, Any]]:
    """
    Получить группу по ID.
    
    Args:
        group_id: ID группы
        active_only: Только активные группы
        
    Returns:
        Словарь с данными группы или None если не найдена
    """
    # ...
```

### Импорты
Порядок импортов:
1. Стандартная библиотека
2. Сторонние библиотеки
3. Локальные импорты

```python
import logging
from typing import Optional, Dict, Any
from datetime import date

from aiogram import Router
from aiogram.types import Message, CallbackQuery

from src.services.group_service import GroupService
from src.utils.auth import require_admin_callback
```

### Docstrings
Все публичные функции и классы должны иметь docstrings с описанием Args, Returns, Raises.

## 📝 Соглашения об именовании

### Переменные и функции
- **snake_case** для переменных и функций: `group_id`, `get_all_groups()`
- **UPPER_CASE** для констант: `MAX_SLOTS = 10`

### Классы
- **PascalCase** для классов: `GroupService`, `GroupRepository`

### Callback data
Формат: `admin:<section>:<action>[:<params>]`
- `admin:groups_menu`
- `admin:groups:create`
- `admin:polls:close:123`
- `admin:curator:add_faq`
- `admin:curator:search_faq`
- `admin:curator:create_info`
- `admin:curator:create_warning`
- `admin:curator:clear_history`
- `admin:curator:stats`

### Состояния FSM
Формат: `waiting_for_<action>`
- `waiting_for_group_name`
- `waiting_for_faq_question`
- `waiting_for_faq_answer`
- `waiting_for_search_query`
- `waiting_for_info_topic`
- `waiting_for_warning_description`
- `waiting_for_warning_user_id`
- `waiting_for_clear_history_user_id`

## 🗄️ Работа с базой данных

### Использование пула соединений
```python
# ✅ Правильно
class GroupRepository:
    def __init__(self, pool: Pool):
        self.pool = pool
    
    async def get_group(self, group_id: int) -> Optional[Dict[str, Any]]:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT * FROM groups WHERE id = $1",
                group_id
            )
            return dict(row) if row else None
```

### SQL запросы
- **Всегда используйте параметризованные запросы** (`$1`, `$2`) для защиты от SQL injection
- Используйте транзакции для множественных операций
- Используйте `IF NOT EXISTS` в миграциях для идемпотентности

```python
# ✅ Правильно
await conn.execute(
    "INSERT INTO groups (name, telegram_chat_id) VALUES ($1, $2)",
    name, chat_id
)

# ❌ Неправильно
await conn.execute(
    f"INSERT INTO groups (name) VALUES ('{name}')"  # SQL injection!
)
```

## 📱 Работа с Telegram API

### Обработка сообщений
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shinshigrisha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
