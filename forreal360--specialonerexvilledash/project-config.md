---
trigger: always_on
description: - Laravel 12.x con PHP 8.2+
---


# Guía de Desarrollo - Hyundai Special One Rexville Dashboard

## Stack Tecnológico

**Backend:**
- Laravel 12.x con PHP 8.2+
- Laravel Sanctum para autenticación
- Laravel Socialite para autenticación social

**Frontend:**
- Livewire 3.6 para componentes reactivos
- Flux 2.2 para componentes de UI
- Tailwind CSS para estilos
- Vite para compilación de assets

**Base de datos:**
- MySQL/MariaDB
- Migraciones de Laravel
- Seeders para datos iniciales

## Arquitectura del Proyecto

### 1. Patrón Action-Service

El proyecto utiliza un patrón Action-Service donde:

**Actions** (`app/Actions/V1/`):
- Manejan la lógica de negocio específica
- Extienden la clase base `App\Actions\V1\Action`
- Retornan `ActionResult` para respuestas consistentes
- Incluyen validación, permisos y manejo de transacciones
- Se organizan por módulos (Admin, Client, Auth, etc.)

**Services** (`app/Services/V1/`):
- Manejan operaciones CRUD y lógica de datos
- Extienden la clase base `App\Services\V1\Service`
- Proporcionan métodos como `getPaginated()`, `findById()`, etc.
- Se configuran con modelo asociado y campos buscables

### 2. Estructura de ActionResult

```php
// Éxito
return $this->successResult(
    data: $result,
    message: 'Operación completada exitosamente'
);

// Error
return $this->errorResult(
    message: 'Error al procesar',
    errors: $errors,
    statusCode: 400
);

// Error de validación
return $this->validationErrorResult(
    errors: $validator->errors(),
    message: 'Datos inválidos'
);
```

### 3. Componentes Livewire

**Organización** (`app/Livewire/V1/`):
- `Auth/`: Componentes de autenticación
- `Panel/`: Componentes del panel administrativo
- `Components/`: Componentes reutilizables

**Patrón de componentes:**
- Utilizan el trait `HandlesActionResults` para manejo de respuestas
- Inyectan Actions y Services via dependency injection
- Manejan estados de formulario y validación en tiempo real

## Componentes Reutilizables

### 1. Componentes de Formularios (`resources/views/components/forms/`)

#### `<x-forms.form-field>`
Wrapper estándar para todos los campos de formulario:
```blade
<x-forms.form-field label="{{ __('panel.name') }}*" for="name" :error="$errors->first('name')">
    <flux:input
        id="name"
        wire:model="name"
        placeholder="Nombre"
        error="{{ $errors->first('name') }}"
    />
</x-forms.form-field>
```

#### `<x-forms.flatpickr-date>`
Componente de selección de fecha con Flatpickr:
```blade
<x-forms.form-field label="{{ __('panel.start_date') }}*" for="start_date" :error="$errors->first('start_date')">
    <x-forms.flatpickr-date
        name="start_date"
        wire:model="start_date"
        dateFormat="m/d/Y"
        placeholder="{{ __('panel.start_date') }}"
        minDate="today"
        error="{{ $errors->first('start_date') }}"
        required
    />
</x-forms.form-field>
```

**Propiedades disponibles:**
- `dateFormat`: Formato de fecha (por defecto 'd/m/Y')
- `placeholder`: Texto placeholder
- `minDate/maxDate`: Fechas límite
- `locale`: Idioma ('es' por defecto)
- `required`: Si es obligatorio
- `size`: Tamaño ('xs', 'sm', 'md', 'lg', 'xl')

#### `<x-forms.file-upload>`
Componente de subida de archivos con drag & drop:
```blade
<x-forms.form-field label="{{ __('panel.image') }}*" for="file" :error="$errors->first('file')">
    <x-forms.file-upload
        name="file"
        wireModel="file"
        accept="image/*"
        :error="$errors->first('file')"
        required
    />
</x-forms.form-field>
```

### 2. Componentes de Tabla (`resources/views/components/table/`)

#### `<x-table.table>`
Componente principal de tabla con paginación, búsqueda y filtros:
```blade
<x-table.table
    :data="$items"
    :perPageOptions="$perPageOptions"
    :currentPerPage="$perPage"
    :search="$search"
    searchPlaceholder="{{ __('panel.search_placeholder') }}"
>
    <x-slot name="filters">
        <!-- Filtros aquí -->
    </x-slot>
    <x-slot name="colums">
        <!-- Columnas aquí -->
    </x-slot>
    <x-slot name="rows">
        <!-- Filas aquí -->
    </x-slot>
</x-table.table>
```

#### Filtros de Tabla
Los filtros van dentro del slot `filters` usando `<flux:field>`:
```blade
<x-slot name="filters">
    <!-- Filtro de estado -->
    <flux:field class="w-full">
        <flux:label>{{ __('panel.status') }}</flux:label>
        <flux:select wire:model.live="status" size="sm" placeholder="{{ __('panel.all_statuses') }}">
            <flux:select.option value="active">{{ __('panel.active') }}</flux:select.option>
            <flux:select.option value="inactive">{{ __('panel.inactive') }}</flux:select.option>
        </flux:select>
    </flux:field>

    <!-- Filtro de fecha desde -->
    <x-forms.flatpickr-date
        name="filter_start_date"
        wire:model.live="filter_start_date"
        size="sm"
        label="{{ __('panel.filter_start_date') }}"
        dateFormat="m/d/Y"
        placeholder="{{ __('panel.filter_start_date') }}"
        locale="{{ app()->getLocale() }}"
    />

    <!-- Filtro de fecha hasta -->
    <x-forms.flatpickr-date
        name="filter_end_date"
        wire:model.live="filter_end_date"
        size="sm"
        label="{{ __('panel.filter_end_date') }}"
        dateFormat="m/d/Y"
        placeholder="{{ __('panel.filter_end_date') }}"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Forreal360/SpecialOneRexvilleDash](https://github.com/Forreal360/SpecialOneRexvilleDash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
