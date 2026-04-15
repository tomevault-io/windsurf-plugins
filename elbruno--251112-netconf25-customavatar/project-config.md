---
trigger: always_on
description: Browser-based talking avatar demo with two implementations: **JavaScript/HTML** (production-ready) and **.NET 9 Blazor** (modern alternative). Both integrate Azure Speech Services (STT/TTS + Avatar) and Azure OpenAI for interactive AI conversations.
---

# Azure AI Avatar Demo - AI Agent Instructions

## Project Overview

Browser-based talking avatar demo with two implementations: **JavaScript/HTML** (production-ready) and **.NET 9 Blazor** (modern alternative). Both integrate Azure Speech Services (STT/TTS + Avatar) and Azure OpenAI for interactive AI conversations.

## Repository Structure

```
├── python/                    # JavaScript implementation (primary)
│   ├── dev-server/           # Express.js local dev server (serves .env as /.env.json)
│   ├── js/                   # Core app logic (chat.js, config.js, theme-manager.js)
│   ├── css/                  # Styles (theme-system.css, theme-*.css)
│   ├── prompts/              # Prompt profiles (*.md templates + index.json)
│   └── .env                  # Azure credentials (gitignored)
├── *.html                    # Root HTML pages (chat.html, config.html, basic.html)
├── dotnet/                           # .NET 9 Blazor implementation
│   ├── AzureAIAvatarBlazor.AppHost/  # Aspire AppHost orchestration
│   ├── AzureAIAvatarBlazor.ServiceDefaults/  # Shared telemetry & resilience
│   └── AzureAIAvatarBlazor/          # Main Blazor app
│       ├── Components/Pages/         # Blazor pages (Chat.razor, Config.razor)
│       ├── Services/                 # Azure service wrappers (OpenAI, Speech, Configuration)
│       └── wwwroot/js/              # JavaScript interop (avatar-interop.js)
└── .vscode/tasks.json        # VS Code tasks for dev server
```

## Architecture

### JavaScript/HTML Implementation (python folder)

- **Static serving pattern**: Express serves `python/` folder FIRST (for js/css/images), then root (for HTML files)
- **Configuration flow**: `.env` file → `dev-server/server.js` → `/.env.json` endpoint → `js/config.js` → localStorage
- **Avatar connection**: `chat.js` uses Azure Speech SDK (loaded via CDN) → WebRTC peer connection → video rendering
- **Prompt system**: Markdown templates in `prompts/*.md` with `{{variable}}` placeholders, managed via `index.json`

### .NET Blazor Implementation

- **Orchestration**: .NET Aspire 9.5 (`AzureAIAvatarBlazor.AppHost`) manages the Blazor app lifecycle
- **Services**: Interface-based DI pattern (`IConfigurationService`, `IAzureSpeechService`, `IAzureOpenAIService`)
- **Configuration**: User Secrets (dev) → Environment Variables → appsettings.json (fallback)
- **Interop**: C# Blazor components ↔ `avatar-interop.js` ↔ Azure Speech SDK (browser)
- **Streaming**: OpenAI responses streamed via `IAsyncEnumerable<ChatStreamingUpdate>`
- **Telemetry**: OpenTelemetry via `ServiceDefaults` (metrics, traces, logs) → Aspire Dashboard
- **Resilience**: HTTP resilience policies via `Microsoft.Extensions.Http.Resilience`

## Critical Workflows

### Running the JavaScript App

```powershell
# Use VS Code Task (recommended - bypasses PowerShell ExecutionPolicy)
Tasks: Run Task → "Dev Server (HTTP)" or "Dev Server (HTTPS)"

# Or manually
cd python/dev-server
npm install
node server.js  # Serves on http://localhost:5173
```

**Key**: Server serves `python/` static assets BEFORE root HTML files. See `python/dev-server/server.js` lines 37-48.

### Running the .NET App

```bash
# Run via Aspire AppHost (recommended - includes dashboard)
cd dotnet/AzureAIAvatarBlazor.AppHost
dotnet run  # Launches app + Aspire Dashboard

# Configure secrets (applies to both AppHost and Blazor app)
cd dotnet/AzureAIAvatarBlazor
dotnet user-secrets set "AzureSpeech:ApiKey" "your-key"
dotnet user-secrets set "AzureOpenAI:Endpoint" "https://..."

# Or run Blazor app directly (without Aspire)
cd dotnet/AzureAIAvatarBlazor
dotnet run  # Serves on https://localhost:5001
```

**Key**:

- **Aspire AppHost** orchestrates the app and provides dashboard at `https://localhost:15216` (or check console output)
- Never edit `appsettings.Development.json` directly (gitignored). Use User Secrets or environment variables
- AppHost uses `AppHost.cs` to define project references: `builder.AddProject<Projects.AzureAIAvatarBlazor>("azureaiavatarblazor")`

### Configuration Priority

1. **JavaScript**: ENV vars from `.env` → `/.env.json` → `js/config.js` → localStorage (browser)
2. **.NET**: User Secrets → ENV vars (AZURE*SPEECH*\*, SYSTEM_PROMPT) → appsettings.json → auto-detection

## Project-Specific Patterns

### Prompt Profile System

- Templates in `python/prompts/*.md` use Mustache-style `{{variables}}`
- `index.json` defines profiles with `id`, `name`, `file`, `defaults` (JSON object)
- Variable precedence: profile defaults < `PROMPT_VAR_*` env vars < UI JSON input
- Force profile on load: Set `PROMPT_ENFORCE_PROFILE=true` in `.env`

### Custom Avatar Handling

**Critical**: Custom avatars + built-in voice requires clearing `customVoiceEndpointId` to avoid silence/403 errors.

- JavaScript: `js/config.js` line ~200 (useBuiltInVoice logic)
- .NET: `Services/ConfigurationService.cs` auto-detects via `DetermineIfCustomAvatar()`

### Audio Gain Fix

Low avatar volume is a common issue. Solution:

- Stored in localStorage as `azureAIFoundryAudioGain` (default 1.8x)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elbruno) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
