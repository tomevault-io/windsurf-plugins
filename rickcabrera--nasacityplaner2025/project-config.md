---
trigger: always_on
description: 1. [Visión General del Proyecto](#visión-general-del-proyecto)
---

# CLAUDE.md - Data Pathways Technical Documentation

## Tabla de Contenidos
1. [Visión General del Proyecto](#visión-general-del-proyecto)
2. [Arquitectura del Sistema](#arquitectura-del-sistema)
3. [Stack Tecnológico](#stack-tecnológico)
4. [Backend - Spring Boot](#backend---spring-boot)
5. [Frontend - React](#frontend---react)
6. [API REST Endpoints](#api-rest-endpoints)
7. [Integración con APIs Externas](#integración-con-apis-externas)
8. [Base de Datos](#base-de-datos)
9. [Configuración y Despliegue](#configuración-y-despliegue)
10. [Guía de Desarrollo](#guía-de-desarrollo)

---

## Visión General del Proyecto

**Data Pathways** es un sistema de análisis y visualización geoespacial diseñado para la Zona Metropolitana de Veracruz que combina datos demográficos, índices de desigualdad social y visualización cartográfica interactiva para apoyar decisiones de planificación urbana.

### Propósito Principal
Identificar oportunidades de expansión urbana con enfoque en:
- Reducción de desigualdad social (IISU - Índice de Inclusión Social Urbana)
- Mejora del acceso a servicios públicos (salud, educación, áreas verdes)
- Optimización de movilidad urbana

### Datos Clave de Veracruz
- **Población**: 939,046 habitantes (Censo INEGI 2020)
- **Índice de Desigualdad**: 4.0/5 (Alto) - Posición 20 de 74 zonas metropolitanas
- **Tiempo promedio al trabajo**: 28.3 minutos
- **Tiempo promedio a escuelas**: 17.6 minutos
- **Uso de transporte público**: 42.9%
- **Uso de automóvil privado**: 35.2%

---

## Arquitectura del Sistema

### Patrón de Arquitectura
El proyecto sigue una arquitectura de **3 capas** (Three-tier architecture):

```
┌─────────────────────────────────────────┐
│         CAPA DE PRESENTACIÓN            │
│    (React + Vite + Mapbox GL JS)       │
│         Puerto: 5173                    │
└────────────────┬────────────────────────┘
                 │ HTTP/REST
┌────────────────▼────────────────────────┐
│         CAPA DE APLICACIÓN              │
│       (Spring Boot 3.5.6)              │
│         Puerto: 8080                    │
│  Controllers → Services → Repositories  │
└────────────────┬────────────────────────┘
                 │ JDBC
┌────────────────▼────────────────────────┐
│         CAPA DE DATOS                   │
│    PostgreSQL (Supabase Cloud)         │
└─────────────────────────────────────────┘
```

### Componentes Principales

#### Backend (Spring Boot)
- **Controllers**: Exponen endpoints REST
- **Services**: Lógica de negocio y orquestación
- **Repositories**: Acceso a datos (JPA)
- **DTOs**: Objetos de transferencia de datos
- **Entities**: Modelos JPA mapeados a PostgreSQL
- **Clients**: Integración con APIs externas (WorldPop)

#### Frontend (React)
- **Components**: UI reutilizables (Dashboard, Map, Cards)
- **Hooks**: Lógica personalizada (useMapbox)
- **Context**: Estado global (MapContext)
- **Pages**: Vistas principales (LandingPage, Dashboard)

---

## Stack Tecnológico

### Backend
| Tecnología | Versión | Propósito |
|-----------|---------|-----------|
| Spring Boot | 3.5.6 | Framework principal |
| Java | 17 | Lenguaje de programación |
| Spring Data JPA | 3.5.6 | ORM y persistencia |
| PostgreSQL | 16+ | Base de datos relacional |
| Maven | 3.9+ | Gestión de dependencias |
| TwelveMonkeys ImageIO | 3.12.0 | Conversión TIFF → PNG |
| Jackson | 2.17+ | Procesamiento JSON |

### Frontend
| Tecnología | Versión | Propósito |
|-----------|---------|-----------|
| React | 19.1.1 (19.2.0 en root) | Framework UI |
| TypeScript | 5.8.3 | Tipado estático |
| Vite | 7.1.7 (7.1.9 en root) | Build tool y dev server |
| Tailwind CSS | 4.1.14 | Framework de estilos |
| Mapbox GL JS | 3.15.0 | Mapas interactivos |
| Lucide React | 0.544.0 | Iconos |

**Nota**: Las versiones entre paréntesis son las del package.json raíz (monorepo), usadas como meta-dependencies. Las versiones principales son las del front-city-planner/package.json.

---

## Backend - Spring Boot

### Estructura de Directorios

```
BackCityPlanner/
├── src/main/java/com/daffidev/backcityplanner/
│   ├── BackCityPlannerApplication.java
│   ├── controllers/
│   │   ├── MapController.java             # WorldPop endpoints
│   │   ├── ImageController.java           # Image URL endpoints
│   │   └── TestingController.java
│   ├── services/
│   │   ├── MapService.java                # Orchestration
│   │   ├── WorldPopClient.java            # External API client
│   │   ├── TiffConverter.java             # Image conversion
│   │   └── GraficoService.java            # DB operations
│   ├── repositories/
│   │   ├── GraficoRepository.java
│   │   └── IMapRepository.java
│   ├── entities/
│   │   └── Grafico.java                   # Population graphics
│   └── dto/
│       └── PopulationImageDto.java
└── src/main/resources/
    └── application.properties
```

### Componentes Clave

#### Controllers

**MapController.java** (`/api/worldpop/`)
- `GET /` - Obtiene datos de población por ISO3
- `GET /files` - Obtiene URLs de archivos WorldPop
- `GET /tiff/convert?url=` - Convierte TIFF a PNG desde URL
- `POST /tiff/upload` - Convierte TIFF a PNG desde archivo subido

**ImageController.java** (`/api/images`)
- `GET /map?iso3=MEX` - Obtiene URLs de imágenes de densidad poblacional

#### Services


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RickCabrera/NasaCityPlaner2025](https://github.com/RickCabrera/NasaCityPlaner2025) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
