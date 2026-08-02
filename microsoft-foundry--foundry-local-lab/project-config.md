---
trigger: always_on
description: Este arquivo fornece contexto para agentes de codificação de IA (GitHub Copilot, Copilot Workspace, Codex, etc.) que trabalham neste repositório.
---

# Instruções para Agentes de Codificação

Este arquivo fornece contexto para agentes de codificação de IA (GitHub Copilot, Copilot Workspace, Codex, etc.) que trabalham neste repositório.

## Visão Geral do Projeto

Este é um **workshop prático** para construir aplicações de IA com o [Foundry Local](https://foundrylocal.ai) — um runtime leve que faz download, gerencia e serve modelos de linguagem inteiramente no dispositivo via uma API compatível com OpenAI. O workshop inclui guias de laboratório passo a passo e exemplos de código executáveis em Python, JavaScript e C#.

## Estrutura do Repositório

```
├── labs/                              # Markdown lab guides (Parts 1–13)
├── python/                            # Python code samples (Parts 2–6, 8–9, 11)
├── javascript/                        # JavaScript/Node.js code samples (Parts 2–6, 8–9, 11)
├── csharp/                            # C# / .NET 9 code samples (Parts 2–6, 8–9, 11)
├── zava-creative-writer-local/        # Part 7 capstone app + Part 12 UI (Python/JS/C#)
│   ├── ui/                            # Shared browser UI (vanilla HTML/CSS/JS)
│   └── src/
│       ├── api/                       # Python FastAPI multi-agent service (serves UI)
│       ├── javascript/                # Node.js CLI + HTTP server (server.mjs)
│       ├── csharp/                    # .NET console multi-agent app
│       └── csharp-web/                # .NET ASP.NET Core minimal API (serves UI)
├── samples/audio/                     # Part 9 sample WAV files + generator script
├── images/                            # Diagrams referenced by lab guides
├── README.md                          # Workshop overview and navigation
├── KNOWN-ISSUES.md                    # Known issues and workarounds
├── package.json                       # Root devDependency (mermaid-cli for diagrams)
└── AGENTS.md                          # This file
```

## Detalhes de Linguagem e Framework

### Python
- **Localização:** `python/`, `zava-creative-writer-local/src/api/`
- **Dependências:** `python/requirements.txt`, `zava-creative-writer-local/src/api/requirements.txt`
- **Principais pacotes:** `foundry-local-sdk`, `openai`, `agent-framework-foundry-local`, `fastapi`, `uvicorn`
- **Versão mínima:** Python 3.9+
- **Execução:** `cd python && pip install -r requirements.txt && python foundry-local.py`

### JavaScript
- **Localização:** `javascript/`, `zava-creative-writer-local/src/javascript/`
- **Dependências:** `javascript/package.json`, `zava-creative-writer-local/src/javascript/package.json`
- **Principais pacotes:** `foundry-local-sdk`, `openai`
- **Sistema de módulos:** Módulos ES (`.mjs` arquivos, `"type": "module"`)
- **Versão mínima:** Node.js 18+
- **Execução:** `cd javascript && npm install && node foundry-local.mjs`

### C#
- **Localização:** `csharp/`, `zava-creative-writer-local/src/csharp/`
- **Arquivos do projeto:** `csharp/csharp.csproj`, `zava-creative-writer-local/src/csharp/ZavaCreativeWriter.csproj`
- **Principais pacotes:** `Microsoft.AI.Foundry.Local` (não Windows), `Microsoft.AI.Foundry.Local.WinML` (Windows — superconjunto com QNN EP), `OpenAI`, `Microsoft.Agents.AI.OpenAI`
- **Alvo:** .NET 9.0 (TFM condicional: `net9.0-windows10.0.26100` no Windows, `net9.0` em outros)
- **Execução:** `cd csharp && dotnet run [chat|rag|agent|multi]`

## Convenções de Codificação

### Geral
- Todos os exemplos de código são **exemplos autossuficientes em arquivo único** — sem bibliotecas utilitárias compartilhadas ou abstrações.
- Cada exemplo roda independentemente após instalar suas próprias dependências.
- As chaves de API são sempre configuradas para `"foundry-local"` — Foundry Local usa isso como placeholder.
- URLs base usam `http://localhost:<port>/v1` — a porta é dinâmica e descoberta em tempo de execução via SDK (`manager.urls[0]` no JS, `manager.endpoint` no Python).
- O Foundry Local SDK gerencia a inicialização do serviço e descoberta do endpoint; prefira padrões do SDK ao invés de portas fixas.

### Python
- Use o SDK `openai` com `OpenAI(base_url=..., api_key="not-required")`.
- Use `FoundryLocalManager()` de `foundry_local` para ciclo de vida gerenciado pelo SDK.
- Streaming: itere sobre o objeto `stream` com `for chunk in stream:`.
- Sem anotações de tipo nos arquivos de exemplo (mantenha os exemplos concisos para aprendizes do workshop).

### JavaScript
- Sintaxe de módulo ES: `import ... from "..."`.
- Use `OpenAI` de `"openai"` e `FoundryLocalManager` de `"foundry-local-sdk"`.
- Padrão de inicialização do SDK: `FoundryLocalManager.create({ appName })` → `FoundryLocalManager.instance` → `manager.startWebService()` → `await catalog.getModel(alias)`.
- Streaming: `for await (const chunk of stream)`.
- `await` de topo é usado amplamente.

### C#
- Nullable ativado, usings implícitos, .NET 9.
- Use `FoundryLocalManager.StartServiceAsync()` para ciclo de vida gerenciado pelo SDK.
- Streaming: `CompleteChatStreaming()` com `foreach (var update in completionUpdates)`.
- O arquivo principal `csharp/Program.cs` é um roteador CLI que despacha para métodos estáticos `RunAsync()`.

### Chamada de Ferramentas

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft-foundry/Foundry-Local-Lab](https://github.com/microsoft-foundry/Foundry-Local-Lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
