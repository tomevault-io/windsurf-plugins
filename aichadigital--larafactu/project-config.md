---
trigger: always_on
description: Larafactu Project - Staging for Billing Packages Development
---


# Larafactu - Staging Project Rules

## 🎯 Project Identity

### Core Purpose
- **Staging environment** for testing billing packages in real-world scenarios
- **Target v1.0**: Hosting companies, Spain/EU, Intracom Operator
- **Critical deadline**: December 15, 2025 - stable version for WHMCS migration
- **Philosophy**: Pragmatic, Laravel-aligned, avoid over-engineering

### Main Packages Under Development
Located at: `/Users/abkrim/development/packages/aichadigital/`

1. **larabill** (core billing) - dev-main
2. **lararoi** (EU VAT/ROI logic) - dev-main
3. **lara-verifactu** (Spain AEAT integration) - dev-main
4. **laratickets** (support tickets) - dev-main
5. **lara100** (base-100 monetary) - v1.0 stable

## 🏗️ Technical Stack

### Framework & Versions
- **PHP**: 8.4.13
- **Laravel**: 12.x
- **Filament**: 4.x
- **Pest**: 4.x (browser testing enabled)
- **Tailwind CSS**: 4.x
- **Database**: MySQL (local: Laravel Herd)

### UUID Strategy - String UUID v7

**IMPORTANTE**: Usamos UUID v7 **STRING** (char 36), NO binary.

#### Implementación

```php
// Model con UUID
use AichaDigital\Larabill\Concerns\HasUuid;

class User extends Authenticatable
{
    use HasUuid;
    // NO necesita $incrementing ni $keyType - el trait lo maneja
}
```

#### Migraciones

```php
// UUID string (36 chars)
$table->uuid('id')->primary();
$table->uuid('user_id');  // FK a users

// Para relaciones
$table->foreignUuid('user_id')->constrained()->cascadeOnDelete();
```

**Modelos con UUID**: User, Invoice, Ticket
**Modelos con Integer ID**: TaxRate, CompanyFiscalConfig, CustomerFiscalData, UnitMeasure

### UUID Binary + Filament (CRITICAL)

Si usas `uuid_binary` (binario 16 bytes), hay reglas especiales para Filament:

#### Select con relaciones de usuario

```php
// ❌ MAL - pluck directo NO aplica el cast, devuelve binario
Forms\Components\Select::make('user_id')
    ->options(fn () => User::pluck('name', 'id'))  // ERROR: binario

// ✅ BIEN - Cargar modelos PRIMERO, luego pluck (aplica cast)
Forms\Components\Select::make('user_id')
    ->options(fn () => User::all()->pluck('name', 'id'))  // OK: UUID string
```

#### Modelos con user_id FK

Usar el trait `HasUserRelation` de larabill:

```php
use AichaDigital\Larabill\Concerns\HasUserRelation;

class CustomerFiscalData extends Model
{
    use HasUserRelation;  // Añade cast EfficientUuid + relación user()
}
```

#### Filament 4 API Changes

```php
// Filament 3 → Filament 4
->actions()     → ->recordActions()
->bulkActions() → ->toolbarActions()

// Columnas con fecha nullable
->default('Texto')      // ❌ MAL - intenta parsear como fecha
->placeholder('Texto')  // ✅ BIEN - muestra texto si null
```

## 💰 Monetary Values - Base 100 (Lara100)

### CRITICAL RULE
**NEVER use float/decimal for monetary values, prices, amounts, tax rates**

### Always Use Integers (Base 100)
- €12.34 → `1234` (integer)
- 21.5% IVA → `2150` (integer)
- €0.99 → `99` (integer)

### Package
- `aichadigital/lara100` (v1.0 - stable)
- Prevents floating-point errors
- Ensures exact fiscal calculations

## 📦 Package Development Mode

### Instalador Inteligente

**Comando principal**: `php artisan larafactu:install`

```bash
# Instalación local interactiva (recomendado)
php artisan larafactu:install

# Instalación local con opciones
php artisan larafactu:install --local --fresh

# Instalación producción (sin symlinks)
php artisan larafactu:install --production

# Especificar ruta de paquetes
php artisan larafactu:install --local --packages-path=/custom/path
```

### Qué hace el instalador en modo local

1. **Crea symlinks** en `packages/aichadigital/` → `/Users/abkrim/development/packages/aichadigital/`
2. **Modifica composer.json** añadiendo path repositories
3. **Ejecuta composer update** para usar paquetes locales
4. **Ejecuta migraciones** (opción fresh o migrate)
5. **Ejecuta seeders** de desarrollo

### Reset Completo (Fresh Install)

```bash
# Preserva .cursor, .claude, .env, .mcp.json
bin/fresh-install.sh

# Luego ejecutar instalador
php artisan larafactu:install --local --fresh
```

### Local Development Setup

**CRITICAL**: Larafactu usa **path repositories con symlinks** para desarrollo local.

```
/Users/abkrim/
├── development/packages/aichadigital/  # Paquetes SOURCE
│   ├── larabill/
│   ├── lararoi/
│   ├── lara-verifactu/
│   └── laratickets/
└── SitesLR12/larafactu/                # App STAGING
    └── packages/aichadigital/          # Symlinks
```

### Workflow de Desarrollo

Ver `docs/WORKFLOW.md` para flujo completo.

**Resumen**:
1. Editar en `packages/aichadigital/` (symlinks)
2. Cambios reflejados inmediatamente
3. Commit SEPARADO: primero paquete, luego app
4. `composer update aichadigital/*` después de commits en paquetes

## 🧪 Testing Standards

### Coverage Goals (Pragmatic)
- **Packages**: 80-95% coverage
- **Staging app**: 60-70% coverage
- **Priority**: Feature tests > Unit tests

### Comandos

```bash
php artisan test                    # Todos los tests
php artisan test --filter=Invoice   # Filtrar
vendor/bin/pint --dirty             # Formatear
```

## 📚 Gestión de Documentación

### Estructura de docs/

```
docs/
├── ADR_*.md              # Decisiones arquitectónicas (permanentes)
├── PRODUCTION_*.md       # Guías de producción (permanentes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AichaDigital) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
