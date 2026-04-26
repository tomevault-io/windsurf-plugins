---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Contexto do Projeto

Esta é uma POC do **PilotoDeVendas.IA** - uma aplicação SaaS para automação de vendas via WhatsApp com IA. A POC foca em validar a arquitetura de autenticação: backend Python (FastAPI) + frontend React (Vite) + Postgres.

## Visão do Produto Completo

**PilotoDeVendas.IA** é uma plataforma SaaS de automação de vendas via WhatsApp com IA para pequenos e médios empresários.

### Modos de Operação
- **Modo Piloto**: IA assume a negociação de forma autônoma (qualificação de leads SDR + vendas)
- **Modo Copiloto**: Auxilia vendedores humanos em tempo real com sugestões de respostas

### Capacidades Principais
- 💬 **WhatsApp**: Integração oficial (WABA) e não-oficial (Z-API, Evolution API)
- 🔄 **CRM Sync**: Sincronização em tempo real via webhooks (Make)
- 📚 **RAG**: Consulta base de conhecimento da empresa para embasar respostas
- 🎙️ **Áudio/Voz**: Processa e responde áudio, clona voz de sócios/vendedores
- 📄 **Documentos**: Interpreta imagens, PDF, DOCX
- 📅 **Follow-up**: Régua customizável quando lead não responde
- 🧠 **Aprendizado**: Delega dúvidas para humano e aprende (não repete perguntas)
- 📊 **Analytics**: Dashboards de performance do piloto
- 🔍 **Auditoria**: Classifica atendimentos humanos e extrai insights
- 🔗 **Integrações**: Google Calendar, CRMs, Stripe, centenas de ferramentas

### Stack Backend Existente (MVP em Produção)
- **Python 3.12** + **FastAPI** (async) + **LangChain/LangGraph** (LLM orchestration)
- **SQLAlchemy ORM** + **PostgreSQL** (CloudSQL na GCP)
- **WhatsApp**: Z-API (não-oficial), Evolution API (container), WABA (oficial)
- **CRM Sync**: Webhooks no Make (endpoints configurados como tools do agente)
- **Infra GCP**: CloudRun, CloudSQL, GTasks, Terraform

**Importante**: O backend Python já está em produção e é o **Source of Truth** para:
- LLM, RAG, get/send WhatsApp messages
- Propriedade do banco de dados (Postgres)

## Contexto da Equipe e Constraints

### Equipe
- **Atual**: 1 dev (expertise Python muito alta, Node/TS/React básico)
- **Em 2 meses no max**: +1 fullstack junior
- **Ferramenta**: Claude Code MAX ilimitado (acelera desenvolvimento, inclusive em techs menos dominadas)

### Timeline e Objetivos
- **Objetivo**: MVP frontend operacional em **2 meses** (clientes podem logar e configurar agentes)
- **Prioridade**: KISS (Keep It Simple Stupid) - entregar rápido, iterar depois
- **Capacitação**: Sem problemas em aprender novas tecnologias, mas custo-benefício e tempo são fatores críticos

### Preferências de Infraestrutura
- **Filosofia**: Soluções versáteis que exigem **menos ops**, mesmo que custem um pouco mais financeiramente
- **Exemplo**: CloudRun (serverless) vs VMs autogerenciadas

### Diretrizes de Desenvolvimento
- **KISS primeiro**: Na dúvida entre simplicidade e boas práticas complexas, tender ao simples (mas consultar em casos críticos)
- **Python first**: Manter stack principal em Python sempre que possível (expertise + código existente)
- **Sem SEO**: App é dashboard interno (não precisa SSR/SEO) - landing page de marketing é separada (NextJS)


## Roadmap e Evolução

### Curto Prazo (2 meses) - **Foco atual**
**Frontend da aplicação:**
- ✅ Autenticação (email/senha + OAuth Google) - POC concluída
- ⏳ Configuração de agentes de IA (prompts, tools, régua de follow-up)
- ⏳ Gestão de RAG (upload de arquivos TXT/PDF + integração Google Drive)
- ⏳ Dashboards de performance dos agentes

### Médio Prazo (4 meses)
**Plataforma completa:**
- Gestão de usuários (vendedores, admins, permissões)
- Interface de atendimento (similar ao WhatsApp Web - usa API oficial ou Evolution container)
- Modo Copiloto (análise em tempo real de atendimentos + sugestões de respostas/procedimentos)
- Logs de atendimentos de vendedores humanos
- Assinaturas e pagamentos (Stripe)
- Integrações nativas com CRMs populares (HubSpot, RD Station, etc.) + Google Calendar

## Arquitetura

### Decisões Técnicas

**Por que Vite (não NextJS)?**
- App é dashboard interno (sem necessidade de SSR/SEO)
- Equipe tem expertise limitada em Node/TS - Vite é mais simples
- Backend Python já existe como API central
- NextJS adicionaria complexidade desnecessária (BFF, deployment extra, learning curve)

**Por que Session-based com cookies HttpOnly (não JWT)?**
- Mais seguro contra XSS (JWT em localStorage é vulnerável)
- Permite revogação instantânea de sessão (crítico para sistema de vendas)
- Mais simples que infraestrutura JWT completa (refresh tokens, rotação, blacklist)

**Por que backend único Python (sem BFF Node)?**
- FastAPI já existente e robusto em produção (LLM, RAG, WhatsApp)
- Python é a stack com maior expertise da equipe
- BFF seria overhead de manutenção/deployment para equipe pequena (1 dev)
- KISS: uma stack, um deploy, um ponto de falha

**Por que Python é a stack principal?**
- **Expertise**: Equipe tem domínio muito alto em Python (vs Node/TS básico)
- **Código existente**: Backend MVP já em produção (LLM orchestration, RAG, WhatsApp integration)
- **Source of Truth**: Postgres é gerenciado pelo backend Python
- **Ecossistema IA**: LangChain, LangGraph, bibliotecas de ML/NLP são Python-first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/italofaguiar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
