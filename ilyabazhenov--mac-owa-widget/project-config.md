---
trigger: always_on
description: Отвечай пользователю всегда на русском языке.
---

# AGENTS.md

## Язык

Отвечай пользователю всегда на русском языке.

Код и комментарии в коде пиши на английском языке.

## Source of truth

Этот файл является основным источником агентских инструкций в репозитории.
Если аналогичные правила встречаются в других файлах, при расхождении следуй этому файлу.

## Приоритет инструкций

Применяй инструкции в таком порядке:

1. Прямой запрос пользователя в текущем чате.
2. Ограничения безопасности и целостности проекта.
3. Процедурные workflow-правила этого файла (сборка, проверка, релиз).

## Контекст проекта

OWAWidget - macOS menu bar приложение на Swift 6 и SwiftUI для просмотра ближайших встреч и быстрого перехода в онлайн-звонки из календаря Microsoft Exchange / OWA.

Основной код находится в `OWAWidget/`.

Ключевые части:

- `OWAWidget/OWAWidgetApp.swift` - точка входа приложения, `MenuBarExtra`, окно настроек, обработка уведомлений.
- `OWAWidget/Services/CalendarService.swift` - главный `@MainActor` источник состояния, аккаунтов, событий и синхронизации.
- `OWAWidget/Providers/CalendarProvider.swift` - общий протокол календарных провайдеров.
- `OWAWidget/Providers/OWA/` - интеграция с OWA: авторизация, CANARY token, запрос календаря и маппинг событий.
- `OWAWidget/Providers/GoogleCalendar/` - заглушка будущего Google Calendar провайдера через прямой API (OAuth). Не используется: календари Google приезжают через EventKit.
- `OWAWidget/Providers/EventKit/` - чтение календарей, которые macOS уже синхронизирует (Google, iCloud, локальные). Провайдер read-only: мутирующие методы `CalendarProvider` остаются `notSupported`.

> **Доступ к календарям требует entitlement.** `Makefile` подписывает с `--options runtime`, а hardened runtime закрывает TCC-ресурсы без явного разрешения - даже вне песочницы. Без `com.apple.security.personal-information.calendars` в `OWAWidget-dev.entitlements` вызов `requestFullAccessToEvents` возвращает `false` за миллисекунды, статус остаётся `notDetermined`, и системный диалог не показывается вообще. Отладка такого молчания легко уходит в ложные версии - проверено на зонде 2026-08-22. `Info.plist` при этом обязан содержать обе строки: `NSCalendarsUsageDescription` (macOS 13) и `NSCalendarsFullAccessUsageDescription` (macOS 14+); отсутствие строки - это крэш в момент запроса, а не отказ.
>
> **Выданный доступ не переживает смену подписи.** TCC привязывает разрешение к подписи бандла, а подпись ad-hoc, поэтому статус возвращается в `notDetermined`, а синхронизация EventKit падает с "Calendar access has not been granted yet". Сбрасывает разрешение любое изменение подписанного содержимого - не только новый код: `make bundle` берёт `CFBundleVersion` из `git rev-list --count HEAD`, так что достаточно одного коммита, чтобы `Info.plist` изменился и подпись стала другой. Повторный `make run` без единого изменения, наоборот, доступ сохраняет: подпись ad-hoc детерминирована. У пользователей то же самое случается после каждого обновления через Sparkle. Это не баг в коде - не ищи его там. Приложение справляется само: `EventKitCalendarProvider.fetchEvents` вызывает `ensureReadAccess()`, и при статусе `notDetermined` система показывает диалог на первом же синке после обновления. Достаточно подтвердить его. Запрос не срабатывает у того, кто уже отказал: отказ - это тоже решение, и переспрашивать каждый синк значило бы донимать.

> **Тесты не должны трогать реальный EventKit.** Причина та же, что у Keychain: `swift test` - обязательный гейт `make release-package`, а диалог доступа к календарям повесит упаковку. Всё, что пересекает границу `EventKitStoring`, - это `Sendable`-снимки (`EventKitSnapshots.swift`), а `EKEventStore` не покидает `SystemEventKitStore`. Инжектируй фейковый стор через `CalendarService(eventKitStore:)` или `EventKitCalendarProvider(account:store:)`.
- `OWAWidget/Services/MeetingURLDetector.swift` - поиск ссылок на Teams, Zoom, Webex, Google Meet и другие платформы.
- `OWAWidget/Services/NotificationService.swift` - локальные уведомления о встречах.
- `OWAWidget/Services/KeychainService.swift` - хранение паролей в Keychain.
- `OWAWidget/Services/SecureStore.swift` - шифрование всех данных на диске (AES-GCM, мастер-ключ в Keychain). Всё, что пишется в `~/Library/Application Support/OWAWidget/<bundle-id>/store/`, проходит через него. Новые хранилища добавляй сюда, а не в `UserDefaults`: там место только для настроек интерфейса.
- `OWAWidget/Services/SecureCodableStore.swift` - `Codable`-обёртка над `SecureStore` с одноразовой миграцией из открытого `UserDefaults`-ключа. Порядок миграции обязателен: записали -> перечитали и сверили -> удалили legacy.
- `OWAWidget/Services/SecureStoreMigrator.swift` - принудительный прогон миграций на старте для хранилищ, которые иначе мигрировали бы лениво.

> **Тесты не должны трогать реальный Keychain.** `swift test` - обязательный гейт `make release-package`, а диалог авторизации связки повесит упаковку. Инжектируй `SecureStore(directory:keyProvider:)` с `InMemorySecureStoreKeyProvider`; `SecureStore.shared` под XCTest сам уходит во временный каталог, но это страховка, а не замена инжекции.
- `OWAWidget/Services/LaunchAtLoginService.swift` - автозапуск при входе (`SMAppService.mainApp`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilyabazhenov/mac-owa-widget](https://github.com/ilyabazhenov/mac-owa-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
