---
trigger: always_on
description: This rule documents the complete implementation of visible, colorful horizontal scrollbars with rainbow gradient styling for tables and scrollable containers. The implementation ensures proper scrollbar visibility, centering of rightmost content, and responsive behavior across all screen sizes.
---

# Horizontal Scrollbar with Rainbow Gradient - Implementation Guide

## Overview
This rule documents the complete implementation of visible, colorful horizontal scrollbars with rainbow gradient styling for tables and scrollable containers. The implementation ensures proper scrollbar visibility, centering of rightmost content, and responsive behavior across all screen sizes.

## Problem Solved
- Horizontal scrollbar thumb was invisible or missing on tables
- Rightmost table content couldn't be centered when scrolled
- Tables forced horizontal scrolling even on large screens
- No visual distinction for scrollable areas

## Complete Implementation

### 1. CSS Styling for Scrollbar

Add this CSS styling using `dangerouslySetInnerHTML` in the component:

```tsx
<style dangerouslySetInnerHTML={{
  __html: `
    .table-scroll-container {
      overflow-x: scroll !important;
      overflow-y: visible !important;
      scrollbar-width: thin !important;
      scrollbar-color: #EC4899 #FCE7F3 !important; /* Pink thumb, pink track (Firefox) */
      -ms-overflow-style: -ms-autohiding-scrollbar !important;
    }

    /* WebKit browsers (Chrome, Safari, Edge) */
    .table-scroll-container::-webkit-scrollbar {
      height: 20px !important; /* Larger for visibility */
      display: block !important;
      -webkit-appearance: none !important;
      appearance: none !important;
    }

    .table-scroll-container::-webkit-scrollbar-track {
      background: linear-gradient(90deg, #DBEAFE, #E9D5FF, #FCE7F3, #FED7AA) !important;
      border-radius: 10px !important;
      -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.15) !important;
      box-shadow: inset 0 0 6px rgba(0,0,0,0.15) !important;
    }

    .table-scroll-container::-webkit-scrollbar-thumb {
      background: linear-gradient(90deg, #3B82F6, #8B5CF6, #EC4899, #F97316) !important;
      border-radius: 10px !important;
      border: 4px solid #F3F4F6 !important;
      -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.4) !important;
      box-shadow: inset 0 0 6px rgba(0,0,0,0.4) !important;
      min-width: 50px !important; /* CRITICAL: Ensures thumb is always visible */
      background-clip: padding-box !important;
    }

    .table-scroll-container::-webkit-scrollbar-thumb:hover {
      background: linear-gradient(90deg, #2563EB, #7C3AED, #DB2777, #EA580C) !important;
      border-color: #E5E7EB !important;
    }

    .table-scroll-container::-webkit-scrollbar-thumb:active {
      background: linear-gradient(90deg, #1D4ED8, #6D28D9, #BE185D, #C2410C) !important;
      border-color: #D1D5DB !important;
    }

    .table-scroll-container::-webkit-scrollbar-button {
      display: none !important;
    }

    .table-scroll-container::-webkit-scrollbar-corner {
      background: #E0E7FF !important;
    }

    /* Flexbox spacer for right-side centering */
    .table-scroll-container::after {
      content: '';
      display: block;
      width: 100vw; /* Full viewport width of scrollable space */
      height: 1px;
      flex-shrink: 0;
    }

    .table-scroll-container {
      display: flex !important;
    }
  `
}} />
```

### 2. HTML Structure with Rainbow Gradient Background

```tsx
{/* Outer wrapper with gradient border */}
<div className="rounded-lg shadow w-full overflow-hidden" style={{
  background: 'linear-gradient(to right, #3B82F6, #8B5CF6, #EC4899, #F97316)',
  padding: '4px'
}}>
  {/* Inner scroll container with gradient background */}
  <div
    className="w-full table-scroll-container"
    style={{
      overflowX: 'scroll',
      overflowY: 'visible',
      WebkitOverflowScrolling: 'touch',
      maxWidth: '100%',
      display: 'flex',
      position: 'relative',
      width: '100%',
      minHeight: '1px',
      scrollbarGutter: 'stable',
      background: 'linear-gradient(to right, #3B82F6, #8B5CF6, #EC4899, #F97316)',
      borderRadius: '8px',
      padding: '20px'
    }}
  >
    {/* Table with semi-transparent white background */}
    <table
      className="divide-y divide-gray-200"
      style={{
        width: 'max-content',
        minWidth: 'fit-content', /* Responsive: fits content naturally */
        flexShrink: 0,
        background: 'rgba(255, 255, 255, 0.95)', /* Semi-transparent white */
        borderRadius: '8px',
        overflow: 'hidden'
      }}
    >
      {/* Table content */}
    </table>
  </div>
</div>
```

### 3. Color Palette

**Rainbow Gradient Colors:**
- Blue-500: `#3B82F6`
- Purple-600: `#8B5CF6`
- Pink-500: `#EC4899`
- Orange-500: `#F97316`

**Pastel Track Colors:**
- Light Blue: `#DBEAFE`
- Light Purple: `#E9D5FF`
- Light Pink: `#FCE7F3`
- Light Orange: `#FED7AA`

**Hover States (Darker):**
- Blue-600: `#2563EB`
- Purple-700: `#7C3AED`
- Pink-600: `#DB2777`
- Orange-600: `#EA580C`

**Active States (Darkest):**
- Blue-700: `#1D4ED8`
- Purple-800: `#6D28D9`
- Pink-700: `#BE185D`
- Orange-700: `#C2410C`

## Key Features

### 1. Visible Scrollbar Thumb
- **Height**: 20px (larger than default for visibility)
- **Minimum Width**: 50px (CRITICAL - ensures thumb is always visible)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
