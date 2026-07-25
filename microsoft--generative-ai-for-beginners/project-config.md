---
trigger: always_on
description: Este repositório contém um currículo abrangente de 21 lições ensinando os fundamentos da IA Generativa e desenvolvimento de aplicações. O curso é destinado a iniciantes e cobre desde conceitos básicos até a construção de aplicações prontas para produção.
---

# AGENTS.md

## Visão Geral do Projeto

Este repositório contém um currículo abrangente de 21 lições ensinando os fundamentos da IA Generativa e desenvolvimento de aplicações. O curso é destinado a iniciantes e cobre desde conceitos básicos até a construção de aplicações prontas para produção.

**Tecnologias Principais:**
- Python 3.9+ com bibliotecas: `openai`, `python-dotenv`, `tiktoken`, `azure-ai-inference`, `pandas`, `numpy`, `matplotlib`
- TypeScript/JavaScript com Node.js e bibliotecas: `openai` (Azure OpenAI via o endpoint v1 + API Responses), `@azure-rest/ai-inference` (Microsoft Foundry Models)
- Azure OpenAI Service, OpenAI API, e Microsoft Foundry Models (GitHub Models será descontinuado no final de julho de 2026)
- Jupyter Notebooks para aprendizagem interativa
- Dev Containers para ambiente de desenvolvimento consistente

**Estrutura do Repositório:**
- 21 diretórios numerados de lições (00-21) contendo README, exemplos de código e tarefas
- Múltiplas implementações: exemplos em Python, TypeScript, e às vezes .NET
- Diretório de traduções com versões para mais de 40 idiomas
- Configuração centralizada via ficheiro `.env` (usar `.env.copy` como modelo)

## Comandos de Configuração

### Configuração Inicial do Repositório

```bash
# Clonar o repositório
git clone https://github.com/microsoft/generative-ai-for-beginners.git
cd generative-ai-for-beginners

# Copiar o modelo de ambiente
cp .env.copy .env
# Editar o .env com as suas chaves API e endpoints
```

### Configuração do Ambiente Python

```bash
# Criar ambiente virtual
python3 -m venv venv

# Ativar ambiente virtual
# No macOS/Linux:
source venv/bin/activate
# No Windows:
venv\Scripts\activate

# Instalar dependências
pip install -r requirements.txt
```

### Configuração Node.js/TypeScript

```bash
# Instale as dependências ao nível root (para ferramentas de documentação)
npm install

# Para exemplos individuais de TypeScript das lições, navegue até à lição específica:
cd 06-text-generation-apps/typescript/recipe-app
npm install
```

### Configuração do Dev Container (Recomendado)

O repositório inclui uma configuração `.devcontainer` para GitHub Codespaces ou VS Code Dev Containers:

1. Abra o repositório no GitHub Codespaces ou no VS Code com a extensão Dev Containers
2. O Dev Container irá automaticamente:
   - Instalar dependências Python do `requirements.txt`
   - Executar o script post-create (`.devcontainer/post-create.sh`)
   - Configurar o kernel do Jupyter

## Fluxo de Desenvolvimento

### Variáveis de Ambiente

Todas as lições que requerem acesso à API usam variáveis de ambiente definidas no `.env`:

- `OPENAI_API_KEY` - Para OpenAI API
- `AZURE_OPENAI_API_KEY` - Para Azure OpenAI no Microsoft Foundry (Azure OpenAI Service agora faz parte do Microsoft Foundry: https://ai.azure.com)
- `AZURE_OPENAI_ENDPOINT` - URL do endpoint Azure OpenAI (endpoint do recurso Foundry)
- `AZURE_OPENAI_DEPLOYMENT` - Nome do deployment do modelo de conclusão de chat
- `AZURE_OPENAI_EMBEDDINGS_DEPLOYMENT` - Nome do deployment do modelo de embeddings
- `AZURE_OPENAI_API_VERSION` - Versão da API (padrão: `2024-10-21`)
- `HUGGING_FACE_API_KEY` - Para modelos Hugging Face
- `AZURE_INFERENCE_ENDPOINT` - Endpoint dos Microsoft Foundry Models (catálogo de modelos multi-fornecedor)
- `AZURE_INFERENCE_CREDENTIAL` - Chave API dos Microsoft Foundry Models (substitui o `GITHUB_TOKEN` que será descontinuado)

### Executar Exemplos Python

```bash
# Navegar para o diretório da lição
cd 06-text-generation-apps/python

# Executar um script Python
python aoai-app.py
```

### Executar Exemplos TypeScript

```bash
# Navegar para o diretório da aplicação TypeScript
cd 06-text-generation-apps/typescript/recipe-app

# Construir o código TypeScript
npm run build

# Executar a aplicação
npm start
```

### Executar Jupyter Notebooks

```bash
# Iniciar o Jupyter na raiz do repositório
jupyter notebook

# Ou usar o VS Code com a extensão Jupyter
```

### Trabalhar com Diferentes Tipos de Lições

- **Lições "Learn"**: Foco na documentação README.md e conceitos
- **Lições "Build"**: Incluem exemplos de código funcionais em Python e TypeScript
- Cada lição tem um README.md com teoria, explicações do código e links para conteúdos em vídeo

## Diretrizes de Estilo de Código

### Python

- Usar `python-dotenv` para gestão de variáveis de ambiente
- Importar a biblioteca `openai` para interações com a API
- Usar `pylint` para linting (alguns exemplos incluem `# pylint: disable=all` para simplificação)
- Seguir convenções de nomenclatura PEP 8
- Guardar credenciais API no ficheiro `.env`, nunca no código

### TypeScript

- Usar o pacote `dotenv` para variáveis de ambiente
- Configuração TypeScript em `tsconfig.json` para cada aplicação
- Usar o pacote `openai` para Azure OpenAI (apontar cliente ao endpoint `/openai/v1/` e usar `client.responses.create`); usar `@azure-rest/ai-inference` para Microsoft Foundry Models
- Usar `nodemon` para desenvolvimento com recarga automática
- Construir antes de executar: `npm run build` depois `npm start`

### Convenções Gerais


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/generative-ai-for-beginners](https://github.com/microsoft/generative-ai-for-beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
