---
trigger: always_on
description: Configuration and settings management patterns
---


# Configuration Management

## ConfigManager Pattern

### Singleton Access
Always access ConfigManager through the singleton instance:

```csharp
var apiKey = ConfigManager.Instance.GetGeminiApiKey();
ConfigManager.Instance.SetGeminiApiKey("new-key");
```

### Configuration Keys
- Define keys as `public const string` constants
- Use descriptive UPPER_SNAKE_CASE names
- Group related keys together

```csharp
public const string GEMINI_API_KEY = "gemini_api_key";
public const string GEMINI_MODEL = "gemini_model";
public const string TRANSLATION_SERVICE = "translation_service";
```

## Getter/Setter Pattern

### Standard Pattern
For each configuration value, provide Get/Set methods:

```csharp
// Get method
public string GetGeminiApiKey()
{
    return GetValue(GEMINI_API_KEY);
}

// Set method (auto-saves)
public void SetGeminiApiKey(string apiKey)
{
    _configValues[GEMINI_API_KEY] = apiKey;
    SaveConfig();
}
```

### Typed Getters
Provide typed getters for common types:

```csharp
// Boolean
public bool GetBoolValue(string key, bool defaultValue = false)
{
    if (_configValues.TryGetValue(key, out var value))
    {
        return value.ToLower() == "true" || value == "1";
    }
    return defaultValue;
}

// Integer
public int GetIntValue(string key, int defaultValue = 0)
{
    if (_configValues.TryGetValue(key, out var value))
    {
        if (int.TryParse(value, out int result))
        {
            return result;
        }
    }
    return defaultValue;
}

// Double/Float
public double GetDoubleValue(string key, double defaultValue = 0.0)
{
    if (_configValues.TryGetValue(key, out var value))
    {
        if (double.TryParse(value, out double result))
        {
            return result;
        }
    }
    return defaultValue;
}
```

## Configuration Files

### File Structure
- Main config: `config.txt` (in app directory)
- Service-specific configs: `gemini_config.txt`, `ollama_config.txt`, etc.
- Format: `key=value` pairs, one per line
- Multiline values use tags: `<tag>content</tag>`

### Loading Configuration
```csharp
private void LoadConfig()
{
    if (!File.Exists(_configFilePath))
    {
        CreateDefaultConfig();
        return;
    }
    
    string content = File.ReadAllText(_configFilePath);
    ProcessMultilineValues(content);
    ProcessSingleLineValues(content);
}
```

### Saving Configuration
```csharp
public void SaveConfig()
{
    try
    {
        var lines = new List<string>();
        foreach (var kvp in _configValues)
        {
            if (kvp.Value.Contains('\n'))
            {
                // Multiline value
                lines.Add($"<{kvp.Key}>");
                lines.Add(kvp.Value);
                lines.Add($"</{kvp.Key}>");
            }
            else
            {
                lines.Add($"{kvp.Key}={kvp.Value}");
            }
        }
        File.WriteAllLines(_configFilePath, lines);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error saving config: {ex.Message}");
    }
}
```

## Service-Specific Configuration

### Separate Config Files
Some services use separate config files for complex settings:

- `gemini_config.txt` - Gemini prompt templates
- `ollama_config.txt` - Ollama prompt templates
- `chatgpt_config.txt` - ChatGPT prompt templates

### Loading Service Configs
```csharp
public string GetGeminiPrompt()
{
    if (File.Exists(_geminiConfigFilePath))
    {
        return File.ReadAllText(_geminiConfigFilePath);
    }
    return GetDefaultGeminiPrompt();
}
```

## Default Values

### Providing Defaults
Always provide sensible defaults:

```csharp
public string GetOllamaModel()
{
    return GetValue(OLLAMA_MODEL, "llama3"); // Default model
}

public int GetCaptureFPS()
{
    return GetIntValue(CAPTURE_FPS, 10); // Default 10 FPS
}
```

## Settings Window Integration

### Binding Pattern
Settings window binds directly to ConfigManager:

```csharp
// In SettingsWindow.xaml.cs
private void LoadSettings()
{
    ApiKeyTextBox.Text = ConfigManager.Instance.GetGeminiApiKey();
    ModelComboBox.SelectedItem = ConfigManager.Instance.GetGeminiModel();
}

private void SaveSettings()
{
    ConfigManager.Instance.SetGeminiApiKey(ApiKeyTextBox.Text);
    ConfigManager.Instance.SetGeminiModel(ModelComboBox.SelectedItem?.ToString() ?? "");
}
```

### Two-Way Binding
For real-time updates, use two-way binding:

```xml
<TextBox Text="{Binding ApiKey, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"/>
```

## Window Position Persistence

### Saving Window State
```csharp
public void SetWindowPosition(string windowName, double left, double top, double width, double height)
{
    SetValue($"{windowName}_left", left.ToString());
    SetValue($"{windowName}_top", top.ToString());
    SetValue($"{windowName}_width", width.ToString());
    SetValue($"{windowName}_height", height.ToString());
    SaveConfig();
}
```

### Loading Window State
```csharp
public WindowPosition? GetWindowPosition(string windowName)
{
    var left = GetDoubleValue($"{windowName}_left", -1);
    var top = GetDoubleValue($"{windowName}_top", -1);
    
    if (left < 0 || top < 0)
    {
        return null; // No saved position
    }
    
    return new WindowPosition
    {
        X = left,
        Y = top,
        Width = GetDoubleValue($"{windowName}_width", 800),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/UGTLive](https://github.com/SethRobinson/UGTLive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
