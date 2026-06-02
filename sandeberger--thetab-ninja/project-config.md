---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TheTab.Ninja is a Chrome browser extension that transforms the new tab page into a customizable bookmark manager. The extension allows users to:

- Manage bookmarks in collections with drag-and-drop functionality
- Sync data via GitHub repositories (no external servers)
- Import bookmarks from other tools like Toby
- Manage Chrome tabs and tab groups directly from the new tab page
- Organize collections into workspaces using Spaces management
- Use Zen Mode for distraction-free browsing with elegant clock display
- Sort collections by name, last modified, or custom order
- Customize background with predefined wallpapers or custom images
- Access quick search shortcuts for Google and ChatGPT
- Enjoy responsive mobile design with touch-optimized interface
- Robust favicon error handling with automatic cleanup and fallback systems
- Modern settings UI with card-based design and improved user experience
- Automatic backup system with customizable folder locations and retention policies

## Architecture

### Core Files Structure

- `manifest.json` - Chrome extension manifest (v3, version 1.2)
- `bm.html` / `bm.js` - Main new tab page interface and logic
- `popup.html` / `popup.js` - Extension popup for saving current tab
- `background.js` - Service worker handling GitHub sync, tab management, favicon fetching
- `styles.css` - External CSS file with modern UI styling, dark mode support, and responsive design
- `tabninja_help.html` - Comprehensive user guide and help documentation

### Key Components

#### Data Management
- All data stored in `localStorage` as `bookmarkManagerData` object
- Collections contain bookmarks with soft-delete functionality (deleted flag)
- Spaces management with object format and soft-delete capability
- GitHub sync uses Git API for remote storage without external dependencies
- Enhanced security with robust merge conflict resolution

#### Favicon Error Handling System
- Comprehensive URL validation and cleanup for problematic favicon URLs
- Automatic detection and replacement of faulty Google favicon service URLs
- Global error handling for image loading failures with graceful fallbacks
- Console error suppression for better user experience
- Fallback SVG icons for failed favicon loads
- Background service worker with multiple fallback sources and timeout handling

#### Drag & Drop System
- Comprehensive drag-and-drop between collections, bookmarks, and Chrome tabs
- Supports moving bookmarks between collections
- Can import Chrome tabs/tab groups directly into collections
- Drag collections between different spaces
- Collection reordering with position management

#### Chrome Integration
- Background service worker communicates with content scripts
- Tab management through Chrome APIs (tabs, windows, tabGroups)
- Favicon fetching via Google's favicon service

#### Automatic Backup System
- Scheduled daily/weekly backup creation with Chrome alarms API
- Customizable backup retention policies (3-30 days)
- Downloads API-based file creation with data URLs (service worker compatible)
- Subfolder support in Downloads directory for organization
- Manual and automatic backups use unified backup functions
- Automatic cleanup of old backups based on retention settings
- Cross-session backup scheduling with persistent alarms

#### Zen Mode
- Distraction-free browsing experience with minimalist interface
- Elegant clock display with date/time
- Dedicated search functionality with smooth animations
- Keyboard navigation with Escape key handling
- Automatic disable on mobile devices for optimal UX

#### Mobile Responsive Design
- Touch-optimized interface for mobile devices
- Responsive layout that adapts to different screen sizes
- Mobile-specific button designs (icon-only for space efficiency)
- Pane toggle system for easy navigation
- Optimized viewport handling for mobile browsers

#### Custom Background System
- Predefined wallpaper gallery with multiple options
- Custom image upload functionality with data URL storage
- Background sync across devices via GitHub
- Thumbnail management with selection states
- Remove custom backgrounds with confirmation dialogs

#### Modern Settings UI Architecture
- Card-based design with clean visual hierarchy and sections
- Settings organized into logical groups: Basic Settings, Sync & Backup, Appearance, Data Management, Help & Support
- Danger zone for destructive actions with clear visual warnings
- Enhanced form controls with proper labels and descriptions
- Responsive design that works on all screen sizes
- Hover effects and smooth transitions for better user experience
- External CSS architecture for better maintainability and performance

## Data Structure

### Main Data Object
```javascript
bookmarkManagerData = {
    collections: [],           // Array of collection objects
    openInNewTab: false,      // User preference for bookmark opening
    closeWhenSaveTab: false,  // Auto-close tabs when saving
    darkMode: false,          // UI theme preference
    leftPaneOpen: true,       // UI state
    rightPaneOpen: true,      // UI state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandeberger/TheTab.Ninja](https://github.com/sandeberger/TheTab.Ninja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
