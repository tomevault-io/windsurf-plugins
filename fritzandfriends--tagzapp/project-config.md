---
trigger: always_on
description: TagzApp is a .NET 9.0 Blazor Server application that aggregates hashtag content from multiple social media platforms including Twitter/X, Mastodon, Bluesky, YouTube, TwitchChat, and more. The application uses .NET Aspire for local orchestration and supports both PostgreSQL and SQLite databases.
---

# TagzApp - Social Media Hashtag Aggregator

TagzApp is a .NET 9.0 Blazor Server application that aggregates hashtag content from multiple social media platforms including Twitter/X, Mastodon, Bluesky, YouTube, TwitchChat, and more. The application uses .NET Aspire for local orchestration and supports both PostgreSQL and SQLite databases.

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Installation
- **CRITICAL**: Install .NET 9.0.200 SDK exactly as specified in `global.json`
  ```bash
  wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh
  chmod +x dotnet-install.sh
  ./dotnet-install.sh --version 9.0.200
  export PATH="/home/runner/.dotnet:$PATH"
  ```
- Git repository must be unshallowed for version calculations: `git fetch --unshallow`
- Docker (optional, for PostgreSQL container via scripts)

### Build and Test Process
**NEVER CANCEL any build or test commands. All timeouts listed include safety margins.**

1. **Restore Dependencies** (50-60 seconds, timeout: 90+ seconds)
   ```bash
   cd src
   dotnet restore
   ```
   - **NEVER CANCEL**: Takes ~51 seconds. Set timeout to 90+ seconds minimum.

2. **Build Main Application** (10-15 seconds after restore, timeout: 30+ seconds)
   ```bash
   cd src
   dotnet build TagzApp.Blazor --no-restore
   ```
   - **NEVER CANCEL**: Takes ~11 seconds after restore. Set timeout to 30+ seconds.
   - Expect 79+ warnings (nullable reference warnings, unused fields) - these are known and do not affect functionality.

3. **Release Build** (10-15 seconds, timeout: 30+ seconds)
   ```bash
   cd src
   dotnet build TagzApp.Blazor --configuration Release --no-restore
   ```

4. **Run Unit Tests** (3-5 seconds, timeout: 15+ seconds)
   ```bash
   cd src/TagzApp.UnitTest
   dotnet test --verbosity normal
   ```
   - **NEVER CANCEL**: Takes ~3.6 seconds. Set timeout to 15+ seconds.
   - **EXPECTED**: 2 test failures in Mastodon provider tests due to external API dependencies - this is normal.
   - **SUCCESS CRITERIA**: 22 total tests, ~18 pass, ~2 fail, ~2 skip.

### Code Quality and Formatting
**ALWAYS run formatting before committing or CI will fail.**

1. **Check Code Formatting** (30-40 seconds, timeout: 60+ seconds)
   ```bash
   cd src
   dotnet format --verbosity normal --exclude **/Migrations/ --verify-no-changes
   ```
   - **NEVER CANCEL**: Takes ~35 seconds. Set timeout to 60+ seconds.
   - Will show formatting errors that must be fixed.

2. **Fix Code Formatting** (30-40 seconds, timeout: 60+ seconds)
   ```bash
   cd src
   dotnet format --verbosity normal --exclude **/Migrations/
   ```
   - **ALWAYS RUN** before committing changes.

### Running the Application

#### Option 1: Using .NET Aspire (Recommended for full development)
```bash
cd src
dotnet run --project TagzApp.AppHost
```
- **NOTE**: Requires Docker for PostgreSQL and Redis containers.
- **LIMITATION**: May fail in containerized environments due to Kubernetes orchestration requirements.
- **ALTERNATIVE**: Use direct Blazor run if Aspire fails.

#### Option 2: Direct Blazor Application (Fallback)
```bash
cd src/TagzApp.Blazor
dotnet run
```
- **USE WHEN**: Aspire orchestration fails or Docker unavailable.
- **LIMITATION**: Requires manual database setup and configuration.

#### Option 3: Using Docker Compose
```bash
docker-compose up
```
- Uses pre-built container image from GitHub Container Registry.

### Database Management Scripts
Located in `scripts/` directory:
- `launchdb` / `launchdb.cmd`: Start PostgreSQL container
- `AddMigration.cmd`: Add new Entity Framework migration
- `UpdateDatabase.cmd`: Apply database migrations
- `RemoveMigration.cmd`: Remove last migration

## Validation and Testing

### Manual Validation Requirements
**ALWAYS perform manual testing after making changes to UI components.**

1. **Basic Application Flow**:
   - Navigate to homepage
   - Attempt to search for a hashtag (may fail without proper configuration - expected)
   - Check that the UI loads without errors
   - Verify moderation panel accessibility (requires authentication)

2. **After UI Changes**:
   - Test the waterfall display (main content view)
   - Test the moderation interface if applicable
   - Verify overlay display functionality
   - Check responsive design on different screen sizes

3. **Authentication Flow** (if providers configured):
   - Test login/logout functionality
   - Verify admin panel access with proper roles

### Pre-Commit Validation Checklist
**ALWAYS complete this checklist before committing:**

1. **Code Quality**:
   ```bash
   cd src
   dotnet format --verbosity normal --exclude **/Migrations/
   dotnet build TagzApp.Blazor --no-restore
   cd TagzApp.UnitTest
   dotnet test --verbosity normal
   ```

2. **Expected Results**:
   - Build succeeds with warnings (acceptable)
   - Unit tests: ~18 pass, ~2 fail (Mastodon), ~2 skip (YouTube)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FritzAndFriends/TagzApp](https://github.com/FritzAndFriends/TagzApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
