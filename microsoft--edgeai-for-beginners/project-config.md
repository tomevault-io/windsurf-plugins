---
trigger: always_on
description: > **Guia do Desenvolvedor para Contribuir com EdgeAI para Iniciantes**
---

# AGENTS.md

> **Guia do Desenvolvedor para Contribuir com EdgeAI para Iniciantes**
> 
> Este documento fornece informações detalhadas para desenvolvedores, agentes de IA e colaboradores que trabalham com este repositório. Ele abrange configuração, fluxos de trabalho de desenvolvimento, testes e melhores práticas.
> 
> **Última Atualização**: 30 de outubro de 2025 | **Versão do Documento**: 3.0

## Índice

- [Visão Geral do Projeto](../..)
- [Estrutura do Repositório](../..)
- [Pré-requisitos](../..)
- [Comandos de Configuração](../..)
- [Fluxo de Trabalho de Desenvolvimento](../..)
- [Instruções de Teste](../..)
- [Diretrizes de Estilo de Código](../..)
- [Diretrizes para Pull Requests](../..)
- [Sistema de Tradução](../..)
- [Integração Local do Foundry](../..)
- [Build e Implantação](../..)
- [Problemas Comuns e Solução de Problemas](../..)
- [Recursos Adicionais](../..)
- [Notas Específicas do Projeto](../..)
- [Obtendo Ajuda](../..)

## Visão Geral do Projeto

EdgeAI para Iniciantes é um repositório educacional abrangente que ensina desenvolvimento de Edge AI com Modelos de Linguagem Pequenos (SLMs). O curso aborda fundamentos de EdgeAI, implantação de modelos, técnicas de otimização e implementações prontas para produção usando Microsoft Foundry Local e vários frameworks de IA.

**Principais Tecnologias:**
- Python 3.8+ (linguagem principal para exemplos de IA/ML)
- .NET C# (exemplos de IA/ML)
- JavaScript/Node.js com Electron (para aplicações desktop)
- Microsoft Foundry Local SDK
- Microsoft Windows ML 
- VSCode AI Toolkit
- OpenAI SDK
- Frameworks de IA: LangChain, Semantic Kernel, Chainlit
- Otimização de Modelos: Llama.cpp, Microsoft Olive, OpenVINO, Apple MLX

**Tipo de Repositório:** Repositório de conteúdo educacional com 8 módulos e 10 aplicações de exemplo abrangentes

**Arquitetura:** Caminho de aprendizado multi-módulo com exemplos práticos demonstrando padrões de implantação de Edge AI

## Estrutura do Repositório

```
edgeai-for-beginners/
├── introduction.md          # Course introduction and overview
├── Module01-07/            # Core educational modules (Markdown)
├── Module08/               # Foundry Local toolkit with 10 samples
│   ├── samples/01-06/     # Foundation samples (Python)
│   ├── samples/07/        # API client (Python)
│   ├── samples/08/        # Windows 11 chat app (Electron)
│   └── samples/09-10/     # Advanced multi-agent systems (Python)
├── Workshop/               # Hands-on workshop materials
│   ├── samples/           # Workshop Python samples with utilities
│   │   ├── session01/     # Chat bootstrap samples
│   │   ├── session02-06/  # Progressive workshop sessions
│   │   └── util/          # Workshop utility modules
│   ├── notebooks/         # Jupyter notebook tutorials
│   └── scripts/           # Validation and testing tools
├── translations/          # Multi-language translations (50+ languages)
├── translated_images/     # Localized images
└── imgs/                  # Course images and assets
```

## Pré-requisitos

### Ferramentas Necessárias

- **Python 3.8+** - Para exemplos e notebooks de IA/ML
- **Node.js 16+** - Para aplicação de exemplo com Electron
- **Git** - Para controle de versão
- **Microsoft Foundry Local** - Para executar modelos de IA localmente

### Ferramentas Recomendadas

- **Visual Studio Code** - Com extensões Python, Jupyter e Pylance
- **Windows Terminal** - Para uma melhor experiência de linha de comando (usuários Windows)
- **Docker** - Para desenvolvimento em contêineres (opcional)

### Requisitos de Sistema

- **RAM**: Mínimo de 8GB, recomendado 16GB+ para cenários com múltiplos modelos
- **Armazenamento**: 10GB+ de espaço livre para modelos e dependências
- **SO**: Windows 10/11, macOS 11+, ou Linux (Ubuntu 20.04+)
- **Hardware**: CPU com suporte AVX2; GPU (CUDA, Qualcomm NPU) opcional, mas recomendada

### Conhecimentos Necessários

- Compreensão básica de programação em Python
- Familiaridade com interfaces de linha de comando
- Entendimento de conceitos de IA/ML (para desenvolvimento de exemplos)
- Fluxos de trabalho com Git e processos de pull request

## Comandos de Configuração

### Configuração do Repositório

```bash
# Clone the repository
git clone https://github.com/microsoft/edgeai-for-beginners.git
cd edgeai-for-beginners

# No build step required - this is primarily an educational content repository
```

### Configuração de Exemplos em Python (Módulo08 e exemplos de Workshop)

```bash
# Create and activate virtual environment
python -m venv .venv
# On Windows
.venv\Scripts\activate
# On macOS/Linux
source .venv/bin/activate

# Install Foundry Local SDK and dependencies
pip install foundry-local-sdk openai

# Install additional dependencies for Module08 samples
cd Module08
pip install -r requirements.txt

# Install Workshop dependencies
cd ../Workshop
pip install -r requirements.txt
```

### Configuração de Exemplos em Node.js (Exemplo 08 - Aplicação de Chat para Windows)

```bash
cd Module08/samples/08
npm install

# Start in development mode
npm run dev

# Build for production
npm run build

# Create installer
npm run dist
```

### Configuração do Foundry Local

Foundry Local é necessário para executar os exemplos. Baixe e instale a partir do repositório oficial:

**Instalação:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/edgeai-for-beginners](https://github.com/microsoft/edgeai-for-beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
