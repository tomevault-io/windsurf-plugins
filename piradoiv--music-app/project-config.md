---
trigger: always_on
description: Music App is a cross-platform desktop application built with Xojo. It consists of a desktop music player and, in the future, a web-based music library management system.
---

# Music App - Xojo-based Cross-Platform Music Player

Music App is a cross-platform desktop application built with Xojo. It consists of a desktop music player and, in the future, a web-based music library management system.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the information here.**

## Working Effectively

### Critical Prerequisites
- **MANDATORY**: Xojo IDE 2025r1.1 or later must be installed to build this application
- This repository CANNOT be built without the Xojo IDE - there are no alternative build tools

### Repository Structure
The repository contains two main Xojo projects:
- **Desktop Application**: `/src/Music.xojo_project` - Cross-platform GUI music player
- **Web Application**: `/server/MusicLibrary.xojo_project` - Web-based music library management, not used at the moment

### Installation and Setup
1. **Install Xojo IDE** (REQUIRED - cannot be automated):
   - Download Xojo 2025r1.1+ from https://www.xojo.com/download/
   - **NEVER CANCEL**: Download takes 15-30 minutes depending on connection
   - **NEVER CANCEL**: Installation takes 10-15 minutes
   - Requires license for building for macOS and Windows

2. **Repository Setup**:
   ```bash
   git clone https://github.com/piradoiv/music-app.git
   cd music-app
   ```

### Testing and Validation

#### Manual Validation Requirements
After building, ALWAYS test these core scenarios:

**Desktop Application:**
1. Launch application and verify UI loads correctly
2. Test music file import functionality (supports common formats via ID3 tag reading)
3. Verify playback controls (play, pause, skip forward/backward)
4. Test volume controls (up, down, mute)
5. Verify library browsing and organization by artist/album/genre
6. Test album artwork display (from ID3 tags or folder images)
7. Verify playlist management and file path handling

### Development Guidelines

#### File Organization
- **GUI Components**: `/src/GUI/` - Desktop interface elements
- **Model Classes**: `/src/Model/` - Data structures and business logic  

#### Key Files to Monitor
- `Music.xojo_project` - Main desktop project file
- `Build Automation.xojo_code` - Build configuration for each project
- `/src/Model/MusicApp.xojo_code` - Core desktop application logic and ID3 tag processing
- `/src/Model/MusicLibrary.xojo_code` - Shared music library functionality
- Always check these files after making changes to model classes

#### Code Structure
- **MusicLibrary Module**: Shared between desktop and web applications
- **GUI Classes**: Desktop-specific interface elements (at `/src/GUI/`)
- **MusicApp Class**: Core desktop application logic including ID3 tag reading and file management
- **Database**: Uses SQLiteDatabase for web application data persistence

### Limitations and Constraints

#### Build Environment Limitations
- **CANNOT build in CI/CD environments** - Xojo IDE required
- **CANNOT install Xojo via package managers** (apt, brew, etc.)

#### Development Constraints
- Debugging requires Xojo IDE
- No unit testing frameworks are used, XojoUnit will be used in the future
- Limited to Xojo's built-in development tools
- Version control best practices: commit `.xojo_project` files, ignore `.xojo_uistate`

#### When Xojo is Not Available
If working in an environment without Xojo:
- Focus on documentation and planning
- Review project structure and file organization
- Analyze code through text editors (`.xojo_code` files are readable)
- Plan features and architectural changes
- **Document clearly**: "Cannot build or test - requires Xojo IDE"

### Common Tasks

#### Troubleshooting Common Issues

**Build Failures:**
- Ensure Xojo IDE version is 2025r1.1 or later
- Check that all .xojo_code files are properly formatted
- Verify build automation settings in Build Automation.xojo_code

**Desktop App Issues:**
- Music file import problems: Check ID3 tag reading in MusicApp.xojo_code
- UI rendering issues: Verify color groups and asset paths in /src/GUI/
- Playback problems: Check audio codec support on target platform
- Missing album art: Verify image extraction from ID3 tags or folder scanning

**Cross-Platform Issues:**
- File path handling: Use FolderItem.NativePath vs ShellPath appropriately
- UI differences: Test on each target platform extensively
- Performance variations: Profile on actual target hardware

## Project Metadata

- **Language**: Xojo (BASIC-like syntax)
- **UI Framework**: Xojo's native controls
- **Database**: Built-in Xojo database classes
- **Platforms**: macOS, Windows, Linux
- **Architecture**: Desktop native application
- **Build Tool**: Xojo IDE (commercial, required)
- **Primary Development Platform**: macOS

## CRITICAL REMINDERS

- **NEVER attempt to build without Xojo IDE** - it will fail
- **NEVER CANCEL long-running Xojo operations** - downloads, installs, and builds take significant time
- **ALWAYS set appropriate timeouts** (30+ minutes for builds, 60+ minutes for installation)
- **ALWAYS manually test** after making changes - automated testing is extremely limited

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piradoiv/music-app](https://github.com/piradoiv/music-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
