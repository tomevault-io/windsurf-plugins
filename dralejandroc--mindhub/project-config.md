---
trigger: always_on
description: MindHub es una plataforma integral de gestión sanitaria que integra múltiples módulos especializados para clínicas y profesionales de la salud.
---

# MindHub - Healthcare Management Platform

## Project Overview

MindHub es una plataforma integral de gestión sanitaria que integra múltiples módulos especializados para clínicas y profesionales de la salud.

## 🚀 ARQUITECTURA ACTUAL - DJANGO BACKEND + REACT CLEAN ARCHITECTURE

### 🏗️ **ARQUITECTURA COMPLETA**

```
┌─ Frontend React/Next.js ──── Vercel (https://mindhub.cloud) - CLEAN ARCHITECTURE
├─ API Proxy Routes ────────── Next.js (/api/*/django/)
├─ Django Backend ──────────── Django REST API (/backend-django/) - TODOS LOS MÓDULOS
├─ Auth Middleware ─────────── Supabase JWT validation
├─ Database ────────────────── Supabase PostgreSQL 
└─ Authentication ──────────── Supabase Auth
```

### 📁 **DOCUMENTACIÓN ARQUITECTÓNICA**

- **Arquitectura APIs**: `docs/architecture/MINDHUB_ARCHITECTURE_MASTER_COMPLETE.md` - 62+ endpoints documentados
- **Esquema Base de Datos**: `docs/architecture/SUPABASE_TABLES_REFERENCE.md` - Estructura exacta
- **Seguridad**: `docs/architecture/MINDHUB_SECURITY_ARCHITECTURE_MASTER.md` - Patrones de seguridad
- **Frontend**: React Clean Architecture (ver principios de desarrollo)
- **Backend**: Django REST Framework con todos los módulos migrados

### URLs de Producción (ACTUALES)

- **Frontend**: https://mindhub.cloud (Vercel)
- **Backend Django**: https://mindhub-django-backend.vercel.app
- **API Proxy**: https://mindhub.cloud/api/*/django/ (Next.js → Django)
- **Database**: Supabase PostgreSQL
- **Auth**: Supabase Auth

### Estado del Deployment

- ✅ **Backend completamente migrado a Django**
- ✅ **Node.js backend movido a legacy-backend**
- ✅ Django REST Framework con autenticación Supabase
- ✅ API proxy routes para integración seamless
- ✅ Sistema híbrido React + Django completamente funcional
- ✅ Todos los módulos (Expedix, Agenda, Resources) en Django
- ✅ Base de datos Supabase PostgreSQL integrada

### 🔐 **SISTEMA DE AUTENTICACIÓN - SUPABASE ÚNICAMENTE**

- **Proveedor**: Supabase Auth (https://supabase.com)
- **Frontend Auth**: `@supabase/auth-helpers-nextjs` con componentes React
- **Backend Auth**: Middleware Supabase en Django REST API
- **Usuario Principal**: Dr. Alejandro (dr_aleks_c@hotmail.com)
- **Funciones**:
  - ✅ Login/Logout automático
  - ✅ JWT tokens para APIs
  - ✅ Gestión de usuarios y sesiones
  - ✅ Row Level Security (RLS) en PostgreSQL
  - ✅ Integración nativa con Next.js
- **URLs de Auth**:
  - Sign In: https://mindhub.cloud/auth/sign-in
  - Sign Up: https://mindhub.cloud/auth/sign-up
  - Dashboard: https://mindhub.cloud/dashboard (post-login)

### Arquitectura del Sistema

```
MindHub-Pro/
├── mindhub/
│   ├── frontend/              # Next.js 14.2.30 + React 18 + TypeScript + Tailwind CSS
│   └── backend-django/        # Django REST API - Backend Principal
└── legacy-backend/            # Node.js backend (DEPRECATED - no usar)
```

### Stack Tecnológico Actual

**Frontend (React/Next.js):**
- Next.js 14.2.30 con App Router
- React 18 con TypeScript
- Tailwind CSS + shadcn/ui components
- Supabase client para auth y operaciones directas
- API proxy routes para Django integration

**Backend (Django REST):**
- Django 5.0.2 + Django REST Framework
- PostgreSQL vía Supabase connection
- Supabase JWT authentication middleware
- CORS configurado para frontend integration
- Modelos Django para todos los módulos (Expedix, Agenda, Resources, ClinimetrixPro)

## 🏥 Módulos Principales (Orden de Interconexión)

### 1. **Agenda** - Sistema de Citas y Programación

- **Frontend URL**: `/hubs/agenda`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/agenda/`
- **Proxy API**: `https://mindhub.cloud/api/agenda/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**
- **Funcionalidades**:
  - Programación de citas médicas - Django scheduling models
  - Gestión de horarios y disponibilidad - Django provider schedules
  - Drag & Drop para reprogramación - Frontend + Django integration
  - Notificaciones automáticas - Django signals
  - Lista de espera inteligente - Django waiting list system
  - Confirmación de citas - Django appointment workflow
  - Integración con Finance para cobros automáticos

### 2. **Expedix** - Gestión de Pacientes y Expedientes Médicos

- **Frontend URL**: `/hubs/expedix`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/expedix/`
- **Proxy API**: `https://mindhub.cloud/api/expedix/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**
- **Funcionalidades**:
  - Gestión completa de pacientes (CRUD) - Django models
  - Expedientes médicos digitales - Django serializers con 33+ campos
  - Sistema de consultas médicas - Django views con examen mental
  - Generación de recetas digitales - Django business logic
  - Historial médico completo - Django relationships
  - Portal de pacientes - Django authentication
  - Documentos médicos encriptados - Django security
  - Integración directa con Agenda para "INICIAR CONSULTA"

### 3. **ClinimetrixPro** - Sistema de Evaluaciones Psicométricas

- **Frontend URL**: `/hubs/clinimetrix`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/clinimetrix/`
- **Proxy API**: `https://mindhub.cloud/api/clinimetrix/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dralejandroc/MINDHUB](https://github.com/dralejandroc/MINDHUB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
