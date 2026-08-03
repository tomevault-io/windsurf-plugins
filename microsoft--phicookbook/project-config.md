---
trigger: always_on
description: O PhiCookBook é um repositório abrangente de cookbooks que contém exemplos práticos, tutoriais e documentação para trabalhar com a família Phi de Modelos Pequenos de Linguagem (SLMs) da Microsoft. O repositório demonstra vários casos de uso, incluindo inferência, fine-tuning, quantização, implementações RAG e aplicações multimodais em diferentes plataformas e frameworks.
---

# AGENTS.md

## Visão Geral do Projeto

O PhiCookBook é um repositório abrangente de cookbooks que contém exemplos práticos, tutoriais e documentação para trabalhar com a família Phi de Modelos Pequenos de Linguagem (SLMs) da Microsoft. O repositório demonstra vários casos de uso, incluindo inferência, fine-tuning, quantização, implementações RAG e aplicações multimodais em diferentes plataformas e frameworks.

**Tecnologias Principais:**
- **Linguagens:** Python, C#/.NET, JavaScript/Node.js
- **Frameworks:** ONNX Runtime, PyTorch, Transformers, MLX, OpenVINO, Semantic Kernel
- **Plataformas:** Microsoft Foundry, GitHub Models, Hugging Face, Ollama
- **Tipos de Modelo:** Phi-3, Phi-3.5, Phi-4 (variantes de texto, visão, multimodal, raciocínio)

**Estrutura do Repositório:**
- `/code/` - Exemplos de código funcional e implementações de amostra
- `/md/` - Documentação detalhada, tutoriais e guias práticos  
- `/translations/` - Traduções multilíngues (mais de 50 idiomas via workflow automatizado)
- `/.devcontainer/` - Configuração do ambiente de desenvolvimento (Python 3.12 com Ollama)

## Configuração do Ambiente de Desenvolvimento

### Utilizando GitHub Codespaces ou Contêineres de Desenvolvimento (Recomendado)

1. Abrir no GitHub Codespaces (mais rápido):
   - Clique no badge "Open in GitHub Codespaces" no README
   - O contêiner configura automaticamente com Python 3.12 e Ollama com Phi-3

2. Abrir no VS Code Dev Containers:
   - Use o badge "Open in Dev Containers" do README
   - O contêiner requer pelo menos 16GB de memória do host

### Configuração Local

**Pré-requisitos:**
- Python 3.12 ou superior
- .NET 8.0 SDK (para exemplos em C#)
- Node.js 18+ e npm (para exemplos em JavaScript)
- 16GB de RAM recomendados no mínimo

**Instalação:**
```bash
git clone https://github.com/microsoft/PhiCookBook.git
cd PhiCookBook
```

**Para Exemplos em Python:**
Navegue até os diretórios específicos dos exemplos e instale as dependências:
```bash
cd code/<example-directory>
pip install -r requirements.txt  # se o ficheiro requirements.txt existir
```

**Para Exemplos em .NET:**
```bash
cd md/04.HOL/dotnet/src
dotnet restore LabsPhi.sln
dotnet build LabsPhi.sln
```

**Para Exemplos em JavaScript/Web:**
```bash
cd code/08.RAG/rag_webgpu_chat
npm install
npm run dev  # Iniciar servidor de desenvolvimento
npm run build  # Construir para produção
```

## Organização do Repositório

### Exemplos de Código (`/code/`)

- **01.Introduce/** - Introduções básicas e amostras de primeiros passos
- **03.Finetuning/** e **04.Finetuning/** - Exemplos de fine-tuning com vários métodos
- **03.Inference/** - Exemplos de inferência em diferentes hardwares (AIPC, MLX)
- **06.E2E/** - Exemplos de aplicações end-to-end
- **07.Lab/** - Implementações laboratoriais/experimentais
- **08.RAG/** - Exemplos de Recuperação-Augmentada de Geração (RAG)
- **09.UpdateSamples/** - Amostras mais recentes atualizadas

### Documentação (`/md/`)

- **01.Introduction/** - Guias introdutórios, configuração do ambiente, guias de plataforma
- **02.Application/** - Exemplos de aplicações organizados por tipo (Texto, Código, Visão, Áudio, etc.)
- **02.QuickStart/** - Guias rápidos para Microsoft Foundry e GitHub Models
- **03.FineTuning/** - Documentação e tutoriais de fine-tuning
- **04.HOL/** - Laboratórios práticos (inclui exemplos .NET)

### Formatos de Ficheiro

- **Jupyter Notebooks (`.ipynb`)** - Tutoriais interativos em Python marcados com 📓 no README
- **Scripts Python (`.py`)** - Exemplos independentes em Python
- **Projetos C# (`.csproj`, `.sln`)** - Aplicações e exemplos .NET
- **JavaScript (`.js`, `package.json`)** - Exemplos web e Node.js
- **Markdown (`.md`)** - Documentação e guias

## Trabalho com Exemplos

### Executar Jupyter Notebooks

A maioria dos exemplos é fornecida como Jupyter notebooks:
```bash
pip install jupyter notebook
jupyter notebook  # Abre a interface do navegador
# Navegue até ao ficheiro .ipynb desejado
```

### Executar Scripts Python

```bash
cd code/<example-directory>
pip install -r requirements.txt
python <script-name>.py
```

### Executar Exemplos .NET

```bash
cd md/04.HOL/dotnet/src/<project-name>
dotnet run
```

Ou construir a solução completa:
```bash
cd md/04.HOL/dotnet/src
dotnet run --project <project-name>
```

### Executar Exemplos JavaScript/Web

```bash
cd code/08.RAG/rag_webgpu_chat
npm install
npm run dev  # Desenvolvimento com recarregamento a quente
```

## Testes

Este repositório contém código de exemplo e tutoriais em vez de um projeto de software tradicional com testes unitários. A validação é normalmente feita por:

1. **Executar os exemplos** - Cada exemplo deve correr sem erros
2. **Verificar as saídas** - Confirmar que as respostas do modelo são apropriadas
3. **Seguir os tutoriais** - Os guias passo a passo devem funcionar conforme documentado

**Abordagem comum de validação:**
- Testar a execução dos exemplos no ambiente alvo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/PhiCookBook](https://github.com/microsoft/PhiCookBook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
