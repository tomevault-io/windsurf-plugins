---
trigger: always_on
description: Debugging, logging, and testing patterns
---


# Debugging and Testing Patterns

## Logging

### LogManager Usage
Use LogManager for structured logging:

```csharp
using UGTLive;

// Info log
LogManager.Instance.LogInfo("Processing started");

// Error log
LogManager.Instance.LogError("Failed to process", exception);

// Debug log
LogManager.Instance.LogDebug("Debug information");
```

### Console Output
Use Console.WriteLine for debug output:

```csharp
Console.WriteLine($"Processing {count} items");
Console.WriteLine($"Config value: {key} = {value}");
```

### Masking Sensitive Data
Never log API keys or sensitive information:

```csharp
// BAD
Console.WriteLine($"API Key: {apiKey}");

// GOOD
Console.WriteLine($"API Key: {(string.IsNullOrEmpty(apiKey) ? "not set" : "***")}");
```

## Error File Writing

### Service Error Files
Some services write error details to files for debugging:

```csharp
try
{
    // API call
}
catch (Exception ex)
{
    string errorFile = "gemini_last_error.txt";
    File.WriteAllText(errorFile, 
        $"Error: {ex.Message}\n\nStack trace: {ex.StackTrace}");
    
    Console.WriteLine($"Error written to {errorFile}");
}
```

## Debug Configuration Files

### Debug Output Files
The app writes debug files for troubleshooting:

- `last_llm_request_sent.txt` - Last LLM request
- `last_llm_reply_received.txt` - Last LLM response
- `last_ocr_response.json` - Last OCR result
- `gemini_last_error.txt` - Last Gemini error
- `openai_audio_log.txt` - OpenAI audio debug log

### Writing Debug Files
```csharp
public void WriteDebugFile(string filename, string content)
{
    try
    {
        string debugPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, filename);
        File.WriteAllText(debugPath, content);
        Console.WriteLine($"Debug file written: {debugPath}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Failed to write debug file: {ex.Message}");
    }
}
```

## Exception Handling

### Unhandled Exception Handler
App.xaml.cs handles unhandled exceptions:

```csharp
private void App_DispatcherUnhandledException(object sender, 
    DispatcherUnhandledExceptionEventArgs e)
{
    Console.WriteLine($"Unhandled exception: {e.Exception.Message}");
    Console.WriteLine($"Stack trace: {e.Exception.StackTrace}");
    
    // Mark as handled to prevent crash
    e.Handled = true;
}
```

### Service Exception Handling
Services should catch and log exceptions:

```csharp
public async Task<Result?> ProcessAsync()
{
    try
    {
        return await DoWorkAsync();
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"HTTP error: {ex.Message}");
        LogManager.Instance.LogError("HTTP request failed", ex);
        return null;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Unexpected error: {ex.Message}");
        LogManager.Instance.LogError("Unexpected error", ex);
        return null;
    }
}
```

## Debug Builds

### Debug vs Release
Debug builds use different assembly names:

```xml
<PropertyGroup Condition="'$(Configuration)'=='Debug'">
    <AssemblyName>ugtlive_debug</AssemblyName>
</PropertyGroup>
```

### Conditional Compilation
```csharp
#if DEBUG
    Console.WriteLine("Debug mode - extra logging enabled");
#endif
```

## Testing Translation Services

### Manual Testing Pattern
1. Set API key in settings
2. Select service and model
3. Use Monitor window to capture text
4. Check ChatBox for results
5. Review logs for errors

### Service Connection Test
```csharp
public async Task<bool> TestConnectionAsync()
{
    try
    {
        var result = await TranslateAsync("test", "en", "");
        return result != null;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Connection test failed: {ex.Message}");
        return false;
    }
}
```

## OCR Testing

### Testing OCR Output
```csharp
// Write OCR results to file for inspection
var ocrResults = await ProcessOCRAsync(bitmap);
var json = JsonSerializer.Serialize(ocrResults, new JsonSerializerOptions 
{ 
    WriteIndented = true 
});
File.WriteAllText("last_ocr_response.json", json);
```

### Visual OCR Feedback
Monitor window shows OCR results visually:

```csharp
private void DrawOCRResults(Graphics g, List<TextObject> textObjects)
{
    foreach (var textObj in textObjects)
    {
        // Draw bounding box
        g.DrawRectangle(Pens.Red, textObj.BoundingBox);
        
        // Draw text
        g.DrawString(textObj.Text, font, Brushes.White, textObj.BoundingBox);
    }
}
```

## Performance Debugging

### Timing Operations
```csharp
var stopwatch = System.Diagnostics.Stopwatch.StartNew();
await ProcessAsync();
stopwatch.Stop();
Console.WriteLine($"Processing took {stopwatch.ElapsedMilliseconds}ms");
```

### Memory Usage
```csharp
using System.Diagnostics;

var process = Process.GetCurrentProcess();
Console.WriteLine($"Memory usage: {process.WorkingSet64 / 1024 / 1024} MB");
```

## Network Debugging

### HTTP Request Logging
```csharp
// Log request details
Console.WriteLine($"Request URL: {url}");
Console.WriteLine($"Request Method: POST");
Console.WriteLine($"Request Body: {requestBody}");

var response = await httpClient.PostAsync(url, content);

Console.WriteLine($"Response Status: {response.StatusCode}");
var responseBody = await response.Content.ReadAsStringAsync();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/UGTLive](https://github.com/SethRobinson/UGTLive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
