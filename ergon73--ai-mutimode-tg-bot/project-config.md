---
trigger: always_on
description: Ты — AI-ассистент для разработки Telegram-бота с LLM. Твоя задача — генерировать чистый, профессиональный код на Python, который будет использоваться в учебном проекте курса вайб-кодинга.
---

# Правила для Cursor AI: Telegram AI Assistant проект

## 🎯 Общие принципы

Ты — AI-ассистент для разработки Telegram-бота с LLM. Твоя задача — генерировать чистый, профессиональный код на Python, который будет использоваться в учебном проекте курса вайб-кодинга.

## 📝 Стиль кода

### Язык комментариев
- **ВСЕ комментарии только на РУССКОМ языке**
- Docstrings на русском в формате:
  ```python
  def function_name(param: str) -> int:
      """
      Краткое описание функции.
      
      Args:
          param: описание параметра
      
      Returns:
          Описание возвращаемого значения
      """
  ```

### Стандарты кодирования
- **PEP 8** — строго следовать стандарту
- Максимальная длина строки: **100 символов**
- Использовать **type hints** для всех функций и методов
- Имена переменных: **snake_case**
- Имена классов: **PascalCase**
- Константы: **UPPER_CASE**

### Структура кода
```python
# 1. Импорты стандартной библиотеки
import asyncio
import logging

# 2. Импорты сторонних библиотек
from aiogram import Bot, Dispatcher
from openai import AsyncOpenAI

# 3. Локальные импорты
from config import settings
from memory import MemoryManager
```

## 🏗 Архитектурные требования

### Асинхронность
- **Всегда используй async/await** для I/O операций
- Используй `AsyncOpenAI` вместо синхронного клиента
- Используй `aiohttp` для HTTP-запросов
- Обработчики aiogram всегда асинхронные

### Модульность
- **Один файл = одна ответственность**
- Не смешивай бизнес-логику и UI-логику
- Классы должны следовать **Single Responsibility Principle**

### Обработка ошибок
```python
# ✅ ПРАВИЛЬНО: Логируем и информируем пользователя
try:
    result = await api_call()
except SomeAPIError as e:
    logging.error(f"Ошибка API: {e}")
    await message.answer("❌ Произошла ошибка. Попробуйте позже.")
    return

# ❌ НЕПРАВИЛЬНО: Молчаливо игнорируем
try:
    result = await api_call()
except:
    pass
```

## 📂 Структура проекта

Придерживайся следующей структуры:

```
telegram-ai-assistant/
├── main.py              # Точка входа, хендлеры Telegram
├── config.py            # Конфигурация через Pydantic Settings
├── memory.py            # MemoryManager класс
├── llm_service.py       # LLMService для работы с OpenAI
├── image_service.py     # (Опционально) ImageService для GNAPI
├── prompts.json         # Ролевые модели
├── requirements.txt     # Python зависимости
├── .env                 # Секреты (НЕ коммитить!)
├── .env.example         # Пример конфигурации
├── .gitignore           # Исключения Git
├── README.md            # Документация
└── memory.json          # Хранилище памяти (НЕ коммитить!)
```

## 🔧 Специфика компонентов

### config.py
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """Конфигурация приложения из .env файла"""
    
    BOT_TOKEN: str
    OPENAI_API_KEY: str
    OPENAI_MODEL: str = "gpt-4o-mini"
    MAX_HISTORY_MESSAGES: int = 10
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

settings = Settings()
```

### memory.py
- Класс `MemoryManager` с методами:
  - `add_message()` — добавить сообщение
  - `get_history()` — получить историю
  - `clear_history()` — очистить для чата
  - `set_mode()` / `get_mode()` — управление режимами
  - `save()` / `load()` — персистентность
- Хранилище: `Dict[int, Dict]` где ключ = chat_id
- Автосохранение после каждого изменения

### llm_service.py
- Класс `LLMService` с методами:
  - `generate_response()` — основной метод генерации
  - `calculate_cost()` — расчёт стоимости (если используется)
- Использовать `AsyncOpenAI` клиент
- Обрабатывать ошибки OpenAI API

### main.py
- Хендлеры команд: `/start`, `/mode`, `/reset`
- Обработчик текстовых сообщений
- InlineKeyboard для выбора режима
- Логика "typing..." при обработке

## 🎨 Форматирование ответов бота

### Приветственное сообщение
```python
WELCOME_MESSAGE = """
👋 Привет! Я AI-ассистент с несколькими режимами работы.

📚 Доступные команды:
/mode — выбрать режим работы
/reset — очистить историю диалога
/start — показать это сообщение

Текущий режим: {current_mode}
"""
```

### Статистика стоимости
```python
COST_TEMPLATE = """
💰 Стоимость запроса:
📥 Входные токены: {input_tokens}
📤 Выходные токены: {output_tokens}
💵 USD: ${cost_usd:.5f}
💸 RUB: ~{cost_rub:.2f}₽
"""
```

## 🔒 Безопасность

### Секреты
- **НИКОГДА не коммить `.env` файл**
- Всегда создавать `.env.example` с заглушками
- В `.gitignore` обязательно:
  ```
  .env
  memory.json
  venv/
  __pycache__/
  *.pyc
  ```

### Валидация входных данных
```python
# ✅ ПРАВИЛЬНО: Валидация через Pydantic
from pydantic import BaseSettings, Field

class Settings(BaseSettings):
    BOT_TOKEN: str = Field(..., min_length=40)
    OPENAI_API_KEY: str = Field(..., min_length=20)

# ❌ НЕПРАВИЛЬНО: Прямое использование без проверок
token = os.getenv("BOT_TOKEN")  # Может быть None!
```

## 📊 Логирование

### Настройка
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

logger = logging.getLogger(__name__)
```

### Что логировать
- ✅ Старт/остановка бота

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ergon73) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
