---
trigger: always_on
description: SEMPRE RESPONDA EM PT BR
---

SEMPRE RESPONDA EM PT BR

# Projeto BIANCA-SANITY - Sistema Integrado de Gestão de Conteúdo e Apresentações

## MCP BiancaTools - Configuração

### Instalação Global do BiancaTools
Para usar o BiancaTools MCP em qualquer projeto, execute:

```bash
claude mcp add BiancaTools /Users/phiz/Desktop/BIANCA-SANITY/mcp-run-ts-tools/run-bianca-tools.sh --env GITHUB_TOKEN=SEU_TOKEN_AQUI -s user
```

### Ferramentas Disponíveis no BiancaTools
- **Puppeteer**: Automação web (navegação, screenshots, cliques, preenchimento de formulários)
- **GitHub**: Criar issues, PRs, repositórios, commits
- **Git**: Status, commit, push, pull
- **Mem0**: Gerenciamento de memória persistente (adicionar, buscar, listar, deletar memórias)
- **Claude**: Executar comandos Claude via CLI

### Estrutura do BiancaTools
```
/mcp-run-ts-tools/
├── src/                    # Código fonte TypeScript
│   ├── tools/             # Implementação das ferramentas
│   │   ├── claude/       # Integração com Claude CLI
│   │   ├── git/          # Comandos Git
│   │   ├── github/       # API do GitHub
│   │   ├── mem0/         # Sistema de memória persistente
│   │   └── puppeteer/    # Automação web
│   └── index.ts          # Entrada principal
├── build/                 # Código compilado
├── run-bianca-tools.sh   # Script wrapper para execução global
└── package.json          # Configuração do projeto
```

## Visão Geral
Projeto integrado que combina:
- **Sanity CMS**: Sistema de gestão de conteúdo (schemas prontos, Studio ainda não instalado)
- **Slidev**: Módulo de apresentações interativas (com cliente Sanity configurado)
- **Sistemas Auxiliares**: Cronogramas e gestão de clientes

### ⚠️ IMPORTANTE: Diferença entre Sanity Studio e Cliente Sanity

**Cliente Sanity** (`/bianca-slidev/sanity/client.ts`) ✅ JÁ CONFIGURADO
- Biblioteca para CONSUMIR dados do Sanity
- Usado no frontend para fazer queries GROQ
- Permite apenas LEITURA de dados

**Sanity Studio** ❌ AINDA NÃO INSTALADO
- Interface administrativa web para CRIAR/EDITAR conteúdo
- Painel onde editores gerenciam o CMS
- Necessário para adicionar dados ao sistema

## Estrutura Completa do Projeto

```
/BIANCA-SANITY/                    # Raiz do projeto
├── /schemas/                      # Schemas do Sanity Studio
│   ├── /documents/               
│   │   ├── cliente.js            # Schema de clientes
│   │   ├── evidenciaServico.js   # Schema de evidências
│   │   └── presentation.js       # Schema de apresentações
│   └── /objects/                 # Objetos reutilizáveis
│
├── /clientes/                     # Dados e configurações de clientes
│   ├── /follow-the-money/        
│   └── /nurnberg/                # Cliente Nurnberg
│       ├── nurnberg-site.json    
│       ├── playbook-replanejamento.json
│       └── workflow-playbook.md
│
├── /planilha/                     # Sistema de cronograma
│   ├── cronograma.db             # Banco de dados SQLite
│   ├── servidor_cronograma.py    # Servidor do cronograma
│   └── [arquivos HTML/JS]        # Interfaces do cronograma
│
├── /bianca-slidev/                # MÓDULO SLIDEV (Apresentações)
│   ├── /components/              # Componentes Vue
│   │   ├── SanityExample.vue    # Exemplo de integração Sanity
│   │   ├── EstrategiaNurnberg.vue
│   │   └── DrawflowEditor.vue   
│   ├── /composables/             
│   │   └── sanity.ts            # Funções para buscar dados do Sanity
│   ├── /sanity/                 
│   │   └── client.ts            # Cliente Sanity (NÃO é o Studio!)
│   ├── slides.md                # Slides principais
│   └── package.json             # Dependências do Slidev
│
├── /icones/                       # Ícones SVG compartilhados
│
└── [FALTA CRIAR] /studio/         # Sanity Studio (interface administrativa)
```

## Integração Sanity + Slidev

### 1. Configuração do Sanity no Slidev

O módulo Slidev já está preparado para integração com Sanity:

```typescript
// bianca-slidev/sanity/client.ts
import { createClient } from '@sanity/client'

export const client = createClient({
  projectId: process.env.VITE_SANITY_PROJECT_ID,
  dataset: process.env.VITE_SANITY_DATASET,
  useCdn: true,
  apiVersion: '2024-01-01'
})
```

### 2. Configurar Variáveis de Ambiente

Criar arquivo `.env` em `/bianca-slidev/`:
```env
VITE_SANITY_PROJECT_ID=seu_project_id
VITE_SANITY_DATASET=production
```

### 3. Usar Dados do Sanity nas Apresentações

```vue
<!-- Em qualquer componente Vue -->
<script setup>
import { useSanity } from '../composables/sanity'

const { data: presentations, loading } = await useSanity().getPresentation('presentation-id')
</script>
```

## Como Instalar o Sanity Studio (NECESSÁRIO!)

### 1. Instalar Sanity CLI
```bash
npm install -g @sanity/cli
```

### 2. Criar o Sanity Studio na raiz do projeto
```bash
cd /Users/phiz/Desktop/BIANCA-SANITY
npm create sanity@latest -- --template clean --create-project "Bianca CMS" --dataset production

# Durante a instalação:
# - Escolha "Yes" para usar os schemas existentes
# - Aponte para a pasta /schemas quando perguntado
```

### 3. Mover os schemas existentes para o Studio
```bash
# Após criar o studio, copie seus schemas
cp -r schemas/* studio/schemas/
```

### 4. Comandos do Sanity Studio (após instalação)
```bash
cd studio
npm run dev          # Iniciar localmente
npm run deploy       # Deploy para sanity.studio

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/biancasivero) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
