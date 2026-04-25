---
trigger: always_on
description: > Язык: Python 3.13+, Windows
---

# Invest Desktop Watcher — Техническое задание

> Версия: v2.4.0
> Язык: Python 3.13+, Windows
> Точка входа: `main.py`
> Репозиторий: https://github.com/Damfler/Invest-Dekstop-Watcher

---

## Контекст для Claude

Это **виджет системного трея** для отслеживания инвестиций. Архитектура:
- **Брокеры**: сейчас только ТБанк (`api.py`), структура подготовлена для других (поле `broker` в конфиге)
- **Данные**: `data_store.py` (DataStore, потокобезопасно) → `menu.py` + `window.py` (только читают через `snapshot()`)
- **GUI**: `pywebview` (HTML-дашборд) + `pystray` (иконка трея), оба в разных потоках
- **Константы**: всё в `constants.py` — алерты, таймеры, GITHUB_REPO, TOKEN_STUB, BROKERS, BROKER_INFO, WM_коды
- **Апдейтер**: `updater.py` → `check_for_update()` → `download_update()` → `apply_update()`. Работает только в `.exe` режиме (`sys.frozen`). Меню показывает кнопку если `update_info.available`
- **Мастер**: `wizard.py` запускается если `token == TOKEN_STUB`. Содержит выбор брокера.
- **APPDATA путь**: `%APPDATA%\InvestDesktopWatcher\` (в `.exe`) или рядом со скриптом (в dev)

Не менять без причины: структуру `snapshot()`, callable-меню pystray, monkey-patch ЛКМ, однопоточность webview.

---

---

## 1. Назначение

Виджет в системном трее Windows для отслеживания инвестиционного портфеля.
Текущий брокер: T-Bank Invest API (REST). Структура подготовлена для подключения других брокеров.
Показывает стоимость счетов, P&L за день или за всё время, события по облигациям (купоны, оферты, погашения).

**Левый клик** на иконке → открывает popup-дашборд.  
**Правый клик** → контекстное меню со всеми данными.

---

## 2. Структура файлов

```
invest-desktop-watcher/
├── main.py            # Точка входа. Логирование, wizard, запуск app.
├── version.py         # Единый источник версии (APP_VERSION, APP_NAME).
├── constants.py       # ВСЕ константы приложения (алерты, таймеры, брокеры, WM_*, TOKEN_STUB, GITHUB_REPO).
├── config.py          # Загрузка/сохранение config.json. Поддержка .env.
├── api.py             # T-Bank Invest REST API. Retry + rate limit 429.
├── cache.py           # Дисковый кэш (cache.json 24ч + history.json бессрочно).
├── utils.py           # Форматирование чисел/дат, parse_ts, days_until.
├── icons_gen.py       # Генерация иконок трея (PIL). Стрелки вверх/вниз.
├── autostart.py       # Автозапуск через реестр Windows (winreg).
├── notifications.py   # Toast-уведомления через plyer.
├── analytics.py       # YTM, аллокация, топ-муверы, денежный поток по месяцам.
├── data_store.py      # Хранилище данных. Потокобезопасный DataStore + кэш инструментов.
├── menu.py            # Построитель меню pystray (callable, подменю).
├── app.py             # Главный класс TBankTrayApp. pywebview main + pystray detached.
├── window.py          # Менеджер окна pywebview + JS API + Excel XML экспорт.
├── dashboard.html     # HTML-дашборд (Chart.js, Lucide Icons, 5 табов).
├── wizard.py          # Мастер первого запуска (выбор брокера + ввод токена).
├── updater.py         # Автообновление через GitHub Releases.
├── .env               # Токен для разработки (в .gitignore).
├── tbank_invest.spec  # PyInstaller spec для сборки .exe.
├── build.bat          # Сборка .exe одной командой.
├── requirements.txt   # Зависимости pip.
├── .github/workflows/
│   └── build.yml      # GitHub Actions: сборка по тегу + ручной запуск.
└── icons/
    ├── positive.png   # Иконка трея: портфель в плюсе
    ├── negative.png   # Иконка трея: портфель в минусе
    ├── warn.png       # Оферта скоро (опционально)
    └── crit.png       # Оферта сегодня (опционально)
```

**При сборке .exe:**
- Пользовательские данные: `%APPDATA%\InvestDesktopWatcher\` (config, cache, logs)
- dashboard.html + icons упакованы внутрь .exe
- Пользователь видит только `tbank_invest.exe`

**Автоматически создаваемые файлы (в gitignore):**
```
cache.json         # Кэш последних данных
dismissed.json     # Скрытые предупреждения об офертах
tbank_errors.log   # Лог ошибок (ротация 1МБ × 3 файла)
config.json.bak    # Бэкап при повреждении конфига
```

---

## 3. Конфигурация (config.json)

```json
{
  "token": "YOUR_TBANK_API_TOKEN_HERE",
  "use_sandbox": false,
  "use_custom_icons": true,
  "bond_horizon_days": 60,
  "bond_sort": "date",
  "notify_move_pct": 1.0,
  "notify_offer_days": 2,
  "notifications": {
    "offer_warn": true,
    "offer_crit": true,
    "coupon_tomorrow": true,
    "portfolio_move": true
  }
}
```

| Поле | Тип | Описание |
|---|---|---|
| `token` | str | API-токен T-Bank (только чтение). Перезаписывается из `.env` |
| `use_sandbox` | bool | Sandbox-режим API |
| `use_custom_icons` | bool | Использовать файлы из `icons/` или рисовать кодом |
| `bond_horizon_days` | int | Горизонт событий облигаций (30-365). Меняется в настройках |
| `bond_sort` | str | Сортировка событий: `"date"` или `"amount"` |
| `notify_move_pct` | float | Порог движения портфеля для уведомления (%) |
| `notify_offer_days` | int | За сколько дней предупреждать об оферте |
| `max_bond_events` | int | Макс. событий в меню трея (по умолчанию 50) |
| `theme` | str | Тема дашборда: `"system"` / `"dark"` / `"light"` |
| `use_logos` | bool | Показывать логотипы инструментов из CDN |
| `app_name` | str | Пользовательское название приложения |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Damfler) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
