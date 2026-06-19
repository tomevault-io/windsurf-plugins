---
trigger: always_on
description: Complete Laravel API architecture with Repository-Service-Controller pattern. Includes BaseRepository/BaseService with inheritance, Searchable/Filterable traits, automatic pagination/filters, Laravel Data (DTOs), type safety, clean code principles, Scramble documentation. Use for: building complete Laravel backend, creating CRUD resources, implementing search/filters, refactoring architecture, ensuring SOLID principles, generating API documentation.
---


# Laravel API Architecture

Arquitectura completa en capas para desarrollo de APIs Laravel con patrón Repository-Service-Controller optimizado.

## 🎯 Cuándo Usar Esta Skill

**Usar automáticamente cuando se trabaje con:**
- Desarrollar backend Laravel completo con arquitectura limpia
- Crear nuevo recurso CRUD desde cero (Model, Repository, Service, Controller, DTOs)
- Implementar sistema de búsqueda y filtros dinámicos
- Configurar paginación automática con metadata
- Refactorizar código legacy a arquitectura en capas
- Eliminar código repetitivo usando herencia (BaseRepository, BaseService)
- Implementar validación con Laravel Data
- Generar documentación API automática con Scramble
- Verificar separación de responsabilidades y principios SOLID
- Escribir tests por cada capa de la arquitectura

**Palabras clave que disparan esta skill:**
- "arquitectura Laravel", "backend completo", "API Laravel"
- "crear recurso", "nuevo CRUD", "Repository", "Service", "Controller"
- "BaseRepository", "BaseService", "herencia", "código repetitivo"
- "Searchable", "Filterable", "traits", "búsqueda", "filtros"
- "paginación", "ordenamiento", "filtros dinámicos"
- "Laravel Data", "DTO", "validación", "type safety"
- "Scramble", "documentación API", "OpenAPI", "Swagger"
- "refactorizar", "separar responsabilidades", "SOLID", "Clean Architecture"
- "testing", "arquitectura en capas"

## 🏗️ Arquitectura Completa

### Componentes Principales

Esta arquitectura combina múltiples patrones y herramientas:

1. **Repository Pattern**: Abstracción de acceso a datos
2. **Service Layer Pattern**: Lógica de negocio centralizada  
3. **Laravel Data (Spatie)**: DTOs con validación y transformación
4. **Herencia con Clases Base**: BaseRepository, BaseService (elimina ~70% código repetitivo)
5. **Traits Reutilizables**: Searchable, Filterable en Models
6. **Paginación Inteligente**: Automática con búsqueda, filtros y ordenamiento
7. **Documentación Automática**: Scramble genera OpenAPI/Swagger
8. **Type Safety Completo**: PHP typed properties throughout
9. **Testing por Capas**: Unit tests independientes por Repository, Service, Controller

### Diagrama de Arquitectura

```
HTTP Request (GET /api/products?search=laptop&category=5&per_page=20)
    ↓ Validado por Laravel Data
ProductIndexQueryData {search, category, per_page, order_by}
    ↓ Controller delega
ProductService::paginate($query)
    ↓ Service usa Repository heredado
BaseRepository::paginateWithQuery($query)
    ↓ Repository usa Traits del Model
Product::search('laptop')->filter(['category'=>5])->paginate(20)
    ↓ Eloquent ejecuta
Database → Collection<Product>
    ↓ Repository retorna
LengthAwarePaginator<Product>
    ↓ Service retorna (Controller transforma para paginación)
Controller: ProductData::collect($products)
    ↓ ApiResponseTrait formatea
JsonResponse {success, data, links, meta}
```

### Principios SOLID Aplicados

1. **Single Responsibility**: Cada capa una responsabilidad clara
   - Controller: Orquestación HTTP
   - Service: Lógica de negocio + Transformación DTO
   - Repository: Acceso a datos
   - Model: Entidad + Scopes

2. **Open/Closed**: Extensible sin modificar
   - BaseRepository: Herencia para nuevos recursos
   - BaseService: Métodos comunes heredados
   - Traits: Comportamiento compartido (Searchable, Filterable)

3. **Liskov Substitution**: Interfaces intercambiables
   - ProductRepositoryInterface → ProductRepository
   - Mock repositories en tests

4. **Interface Segregation**: Interfaces específicas
   - BaseRepositoryInterface: Métodos comunes
   - ProductRepositoryInterface: Métodos específicos de Product

5. **Dependency Inversion**: Depender de abstracciones
   - Services dependen de RepositoryInterface, no implementación concreta
   - Controllers dependen de Services, no Repositories directamente

### Responsabilidades por Capa

| Capa | Responsabilidad Principal | Retorna | NO Debe Hacer |
|------|--------------------------|---------|---------------|
| **Controller** | Orquestación HTTP, validación request, autorización | JsonResponse | Lógica de negocio, queries DB, transformar Models manualmente |
| **Service** | Lógica de negocio, transacciones, transformar Model→DTO | **DTO** | Queries directas, formateo HTTP, acceso a Request |
| **Repository** | Queries DB, acceso a datos | **Model** | Lógica de negocio, transformar a DTO, validaciones |
| **Model** | Entidad, relaciones, scopes (search, filter) | - | Lógica de negocio compleja, acceso a otros services |

### Reglas de Oro

1. **Services retornan DTOs** (no Models Eloquent)
2. **Controllers solo orquestan** (no transforman Models)
3. **Repositories retornan Models** (no DTOs)
4. **Type hints estrictos** en todos los métodos
5. **BaseRepository/BaseService** eliminan código repetitivo
6. **Traits Searchable/Filterable** automatizan búsqueda y filtros

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RuddyG4/laravel-api-architecture](https://github.com/RuddyG4/laravel-api-architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
