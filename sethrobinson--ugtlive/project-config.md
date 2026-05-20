---
trigger: always_on
description: Translation workflow and OCR processing pipeline
---


# Translation Workflow and API Integration Guide

## Translation Pipeline Overview

The translation process in UGTLive follows this pipeline:
1. **Screen Capture** → 2. **OCR Processing** → 3. **Block Detection** → 4. **Translation** → 5. **Display**

## Screen Capture

### Monitor Window ([src/MonitorWindow.xaml.cs](mdc:src/MonitorWindow.xaml.cs))
- Captures screen region at configurable FPS
- Uses `System.Drawing` for bitmap operations
- Sends captured images to OCR services
- Provides visual feedback of capture area

### Capture Methods
```csharp
// Main capture triggered by timer
private void Timer_Tick(object sender, EventArgs e)
// Captures bitmap from screen region
private Bitmap CaptureScreen(int x, int y, int width, int height)
```

## OCR Processing

### OCR Service Selection
UGTLive supports six OCR backends configured in [src/ConfigManager.cs](mdc:src/ConfigManager.cs):

1. **Windows OCR** ([src/WindowsOCRManager.cs](mdc:src/WindowsOCRManager.cs))
   - Uses Windows.Media.Ocr API
   - Faster but less accurate for some languages
   - No external dependencies

2. **EasyOCR** (via PythonServicesManager)
   - Python server running locally
   - Better accuracy for Asian languages
   - Requires conda environment setup

3. **MangaOCR** (via PythonServicesManager)
   - Specialized for vertical Japanese manga text
   - YOLO-based text detection
   - Configurable region size and overlap settings

4. **PaddleOCR** (via PythonServicesManager)
   - Multi-language OCR with 100+ language support
   - Optional angle classification for rotated text
   - GPU acceleration support

5. **docTR** (via PythonServicesManager)
   - Great for non-Asian languages
   - Document-oriented OCR
   - High accuracy for printed text

6. **Google Vision** ([src/GoogleVisionOCRService.cs](mdc:src/GoogleVisionOCRService.cs))
   - Cloud-based OCR service
   - Requires API key and costs money
   - High accuracy but not local

### OCR Data Flow
```
Bitmap → OCR Service → List<TextObject> → UniversalBlockDetector
```

### Python OCR Services
All Python OCR services are managed by `PythonServicesManager`:
- Services discovered automatically on startup
- Each service runs on its own port
- Services can be installed/uninstalled via UI
- Health checks and diagnostics available

## Block Detection ([src/UniversalBlockDetector.cs](mdc:src/UniversalBlockDetector.cs))

The UniversalBlockDetector groups individual characters/words/lines into meaningful text blocks:

### Key Features
- **Universal Detection**: Handles mixed input types (Words, Lines, Characters)
- **Intelligent Grouping**: Groups based on proximity, alignment, and reading order
- **Configurable Thresholds**: Per-OCR method settings for glue distances
- **Height Similarity**: Prevents merging text with very different sizes
- **Large Gap Detection**: Splits text at large horizontal gaps

### Key Parameters
- **Block Detection Scale**: Controls grouping aggressiveness (0.0 - 1.0)
- **Horizontal/Vertical Glue**: Per-OCR method glue distances
- **Height Similarity Threshold**: Percentage for height matching
- **Settle Time**: Time to wait before processing blocks

### Grouping Algorithm
1. Sorts text objects by position
2. Groups based on proximity and alignment
3. Merges overlapping blocks
4. Filters by minimum size
5. Handles mixed input types intelligently

## Translation Services

### Service Interface ([src/ITranslationService.cs](mdc:src/ITranslationService.cs))
```csharp
public interface ITranslationService
{
    Task<TranslationResult?> TranslateAsync(
        string sourceText, 
        string targetLanguage, 
        string context);
}
```

### Available Services

#### Gemini ([src/GeminiTranslationService.cs](mdc:src/GeminiTranslationService.cs))
- Endpoint: `https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent`
- Models: gemini-1.5-flash, gemini-1.5-pro, gemini-2.0-flash
- Supports custom prompts and context

#### ChatGPT ([src/ChatGptTranslationService.cs](mdc:src/ChatGptTranslationService.cs))
- Endpoint: `https://api.openai.com/v1/chat/completions`
- Models: gpt-4o, gpt-4o-mini, o1-preview, o1-mini
- Supports streaming responses
- Configurable max completion tokens

#### Ollama ([src/OllamaTranslationService.cs](mdc:src/OllamaTranslationService.cs))
- Local endpoint: `http://localhost:11434/api/generate`
- Models: Various local models (llama, gemma, etc.)
- Privacy-focused, no cloud dependency
- Configurable URL and port

#### Google Translate ([src/GoogleTranslateService.cs](mdc:src/GoogleTranslateService.cs))
- Can use Cloud API or free web API
- Auto language mapping support
- Fast translation for simple use cases

#### llama.cpp ([src/LlamaCppTranslationService.cs](mdc:src/LlamaCppTranslationService.cs))
- Local llama.cpp server endpoint
- Configurable URL and port
- Privacy-focused local translation

### Translation Request Format
All services receive:
- Source text with position data
- Target language
- Previous context (configurable length)
- Game name for context
- Custom prompt template

## Display Systems

### ChatBox Window ([src/ChatBoxWindow.xaml.cs](mdc:src/ChatBoxWindow.xaml.cs))
- Overlay window for displaying translations
- Customizable appearance (color, transparency, font)
- Auto-scroll and history management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SethRobinson/UGTLive](https://github.com/SethRobinson/UGTLive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
