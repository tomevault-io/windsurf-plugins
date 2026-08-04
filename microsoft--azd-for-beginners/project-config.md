---
trigger: always_on
description: **Navegação do Capítulo:**
---

# Agentes de IA com Azure Developer CLI

**Navegação do Capítulo:**
- **📚 Início do Curso**: [AZD Para Iniciantes](../../README.md)
- **📖 Capítulo Atual**: Capítulo 2 - Desenvolvimento AI-First
- **⬅️ Anterior**: [Integração Microsoft Foundry](microsoft-foundry-integration.md)
- **➡️ Próximo**: [Implantação de Modelo AI](ai-model-deployment.md)
- **🚀 Avançado**: [Soluções Multi-Agente](../../examples/retail-scenario.md)

---

## Introdução

Agentes de IA são programas autônomos que podem perceber seu ambiente, tomar decisões e executar ações para atingir objetivos específicos. Diferente de simples chatbots que respondem a comandos, agentes podem:

- **Usar ferramentas** - Chamar APIs, buscar em bases de dados, executar código
- **Planejar e raciocinar** - Dividir tarefas complexas em etapas
- **Aprender com o contexto** - Manter memória e adaptar comportamento
- **Colaborar** - Trabalhar com outros agentes (sistemas multi-agente)

Este guia mostra como implantar agentes de IA no Azure usando Azure Developer CLI (azd).

> **Nota de validação (13-07-2026):** Este guia foi revisado com as versões `azd` `1.27.1` e `azure.ai.agents` `1.0.0-beta.5`. A experiência `azd ai` ainda está em pré-visualização, portanto, verifique a ajuda da extensão se suas flags instaladas forem diferentes.

## Objetivos de Aprendizagem

Ao completar este guia, você irá:
- Entender o que são agentes de IA e como eles se diferenciam de chatbots
- Implantar modelos predefinidos de agentes de IA usando AZD
- Configurar Agentes Foundry para agentes personalizados
- Implementar padrões básicos de agentes (uso de ferramentas, RAG, multi-agente)
- Monitorar e depurar agentes implantados

## Resultados do Aprendizado

Ao final, você será capaz de:
- Implantar aplicações de agente IA no Azure com um único comando
- Configurar ferramentas e capacidades do agente
- Implementar geração aumentada por recuperação (RAG) com agentes
- Projetar arquiteturas multi-agente para fluxos de trabalho complexos
- Solucionar problemas comuns em implantações de agentes

---

## 🤖 O Que Torna um Agente Diferente de um Chatbot?

| Recurso | Chatbot | Agente de IA |
|---------|---------|----------|
| **Comportamento** | Responde a comandos | Executa ações autônomas |
| **Ferramentas** | Nenhuma | Pode chamar APIs, buscar, executar código |
| **Memória** | Apenas por sessão | Memória persistente entre sessões |
| **Planejamento** | Resposta única | Raciocínio em múltiplas etapas |
| **Colaboração** | Entidade única | Pode trabalhar com outros agentes |

### Analogia Simples

- **Chatbot** = Uma pessoa prestativa respondendo perguntas em uma mesa de informações
- **Agente de IA** = Um assistente pessoal que pode fazer chamadas, agendar compromissos e completar tarefas para você

---

## 🚀 Início Rápido: Implante Seu Primeiro Agente

### Opção 1: Modelo Foundry Agents (Recomendado)

```bash
# Inicializar o modelo de agentes de IA
azd init --template get-started-with-ai-agents

# Implantar no Azure
azd up
```

**O que é implantado:**
- ✅ Foundry Agents
- ✅ Modelos Microsoft Foundry (gpt-4.1)
- ✅ Azure AI Search (para RAG)
- ✅ Azure Container Apps (interface web)
- ✅ Application Insights (monitoramento)

**Tempo:** ~15-20 minutos
**Custo:** ~$100-150/mês (desenvolvimento)

### Opção 2: Agente OpenAI com Prompty

```bash
# Inicialize o modelo do agente baseado em Prompty
azd init --template agent-openai-python-prompty

# Implantar no Azure
azd up
```

**O que é implantado:**
- ✅ Azure Functions (execução de agente serverless)
- ✅ Modelos Microsoft Foundry
- ✅ Arquivos de configuração Prompty
- ✅ Implementação de agente de exemplo

**Tempo:** ~10-15 minutos
**Custo:** ~$50-100/mês (desenvolvimento)

### Opção 3: Agente RAG de Chat

```bash
# Inicializar modelo de chat RAG
azd init --template azure-search-openai-demo

# Implantar no Azure
azd up
```

**O que é implantado:**
- ✅ Modelos Microsoft Foundry
- ✅ Azure AI Search com dados de exemplo
- ✅ Pipeline de processamento de documentos
- ✅ Interface de chat com citações

**Tempo:** ~15-25 minutos
**Custo:** ~$80-150/mês (desenvolvimento)

### Opção 4: AZD AI Agent Init (Pré-visualização baseadas em Manifesto ou Template)

Se você tem um arquivo manifesto de agente, pode usar o comando `azd ai` para estruturar um projeto Foundry Agent Service diretamente. As versões recentes de pré-visualização também adicionaram suporte à inicialização baseada em template, então o fluxo exato de prompts pode variar conforme sua versão instalada da extensão.

```bash
# Instale a extensão dos agentes de IA
azd extension install azure.ai.agents

# Opcional: verifique a versão de visualização instalada
azd extension show azure.ai.agents

# Inicialize a partir de um manifesto de agente
azd ai agent init -m agent-manifest.yaml

# Faça o deploy no Azure
azd up

# Teste o agente implantado (exibe latência + tempo para o primeiro byte)
azd ai agent invoke
```

**Quando usar `azd ai agent init` vs `azd init --template`:**

| Abordagem | Melhor Para | Como Funciona |
|----------|------------|--------------|
| `azd init --template` | Começando de um app de exemplo funcional | Clona um repositório template completo com código + infra |
| `azd ai agent init -m` | Construindo a partir do seu próprio manifesto de agente | Estrutura o projeto a partir da definição do seu agente |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/AZD-for-beginners](https://github.com/microsoft/AZD-for-beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
