---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important: Documentation Updates

**CRITICAL RULE**: Every change, feature addition, or update made to this project MUST be reflected in the documentation. When making any modifications:
1. Update CLAUDE.md if architecture, structure, or processes change
2. Update PRD.md if requirements or features are modified
3. Keep all documentation synchronized with the actual codebase

## Project Overview

**Varnion QR Code Generator** is a web-based application that enables users to create customizable QR codes with advanced styling options. The application provides real-time preview, logo integration, and multiple export formats without requiring user authentication.

**Key Links:**
- Product Requirements: `/home/bintt/projects/node/varnion-qr-generator/PRD.md`

## Current Status

**Project Status:** Planning Phase  
**Development Stage:** Not yet started - awaiting initial setup and development  
**Repository Type:** Brand new project (no existing codebase)

## Project Structure

Currently, the project only contains the Product Requirements Document (PRD.md). The following directories and files will be created during development:

```
varnion-qr-generator/
├── PRD.md                              # Product Requirements Document
├── package.json                        # Dependencies and scripts
├── next.config.js                      # Next.js configuration
├── tailwind.config.js                  # Tailwind CSS configuration
├── .gitignore                          # Git ignore rules
├── .env.local                          # Environment variables (local)
├── README.md                           # Project documentation
├── CLAUDE.md                           # This file
│
├── public/
│   └── assets/
│       └── default-logo.jpg            # Varnion default logo
│
├── app/
│   ├── page.js                         # Main QR generator page
│   ├── layout.js                       # Root layout
│   └── api/
│       ├── generate-qr/route.js        # QR generation endpoint (optional)
│       └── health/route.js             # Health check endpoint
│
├── components/
│   ├── qr-generator/                   # Main QR generation component
│   ├── qr-preview/                     # Real-time preview canvas
│   ├── qr-type-selector/               # Type selection UI
│   ├── customization/                  # Customization panels
│   │   ├── color-customizer.js
│   │   ├── shape-customizer.js
│   │   ├── logo-uploader.js
│   │   └── error-correction.js
│   └── export/                         # Export controls
│
└── lib/
    ├── qr-generator.js                 # QR generation utilities
    ├── validators.js                   # Input validation logic
    └── utils.js                        # Helper functions
│
└── tests/ (future)
    ├── unit/
    ├── integration/
    └── e2e/
```

## Technology Stack

### Frontend
- **Framework:** Next.js 14+ (React) with App Router
- **Language:** JavaScript (ES6+) - No TypeScript/compilation for faster development
- **Styling:** Tailwind CSS
- **UI Components:** Radix UI or shadcn/ui (accessible components)
- **Form Handling:** React Hook Form + Zod (runtime validation)
- **Color Picker:** react-colorful or similar

### Backend
- **Runtime:** Node.js 18+ LTS
- **Server Framework:** Next.js API Routes
- **Image Processing:** sharp (logo integration, format conversion)
- **PDF Generation:** jspdf
- **QR Generation:** qr-code-styling (supports advanced customization)
- **File Handling:** file-saver (client-side downloads)

### Deployment
- **Target Environment:** VPS (Ubuntu 20.04+ / Debian 11+)
- **Process Manager:** PM2
- **Web Server:** Nginx (reverse proxy)
- **No Docker:** Direct Node.js deployment
- **SSL:** Let's Encrypt (Certbot)

### Development Tools
- **Package Manager:** npm or yarn
- **Version Control:** Git
- **Code Quality:** ESLint, Prettier (recommended)
- **Testing:** Jest, React Testing Library (recommended)
- **No TypeScript:** Pure JavaScript for faster iteration without compilation overhead

## Core Features

### 1. QR Code Types (7 types)
- **URL**: Website URL generation
- **Plain Text**: Up to 500 characters
- **Email**: With optional subject and body
- **Phone Number**: Tel format with validation
- **WiFi**: Network config (SSID, password, encryption)
- **vCard (Contact)**: Full contact information
- **SMS**: With optional message body

### 2. Customization Features
- **Colors**: Foreground, background, and gradient support
- **Gradient Types**: Linear with direction options (horizontal, vertical, diagonal)
- **Shapes**: Dot styles (square, rounded, dots, extra-rounded)
- **Corner Styles**: For corner squares and dots
- **Logo Integration**: 
  - Default Varnion logo pre-loaded
  - Upload custom logos (PNG, JPG, JPEG, SVG)
  - Logo size: 10-30% of QR code
  - Support for square, circular, and rectangular logos
  - Option to add white padding
  - Remove or reset to default logo
- **Error Correction Level**: L (7%), M (15%), Q (25%), H (30%)

### 3. Real-Time Preview
- Live canvas showing QR code changes
- Debounced updates (300ms)
- Responsive preview (minimum 300x300px)
- Loading indicator during generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [varnionjgj/varnion-qr-generator](https://github.com/varnionjgj/varnion-qr-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
