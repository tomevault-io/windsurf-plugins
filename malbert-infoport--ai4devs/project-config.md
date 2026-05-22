---
trigger: always_on
description: Este proyecto está basado en el **Framework Helix6**, un framework empresarial para desarrollo de Web APIs con .NET 8 que implementa una arquitectura en capas con Clean Architecture y principios DDD.
---

# GitHub Copilot Instructions - Helix6 Framework

Este proyecto está basado en el **Framework Helix6**, un framework empresarial para desarrollo de Web APIs con .NET 8 que implementa una arquitectura en capas con Clean Architecture y principios DDD.

## Contexto del Proyecto

**Framework**: Helix6 v1.0  
**Runtime**: .NET 8.0  
**Arquitectura**: N-Layer Architecture (Api → Services → Data → DataModel)  
**Patrones principales**: Repository Pattern, Service Layer, Unit of Work, DDD

## Principios Fundamentales de Helix6

### 1. Convención sobre Configuración
- Las clases siguen convenciones estrictas de nomenclatura que permiten autodescubrimiento
- Repositorios: `[Entidad]Repository` implementa `IBaseRepository<Entity>`
- Servicios: `[Entidad]Service` hereda de `BaseService<View, Entity, Metadata>`
- Views (DTOs): `[Entidad]View` implementa `IViewBase`
- Endpoints: `/api/[Entidad]/[Método]`

### 2. DRY (Don't Repeat Yourself)
- El framework proporciona clases base genéricas para CRUD automático
- No duplicar código que ya existe en `BaseService` o `BaseRepository`
- Sobrescribir solo los hooks necesarios: `ValidateView`, `PreviousActions`, `PostActions`

### 3. Separación de Responsabilidades
```
┌─────────────────────────────────────────┐
│  Api (Presentación)                     │  ← Endpoints HTTP, configuración
├─────────────────────────────────────────┤
│  Entities (DTOs/Views)                  │  ← Transferencia de datos
├─────────────────────────────────────────┤
│  Services (Lógica de Negocio)           │  ← Validaciones, reglas de dominio
├─────────────────────────────────────────┤
│  Data (Acceso a Datos)                  │  ← Repositorios, EF Core, Dapper
├─────────────────────────────────────────┤
│  DataModel (Modelo de Base de Datos)    │  ← Entidades que mapean a tablas
└─────────────────────────────────────────┘
```

**Regla crítica**: NUNCA saltar capas (ej: no llamar repositorios desde endpoints)

## Estructura de Carpetas Estándar

```
[Proyecto].Api/
├── Program.cs                    # Bootstrapping, DI, middleware
├── Endpoints/
│   ├── Endpoints.cs             # Endpoints personalizados
│   └── Base/Generator/          # Endpoints auto-generados (NO MODIFICAR)
├── Extensions/
│   ├── DependencyInjection.cs   # Registro de servicios/repositorios
│   ├── AuthConfiguration.cs     # JWT, autenticación
│   └── SwaggerConfiguration.cs  # OpenAPI
└── Security/                    # UserContext, claims mapping

[Proyecto].DataModel/
└── [Entidad].cs                 # Clases POCO que mapean a BD (implementan IEntityBase)

[Proyecto].Entities/
├── Views/                       # DTOs generados
│   ├── [Entidad]View.cs        # Auto-generado (NO MODIFICAR directamente)
│   └── Metadata/               # Atributos de validación
└── PartialViews/               # Extensiones personalizadas ([Entidad]View.Custom.cs)

[Proyecto].Data/
├── EntityModel.cs               # DbContext de EF Core
└── Repository/
    ├── [Entidad]Repository.cs   # Implementación concreta
    └── Interfaces/              # Contratos de repositorios

[Proyecto].Services/
├── [Entidad]Service.cs          # Lógica de negocio
├── ServiceConsts.cs             # Constantes de validación
└── Base/                        # Servicios del framework (seguridad, attachments)
```

## Convenciones de Código (OBLIGATORIAS)

### Nomenclatura de Clases
| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| Entidad | PascalCase singular | `Worker`, `Invoice` |
| View | `[Entidad]View` | `WorkerView` |
| Servicio | `[Entidad]Service` | `WorkerService` |
| Repositorio | `[Entidad]Repository` | `WorkerRepository` |
| Interfaz | `I[Nombre]` | `IWorkerService` |
| Constantes | `[Contexto]Consts` | `ServiceConsts` |

### Nomenclatura en Código C#
```csharp
// ✅ CORRECTO
public class WorkerService : BaseService<WorkerView, Worker, WorkerViewMetadata>
{
    private readonly IWorkerRepository _workerRepository;  // ✅ Prefijo _ para privados
    private readonly ILogger<WorkerService> _logger;       // ✅ camelCase con _
    
    public WorkerService(                                  // ✅ Parámetros en camelCase
        IApplicationContext applicationContext,
        IUserContext userContext,
        IWorkerRepository repository)
        : base(applicationContext, userContext, repository)
    {
        _workerRepository = repository;
    }
    
    public override async Task ValidateView(               // ✅ PascalCase para métodos
        HelixValidationProblem validations,
        WorkerView? view,
        EnumActionType actionType,
        string? configurationName = null)
    {
        if (view != null)
        {
            var minimumAge = 18;                           // ✅ Variables locales camelCase
            // Validaciones personalizadas
        }
        
        await base.ValidateView(validations, view, actionType, configurationName);  // ✅ SIEMPRE llamar a base
    }
}

// ❌ INCORRECTO
public class WorkerService
{
    private IWorkerRepository workerRepository;  // ❌ Falta prefijo _
    
    public WorkerService(IWorkerRepository WorkerRepository)  // ❌ Parámetro en PascalCase
    {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malbert-infoport/AI4Devs](https://github.com/malbert-infoport/AI4Devs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
