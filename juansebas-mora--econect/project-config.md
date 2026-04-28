---
trigger: always_on
description: EConect es una aplicación Android nativa que conecta ciudadanos comunes con recicladores profesionales.
---

# CLAUDE.md — EConect

## Descripción general del proyecto

EConect es una aplicación Android nativa que conecta ciudadanos comunes con recicladores profesionales.
Los ciudadanos registran material reciclable y su ubicación; los recicladores consultan rutas asignadas,
recogen el material y confirman la entrega con precio en el centro de acopio.

---

## Stack tecnológico

| Componente | Tecnología |
|---|---|
| Lenguaje | Kotlin |
| UI | Jetpack Compose + Material 3 |
| Arquitectura | MVVM + Clean Architecture (capas: UI → ViewModel → Repository → DataSource) |
| Navegación | Navigation Compose con deep links |
| Inyección de dependencias | Hilt |
| Base de datos local | Room |
| Preferencias | DataStore (Proto) |
| Backend / Auth | Firebase Auth (email + contraseña) |
| Base de datos remota | Cloud Firestore |
| Notificaciones push | Firebase Cloud Messaging (FCM) |
| Mapas y geolocalización | Google Maps SDK + Google Places API + FusedLocationProviderClient |
| Chat en tiempo real | Firestore (colección de mensajes con listeners en tiempo real) |
| Tareas en segundo plano | WorkManager |
| Imágenes | Coil |
| Testing | JUnit 4/5 · MockK · Turbine · Compose UI Test |
| Build | Gradle (Kotlin DSL) · compileSdk 35 · minSdk 24 |

---

## Arquitectura MVVM + Clean Architecture

```
app/
├── di/                          # Módulos Hilt (AppModule, FirebaseModule, etc.)
├── data/
│   ├── local/
│   │   ├── db/                  # Room Database, DAOs, Entities
│   │   └── datastore/           # DataStore Proto para preferencias de usuario
│   ├── remote/
│   │   ├── firebase/            # FirebaseAuth, Firestore data sources
│   │   └── fcm/                 # FCM service, token management
│   ├── repository/              # Implementaciones de repositorios
│   └── model/                   # DTOs y mapeadores de datos
├── domain/
│   ├── model/                   # Modelos de dominio (User, Material, Ruta, etc.)
│   ├── repository/              # Interfaces de repositorio
│   └── usecase/                 # Casos de uso (un archivo por caso de uso)
├── presentation/
│   ├── auth/                    # Login, Registro, recuperación de contraseña
│   ├── citizen/
│   │   ├── dashboard/           # Dashboard ciudadano
│   │   ├── material/            # Añadir / ver material reciclable
│   │   └── profile/             # Perfil ciudadano
│   ├── recycler/
│   │   ├── dashboard/           # Dashboard reciclador
│   │   ├── routes/              # Rutas asignadas y mapa
│   │   ├── pickup/              # Recoger material, confirmar cantidad y precio
│   │   └── profile/             # Perfil reciclador
│   ├── shared/
│   │   ├── chat/                # Chat entre reciclador y ciudadano
│   │   ├── notifications/       # Centro de notificaciones
│   │   ├── history/             # Historial de transacciones
│   │   └── rating/              # Sistema de calificaciones
│   └── navigation/              # NavGraph, rutas, argumentos
└── util/                        # Extensions, constantes, formatters
```

### Reglas por capa

- **UI (Composables):** Solo observa StateFlow/LiveData del ViewModel. No tiene lógica de negocio ni acceso directo a repositorios.
- **ViewModel:** Expone `UiState` como `StateFlow<T>`. Llama únicamente a casos de uso o repositorios. No importa clases de `data/`.
- **Domain (UseCases):** Clases con un único método `invoke()`. Sin dependencias de Android framework.
- **Repository:** La interfaz vive en `domain/`; la implementación en `data/`. Decide entre caché local y fuente remota.
- **DataSource:** Acceso directo a Room, Firestore o FCM. Sin lógica de negocio.

---

## Modelos de dominio clave

```kotlin
// Tipos de usuario
enum class UserType { CITIZEN, RECYCLER }

data class User(
    val id: String,
    val name: String,
    val email: String,
    val phone: String,
    val userType: UserType,
    val preferredLocations: List<LatLng>,
    val availableSchedules: List<Schedule>,
    val rating: Float,
    val ratingCount: Int
)

data class RecyclableMaterial(
    val id: String,
    val citizenId: String,
    val type: MaterialType,         // PAPER, PLASTIC, GLASS, METAL, etc.
    val condition: MaterialCondition,
    val quantity: MaterialQuantity, // kg o unidades (cajas, botellas, etc.)
    val pickupLocation: LatLng,
    val status: MaterialStatus,     // AVAILABLE, ASSIGNED, COLLECTED
    val createdAt: Timestamp
)

data class Route(
    val id: String,
    val recyclerId: String,
    val stops: List<RouteStop>,     // Cada stop tiene material + ciudadano + horario
    val date: LocalDate,
    val status: RouteStatus         // PENDING, IN_PROGRESS, COMPLETED
)

data class Transaction(
    val id: String,
    val routeId: String,
    val recyclerId: String,
    val citizenId: String,
    val materialId: String,
    val confirmedQuantity: MaterialQuantity,
    val pricePerUnit: Double,
    val totalAmount: Double,
    val recyclingCenterId: String,
    val completedAt: Timestamp
)

data class ChatMessage(
    val id: String,
    val routeId: String,
    val senderId: String,
    val content: String,
    val timestamp: Timestamp,
    val read: Boolean
)

data class Rating(
    val id: String,
    val fromUserId: String,
    val toUserId: String,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juansebas-mora) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
