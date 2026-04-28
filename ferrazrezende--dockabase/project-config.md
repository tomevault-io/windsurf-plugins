---
trigger: always_on
description: DockaBase é um clone funcional e simplificado do Supabase, construído com Laravel 13. O objetivo é criar uma plataforma BaaS que fornece:
---

# DockaBase - BaaS Platform (Backend as a Service)

## Visão Geral

DockaBase é um clone funcional e simplificado do Supabase, construído com Laravel 13. O objetivo é criar uma plataforma BaaS que fornece:

- **Database Manager** - Interface visual para gerenciar múltiplos databases PostgreSQL
- **Schema Builder** - Interface visual para criar tabelas e colunas
- **Auth Provider** - Autenticação multi-tenant para usuários finais (OTP-based)
- **Dynamic REST API** - API auto-gerada a partir do schema do banco
- **Realtime** - Websockets com LISTEN/NOTIFY do PostgreSQL
- **Storage** - Abstração S3/MinIO com políticas de acesso

## Estratégia de Distribuição

**Modelo: Single-Tenant Self-Hosted com Múltiplos Databases**

Cada instância do DockaBase pode gerenciar **múltiplos databases PostgreSQL** na mesma instalação (ex: `dev`, `prod`).

### Opções de Deploy

| Modalidade | Descrição |
|------------|-----------|
| **Self-Hosted (Free)** | Cliente roda localmente via Docker |
| **Cloud Managed (Paid)** | Subimos em servidor dedicado para o cliente |

### Implicações Arquiteturais

- **Múltiplos databases por instância:** Uma instalação pode gerenciar N databases
- **Features híbridas:** Features globais com override por database
- **Rotas simplificadas:** `/system/features` ao invés de `/system/projects/{id}/features`
- **Isolamento por database:** Cada database tem suas tabelas, schemas e dados

### Comparação

| Aspecto | DockaBase | Supabase |
|---------|-----------|----------|
| Modelo | Single-tenant, múltiplos databases | Multi-tenant |
| Isolamento | Por database | Por projeto |
| Features | Globais + override por database | Por projeto |

## Stack Tecnológica

| Componente | Tecnologia |
|------------|------------|
| Backend | Laravel 13+ / PHP 8.4+ |
| Performance | Laravel Octane (Swoole) |
| Database | PostgreSQL 16+ |
| Cache | Redis 7+ |
| Queue | RabbitMQ 7+ |
| Frontend | Inertia.js + Vue 3 + Pinia + TypeScript (strict) |
| UI | shadcn-vue + Tailwind CSS 4.x |
| Feature Flags | Laravel Pennant |
| RBAC | Spatie Permission |
| Storage | MinIO (Self-hosted S3) |
| AI Tools | Laravel MCP Server |
| Observabilidade | Laravel Pulse |

## Arquitetura de Acesso - Dois Níveis

O DockaBase separa claramente dois níveis de acesso:

### Nível 1: Aplicação (Spatie RBAC)

Para **System Users** que acessam o painel administrativo do DockaBase.

| Role | Descrição | Permissões |
|------|-----------|------------|
| `super-admin` | Acesso total | Todas as permissões |
| `admin` | Gerencia instância | create-database, manage-credentials, manage-features |
| `manager` | Gerencia usuários | manage-users, view-databases |
| `user` | Usuário comum | view-databases |

**Uso:** Controle de quem pode fazer o que no painel administrativo.

### Nível 2: Database Access (Credentials)

Para **End Users** que acessam databases PostgreSQL (frontend-only apps, DBeaver, etc).

**ATENÇÃO:** Credentials definem permissões de acesso aos databases. O acesso via API será implementado através da Dynamic REST API (Fase 5 - planejada).

```
┌─────────────────────────────────────────────────────────────┐
│                      CREDENTIAL                              │
│  Name: "Dev Team"                                           │
│  Permission: read-write                                     │
├─────────────────────────────────────────────────────────────┤
│  Users:              │  Databases:                          │
│  • alice@company     │  • dev (read-write)                  │
│  • bob@company       │  • prod (read-write)                 │
└─────────────────────────────────────────────────────────────┘
```

| Permissão | Descrição | Operações |
|-----------|-----------|-----------|
| `read` | Apenas leitura | GET |
| `write` | Apenas escrita | POST, PUT, PATCH, DELETE |
| `read-write` | Leitura e escrita | Todas |

**Fluxo de Criação:**
1. Admin cria **Credential** com permissão (R, W, RW) e atrela usuários
2. Admin cria **Database** e atrela credentials
3. Usuários das credentials ganham acesso ao database

**Importante:**
- Usuários são atrelados à Credential na criação da Credential
- Credentials são atreladas ao Database na criação do Database
- Um usuário pode ter múltiplas credentials (access groups diferentes)
- Uma credential pode ser atrelada a múltiplos databases

### Modelo de Dados

```
users ──┐
        ├── credential_user (pivot) ──► credentials
        │                                    │
        │                                    ├── permission: enum(read, write, read-write)
        │                                    │
        └────────────────────────────────────┴──► credential_database (pivot) ──► databases
```

### Endpoints por Nível

| Nível | Prefixo | Autenticação | Uso |
|-------|---------|--------------|-----|
| Aplicação | `/system/*` | Session (web) | Painel admin |
| API | `/api/v1/{database}/*` | Sanctum (token) | End users |

## Design System

### UI Components
Usar **shadcn-vue** (https://www.shadcn-vue.com/) - componentes copiáveis, não dependência npm.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FerrazRezende) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
