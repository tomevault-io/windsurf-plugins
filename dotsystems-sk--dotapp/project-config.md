---
trigger: always_on
description: **You are working with the DOTAPP PHP FRAMEWORK - NOT Laravel, Symfony, CodeIgniter, or any other framework!**
---

# DotApp Framework - AI Assistant Configuration
# ⚠️  READ THIS ENTIRE FILE BEFORE EVERY CODE CHANGE! ⚠️
# ⚠️  RE-READ THESE RULES FOR EVERY REQUEST! ⚠️

## 🚨 CRITICAL FRAMEWORK IDENTITY
**You are working with the DOTAPP PHP FRAMEWORK - NOT Laravel, Symfony, CodeIgniter, or any other framework!**

**DOTAPP is a proprietary framework with unique syntax and patterns.** If you don't understand how something works, **study the actual code in `/app/parts/`** - do NOT assume it works like other frameworks.

## 🤖 AI ASSISTANT CRITICAL WARNING
**BEFORE ANY WORK ON A MODULE: Always read the MD guide files in that specific module!**
- Each module has its own `*_AI_guide.md` files (e.g., `CONTROLLERS_AI_guide.md`, `VIEWS_AI_guide.md`)
- **Example:** Working on `TestPaka` module? Read `TestPaka/CONTROLLERS_AI_guide.md` FIRST!
- **Why?** Module-specific guides override general rules and contain exact syntax examples!
- **AI Memory Tip:** If you forget this, you'll use wrong syntax and create bugs!

### Common Misconceptions to Avoid:
- ❌ NO `$this->db` or `DB::table()` like Laravel
- ❌ NO Doctrine ORM syntax
- ❌ NO Symfony dependency injection containers
- ❌ NO CodeIgniter active records
- ✅ YES `DB::module("RAW")` and `DB::module("ORM")`
- ✅ YES Static controllers with DI
- ✅ YES Custom template syntax `{{ variable }}`

---

## 🏗️ FRAMEWORK ARCHITECTURE

### Core Structure (`/app/parts/` - 🔴 DO NOT TOUCH!)
```
/app/parts/                    # 🔴 CORE FRAMEWORK - NEVER EDIT!
├── Controllers/              # Core controller classes
├── Middleware/               # Core middleware classes
├── Models/                   # Core model classes
├── Databaser.php             # Database abstraction layer
├── QueryBuilder.php          # SQL query builder
├── Entity.php               # ORM Entity class
├── Collection.php           # ORM Collection class
├── Renderer.php             # Template rendering engine
├── Router.php               # Route handling
├── DI.php                   # Dependency injection
├── DB.php                   # Database facade
├── Request.php              # HTTP request handling
├── Response.php             # HTTP response handling
├── Validator.php            # Data validation
├── Translator.php           # Internationalization
└── [many more core files...]
```

### Module Structure (`/app/modules/` - 🟢 SAFE TO EDIT!)
```
/app/modules/                 # 🟢 MODULES - EDIT HERE!
├── ModuleName/
│   ├── Controllers/          # Module controllers
│   │   └── CONTROLLERS_GUIDE.md  # Controller documentation
│   ├── Models/               # Module models
│   ├── views/                # Module templates
│   ├── assets/               # CSS, JS, images
│   │   └── ASSETS_GUIDE.md   # Asset management
│   ├── translations/         # Language files
│   │   └── TRANSLATOR_GUIDE.md   # Translation documentation
│   ├── tests/                # Module tests
│   │   └── TESTS_GUIDE.md    # Testing documentation
│   ├── MODULE_GUIDE.md       # Module documentation
│   └── module.init.php       # Module initialization
```

---

## 📚 DOCUMENTATION MAP

### Core Framework Guides
- **`database_guide.md`** - Complete database usage guide (RAW, ORM, QueryBuilder)
- **`ai_database_guide.md`** - Database operations guide specifically for AI assistants

### Module-Specific Guides
## 🔴 CRITICAL: ALWAYS READ MODULE MD FILES FIRST!
**Each module in `/app/modules/` contains its own documentation that YOU MUST READ before working on that module!**

**Example workflow:**
1. User asks: "Create a route in TestPaka module"
2. YOU: Read `TestPaka/CONTROLLERS_AI_guide.md` and `TestPaka/views/VIEWS_AI_guide.md`
3. Only then start coding with the correct syntax from those files

**If you skip this step, you'll use wrong syntax and create bugs!**

#### Controllers (`*/Controllers/CONTROLLERS_GUIDE.md` or `*_AI_guide.md`)
- Static class structure
- Dependency injection usage
- Request/response handling
- API dispatch patterns

#### Views & Templates (`*/views/` or `*/ASSETS_GUIDE.md`)
- Custom template syntax `{{ variable }}`
- Layout system
- Asset management

#### Translations (`*/translations/TRANSLATOR_GUIDE.md`)
- Key-based translation system
- JSON format for language files
- Fallback mechanisms

#### Tests (`*/tests/TESTS_GUIDE.md` or `*_AI_guide.md`)
- Testing framework usage
- Module-specific test patterns

#### Module Initialization (`*/MODULE_GUIDE.md` or `*_AI_guide.md`)
- Module lifecycle
- Configuration options
- Event listeners setup

---

## ⚠️ ABSOLUTE SECURITY RULES

### 1. NEVER EDIT CORE FILES
```bash
❌ NEVER: /app/parts/*.php
❌ NEVER: /app/DotApp.php
❌ NEVER: /app/config.php
✅ SAFE: /app/modules/*/*
```

**If you think there's an error in core files, ASK THE USER - DO NOT EDIT!**

### 2. CONTROLLER CALLING SYNTAX (CRITICAL!)
**NEVER use `Controller@function` format!**

**✅ CORRECT DotApp syntax:**
```php
// Module:Controller@function!
PharmListWeb:Home@indexSk!
PharmListWeb:Search@indexEn!

// Use ! at end to skip DI container (saves memory)
```

**❌ WRONG (Laravel/Symfony style):**
```php
// DO NOT USE THESE!
HomeController@index
Controller@function
Home@index
```

**❗ IMPORTANT: When using `!` (no DI):**
- **NO dependency injection parameters** in controller methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dotsystems-sk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
