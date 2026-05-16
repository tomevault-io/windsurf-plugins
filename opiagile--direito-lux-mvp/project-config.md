---
trigger: always_on
description: O Direito Lux é uma plataforma SaaS para monitoramento automatizado de processos jurídicos, integrada com a API DataJud do CNJ, oferecendo notificações multicanal e análise com IA.
---

# Contexto para Claude - Projeto Direito Lux

## 🎯 Sobre o Projeto

O Direito Lux é uma plataforma SaaS para monitoramento automatizado de processos jurídicos, integrada com a API DataJud do CNJ, oferecendo notificações multicanal e análise com IA.

## 🏗️ Arquitetura

- **Microserviços** em Go (Hexagonal Architecture)
- **Event-Driven** com RabbitMQ
- **Multi-tenant** com isolamento por schema PostgreSQL
- **Cloud-native** para GCP com Kubernetes
- **AI Service** em Python para análises

## 📋 Processo de Desenvolvimento

### 🚨 **REGRA FUNDAMENTAL - DOCUMENTAÇÃO CONTÍNUA OBRIGATÓRIA**

**⚠️ CRÍTICO**: NUNCA DESENVOLVER SEM DOCUMENTAR EM TEMPO REAL

**Durante QUALQUER desenvolvimento, é OBRIGATÓRIO**:

1. **📝 Criar STATUS por microserviço**: `services/[nome-service]/STATUS_[NOME]_SERVICE.md`
2. **⏰ Atualizar documentação a cada 2 horas** de desenvolvimento
3. **📊 Documentar TODA decisão técnica** e problema resolvido
4. **📈 Manter STATUS_IMPLEMENTACAO.md sempre atualizado**
5. **🔄 Evitar perda de contexto** entre sessões de desenvolvimento

**📋 Template**: Consultar `PROCESSO_DOCUMENTACAO.md` no diretório `documentos_direito_lux_contexto_completo/`

**🔥 SEM DOCUMENTAÇÃO = SEM DESENVOLVIMENTO!**

### 🔄 Ao Finalizar Cada Módulo/Serviço

**IMPORTANTE**: Sempre atualizar a documentação após implementar qualquer componente!

1. **🗄️ MIGRATIONS E DATABASE (OBRIGATÓRIO)**
   - **CRÍTICO**: Executar `./scripts/setup-staging-database.sh` após qualquer novo serviço
   - Verificar que todas as tabelas necessárias foram criadas
   - Testar endpoints críticos do serviço para confirmar funcionamento
   - **PROBLEMA RESOLVIDO**: Colunas faltantes em tabelas não ocorrerão mais

2. **STATUS_IMPLEMENTACAO.md**
   - Mover item de "O que Falta" para "O que está Implementado"
   - Atualizar percentual de progresso
   - Adicionar detalhes do que foi implementado

3. **README.md**
   - Atualizar seção "Status do Projeto"
   - Adicionar URLs de desenvolvimento
   - Atualizar comandos úteis

4. **SETUP_AMBIENTE.md**
   - Adicionar instruções de setup do novo módulo
   - Incluir novas variáveis de ambiente
   - Documentar troubleshooting

5. **Documentação do Módulo**
   - Criar README.md específico no diretório do serviço
   - Documentar APIs e eventos
   - Incluir exemplos de uso

### 📝 Padrões de Código

- **Go**: Seguir template em `template-service/`
- **Comentários**: Sempre em português
- **Commits**: Conventional Commits
- **Testes**: Mínimo 80% coverage
- **APIs**: Documentar com Swagger/OpenAPI
- **Snippets**: Máximo 40 linhas por vez

### 🚀 Comandos Importantes

```bash
# Setup completo de database (EXECUTAR SEMPRE)
./scripts/setup-staging-database.sh

# Criar novo serviço
./scripts/create-service.sh nome-service

# Rodar migrações individuais
cd services/[nome-service]
migrate -path migrations -database "postgres://..." up

# Executar testes
make test
make test-coverage
```

## 📊 Status Atual (Atualizado 14/07/2025)

- ✅ **Implementado (100% do projeto - STAGING FUNCIONAL)**: 
  - Documentação completa (visão, arquitetura, roadmap)
  - Event Storming e Domain Modeling
  - Docker Compose com 15+ serviços
  - Template de microserviço Go
  - **✅ 10 Microserviços Core 100% funcionais**: Auth, Tenant, Process, DataJud, Notification, AI, Search, MCP, Report, **Billing**
  - **Frontend Next.js 14 completo** - CRUD processos, busca, billing, dashboard
  - **Infrastructure completa**: K8s, Terraform, CI/CD GitHub Actions
  - **✅ STAGING DEPLOY COMPLETO** - Sistema online em https://35.188.198.87
  - **✅ GKE Cluster operacional** - 6 nodes no GCP funcionando
  - **✅ Bot Telegram TOTALMENTE funcional** - @direitolux_staging_bot
  - **✅ GitHub Secrets implementado** - Solução profissional
  - **✅ Gateways de pagamento** - ASAAS + NOWPayments configurados
  - **✅ Email corporativo** - contato@direitolux.com.br funcionando
  - **✅ DNS configurado** - staging.direitolux.com.br apontando para GCP
  - **✅ Documentação de segredos** - SECRETS_DOCUMENTATION.md
  - **✅ Scripts de automação** - Setup e deploy automatizados
  
- 🎉 **DEBUGGING SESSION COMPLETA (09/07/2025)**: 
  - ✅ **Auth Service** - Hash bcrypt corrigido, login 100% funcional
  - ✅ **DataJud Service** - Todos erros de compilação resolvidos (domain types, UUID conversion, mock client)
  - ✅ **Notification Service** - Dependency injection Fx corrigida, rotas funcionais
  - ✅ **Search Service** - Bug dependency injection resolvido
  - ✅ **MCP Service** - Compilação corrigida
  - ✅ **RESULTADO**: 9/9 serviços 100% operacionais (era 6/9)

- 💰 **BILLING SERVICE IMPLEMENTADO (11/07/2025 - MARCO CRÍTICO)**:
  - ✅ **Sistema Multi-Gateway** - ASAAS + NOWPayments integrados
  - ✅ **8+ Criptomoedas** - BTC, XRP, XLM, XDC, ADA, HBAR, ETH, SOL
  - ✅ **Trial 15 dias** - Sistema completo implementado
  - ✅ **Emissão NF-e** - Automática para Curitiba/PR
  - ✅ **Webhooks** - Confirmações automáticas de pagamento
  - ✅ **20+ APIs** - Endpoints REST completos
  - ✅ **Docker Integration** - Porta 8089 funcionando
  - ✅ **RESULTADO**: 10/10 serviços 100% operacionais

- 🚀 **DATAJUD API REAL ATIVADA (09/07/2025 - MARCO HISTÓRICO)**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opiagile/direito-lux-mvp](https://github.com/opiagile/direito-lux-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
