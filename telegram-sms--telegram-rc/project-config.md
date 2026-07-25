---
trigger: always_on
description: This is an Android application that enables remote control of Android devices via Telegram Bot API. The app is written in Kotlin and uses modern Android development practices.
---

# GitHub Copilot Instructions for Telegram Remote Control

## Project Overview

This is an Android application that enables remote control of Android devices via Telegram Bot API. The app is written in Kotlin and uses modern Android development practices.

**Key Technologies:**
- Language: Kotlin 2.3.0
- Min SDK: Android 10 (API 29)
- Target SDK: Android 14+ (API 36)
- Architecture: Service-based with MMKV for persistent storage
- Primary Libraries: OkHttp, Gson, Shizuku, Room Database, MMKV

## Coding Standards and Conventions

### General Guidelines

1. **Language**: All code must be written in Kotlin
2. **Null Safety**: Leverage Kotlin's null safety features; use `?` and `!!` appropriately
3. **Code Style**: Follow Android Kotlin Style Guide
4. **Naming Conventions**:
   - Classes: PascalCase (e.g., `ChatService`, `MainActivity`)
   - Functions: camelCase (e.g., `sendMessage`, `getNetworkType`)
   - Constants: SCREAMING_SNAKE_CASE (e.g., `TAG`, `CHAT_INFO_MMKV_ID`)
   - Private variables: start with lowercase (e.g., `preferences`, `botToken`)

### Android-Specific Patterns

1. **Context Usage**:
   - Use `applicationContext` for long-lived operations
   - Use activity context only when necessary for UI operations
   - Always pass context as parameter instead of storing static references

2. **Permission Handling**:
   - Always check permissions before using protected APIs
   - Use `ActivityCompat.checkSelfPermission()` pattern
   - Log permission denials with appropriate messages

3. **Background Work**:
   - Use Services for long-running operations
   - Implement JobScheduler/WorkManager for scheduled tasks
   - Always acquire WakeLock and WifiLock when needed
   - Release locks in `onDestroy()`

4. **Threading**:
   - Network calls must be on background threads
   - UI updates must use `runOnUiThread { }`
   - Use Kotlin Coroutines when appropriate

### Project-Specific Patterns

#### 1. Telegram API Communication

```kotlin
// Always use this pattern for Telegram API calls
val requestUri = getUrl(botToken, "sendMessage")
val requestBody = RequestMessage().apply {
    chatId = chatID
    messageThreadId = threadID
    text = messageText
    parseMode = "HTML"
}
val body = Gson().toJson(requestBody).toRequestBody(Const.JSON)
val request = Request.Builder().url(requestUri).method("POST", body).build()
```

#### 2. MMKV Storage Access

```kotlin
// Use named MMKV instances for different data types
val preferences = MMKV.mmkvWithID(Const.SETTINGS_MMKV_ID)
val chatInfo = MMKV.mmkvWithID(Const.CHAT_INFO_MMKV_ID)
val statusMMKV = MMKV.mmkvWithID(Const.STATUS_MMKV_ID)
```

#### 3. Shizuku Integration

```kotlin
// Always check Shizuku availability
if (Shizuku.pingBinder() && Shizuku.checkSelfPermission() == PackageManager.PERMISSION_GRANTED) {
    // Use Shizuku APIs
}
```

#### 4. Logging

```kotlin
// Use consistent logging pattern
Log.d(Const.TAG, "Description: $variable")
Log.e(Const.TAG, "Error description: ", exception)
Log.i(Const.TAG, "Info message")
Log.w(Const.TAG, "Warning message")
```

#### 5. Message Formatting

```kotlin
// System messages always use this header
requestBody.text = "${getString(R.string.system_message_head)}\n$actualMessage"

// Use HTML formatting for Telegram
parseMode = "HTML"
// Use <code>, <b>, <i> tags as needed
```

#### 6. Error Handling

```kotlin
// Always provide user-friendly error messages
try {
    // operation
} catch (e: Exception) {
    Log.e(Const.TAG, "operation_name: ", e)
    runOnUiThread {
        showErrorDialog("User-friendly error: ${e.message}")
    }
}
```

## File Structure Guidelines

### Activity Classes
- Implement proper lifecycle methods
- Clean up resources in `onDestroy()`
- Use `lateinit var` for views that are initialized in `onCreate()`
- Request permissions appropriately

### Service Classes
- Always call `startForeground()` for foreground services
- Specify service type for Android 14+ (`FOREGROUND_SERVICE_TYPE_*`)
- Implement `onStartCommand()` to return `START_STICKY` for persistent services
- Release all locks and resources in `onDestroy()`

### Static Utility Classes
- Place in `static_class` package
- Use `object` for singletons
- Make functions self-contained with all dependencies passed as parameters

### Data Classes
- Place in `data_structure` package
- Use `@SerializedName` for Gson serialization
- Keep data classes simple (no business logic)

## API Integration Guidelines

### Telegram Bot API

1. **Message Sending**:
   - Always include error handling
   - Check response codes
   - Parse error descriptions from response JSON
   - Log all API calls

2. **Keyboard Markup**:
   - Use `ReplyMarkupKeyboard.getReplyKeyboardMarkup()` for persistent keyboards
   - Use `ReplyMarkupKeyboard.getRemoveKeyboardMarkup()` to remove keyboards
   - Use inline keyboards for temporary actions

3. **Message Threading**:
   - Support `message_thread_id` for topic-based groups
   - Always validate topic ID matches configured value

### Shizuku API

1. **Telephony Operations**:
   - Use `ITelephony` interface via Shizuku
   - Handle dual-SIM scenarios
   - Check subscription validity

2. **Network Operations**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telegram-sms/telegram-rc](https://github.com/telegram-sms/telegram-rc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
