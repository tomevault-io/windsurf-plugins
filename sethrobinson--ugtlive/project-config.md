---
trigger: always_on
description: Architecture patterns and project structure guidelines
---


# Architecture Patterns

## Project Structure

### Directory Organization
```
src/
  - Core application files (App.xaml, Logic.cs, ConfigManager.cs)
  - Window classes (MainWindow, MonitorWindow, ChatBoxWindow, SettingsWindow, etc.)
  - Translation services (Gemini, ChatGPT, Ollama, GoogleTranslate, LlamaCpp)
  - OCR services (WindowsOCRManager, GoogleVisionOCRService)
  - Managers (UniversalBlockDetector, LogManager, HotkeyManager, PythonServicesManager, AudioPlaybackManager, etc.)
  - Utilities (TextObject, TranslationEventArgs, etc.)
app/
  - Compiled binaries
  - services/ (Python OCR services - EasyOCR, MangaOCR, PaddleOCR, DocTR)
    - shared/ (Shared Python utilities)
    - util/ (Python installation utilities)
    - EasyOCR/, MangaOCR/, PaddleOCR/, DocTR/ (Individual service directories)
  - media/ (Resources)
```

### File Naming
- One class per file
- File name matches class name
- XAML files: `WindowName.xaml` and `WindowName.xaml.cs`

## Design Patterns

### Singleton Pattern
Used for core managers:
- `ConfigManager.Instance`
- `Logic.Instance`
- `UniversalBlockDetector.Instance` - Advanced text block detection (replaces BlockDetectionManager)
- `LogManager.Instance`
- `PythonServicesManager.Instance` - Manages Python OCR services
- `HotkeyManager.Instance` - Manages global keyboard shortcuts (replaces KeyboardShortcuts)
- `AudioPlaybackManager.Instance` - Manages audio playback
- `AudioPreloadService.Instance` - Preloads TTS audio
- `ErrorPopupManager` - Static class for error popups
- `GamepadManager.Instance` - Manages gamepad input
- `WebViewEnvironmentManager` - Static class for WebView2 environment

**Implementation:**
```csharp
private static ConfigManager? _instance;

public static ConfigManager Instance
{
    get
    {
        if (_instance == null)
        {
            _instance = new ConfigManager();
        }
        return _instance;
    }
}

private ConfigManager() { }
```

### Factory Pattern
Used for creating service instances:

```csharp
public static class TranslationServiceFactory
{
    public static ITranslationService CreateService(string serviceName)
    {
        return serviceName switch
        {
            "Gemini" => new GeminiTranslationService(),
            "ChatGPT" => new ChatGptTranslationService(),
            "Ollama" => new OllamaTranslationService(),
            "Google Translate" => new GoogleTranslateService(),
            "llama.cpp" => new LlamaCppTranslationService(),
            _ => throw new ArgumentException(...)
        };
    }
}
```

### Strategy Pattern
Translation services implement common interface:

```csharp
public interface ITranslationService
{
    Task<TranslationResult?> TranslateAsync(
        string sourceText,
        string targetLanguage,
        string context);
}
```

## Separation of Concerns

### UI Layer
- XAML files define UI structure
- Code-behind handles UI events
- Minimal business logic in UI code

### Business Logic Layer
- `Logic.cs` - Core translation workflow
- `UniversalBlockDetector.cs` - Advanced text block detection and grouping
- Service classes - External API integration

### Data Layer
- `ConfigManager.cs` - Configuration persistence
- File-based storage (config.txt, service-specific configs)

## Dependency Management

### Service Dependencies
Services depend on ConfigManager, not each other:

```csharp
public class GeminiTranslationService
{
    private readonly string _apiKey;
    
    public GeminiTranslationService()
    {
        _apiKey = ConfigManager.Instance.GetGeminiApiKey();
    }
}
```

### Manager Dependencies
Managers can depend on other managers:

```csharp
public class Logic
{
    private readonly ConfigManager _configManager;
    private readonly UniversalBlockDetector _blockDetector;
    
    public Logic()
    {
        _configManager = ConfigManager.Instance;
        _blockDetector = UniversalBlockDetector.Instance;
    }
}
```

## Communication Patterns

### Window Communication
Windows communicate through Logic.Instance:

```csharp
// In ChatBoxWindow
Logic.Instance.OnTranslationReceived += HandleTranslation;

// In Logic
public event EventHandler<TranslationEventArgs>? OnTranslationReceived;
```

### Event-Driven Architecture
Use events for loose coupling:

```csharp
public class TranslationEventArgs : EventArgs
{
    public string TranslatedText { get; set; } = "";
    public string SourceText { get; set; } = "";
}
```

### Hotkey System
Hotkeys managed through HotkeyManager events:

```csharp
HotkeyManager.Instance.StartStopRequested += OnStartStopRequested;
HotkeyManager.Instance.MonitorToggleRequested += OnMonitorToggleRequested;
```

## Resource Management

### Disposable Resources
Implement IDisposable for resources:

```csharp
public class ResourceManager : IDisposable
{
    private bool _disposed = false;
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Dispose managed resources
            }
            // Dispose unmanaged resources
            _disposed = true;
        }
    }
}
```

### Using Statements
Always use `using` for disposable resources:

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/UGTLive](https://github.com/SethRobinson/UGTLive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
