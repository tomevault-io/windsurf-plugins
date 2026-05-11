---
trigger: always_on
description: Sistema de Punto de Venta (POS) para pequenas y medianas empresas.
---

# CLAUDE.md - Directivas del Proyecto PuntoDeVenta

## Descripcion del Proyecto
Sistema de Punto de Venta (POS) para pequenas y medianas empresas.
Arquitectura multicapa con cliente WPF y API REST.

## Stack Tecnologico Actual
- **Framework**: .NET 8
- **UI Desktop**: WPF (Windows Presentation Foundation) + XAML
- **UI Web**: Blazor WebAssembly + nginx
- **API REST**: ASP.NET Core 8 Web API
- **ORM**: Entity Framework Core 8
- **Base de datos**: PostgreSQL
- **Autenticacion**: JWT Bearer Tokens
- **PDF**: QuestPDF / iTextSharp 5.5.13
- **Graficos**: LiveCharts 0.9.7, Blazor-ApexCharts (Web)
- **Deploy**: Railway (API + Web como servicios separados)
- **Lenguaje**: C#

## Estructura del Proyecto

```
PuntoDeVenta/
├── Capa Entidad/              # Modelos de dominio (CE_*)
│   ├── CE_Usuarios.cs
│   ├── CE_Productos.cs
│   ├── CE_Clientes.cs
│   ├── CE_Grupos.cs
│   └── CE_Movimiento.cs
│
├── Capa Datos/                # Acceso a datos
│   ├── Context/
│   │   └── ApplicationDbContext.cs   # EF Core DbContext
│   ├── Interfaces/            # Contratos
│   │   ├── IRepository.cs
│   │   ├── IUsuarioRepository.cs
│   │   ├── IProductoRepository.cs
│   │   └── IUnitOfWork.cs
│   ├── Repositories/          # Implementaciones
│   │   ├── Repository.cs      # Generico
│   │   ├── UsuarioRepository.cs
│   │   ├── ProductoRepository.cs
│   │   └── UnitOfWork.cs
│   ├── CD_*.cs                # Legacy (ADO.NET)
│   └── ConfigurationHelper.cs # Helper de configuracion
│
├── Capa Negocio/              # Logica de negocio (CN_*)
│
├── PuntoDeVenta/              # Cliente WPF
│   ├── Views/                 # Vistas XAML
│   ├── src/
│   │   ├── Boxes/             # Dialogos modales
│   │   ├── Themes/            # Temas (Dark, Green, Red)
│   │   └── img/               # Imagenes
│   └── Resources/             # Templates PDF
│
├── PuntoDeVenta.API/          # API REST
│   ├── Controllers/           # Endpoints
│   │   ├── AuthController.cs
│   │   ├── ProductosController.cs
│   │   ├── UsuariosController.cs
│   │   ├── ClientesController.cs
│   │   ├── GruposController.cs
│   │   └── MovimientosController.cs
│   ├── DTOs/                  # Data Transfer Objects
│   ├── Auth/
│   │   └── JwtService.cs      # Generacion de tokens
│   └── Program.cs             # Configuracion de la API
│
├── docs/                      # Documentacion tecnica
│   ├── Fase1_TecnologiasNuevas.md
│   ├── Fase2_TecnologiasNuevas.md
│   └── Fase3_TecnologiasNuevas.md
│
├── Query Inicial/             # Scripts SQL PostgreSQL
├── appsettings.json           # Configuracion (NO commitear)
├── appsettings.example.json   # Template de configuracion
└── .gitignore
```

## Convenciones de Nomenclatura

### Clases por Capa
- `CE_*` = Capa Entidad (ej: CE_Usuarios, CE_Productos)
- `CN_*` = Capa Negocio (ej: CN_Usuarios, CN_Productos)
- `CD_*` = Capa Datos Legacy (ej: CD_Usuarios, CD_Productos)
- `I*Repository` = Interfaces de repositorio
- `*Repository` = Implementaciones de repositorio
- `*DTO` = Data Transfer Objects
- `*Controller` = Controladores API

### Archivos
- Views: PascalCase (ej: `Dashboard.xaml`, `POS.xaml`)
- Temas: PascalCase (ej: `Dark.xaml`, `Green.xaml`)
- DTOs: PascalCase con sufijo DTO (ej: `ProductoDTO.cs`)

## Patrones de Diseno

### Repository Pattern
```csharp
// Interfaz
public interface IProductoRepository : IRepository<CE_Productos>
{
    Task<CE_Productos> GetByCodigoAsync(string codigo);
}

// Implementacion
public class ProductoRepository : Repository<CE_Productos>, IProductoRepository
{
    public async Task<CE_Productos> GetByCodigoAsync(string codigo)
    {
        return await _dbSet.FirstOrDefaultAsync(p => p.Codigo == codigo);
    }
}
```

### Unit of Work
```csharp
public interface IUnitOfWork : IDisposable
{
    IUsuarioRepository Usuarios { get; }
    IProductoRepository Productos { get; }
    Task<int> SaveChangesAsync();
}
```

### Dependency Injection
```csharp
// En Program.cs
builder.Services.AddScoped<ApplicationDbContext>();
builder.Services.AddScoped<IUsuarioRepository, UsuarioRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

### DTO Pattern
```csharp
// Entidad (BD)
public class CE_Productos { public byte[] Img; ... }

// DTO (API)
public class ProductoDTO { public bool TieneImagen; ... }
```

## Reglas de Desarrollo

### Seguridad
1. **NUNCA** hardcodear credenciales en el codigo
2. Usar `appsettings.json` para configuracion sensible
3. `appsettings.json` esta en `.gitignore`
4. Usar JWT para autenticacion en la API
5. Las contrasenas en BD estan encriptadas con pgcrypto

### Arquitectura
1. Mantener separacion estricta de capas
2. La Capa Presentacion NO accede directamente a Capa Datos
3. Usar interfaces para abstraer dependencias (DI)
4. La API usa DTOs, nunca expone entidades directamente
5. Validacion en DTOs con DataAnnotations

### Base de Datos
1. EF Core para nuevas funcionalidades
2. Usar async/await para todas las operaciones
3. Los nombres de tablas/columnas usan PascalCase con comillas en PostgreSQL
4. Zona horaria: America/Argentina/Buenos_Aires (UTC-3)
5. Extension pgcrypto para encriptacion de contrasenas

### Mapeo EF Core - PostgreSQL
```csharp
// Los nombres deben coincidir exactamente con el SQL
entity.ToTable("Usuarios");  // No "usuarios"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EmmaVZ89/PointOfSale](https://github.com/EmmaVZ89/PointOfSale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
