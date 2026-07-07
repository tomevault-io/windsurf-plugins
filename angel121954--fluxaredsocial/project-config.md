---
trigger: always_on
description: Red social para desarrolladores latinoamericanos que combina conceptos de LinkedIn, GitHub y Dev.to.
---

# Fluxa — Reglas del Proyecto

## Descripción del proyecto
Red social para desarrolladores latinoamericanos que combina conceptos de LinkedIn, GitHub y Dev.to.
Proyecto final de grado SENA (programa ADSO). Dominio en evaluación: getfluxa.com / fluxahq.com.

## Filosofía del proyecto

### YAGNI — You Ain't Gonna Need It
No construir para el futuro incierto. Cada línea de código debe resolver un problema **actual**, no uno imaginario. Si una abstracción solo sirve "por si acaso", no se hace.

### KISS — Keep It Simple, Stupid
La solución más simple que funciona es la mejor. No agregar capas de indirección, interfaces, herencia, patrones de diseño o dependencias sin que exista una necesidad real y presente.

### Overengineering está prohibido
- No crear interfaces o contratos hasta que existan **2+ implementaciones concretas** reales
- No crear Services genéricos tipo `BaseService` o `CrudService` — cada Service tiene una responsabilidad única y concreta
- No usar patrones de diseño como decorators, strategies, observers (PHP) a menos que el código actual lo exija, no "por si acaso"
- No abstraer lógica que solo se usa en un lugar
- No añadir repositorios — Eloquent **es** la capa de datos, no necesitas otra
- No crear DTOs a menos que un método reciba 4+ parámetros del mismo tipo
- No usar Traits para "compartir código bonito" — prefiere composición o duplicación controlada (rule of three)
- La duplicación de código es aceptable hasta 3 veces; después de la 3ra ocurrencia, refactoriza

### Principios generales
- **Código explícito > código clever/ingenioso** — Un bloque `if` sencillo vale más que una expresión ternaria anidada de una línea
- **Menos dependencias, mejor** — Cada dependencia nueva es un riesgo de seguridad, mantenimiento y breaking changes. Preguntar siempre ¿podemos hacerlo con lo que ya tenemos?
- **Las reglas están para romperse, pero con permiso** — Si necesitas desviarte de estas convenciones, discútelo antes
- **Cada archivo debe tener una responsabilidad única** — Si no puedes explicar qué hace en una frase, está haciendo demasiado
- **Las vistas Blade no tienen lógica** — Sin condicionales complejos, sin consultas, sin procesamiento de datos. Solo `@if` simples y bucles `@foreach`

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Framework | Laravel 11 + Laravel Sail (Docker + WSL2) |
| PHP mínimo | 8.2 |
| Frontend | Blade + Tailwind CSS 3 + Vanilla JS |
| Reactividad | Livewire 4 |
| WebSockets | Laravel Reverb 1.x (puerto 8080, Supervisor en Docker) |
| Echo cliente | laravel-echo + pusher-js |
| Build | Vite 4 + laravel-vite-plugin |
| Base de datos | MySQL (via Sail) |
| Medios | Cloudinary (cloudinary-labs/cloudinary-laravel 3) |
| Colas | Laravel Queues |
| PDF | Spatie Browsershot 5 + Puppeteer |
| Push notifications | laravel-notification-channels/webpush (VAPID) |
| Auth | Laravel Fortify 1 + Laravel Socialite 5 |
| Códigos QR | endroid/qr-code 5 |
| Rutas JS | ziggy-js 2 |
| Observabilidad | Laravel Telescope 5 (dev) |
| Testing | PestPHP 3 |
| Linting PHP | Laravel Pint |
| Fuente principal | Figtree (Tailwind sans) |
| Fuente monospace | JetBrains Mono (--mono) |

### Paquetes (Composer)

| Paquete | Propósito |
|---------|-----------|
| Laravel Framework | Framework base |
| Laravel Fortify | Autenticación (login, registro, 2FA) |
| Laravel Socialite | OAuth social (GitHub, Google, etc.) |
| Laravel Reverb | WebSockets en tiempo real |
| Laravel Sanctum | API tokens |
| Laravel Telescope | Debug y observabilidad (dev) |
| Laravel Sail | Entorno Docker (dev) |
| Laravel Pint | Linting PHP (dev) |
| Livewire | Framework reactivo del lado del servidor |
| Spatie Browsershot | Generación de PDF con Puppeteer |
| Cloudinary Laravel | Subida y transformación de imágenes |
| Endroid QR Code | Generación de códigos QR |
| Pest PHP | Testing (dev) |
| Guzzle | Cliente HTTP |
| Faker PHP | Generación de datos falsos (dev) |
| PHPUnit | Testing base (dev) |
| Laravel Tinker | REPL interactivo (dev) |

### Paquetes JS (NPM)

| Paquete | Propósito |
|---------|-----------|
| Vite | Build tool y dev server |
| Tailwind CSS | Framework CSS utilitario |
| Axios | Cliente HTTP para el frontend |
| Laravel Echo | Cliente WebSocket (Reverb/Pusher) |
| Pusher JS | Protocolo WebSocket |
| Chart.js | Gráficos interactivos (admin) |
| Ziggy | Generación de rutas Laravel en JS |
| Puppeteer | Headless Chrome (para PDF con Browsershot) |
| Lodash | Utilidades JS |

### Librerías CDN

| Librería | Propósito |
|----------|-----------|
| SweetAlert2 | Alertas y modales personalizados |
| Devicon | Íconos de tecnologías de programación |

### APIs externas

| API | Propósito |
|-----|-----------|
| Cloudinary | Almacenamiento, transformación y entrega de imágenes |
| GIPHY | Búsqueda e inserción de GIFs en mensajería |

## Convenciones de código

### PHP / Laravel
- `declare(strict_types=1)` en **todos** los archivos PHP nuevos
- Type hints y return types en **todos** los métodos, sin excepción
- `readonly` properties donde el valor no cambia después de construcción
- Lógica de negocio en `app/Services/` o `app/Actions/`, **nunca** en controladores
- Observers para efectos secundarios en modelos (ej. sincronización Cloudinary)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Angel121954/fluxaRedSocial](https://github.com/Angel121954/fluxaRedSocial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
