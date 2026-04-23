---
trigger: always_on
description: WeatherLockscreen is a KOReader plugin that displays weather information on e-reader sleep screens. It's written in Lua and integrates with KOReader's widget system to create custom lockscreen displays. The plugin supports multiple display modes, periodic refresh features, and multi-language localization.
---

# GitHub Copilot Instructions for WeatherLockscreen

## Project Overview
WeatherLockscreen is a KOReader plugin that displays weather information on e-reader sleep screens. It's written in Lua and integrates with KOReader's widget system to create custom lockscreen displays. The plugin supports multiple display modes, periodic refresh features, and multi-language localization.

## Technology Stack
- **Language**: Lua
- **Platform**: KOReader (e-reader framework)
- **API**: WeatherAPI.com (forecast endpoint)
- **UI Framework**: KOReader widget system (ImageWidget, TextWidget, containers, etc.)
- **Localization**: gettext (.po/.mo files)

## Architecture

### Core Modules
1. **main.lua**: Plugin entry point, screensaver patching, event handlers, settings initialization
2. **weather_menu.lua**: Menu system and UI dialogs for settings
3. **weather_api.lua**: API fetching, JSON parsing, data processing
4. **weather_utils.lua**: Utility functions for caching, icons, time formatting, localization, device detection
5. **weather_dashboard.lua**: Full-screen dashboard mode with periodic refresh
6. **display_*.lua**: Display mode implementations (default, card, nightowl, retro, reading)
7. **display_helper.lua**: Shared display utilities
8. **_meta.lua**: Plugin metadata (name, version, description)

### Key Design Patterns
- **Display Strategy Pattern**: Each display mode is a separate module with a `create(weather_lockscreen, weather_data)` function
- **Widget Composition**: UI built using nested widget containers (VerticalGroup, HorizontalGroup, OverlapGroup, etc.)
- **Dynamic Scaling**: Content scales to fit different screen sizes using DPI-independent base sizes and calculated scale factors
- **Two-Tier Caching**: Minimum delay between API calls + configurable max cache age for offline use
- **Safe Network Wrapper**: All network calls wrapped in pcall to prevent crashes

## Code Style & Conventions

### Lua Best Practices
- Use `local` for all variables and functions unless global scope is required
- Follow KOReader naming conventions: snake_case for variables/functions
- Comment blocks use `--[[  ]]` format with module description
- Inline comments use `--` prefix
- Always check for nil before accessing nested tables: `if data and data.current and data.current.field then`

### Widget Creation Pattern
```lua
local widget = WidgetType:new {
    property = value,
    nested_widget = OtherWidget:new { ... }
}
```

### Scaling Pattern for Display Modes
```lua
-- Define base sizes (DPI-independent)
local base_font_size = 24
local base_icon_size = 100

-- Function to build content with scale factor
local function buildContent(scale_factor)
    local font_size = math.floor(base_font_size * scale_factor)
    local icon_size = math.floor(base_icon_size * scale_factor)
    -- ... build widgets with scaled sizes
    return VerticalGroup:new { ... }
end

-- Measure content and calculate optimal scale
local content = buildContent(1.0)
local content_height = content:getSize().h
-- ... calculate scale based on available_height
-- Rebuild if needed: content = buildContent(new_scale)
```

## Important Implementation Details

### Settings System
All settings are prefixed with `weather_` and initialized in `main.lua:initDefaultSettings()`:

```lua
-- Core settings
weather_location           -- Location string (e.g., "London" or coordinates)
weather_temp_scale         -- "C" or "F"
weather_display_mode       -- "default", "card", "nightowl", "retro", "reading"

-- Display settings
weather_show_header        -- boolean, show status bar
weather_override_scaling   -- boolean, enable manual scaling
weather_fill_percent       -- 50-100, content fill percentage
weather_cover_scaling      -- "fit" or "zoom" (for reading mode)

-- Cache settings
weather_cache_max_age      -- hours (1-24), max cache age for offline
weather_min_update_delay   -- minutes (1-60), min time between API calls

-- Periodic refresh settings
weather_rtc_mode           -- nil/false (off) or interval in minutes
weather_dashboard_mode     -- nil/false (off) or interval in minutes
weather_custom_rtc_interval    -- custom interval minutes
weather_custom_dashboard_interval

-- Debug settings
weather_debug_options      -- boolean, show advanced options in menu
```

### Weather Data Structure
The processed weather data has this structure:
```lua
{
    lang = "en",  -- Language code
    is_cached = false,  -- Whether data is from cache
    current = {
        icon_path = "/path/to/icon.png",
        temperature = "20°C",
        condition = "Partly cloudy",
        location = "London",
        timestamp = "2024-11-18 14:30",
        feels_like = "18°C",
        humidity = "65%",
        wind = "15 km/h",
        wind_dir = "NW"
    },
    hourly_today_all = { {hour="6:00", hour_num=6, icon_path="...", temperature="15°", condition="..."}, ... },
    hourly_tomorrow_all = { ... },
    forecast_days = { {day_name="Today", icon_path="...", high_low="20° / 10°", condition="..."}, ... },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loeffner/WeatherLockscreen](https://github.com/loeffner/WeatherLockscreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
