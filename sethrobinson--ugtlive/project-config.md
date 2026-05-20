---
trigger: always_on
description: Patterns for integrating external services (APIs, OCR, TTS)
---


# Service Integration Patterns

## Translation Service Interface

### ITranslationService Pattern
All translation services implement a common interface:

```csharp
public interface ITranslationService
{
    Task<TranslationResult?> TranslateAsync(
        string sourceText,
        string targetLanguage,
        string context);
}
```

### TranslationResult Structure
```csharp
public class TranslationResult
{
    public string TranslatedText { get; set; } = "";
    public string SourceLanguage { get; set; } = "";
    public double Confidence { get; set; } = 1.0;
}
```

## HTTP Client Usage

### Service HTTP Client Pattern
Each service creates its own HttpClient:

```csharp
public class GeminiTranslationService : ITranslationService
{
    private static readonly HttpClient _httpClient = new HttpClient();
    private readonly string _apiKey;
    
    public GeminiTranslationService()
    {
        _apiKey = ConfigManager.Instance.GetGeminiApiKey();
    }
    
    public async Task<TranslationResult?> TranslateAsync(string text, string lang, string context)
    {
        // Implementation
    }
}
```

### Request Building
```csharp
var requestBody = new
{
    contents = new[]
    {
        new
        {
            parts = new[]
            {
                new { text = prompt }
            }
        }
    }
};

var json = JsonSerializer.Serialize(requestBody);
var content = new StringContent(json, Encoding.UTF8, "application/json");
var response = await _httpClient.PostAsync(url, content);
```

## Error Handling in Services

### HTTP Error Handling Pattern
```csharp
if (response.IsSuccessStatusCode)
{
    var result = await response.Content.ReadAsStringAsync();
    return ParseResult(result);
}
else
{
    string errorMessage = await response.Content.ReadAsStringAsync();
    Console.WriteLine($"API error: {response.StatusCode}, {errorMessage}");
    
    // Try to parse JSON error
    try
    {
        using JsonDocument errorDoc = JsonDocument.Parse(errorMessage);
        if (errorDoc.RootElement.TryGetProperty("error", out JsonElement errorElement))
        {
            // Extract detailed error
            return null;
        }
    }
    catch (JsonException)
    {
        // Fallback to raw message
    }
    
    // Show user-friendly error
    ErrorPopupManager.ShowError("Translation Error", errorMessage);
    
    return null;
}
```

### Exception Handling
```csharp
try
{
    var response = await _httpClient.PostAsync(url, content);
    // Process response
}
catch (Exception ex)
{
    Console.WriteLine($"Service error: {ex.Message}");
    LogManager.Instance.LogError("Translation failed", ex);
    
    ErrorPopupManager.ShowError("Translation Error", ex.Message);
    
    return null;
}
```

## Service Factory Pattern

### TranslationServiceFactory
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
            _ => throw new ArgumentException($"Unknown service: {serviceName}")
        };
    }
}
```

## OCR Service Integration

### Windows OCR Pattern
```csharp
public class WindowsOCRManager
{
    public async Task<List<TextObject>> ProcessImageAsync(Bitmap bitmap)
    {
        return await Task.Run(() =>
        {
            // Convert bitmap to Windows bitmap
            // Process with Windows OCR API
            // Return TextObject list
        });
    }
}
```

### Google Vision OCR Pattern
```csharp
public class GoogleVisionOCRService
{
    public async Task<List<TextObject>> ProcessImageAsync(Bitmap bitmap)
    {
        // Convert bitmap to base64
        // Send to Google Vision API
        // Parse response and return TextObject list
    }
}
```

### Python OCR Service Pattern (HTTP-based)
```csharp
// Services are managed via PythonServicesManager
// Discover services on startup
PythonServicesManager.Instance.DiscoverServices();

// Get service by name
var service = PythonServicesManager.Instance.GetServiceByName("EasyOCR");

// Check if service is running
if (!service.IsRunning)
{
    bool isRunning = await service.CheckIsRunningAsync();
    if (!isRunning)
    {
        // Show error dialog or start service
        await service.StartAsync(showWindow: false);
    }
}

// Process image with HTTP service
private async Task<string?> ProcessImageWithHttpServiceAsync(byte[] imageBytes, string serviceName, string language)
{
    var service = PythonServicesManager.Instance.GetServiceByName(serviceName);
    if (service == null) return null;
    
    // Build URL with query parameters
    string langParam = MapLanguageForService(language);
    string url = $"{service.ServerUrl}:{service.Port}/process?lang={langParam}&char_level=true";
    
    // Add service-specific parameters (e.g., MangaOCR)
    if (serviceName == "MangaOCR")
    {
        url += $"&min_region_width={minWidth}&min_region_height={minHeight}&overlap_allowed_percent={overlapPercent}";
    }
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/UGTLive](https://github.com/SethRobinson/UGTLive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
