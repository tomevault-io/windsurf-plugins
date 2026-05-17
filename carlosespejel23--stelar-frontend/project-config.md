---
trigger: always_on
description: Frontend de Stellar, la plataforma SaaS edtech para docentes en México. Construido con React + TypeScript + MUI, basado en un paquete de componentes comercial que incluye dashboard, páginas y componentes pre-construidos.
---

# CLAUDE.md — Stellar Frontend

## Identidad

Frontend de Stellar, la plataforma SaaS edtech para docentes en México. Construido con React + TypeScript + MUI, basado en un paquete de componentes comercial que incluye dashboard, páginas y componentes pre-construidos.

---

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Framework | React 18+ con TypeScript |
| UI Library | MUI (Material UI) v5/v6 |
| Routing | React Router v6 |
| HTTP Client | Axios (ya configurado en el paquete) |
| State Management | React Context |
| Formularios | React Hook Form + Yup/Zod (para validaciones) |
| Idioma de la UI | Español (México) |

---

## Arquitectura del frontend

### Estructura de carpetas

```
frontend/src/
├── app.tsx                         # Componente principal de la app
├── main.tsx                        # Entry point (ReactDOM.render)
├── global-config.ts                # Configuración global (API URL, etc.)
├── global.css                      # Estilos globales
├── vite-env.d.ts                   # Tipos de Vite
│
├── _mock/                          # Datos mock para desarrollo
│   ├── _user.ts, _overview.ts...   # Mock data por dominio
│   └── index.ts
│
├── assets/                         # Assets estáticos
│   ├── data/                       # Datos estáticos (countries.ts, etc.)
│   ├── icons/                      # Iconos SVG como componentes React
│   └── illustrations/              # Ilustraciones SVG (error pages, onboarding, etc.)
│
├── auth/                           # Módulo de autenticación (contexto, guards, vistas)
│   ├── components/                 # Componentes de UI de auth (form-head, form-divider, etc.)
│   ├── context/
│   │   ├── auth-context.tsx        # Context base de autenticación
│   │   └── jwt/                    # Implementación JWT (provider, actions, utils)
│   │       ├── auth-provider.tsx   # JWT AuthProvider principal
│   │       ├── action.ts           # Acciones de auth (login, register, logout)
│   │       └── utils.ts            # Utilidades JWT (decode, isExpired, etc.)
│   ├── guard/                      # Guards de ruta
│   │   ├── auth-guard.tsx          # Redirige a login si no autenticado
│   │   ├── guest-guard.tsx         # Redirige a dashboard si ya autenticado
│   │   └── role-based-guard.tsx    # Verifica roles/permisos
│   ├── hooks/                      # Hooks de auth
│   │   ├── use-auth-context.ts     # Hook para acceder al AuthContext
│   │   └── use-mocked-user.ts     # Hook de usuario mock (dev)
│   ├── types.ts                    # Tipos de autenticación
│   ├── utils/
│   │   └── error-message.ts        # Formateo de errores de auth
│   └── view/
│       └── jwt/                    # Vistas de auth JWT
│           ├── jwt-sign-in-view.tsx
│           └── jwt-sign-up-view.tsx
│
├── components/                     # Componentes reutilizables del paquete
│   ├── animate/                    # Animaciones (Framer Motion)
│   │   ├── motion-container.tsx, motion-viewport.tsx, motion-lazy.tsx
│   │   ├── scroll-progress/       # Barra de progreso de scroll
│   │   └── variants/              # Variantes de animación (fade, slide, zoom, etc.)
│   ├── custom-popover/             # Popover personalizado
│   ├── file-thumbnail/             # Preview de archivos
│   ├── flag-icon/                  # Iconos de banderas
│   ├── hook-form/                  # Integración React Hook Form + MUI
│   │   ├── form-provider.tsx       # FormProvider wrapper
│   │   ├── rhf-text-field.tsx      # TextField controlado por RHF
│   │   ├── rhf-select.tsx          # Select controlado por RHF
│   │   ├── rhf-checkbox.tsx        # Checkbox controlado por RHF
│   │   ├── rhf-autocomplete.tsx    # Autocomplete controlado por RHF
│   │   ├── rhf-date-picker.tsx     # DatePicker controlado por RHF
│   │   ├── rhf-radio-group.tsx     # RadioGroup controlado por RHF
│   │   ├── rhf-switch.tsx          # Switch controlado por RHF
│   │   ├── rhf-slider.tsx          # Slider controlado por RHF
│   │   ├── rhf-rating.tsx          # Rating controlado por RHF
│   │   └── schema-utils.ts        # Utilidades de validación (Zod)
│   ├── iconify/                    # Sistema de iconos (Iconify)
│   ├── label/                      # Componente Label (badges/chips estilizados)
│   ├── loading-screen/             # Pantallas de carga y splash
│   ├── logo/                       # Logo de la app
│   ├── nav-section/                # Componentes de navegación
│   │   ├── components/             # Subcomponentes (collapse, dropdown, subheader)
│   │   ├── horizontal/             # Navegación horizontal
│   │   ├── mini/                   # Navegación mini/colapsada
│   │   ├── vertical/              # Navegación vertical (sidebar)
│   │   └── styles/                 # Estilos y CSS vars de nav
│   ├── progress-bar/               # Barra de progreso (top of page)
│   ├── scrollbar/                  # Scrollbar personalizado
│   ├── search-not-found/           # Estado vacío de búsqueda
│   ├── settings/                   # Panel de configuración visual

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlosespejel23/stelar_frontend](https://github.com/carlosespejel23/stelar_frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
