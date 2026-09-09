---
trigger: always_on
description: Paozhu (炮竹) is a high-performance C++20 web framework built on the Asio asynchronous networking library, supporting HTTP/HTTPS/HTTP2 protocols. The framework adopts an MVC architecture and integrates ORM, template engine, coroutine support, and a rich set of functional modules.
---

# Paozhu C++ Web Framework - AI Development Guide

## I. Project Overview

Paozhu (炮竹) is a high-performance C++20 web framework built on the Asio asynchronous networking library, supporting HTTP/HTTPS/HTTP2 protocols. The framework adopts an MVC architecture and integrates ORM, template engine, coroutine support, and a rich set of functional modules.

### Core Features
- **Asynchronous I/O**: Asio‑based non‑blocking network model
- **Coroutine Support**: Use C++20 `co_await` for asynchronous code
- **HTTP/HTTP2**: Native support for HTTP/1.1 and HTTP/2
- **ORM**: Built‑in ORM with MySQL PostgreSQL and SQLite support
- **OCSP Stapling**: Automatic OCSP stapling
- **ACME**: Automatic SSL certificate issuance and renewal
- **Modular Extensions**: Image processing, Excel, Word, PDF, chart generation, etc.

### Compilation Standards
- C++20 or later
- OpenSSL 3.0+ (for SSL/TLS)
- Cross‑platform: macOS, Linux, Windows

---

## II. Directory Structure

```
paozhu/
├── conf/                          # Configuration files
│   ├── server.conf                # Main server configuration
│   ├── orm.conf                   # Database connection settings
│   ├── acme.conf                  # ACME certificate settings
│   └── ...                        # Other configuration and SQL files
│
├── controller/                    # Controller layer
│   ├── include/                   # Controller headers
│   │   ├── admin/                 # Admin controllers
│   │   └── test*.h               # Functional test controllers
│   └── src/                       # Controller implementations
│       ├── admin/                 # Admin implementations
│       └── test*.cpp              # Functional test implementations
│
├── models/                        # Data model layer
│   ├── cms/                       # CMS‑related models
│   ├── ph/                        # PostgreSQL test models
│   └── include/                   # Model headers
│
├── orm/                           # ORM layer (auto‑generated)
│   ├── cms/include/               # CMS ORM operation classes
│   │   ├── *_base.h               # Base field definitions (auto‑generated)
│   │   └── *_opsql.h              # SQL operation intermediate layer (auto‑generated)
│   ├── include/                   # Common ORM files
│   └── orm.h                      # ORM unified entry
│
├── view/                          # View templates (HTML)
│   ├── admin/                     # Admin views
│   ├── home/                      # Home views
│   └── ...                        # Other views
│
├── viewsrc/                       # Compiled view artifacts (C++ sources)
│   ├── include/                   # View registration headers
│   └── view/                      # View implementations
│
├── vendor/                        # Third‑party / internal libraries
│   ├── httpserver/                # HTTP server core
│   │   ├── include/               # Server headers
│   │   │   ├── request.h          # HTTP request handling
│   │   │   ├── httppeer.h         # HTTP request/response handler
│   │   │   ├── router.h           # URL routing
│   │   │   └── ...               # Other headers
│   │   └── src/                   # Server implementations
│   ├── httpcli/                   # CLI tools (code generators)
│   ├── pzexcel/                   # Excel read/write
│   ├── pzword/                    # Word document generation
│   ├── pzzip/                     # ZIP compression/decompression
│   ├── pzimage/                   # Image processing (JPG/PNG)
│   ├── pzcharts/                  # SVG chart generation
│   ├── webpdf/                    # HTML to PDF conversion
│   ├── sms/                       # SMS sending
│   └── webpay/                    # WeChat / Alipay payments
│
├── libs/                          # Business utility libraries
│   ├── img/                       # Image upload handling
│   ├── markdown/                  # Markdown to HTML conversion
│   ├── pinyin/                    # Chinese to Pinyin conversion
│   ├── ipdata/                    # IP geolocation lookup
│   └── types/                     # Common type definitions
│
├── common/                        # Common definitions
│   ├── autorestfulpaths.hpp       # RESTful path registration
│   ├── reghttpmethod.hpp          # HTTP method registration
│   └── ...
│
├── sockets/                       # Socket extensions
├── websockets/                    # WebSocket support
├── startup/                       # Program entry points
│   ├── main_dev.cpp               # Development mode entry
│   ├── main_daemon.cpp            # Daemon mode entry
│   └── main_docker.cpp            # Docker entry
│
├── www/                           # Web static file root
├── docs/                          # Example resources (images, fonts, etc.)
├── CMakeLists.txt                 # CMake build configuration
└── AGENTS.md                      # This document
```

---

## III. Configuration File Reference

### 3.1 server.conf – Server Configuration

```ini
[default]
; Thread settings
threadmax = 1024          ; Maximum threads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hggq/paozhu](https://github.com/hggq/paozhu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
