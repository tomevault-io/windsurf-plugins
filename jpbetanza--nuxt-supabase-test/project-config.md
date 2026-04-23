---
trigger: always_on
description: Este documento serve como referência para AIs desenvolvendo novas funcionalidades neste aplicativo Nuxt.js que utiliza Nuxt UI como sistema de componentes e Supabase como banco de dados e autenticador.
---

# Guia de Desenvolvimento - Nuxt.js + Nuxt UI + Supabase

Este documento serve como referência para AIs desenvolvendo novas funcionalidades neste aplicativo Nuxt.js que utiliza Nuxt UI como sistema de componentes e Supabase como banco de dados e autenticador.

## 📋 Visão Geral do Projeto

**Stack Tecnológica:**
- **Nuxt 4** - Framework Vue.js full-stack com SSR/SSG
- **Nuxt UI** - Biblioteca de componentes acessíveis e responsivos
- **Supabase** - Backend-as-a-Service (banco de dados PostgreSQL + autenticação)
- **Stripe** - Plataforma de pagamentos via módulo `@unlok-co/nuxt-stripe`
- **TypeScript** - Tipagem estática
- **Tailwind CSS** - Framework CSS (integrado ao Nuxt UI)
- **ESLint** - Linting e formatação de código

**Estrutura do Projeto:**
```
app/
├── app.vue           # Layout principal da aplicação
├── app.config.ts     # Configurações do Nuxt UI
├── pages/            # Páginas da aplicação (file-based routing)
├── components/       # Componentes reutilizáveis
└── assets/css/       # Estilos customizados

nuxt.config.ts        # Configurações do Nuxt
package.json          # Dependências e scripts
```

## 🔧 Servidores MCP Disponíveis

**IMPORTANTE:** Sempre utilize os servidores MCP disponíveis para entender o comportamento dos frameworks antes de implementar qualquer funcionalidade.

### 1. MCP Supabase (`mcp_supabase_*`)
- **Uso:** Consultas ao banco de dados, autenticação, gerenciamento de projetos
- **Quando usar:**
  - Antes de implementar funcionalidades que envolvam dados
  - Para verificar estrutura de tabelas e políticas RLS
  - Para entender configurações de autenticação
  - Para executar queries e migrations

**Comandos essenciais:**
- `mcp_supabase_list_tables` - Ver estrutura das tabelas
- `mcp_supabase_execute_sql` - Executar queries SQL
- `mcp_supabase_apply_migration` - Aplicar migrations
- `mcp_supabase_get_advisors` - Verificar vulnerabilidades e performance

### 2. MCP Stripe (`mcp_stripe_*`)
- **Uso:** Documentação oficial do Stripe, integração de pagamentos, produtos Stripe
- **Quando usar:**
  - Antes de implementar funcionalidades de pagamento
  - Para entender produtos Stripe (Payments, Billing, Connect, etc.)
  - Para verificar métodos de pagamento disponíveis
  - Para consultar APIs e exemplos de implementação
  - Para entender fluxos de pagamento e melhores práticas

**Comandos essenciais:**
- `mcp_stripe_search_stripe_documentation` - Buscar documentação oficial do Stripe
- `mcp_stripe_get_stripe_account_info` - Informações da conta Stripe conectada
- `mcp_stripe_list_customers` - Listar clientes
- `mcp_stripe_list_products` - Listar produtos
- `mcp_stripe_list_prices` - Listar preços
- `mcp_stripe_list_payment_intents` - Listar intents de pagamento
- `mcp_stripe_list_subscriptions` - Listar assinaturas
- `mcp_stripe_retrieve_balance` - Ver saldo da conta

### 3. MCP Nuxt (`mcp_nuxt_*`)
- **Uso:** Documentação e guias do Nuxt
- **Quando usar:**
  - Antes de implementar novas páginas ou rotas
  - Para entender conceitos do Nuxt (SSR, composables, etc.)
  - Para verificar compatibilidade de módulos

**Comandos essenciais:**
- `mcp_nuxt_get_documentation_page` - Documentação específica
- `mcp_nuxt_list_modules` - Ver módulos disponíveis
- `mcp_nuxt_get_getting_started_guide` - Guias de início

### 3. MCP Nuxt UI (`mcp_nuxt-ui_*`)
- **Uso:** Componentes, templates e documentação do Nuxt UI
- **Quando usar:**
  - Antes de criar novos componentes
  - Para verificar props, slots e eventos de componentes
  - Para explorar templates e exemplos

**Comandos essenciais:**
- `mcp_nuxt-ui_list_components` - Listar componentes disponíveis
- `mcp_nuxt-ui_get_component` - Detalhes de um componente específico
- `mcp_nuxt-ui_list_examples` - Exemplos de implementação

## 🚀 Workflow de Desenvolvimento

### 1. Planejamento da Funcionalidade
```bash
# Sempre comece verificando os servidores MCP relevantes

# Para funcionalidades de pagamento:
1. mcp_stripe_search_stripe_documentation (consultar documentação oficial)
2. mcp_nuxt-ui_list_components (ver componentes de UI)
3. mcp_supabase_list_tables (verificar estrutura de dados)

# Para páginas gerais:
1. mcp_nuxt-ui_list_components (ver componentes de formulário)
2. mcp_supabase_list_tables (verificar estrutura de dados)
3. mcp_nuxt_get_documentation_page (entender roteamento)
```

### 2. Implementação
```typescript
// Estrutura típica de uma página
<script setup lang="ts">
// 1. Imports de composables
const supabase = useSupabaseClient()
const user = useSupabaseUser()

// 2. Reatividade
const data = ref([])
const loading = ref(false)

// 3. Funções assíncronas
const fetchData = async () => {
  loading.value = true
  try {
    const { data: result, error } = await supabase
      .from('table_name')
      .select('*')
    if (error) throw error
    data.value = result
  } catch (error) {
    console.error('Erro:', error)
  } finally {
    loading.value = false
  }
}

// 4. Lifecycle hooks
onMounted(() => {
  fetchData()
})
</script>

<template>
  <!-- 5. Template usando componentes Nuxt UI -->
  <UPage>
    <UPageHero title="Minha Página" />

    <UPageSection>
      <!-- Conteúdo da página -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jpbetanza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
