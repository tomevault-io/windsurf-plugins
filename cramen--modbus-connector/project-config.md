---
trigger: always_on
description: GUI-приложение на PySide6 для отладки шины Modbus и разработки Modbus-устройств.
---

# Agent Guidance: modbus_connector

## Назначение

GUI-приложение на PySide6 для отладки шины Modbus и разработки Modbus-устройств.
Каждая вкладка — сессия в режиме Master (опрос устройства), Slave
(встроенный симулятор устройства: Modbus TCP/RTU сервер на pymodbus с
редактируемой картой значений и правилами-выражениями), Sniffer (пассивное
прослушивание RTU-шины через отдельный serial-адаптер: карта регистров
восстанавливается из подслушанного трафика чужого мастера) или Gateway
(прозрачный шлюз: listen-сервер TCP/RTU over TCP/RTU транслирует каждый
запрос в target-устройство любого типа подключения, с фильтром unit-адресов).
Подключение к устройствам по Modbus TCP и RTU (по умолчанию RTU), чтение/запись
регистров из таблицы, поллинг с интервалом, сканер unit-адресов в отдельном окне.
Настройки соединений (вкладки), списки регистров и состояния сканеров
сохраняются между запусками в `~/.modbus_connector/settings.json`
(`settings_store.py`, формат `{"tabs": [...], "active_tab": i}`); через
меню File их же можно сохранить/загрузить в произвольный JSON-файл.

## Архитектура

- Вся Modbus-логика — в `backend.py` на **синхронных** клиентах pymodbus
  (`ModbusTcpClient`, `ModbusSerialClient`). backend.py и models.py не импортируют Qt.
- Qt-слой тонкий: `worker.py` (QObject в отдельном QThread, сигналы/слоты над
  backend) + виджеты (panels, main_window, app). Никакой Modbus-логики в виджетах.
- Async pymodbus не используется.

## Стек

- Python 3.11+, PySide6 (полный метапакет — включает Addons; alarm_sound.py
  использует QtMultimedia QSoundEffect, PyInstaller подхватывает импорт сам)
- `pyqtgraph` (тянет numpy) — живые графики значений регистров
- `pyqtdarktheme` — темы System/Light/Dark (theme.py)
- `pymodbus[serial]==3.6.9` — sync-клиенты; методы чтения/записи принимают
  ключевой `slave=`, результат: `.registers` (регистры) / `.bits` (coils,
  discrete inputs), ошибки — `result.isError()`; extra `serial` = pyserial;
  это ЕДИНСТВЕННАЯ базовая зависимость — Qt-стек (PySide6/pyqtgraph/
  pyqtdarktheme) вынесен в extra `gui`, чтобы headless-CLI ставился lean;
  без `gui` gui-script `modbus-connector` падает с подсказкой (app.py,
  exit 4); console-script `modbus-connector-cli` → cli.py (без Qt)
- dev (extra `dev`): Qt-стек (дублирует `gui` — extras не ссылаются друг на
  друга) + `pytest`, `pytest-asyncio`, `ruff`
- build (extra `build`): `pyinstaller>=6`

## Структура (src-layout)

```
pyproject.toml
src/modbus_connector/
  models.py       # без Qt: RegisterKind, TcpParams/RtuParams,
                  # RtuOverTcpParams/RtuOverUdpParams, ConnectionParams,
                  # describe_connection(params) — "tcp host:port" и т.п.,
                  # RegisterRow, ScanProbe, DEFAULT_SCAN_PROBES, DisplayFormat,
                  # ReadSpec/ReadMember/ReadPlan + plan_grouped_reads(rows,
                  # max_gap=8) — объединение чтений соседних адресов в один
                  # запрос: группировка по (unit, kind), сортировка по адресу,
                  # мердж при зазоре ≤ max_gap (перекрытия тоже; members
                  # помнят offset/count), кап длины плана 125 регистров /
                  # 2000 бит, count<=0/address<0 пропускаются,
                  # ByteOrder, parse_values(kind, text), format_values(values),
                  # decode_register_values(values, fmt, order) — decode до чисел,
                  # encode_register_values(value, fmt, order) — обратный encode
                  # числа в регистры (round+clamp для целых, f32/f64 через
                  # struct.pack, OverflowError на непредставимом; hex/ascii —
                  # ValueError), для правил симулятора,
                  # register_width(fmt) — регистров на значение (1/2/4),
                  # parse_formatted_values(text, fmt, count) — ввод чисел в
                  # формате отображения (поверх encode),
                  # encode_ascii_values(text, count) — текст → регистры
                  # (2 символа на регистр, NUL-pad, не-ASCII → «?»),
                  # format_register_values (поверх decode),
                  # format_scaled_values (x*scale+offset по decoded),
                  # rows_to_csv/rows_from_csv/row_to_csv_record/CSV_COLUMNS —
                  # CSV таблицы регистров (толерантный разбор,
                  # guess_column_mapping/csv_header, subset колонок),
                  # EXCEPTION_CODES/describe_exception — имена Modbus-исключений,
                  # AlarmRule/evaluate_alarm/rule_matches/alarm_rule_to_json/
                  # alarm_rules_from_json — правила алармов (иммутабельные,
                  # приоритет = порядок, границы диапазонов включительные),
                  # Stats/StatsSnapshot — счётчики транзакций (ok/err, avg ms,
                  # разбивка ошибок по видам),
                  # diff_snapshots(old, new) — сравнение двух RAW-снимков
                  # значений строки (None = «нет данных», None/None = нет
                  # различия) для snapshot diff;
                  # parse_expression/Expression (.text/.deps/.evaluate) —
                  # движок выражений: ссылки [имя] на строки, whitelisted
                  # арифметика/функции (AST-валидация, eval без builtins),
                  # ValueError на мусоре, KeyError = нет строки, nan = мат.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cramen/modbus_connector](https://github.com/cramen/modbus_connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
