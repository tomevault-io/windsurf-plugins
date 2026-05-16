---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ThreadUI is an Android application built with Kotlin and Jetpack Compose that serves as a configuration manager for AppList configurations. The app allows users to view and edit game optimization settings with dynamic path configuration, comprehensive error handling, and enhanced user interface features.

**Package**: `com.reddoctor.threadui`
**Min SDK**: 35 (Android 14)
**Target SDK**: 36
**Compile SDK**: 36
**Build Tools**: Gradle 8.11.0
**Kotlin Version**: 2.0.21

## Core Functionality

The application provides the following features:
- **Root Permission Management**: Detects and manages root access with refresh capabilities and contextual UI
- **Dynamic Configuration Path Management**: Customizable module paths with settings dialog and validation
- **Configuration File Parsing**: Reads and parses applist.conf files containing game thread configurations
- **Visual Configuration Editor**: Card-based UI with floating scrollbar and enhanced navigation
- **Intelligent Search**: Real-time search functionality with highlighting and quick search tags
- **Real-time Editing**: Add, modify, and delete thread configurations for games
- **System File Operations**: Read from and write to protected system directories
- **Configuration Sharing**: Export and import game configurations via JSON files with secure validation
- **Batch Operations**: Support for bulk export/import of multiple game configurations
- **Error Logging & Sharing**: Local error logging with shareable log files via hidden gesture
- **Enhanced Module Management**: Download links, custom path validation, and status checking
- **Global Exception Handling**: Centralized error management across the entire application
- **Advanced UI Components**: Floating scrollbar with alphabet indexing and directional arrows
- **About Dialog with QQ Support**: Comprehensive app information with QQ group integration
- **Hidden Debug Features**: Log sharing via consecutive clicks (5x) on version information

## Search Functionality

The app includes comprehensive search capabilities:
- **Real-time Search**: Filter games by name or package name with instant results
- **Search Highlighting**: Visual highlighting of search terms in results
- **Quick Search Tags**: Pre-defined tags for common developers and game types
- **Search State Management**: Toggle between normal and search modes
- **Empty State Handling**: User-friendly messages when no results are found

## User Interface Features

### Advanced Scrolling System
- **Floating Scrollbar**: Non-intrusive scrollbar that floats over content without reserving space
- **Alphabet Indexing**: Quick navigation using alphabetical letter indicators
- **Directional Arrows**: Visual up/down arrow indicators on the scrollbar
- **Auto-hide**: Scrollbar automatically appears during scrolling and hides after inactivity
- **Letter Indicator**: Large centered letter display during alphabet navigation
- **Transparent Track**: Minimalist design with near-transparent track (alpha = 0.1f)

### Enhanced About Dialog
- **Project Information**: Complete application details and build information including tech stack
- **QQ Group Integration**: Direct access to support QQ group (975905874) with join link and proper centering
- **GitHub Repository**: Direct link to project repository (https://github.com/reddoctor/ThreadUI)
- **Hidden Log Access**: Consecutive clicks (5x) on version info card triggers log sharing with progress indicator
- **Device Information**: Comprehensive device and app information for debugging
- **Core Features**: Visual list of all major application capabilities

### Module Status Management
- **Dynamic Path Configuration**: Settings dialog for custom module paths with real-time validation
- **Module Detection**: Intelligent detection of module installation status based on dynamic paths
- **Multiple Action Buttons**: Custom path settings (with distinct color), module download, and status checking
- **Contextual Menus**: Simplified menu options (only "关于") for Root/Module detection pages
- **Refresh Functionality**: Manual refresh buttons for permissions and module status
- **Download Integration**: Direct links to module download site (http://appopt.suto.top/#download)

## Architecture

### Project Structure
- **Main Application**: `app/src/main/java/com/reddoctor/threadui/MainActivity.kt`
- **Application Class**: `app/src/main/java/com/reddoctor/threadui/ThreadUIApplication.kt`
- **Data Models**: `app/src/main/java/com/reddoctor/threadui/data/GameConfig.kt`
- **Share Configuration**: `app/src/main/java/com/reddoctor/threadui/data/ShareConfig.kt`
- **Root Utilities**: `app/src/main/java/com/reddoctor/threadui/utils/RootUtils.kt`
- **Share Utilities**: `app/src/main/java/com/reddoctor/threadui/utils/ShareUtils.kt`
- **Import Utilities**: `app/src/main/java/com/reddoctor/threadui/utils/ImportUtils.kt`
- **Permission Utilities**: `app/src/main/java/com/reddoctor/threadui/utils/PermissionUtils.kt`
- **Configuration Management**: `app/src/main/java/com/reddoctor/threadui/utils/ConfigManager.kt`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reddoctor/ThreadUI](https://github.com/reddoctor/ThreadUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
