---
trigger: always_on
description: Quando receber este prompt @UltraThink, ative o modo de **análise profunda**:
---

# CLAUDE.md - Instruções Detalhadas para Claude

## 🧠 Modo de Operação: @UltraThink

Quando receber este prompt @UltraThink, ative o modo de **análise profunda**:

1. **PARE** e pense profundamente
2. **ANALISE** o problema por múltiplos ângulos
3. **CONSIDERE** edge cases e corner cases
4. **PLANEJE** a solução completa
5. **VERIFIQUE** todas as dependências
6. **EXECUTE** com precisão

### Processo @UltraThink:
```
1. ENTENDER o problema real
2. DECOMPOR em partes menores
3. IDENTIFICAR riscos e complexidade
4. PESQUISAR soluções disponíveis
5. DESIGN a arquitetura
6. IMPLEMENTAR gradualmente
7. TESTAR exaustivamente
8. DOCUMENTAR adequadamente
```

## 🏗️ Estrutura do Projeto Detalhada

### Apps (Aplicações)
```typescript
// apps/web - Frontend Nuxt 4
export interface WebConfig {
  ssr: true
  typescript: {
    strict: true
    typeCheck: true
  }
  modules: [
    '@nuxtjs/kit',
    '@nuxt/image',
    '@pinia/nuxt',
    '@vueuse/nuxt'
  ]
}

// apps/api - Backend Nitro
export interface ApiConfig {
  runtimeConfig: {
    database: DatabaseConfig
    auth: AuthConfig
    external: ExternalConfig
  }
  nitro: {
    experimental: {
      openAPI: true
    }
  }
}
```

### Packages (Bibliotecas)
```typescript
// packages/ui - Componentes Vue
// packages/types - Tipos TypeScript compartilhados
// packages/config - Configurações ESLint/Prettier
// packages/utils - Utilitários comuns
```

### Database (Drizzle ORM)
```typescript
// drizzle/schema/ - Definições de tabelas
// drizzle/migrations/ - Migrações versionadas
// drizzle/seed/ - Dados de teste
```

## 📦 Comandos pnpm Disponíveis

### Scripts Principais
```bash
# Desenvolvimento
pnpm dev                  # Inicia todos os apps em paralelo
pnpm dev:web             # Frontend (porta 3000)
pnpm dev:api             # Backend API (porta 3001)
pnpm dev:docs            # Documentação (porta 3002)

# Build e Produção
pnpm build               # Build all packages/apps
pnpm build:web          # Build frontend
pnpm build:api          # Build backend
pnpm preview            # Preview build

# Database (Drizzle)
pnpm db:generate        # Gerar migrations
pnpm db:migrate         # Aplicar migrations
pnpm db:push            # Push schema para DB
pnpm db:seed            # Popular database
pnpm db:studio          # Interface visual DB
pnpm db:reset           # Reset completo

# Testes
pnpm test               # Unit tests (Vitest)
pnpm test:e2e          # E2E tests (Playwright)
pnpm test:coverage     # Coverage report
pnpm test:ui           # Test UI

# Linting e Formatação
pnpm lint               # ESLint all
pnpm lint:fix           # ESLint fix
pnpm format             # Prettier format
pnpm typecheck          # TypeScript check

# Utilitários
pnpm clean              # Limpar dist/
pnpm clean:all          # Limpar tudo (node_modules, dist, .cache)
pnpm deps:update        # Update dependencies
```

### Scripts Monorepo
```bash
# Filtrar por package
pnpm --filter web build
pnpm --filter api dev
pnpm --filter ui test

# Parallel execution
pnpm dev --parallel
pnpm build --parallel

# Affected (Turbo)
pnpm build --filter=[origin/main]
```

## 🔄 Workflow de Desenvolvimento

### 1. Setup Inicial
```bash
# 1. Clone e install
git clone <repo>
pnpm install

# 2. Configure env
cp .env.example .env
# Editar .env com suas credenciais

# 3. Setup database
pnpm db:generate
pnpm db:migrate
pnpm db:seed

# 4. Start dev
pnpm dev
```

### 2. Desenvolvimento Diário
```bash
# Start dev (parallel)
pnpm dev

# Em outro terminal
pnpm test --watch        # Watch mode
pnpm lint --watch        # Watch lint

# Antes de commit
pnpm typecheck
pnpm test
pnpm lint:fix
pnpm format
```

### 3. Criando Novo Recurso
```bash
# 1. Criar branch
git checkout -b feature/novo-recurso

# 2. Gerar migration (se necessário)
pnpm db:generate --name add-novo-recurso

# 3. Desenvolver
pnpm dev

# 4. Testar
pnpm test
pnpm test:e2e

# 5. Commit
git add .
git commit -m "feat: add novo recurso"
```

### 4. Deploy
```bash
# Build all
pnpm build

# Preview
pnpm preview

# Deploy (exemplo Vercel)
vercel --prod
```

## 🎯 Mega-Prompts com Instruções

### Mega-Prompt 1: Novo Módulo CRUD
```markdown
"Crie um módulo CRUD completo para [ENTIDADE]:

INCLUIR:
✓ Schema Drizzle (drizzle/schema/)
✓ Migration (drizzle/migrations/)
✓ API endpoints (apps/api/server/api/)
✓ Tipos TypeScript (packages/types/)
✓ Página Vue (apps/web/pages/)
✓ Componentes UI (packages/ui/)
✓ Composables (apps/web/composables/)
✓ Validação Zod (apps/web/validation/)
✓ Testes unitários (test/)
✓ Testes E2E (e2e/)
✓ Documentação (docs/)

ARQUIVOS OBRIGATÓRIOS:
- Schema: drizzle/schema/entidade.ts
- Migration: drizzle/migrations/xxx_add_entidade.sql
- API: apps/api/server/api/entidade/index.get.ts
- API: apps/api/server/api/entidade/index.post.ts
- API: apps/api/server/api/entidade/[id].get.ts
- API: apps/api/server/api/entidade/[id].put.ts
- API: apps/api/server/api/entidade/[id].delete.ts
- Tipos: packages/types/entidade.ts
- Página: apps/web/pages/entidade/index.vue
- Form: apps/web/pages/entidade/form.vue
- Test: test/entidade.test.ts
- E2E: e2e/entidade.spec.ts
- Docs: docs/entidade.md"
```

### Mega-Prompt 2: Setup Projeto
```markdown
"Configure projeto completo Nuxt 4 enterprise:

CONFIGURAR:
✓ Nuxt 4 + TypeScript strict

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neoand) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
