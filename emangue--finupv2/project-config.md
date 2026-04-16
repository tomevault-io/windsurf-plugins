---
trigger: always_on
description: - ✅ PRD: `/docs/features/PRD_MOBILE_EXPERIENCE.md` (135+ páginas)
---

# Cursor Rules - Mobile Experience Project

## 📱 Projeto Atual: Mobile Experience V1.0

### Status: PRD + Style Guide Completos
- ✅ PRD: `/docs/features/PRD_MOBILE_EXPERIENCE.md` (135+ páginas)
- ✅ Style Guide: `/docs/features/MOBILE_STYLE_GUIDE.md` (45+ páginas)
- ✅ Index: `/docs/features/MOBILE_INDEX.md` (resumo executivo)
- ⏭️ Próximo: TECH_SPEC (Especificação Técnica)

---

## 🎨 Design System - Imagem "Trackers"

### Paleta de Cores Extraída (SEMPRE usar estas cores)
```typescript
const trackerColors = {
  casa: { bg: '#DDD6FE', icon: '#6B21A8', progress: '#9F7AEA' },      // Roxo
  alimentacao: { bg: '#DBEAFE', icon: '#1E40AF', progress: '#60A5FA' }, // Azul
  compras: { bg: '#FCE7F3', icon: '#BE185D', progress: '#F472B6' },   // Rosa
  transporte: { bg: '#E7E5E4', icon: '#78716C', progress: '#A8A29E' }, // Bege
  contas: { bg: '#FEF3C7', icon: '#D97706', progress: '#FCD34D' },    // Amarelo
  lazer: { bg: '#D1FAE5', icon: '#047857', progress: '#6EE7B7' },     // Verde
};
```

### Dimensões Críticas (SEMPRE usar estas medidas)
```typescript
const dimensions = {
  iconCircle: '48px',         // Ícones circulares
  progressHeight: '6px',      // Barras de progresso
  cardRadius: '16px',         // Border radius cards
  touchTarget: '44px',        // Mínimo WCAG 2.5.5
  screenPadding: '20px',      // Padding lateral
  cardGap: '16px',            // Gap entre cards
};
```

### Tipografia (SEMPRE usar estas classes Tailwind)
```typescript
const typography = {
  pageTitle: 'text-[34px] font-bold leading-tight text-black',
  categoryName: 'text-[17px] font-semibold leading-snug text-black',
  frequency: 'text-[13px] font-normal leading-relaxed text-gray-400',
  amountPrimary: 'text-[17px] font-semibold leading-snug text-black',
  amountSecondary: 'text-[13px] font-normal leading-relaxed text-gray-400',
};
```

---

## 🚨 REGRAS CRÍTICAS (Do copilot-instructions.md)

### 1. Sincronização Git
**SEMPRE:** Local → Git → Servidor  
**NUNCA:** Editar código diretamente no servidor

### 2. Estrutura de Pastas

**REGRA CRÍTICA:** Organização de Features/Projetos

**Estrutura OBRIGATÓRIA para novos projetos:**
```
/docs/features/[nome-projeto]/
├── README.md                    # START HERE - navegação rápida
├── 01-PRD/                      # Product Requirements Document
│   ├── PRD.md                   # Especificação principal
│   ├── STYLE_GUIDE.md          # Design System (se aplicável)
│   ├── FACTIBILIDADE.md        # Análise técnica
│   ├── ANALISE_STAKEHOLDER.md  # Respostas a perguntas específicas
│   ├── SUMMARY.md              # Resumo executivo
│   ├── CHECKLIST.md            # Checklist de implementação
│   └── INDEX.md                # Índice e quick reference
├── 02-TECH_SPEC/               # Especificação Técnica
│   ├── TECH_SPEC.md            # Arquitetura, endpoints, componentes
│   ├── API_SPEC.md             # Especificação de APIs (se necessário)
│   └── DATABASE_CHANGES.md     # Mudanças no banco (se necessário)
└── 03-DEPLOY/                  # Deploy e Produção
    ├── DEPLOY_PLAN.md          # Plano de deploy
    ├── ROLLBACK_PLAN.md        # Plano de rollback
    └── MONITORING.md           # Métricas e monitoramento
```

**Exemplo (Mobile V1.0):**
```
/docs/features/mobile-v1/
├── README.md
├── 01-PRD/
│   ├── PRD.md
│   ├── STYLE_GUIDE.md
│   ├── FACTIBILIDADE.md
│   ├── ANALISE_STAKEHOLDER.md
│   ├── SUMMARY.md
│   ├── CHECKLIST.md
│   └── INDEX.md
├── 02-TECH_SPEC/ (a criar)
└── 03-DEPLOY/ (a criar)
```

**NUNCA:**
- ❌ Criar múltiplos arquivos soltos em `/docs/features/`
- ❌ Arquivos com prefixos como `MOBILE_*`, `PROJETO_*`, etc
- ❌ Duplicar informação em múltiplos arquivos
- ❌ Criar .md na raiz do projeto

**SEMPRE:**
- ✅ 1 pasta por projeto/feature
- ✅ Estrutura 01-PRD, 02-TECH_SPEC, 03-DEPLOY
- ✅ README.md como ponto de entrada
- ✅ Consolidar informações (máximo 7-8 arquivos por pasta PRD)

### 3. Banco de Dados — SEMPRE PostgreSQL via Docker
**SEMPRE usar PostgreSQL** (espelho da VM/produção). **NUNCA** SQLite em dev.
- **Local:** PostgreSQL via Docker — `docker compose up -d postgres` (antes de `quick_start.sh`)
- **Produção/VM:** PostgreSQL em `127.0.0.1:5432/finup_db`
- **URL:** `postgresql://finup_user:finup_password_dev_2026@localhost:5432/finup_db`
- **Como subir:** Docker Desktop aberto → `docker compose up -d postgres` (ou `quick_start.sh` faz isso)

### 4. Arquitetura
- **Backend:** DDD com domínios isolados (`app_dev/backend/app/domains/`)
- **Frontend:** Feature-based (`app_dev/frontend/src/features/`)
- **NUNCA:** Imports cruzados entre domínios ou features

### 5. Versionamento
**SEMPRE:** Usar `version_manager.py` para arquivos críticos  
**NUNCA:** Commitar versões `-dev` ou `-test`

### 6. Migrations
**SEMPRE:** Usar Alembic (`alembic revision --autogenerate`)  
**NUNCA:** Modificar schema SQL diretamente

### 7. Deploy
**FONTE ÚNICA DE VERDADE:** `deploy/` (pasta na raiz do projeto)

```
deploy/
├── README.md                         ← guia mestre
├── scripts/
│   ├── predeploy.sh                  ← RODAR ANTES DE TODO DEPLOY
│   ├── predeploy.py                  ← 22 testes auto + 13 UI (Playwright)
│   ├── deploy_docker_build_local.sh  ← DEPLOY PRINCIPAL
│   ├── deploy_docker_vm.sh           ← alternativo
│   └── validate_deploy.sh
├── validations/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emangue) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
