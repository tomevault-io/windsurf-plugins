---
trigger: always_on
description: This is a complete C# .NET Core Web API backend with Vue.js frontend for automated audiobook downloading and processing.
---

# Listenarr Project Instructions

This is a complete C# .NET Core Web API backend with Vue.js frontend for automated audiobook downloading and processing.

## Project Overview
- **Backend**: ASP.NET Core Web API (.NET 8.0+ / net8.0) with modular service architecture
- **Frontend**: Vue.js 3 + TypeScript + Pinia + Vue Router + Vite
- **Purpose**: Search multiple APIs for audiobook torrents/NZBs, manage downloads via clients (qBittorrent, Transmission, SABnzbd, NZBGet), and process files with metadata using Audnexus API
- **Database**: SQLite with Entity Framework Core (ListenArrDbContext)

## Project Structure
```
Listenarr/
├── listenarr.api/                 # Backend API (Note: lowercase directory name!)
│   ├── Controllers/               # API endpoints
│   ├── Models/                    # Data models (Audiobook, SearchResult, Download, etc.)
│   ├── Services/                  # Business logic (Search, Metadata, DownloadMonitor, adapters)
│   ├── tools/                    # Development utilities housed with API (discord-bot)
│   ├── wwwroot/cache/            # Image cache directory (gitignored)
│   └── Program.cs                # Application entry point
├── fe/                           # Frontend Vue application
│   ├── src/
│   │   ├── components/           # Vue components (AudiobookModal, FolderBrowser, etc.)
│   │   ├── views/                # Pages (Dashboard, Search, Downloads, Settings)
│   │   ├── stores/               # Pinia stores for state management
│   │   ├── services/             # API client services
│   │   ├── types/                # TypeScript type definitions
│   │   └── router/               # Vue Router configuration
│   └── public/                   # Static assets (icon.png, logo.png)
│   └── package.json
├── .github/                      # GitHub configuration and assets
│   ├── copilot-instructions.md  # This file
│   ├── BRANDING.md              # Logo and branding guidelines
│   ├── logo-icon.png            # Brand icon (square format)
│   └── logo-full.png            # Full logo with text (horizontal)
├── start-dev.bat                 # Windows startup script
├── start-dev.ps1                 # PowerShell startup script
├── start-dev.sh                  # Linux/macOS startup script
├── package.json                  # Root package with concurrently scripts
├── docker-compose.yml            # Docker orchestration
├── listenarr.application/        # Application layer (services, interfaces)
├── listenarr.domain/             # Domain models and enums
├── listenarr.infrastructure/     # Persistence, adapters, EF Core configs
├── tests/                        # Unit and integration tests
└── listenarr.sln                 # Visual Studio solution file
```

## Branding
The Listenarr logo combines headphones and a book to represent audiobook listening:
- **Primary Color**: `#2196F3` (Blue)
- **Icon**: `icon.png` - Square format for favicons and app icons
- **Full Logo**: `logo.png` - Horizontal format with text for headers
- **Format**: PNG with transparency for universal compatibility
- See `.github/BRANDING.md` for complete guidelines

## Key Features Implemented
- 🔍 **Multi-API Search**: Search across multiple torrent/NZB APIs simultaneously
- 📥 **Download Management**: Support for qBittorrent, Transmission, SABnzbd, NZBGet
- 🎵 **Metadata Integration**: Audible metadata via AudibleMetadataService and Audnexus API
- 🖼️ **Image Caching**: Automatic image caching with cleanup service
- 📁 **File Browser**: FolderBrowser component for path selection
- 📚 **Library Management**: AudiobookRepository with SQLite persistence
- ⚙️ **Configuration Management**: APIs, download clients, and settings via JSON
- 🖥️ **Modern Dashboard**: Statistics and quick actions
- 📱 **Responsive Design**: Mobile and desktop support

## Architecture Details

### Backend Services
- **SearchService**: Multi-API search coordination
- **AudibleMetadataService**: Fetch metadata from Audible/Audnexus
- **AmazonAsinService**: ASIN extraction from Amazon URLs
- **ImageCacheService**: Download and cache book cover images
- **ConfigurationService**: JSON-based settings management
- **AudiobookRepository**: Database operations (EF Core)

### Frontend State Management
- **Pinia Stores**: search, downloads, configuration, library
- **API Communication**: Type-safe HTTP client with Axios-style error handling
- **Reactive Updates**: Automatic refresh for active downloads

### Database
- **SQLite** via Entity Framework Core
- **Models**: Audiobook, SearchResult, Download, Configuration
- **Context**: ListenArrDbContext with automatic migrations

## How to Run This Project

### Prerequisites
- **.NET 8.0 SDK or later (net8.0)** - [Download](https://dotnet.microsoft.com/download)  

**Note:** Build/test environments in this repo target net8.0. Running with a different SDK may create build/run inconsistencies.
- **Node.js 20.x or later** - [Download](https://nodejs.org/)
- **npm** (comes with Node.js)

### Recommended: Single Command Start

Use the provided startup scripts that handle everything automatically:

**Windows (Command Prompt):**
```bash
start-dev.bat
```

**Windows (PowerShell):**
```bash
.\start-dev.ps1
```

**Linux/macOS:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Listenarrs/Listenarr](https://github.com/Listenarrs/Listenarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
