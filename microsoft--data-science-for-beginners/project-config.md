---
trigger: always_on
description: Data Science for Beginners é um currículo abrangente de 10 semanas e 20 lições, criado pelos Microsoft Azure Cloud Advocates. Este repositório é um recurso de aprendizagem que ensina conceitos fundamentais de ciência de dados através de lições baseadas em projetos, incluindo notebooks Jupyter, questionários interativos e tarefas práticas.
---

# AGENTS.md

## Visão Geral do Projeto

Data Science for Beginners é um currículo abrangente de 10 semanas e 20 lições, criado pelos Microsoft Azure Cloud Advocates. Este repositório é um recurso de aprendizagem que ensina conceitos fundamentais de ciência de dados através de lições baseadas em projetos, incluindo notebooks Jupyter, questionários interativos e tarefas práticas.

**Principais Tecnologias:**
- **Jupyter Notebooks**: Meio principal de aprendizagem utilizando Python 3
- **Bibliotecas Python**: pandas, numpy, matplotlib para análise e visualização de dados
- **Vue.js 2**: Aplicação de questionários (pasta quiz-app)
- **Docsify**: Gerador de sites de documentação para acesso offline
- **Node.js/npm**: Gestão de pacotes para componentes JavaScript
- **Markdown**: Todo o conteúdo das lições e documentação

**Arquitetura:**
- Repositório educacional multilíngue com extensas traduções
- Estruturado em módulos de lições (1-Introdução até 6-Ciência-de-Dados-no-Mundo-Real)
- Cada lição inclui README, notebooks, tarefas e questionários
- Aplicação de questionários Vue.js independente para avaliações pré/pós-lição
- Suporte para GitHub Codespaces e contêineres de desenvolvimento do VS Code

## Comandos de Configuração

### Configuração do Repositório
```bash
# Clone the repository (if not already cloned)
git clone https://github.com/microsoft/Data-Science-For-Beginners.git
cd Data-Science-For-Beginners
```

### Configuração do Ambiente Python
```bash
# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install common data science libraries (no requirements.txt exists)
pip install jupyter pandas numpy matplotlib seaborn scikit-learn
```

### Configuração da Aplicação de Questionários
```bash
# Navigate to quiz app
cd quiz-app

# Install dependencies
npm install

# Start development server
npm run serve

# Build for production
npm run build

# Lint and fix files
npm run lint
```

### Servidor de Documentação Docsify
```bash
# Install Docsify globally
npm install -g docsify-cli

# Serve documentation locally
docsify serve

# Documentation will be available at localhost:3000
```

### Configuração de Projetos de Visualização
Para projetos de visualização como meaningful-visualizations (lição 13):
```bash
# Navigate to starter or solution folder
cd 3-Data-Visualization/13-meaningful-visualizations/starter

# Install dependencies
npm install

# Start development server
npm run serve

# Build for production
npm run build

# Lint files
npm run lint
```


## Fluxo de Trabalho de Desenvolvimento

### Trabalhando com Jupyter Notebooks
1. Inicie o Jupyter na raiz do repositório: `jupyter notebook`
2. Navegue até a pasta da lição desejada
3. Abra os ficheiros `.ipynb` para realizar os exercícios
4. Os notebooks são autossuficientes, com explicações e células de código
5. A maioria dos notebooks utiliza pandas, numpy e matplotlib - certifique-se de que estão instalados

### Estrutura das Lições
Cada lição geralmente contém:
- `README.md` - Conteúdo principal da lição com teoria e exemplos
- `notebook.ipynb` - Exercícios práticos em Jupyter Notebook
- `assignment.ipynb` ou `assignment.md` - Tarefas práticas
- Pasta `solution/` - Notebooks e códigos de solução
- Pasta `images/` - Materiais visuais de suporte

### Desenvolvimento da Aplicação de Questionários
- Aplicação Vue.js 2 com recarregamento automático durante o desenvolvimento
- Questionários armazenados em `quiz-app/src/assets/translations/`
- Cada idioma tem a sua própria pasta de tradução (en, fr, es, etc.)
- A numeração dos questionários começa em 0 e vai até 39 (40 questionários no total)

### Adicionando Traduções
- Traduções vão para a pasta `translations/` na raiz do repositório
- Cada idioma tem uma estrutura completa de lições espelhada do inglês
- Tradução automatizada via GitHub Actions (co-op-translator.yml)

## Instruções de Teste

### Teste da Aplicação de Questionários
```bash
cd quiz-app

# Run lint checks
npm run lint

# Test build process
npm run build

# Manual testing: Start dev server and verify quiz functionality
npm run serve
```

### Teste de Notebooks
- Não existe um framework de teste automatizado para notebooks
- Validação manual: Execute todas as células em sequência para garantir que não há erros
- Verifique se os ficheiros de dados estão acessíveis e se os resultados são gerados corretamente
- Confirme que as visualizações são renderizadas corretamente

### Teste de Documentação
```bash
# Verify Docsify renders correctly
docsify serve

# Check for broken links manually by navigating through content
# Verify all lesson links work in the rendered documentation
```

### Verificações de Qualidade de Código
```bash
# Vue.js projects (quiz-app and visualization projects)
cd quiz-app  # or visualization project folder
npm run lint

# Python notebooks - manual verification recommended
# Ensure imports work and cells execute without errors
```


## Diretrizes de Estilo de Código


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Data-Science-For-Beginners](https://github.com/microsoft/Data-Science-For-Beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
