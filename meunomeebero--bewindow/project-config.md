---
trigger: always_on
description: - Always prioritize native macOS appearance and behavior
---

# 🍎 macOS Native Interface Specifications for Tauri Apps

## Core Principles
- Always prioritize native macOS appearance and behavior
- Use glassmorphism effects with backdrop-filter for authentic macOS look
- Implement traffic lights with native colors and behaviors
- Follow Apple's Human Interface Guidelines for typography, spacing, and interactions
- Ensure accessibility and responsiveness across all screen sizes

## 🎨 Visual Design Standards

### Glassmorphism Effect
```css
/* Native macOS glassmorphism */
background: rgba(255, 255, 255, 0.12);
backdrop-filter: blur(20px);
-webkit-backdrop-filter: blur(20px);
border: 1px solid rgba(255, 255, 255, 0.25);
border-radius: 1rem; /* 16px - macOS Big Sur+ standard */

/* Native shadows */
box-shadow:
    0 0.5rem 2rem rgba(0, 0, 0, 0.15),      /* Main shadow */
    0 0.25rem 0.5rem rgba(0, 0, 0, 0.1),    /* Close shadow */
    inset 0 1px 0 rgba(255, 255, 255, 0.1); /* Internal highlight */
```

### Traffic Lights Specifications
```css
/* Native macOS traffic lights */
.traffic-light {
    width: 0.75rem;  /* 12px */
    height: 0.75rem; /* 12px */
    border-radius: 50%;
    opacity: 0.7; /* Default state */
    transition: all 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

/* Native colors */
.traffic-light.close { background: #ff5f57; }
.traffic-light.minimize { background: #ffbd2e; }
.traffic-light.maximize { background: #28c840; }

/* Icons only visible on hover (native behavior) */
.traffic-light .icon {
    opacity: 0;
    transition: opacity 0.2s ease;
}
.traffic-light:hover .icon { opacity: 1; }
```

### Typography Standards
```css
/* Native macOS font stack */
font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'Segoe UI', 'Roboto', 'Helvetica Neue', Arial, sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;

/* Title bar text */
font-size: 0.8125rem; /* 13px */
font-weight: 500;

/* Main content heading */
font-size: 1.5rem; /* 24px */
font-weight: 300; /* Light weight */

/* Secondary text */
font-size: 0.875rem; /* 14px */
font-weight: 400;
```

### Spacing and Layout
```css
/* Titlebar height */
height: 3.25rem; /* 52px - macOS standard */

/* Window dimensions */
width: 37.5rem;  /* 600px */
height: 25rem;   /* 400px */

/* Traffic lights spacing */
gap: 0.5rem; /* 8px between lights */

/* Content padding */
padding: 2rem; /* 32px */

/* Responsive breakpoint */
@media (max-width: 48rem) { /* 768px */ }
```

## 🔧 Tauri Configuration

### tauri.conf.json
```json
{
  "app": {
    "withGlobalTauri": true,
    "windows": [
      {
        "decorations": false,
        "transparent": true,
        "titleBarStyle": "Overlay",
        "hiddenTitle": true,
        "resizable": true,
        "center": true
      }
    ],
    "macOSPrivateApi": true
  }
}
```

### Rust Dependencies
```toml
[dependencies]
tauri = { version = "2", features = ["macos-private-api"] }
tauri-plugin-decorum = "1.1.0"
window-vibrancy = "0.6.0"
```

### Rust Setup
```rust
use tauri::Manager;
use tauri_plugin_decorum::WebviewWindowExt;

// Create overlay titlebar
window.create_overlay_titlebar().unwrap();

// Set traffic lights position
window.set_traffic_lights_inset(12.0, 16.0).unwrap();

// Apply vibrancy effect
use window_vibrancy::{apply_vibrancy, NSVisualEffectMaterial};
apply_vibrancy(&window, NSVisualEffectMaterial::HudWindow, None, None)
    .expect("Unsupported platform! 'apply_vibrancy' is only supported on macOS");
```

## 🎯 Interaction Patterns

### Draggable Regions
```css
/* Make titlebar draggable */
-webkit-app-region: drag;

/* Remove drag from interactive elements */
-webkit-app-region: no-drag;
```

### Hover Effects
```css
/* Subtle window hover effect */
.macos-window:hover {
    transform: translateY(-0.125rem); /* -2px */
    box-shadow: /* Enhanced shadow */;
}

/* Traffic light hover */
.traffic-light:hover {
    opacity: 1;
    transform: scale(1.1);
}
```

### Animations
```css
/* Window appear animation */
@keyframes windowAppear {
    from {
        opacity: 0;
        transform: scale(0.95) translateY(1rem);
    }
    to {
        opacity: 1;
        transform: scale(1) translateY(0);
    }
}

/* Use native easing */
transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
```

## 🌓 Dark/Light Mode Support

### Dark Mode
```css
@media (prefers-color-scheme: dark) {
    .macos-window {
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        box-shadow: /* Darker shadows */;
    }
}
```

### Light Mode
```css
@media (prefers-color-scheme: light) {
    .macos-window {
        background: rgba(255, 255, 255, 0.2);
        border: 1px solid rgba(255, 255, 255, 0.3);
    }

    /* Adjust text colors for light background */
    .window-content h1 {
        color: rgba(0, 0, 0, 0.9);
        text-shadow: 0 1px 3px rgba(255, 255, 255, 0.5);
    }
}
```

## 📱 Responsive Design

### Mobile Adaptations
```css
@media (max-width: 48rem) {
    .window-container {
        width: 95vw;
        height: 80vh;
    }

    .titlebar {
        height: 2.75rem; /* 44px - touch-friendly */
    }

    .traffic-light {
        width: 1rem;
        height: 1rem;
    }
}
```

## 🔍 Focus States

### Accessibility
```css
.macos-window:focus-within {
    box-shadow:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meunomeebero/bewindow](https://github.com/meunomeebero/bewindow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
