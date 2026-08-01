---
trigger: always_on
description: Este é o **Machine Learning para Principiantes**, um currículo abrangente de 12 semanas e 26 lições que cobre conceitos clássicos de machine learning utilizando Python (principalmente com Scikit-learn) e R. O repositório foi concebido como um recurso de aprendizagem autodidata, com projetos práticos, questionários e tarefas. Cada lição explora conceitos de ML através de dados reais provenientes de várias culturas e regiões do mundo.
---

# AGENTS.md

## Visão Geral do Projeto

Este é o **Machine Learning para Principiantes**, um currículo abrangente de 12 semanas e 26 lições que cobre conceitos clássicos de machine learning utilizando Python (principalmente com Scikit-learn) e R. O repositório foi concebido como um recurso de aprendizagem autodidata, com projetos práticos, questionários e tarefas. Cada lição explora conceitos de ML através de dados reais provenientes de várias culturas e regiões do mundo.

Componentes principais:
- **Conteúdo Educacional**: 26 lições que abrangem introdução ao ML, regressão, classificação, clustering, NLP, séries temporais e aprendizagem por reforço
- **Aplicação de Questionários**: Aplicação de questionários baseada em Vue.js com avaliações antes e depois das lições
- **Suporte Multilíngue**: Traduções automáticas para mais de 40 idiomas via GitHub Actions
- **Suporte a Duas Linguagens**: Lições disponíveis tanto em Python (notebooks Jupyter) quanto em R (ficheiros R Markdown)
- **Aprendizagem Baseada em Projetos**: Cada tópico inclui projetos práticos e tarefas

## Estrutura do Repositório

```
ML-For-Beginners/
├── 1-Introduction/         # ML basics, history, fairness, techniques
├── 2-Regression/          # Regression models with Python/R
├── 3-Web-App/            # Flask web app for ML model deployment
├── 4-Classification/      # Classification algorithms
├── 5-Clustering/         # Clustering techniques
├── 6-NLP/               # Natural Language Processing
├── 7-TimeSeries/        # Time series forecasting
├── 8-Reinforcement/     # Reinforcement learning
├── 9-Real-World/        # Real-world ML applications
├── quiz-app/           # Vue.js quiz application
├── translations/       # Auto-generated translations
└── sketchnotes/       # Visual learning aids
```

Cada pasta de lição geralmente contém:
- `README.md` - Conteúdo principal da lição
- `notebook.ipynb` - Notebook Jupyter em Python
- `solution/` - Código de solução (versões em Python e R)
- `assignment.md` - Exercícios práticos
- `images/` - Recursos visuais

## Comandos de Configuração

### Para Lições em Python

A maioria das lições utiliza notebooks Jupyter. Instale as dependências necessárias:

```bash
# Install Python 3.8+ if not already installed
python --version

# Install Jupyter
pip install jupyter

# Install common ML libraries
pip install scikit-learn pandas numpy matplotlib seaborn

# For specific lessons, check lesson-specific requirements
# Example: Web App lesson
pip install flask
```

### Para Lições em R

As lições em R estão nas pastas `solution/R/` como ficheiros `.rmd` ou `.ipynb`:

```bash
# Install R and required packages
# In R console:
install.packages(c("tidyverse", "tidymodels", "caret"))
```

### Para a Aplicação de Questionários

A aplicação de questionários é uma aplicação Vue.js localizada no diretório `quiz-app/`:

```bash
cd quiz-app
npm install
```

### Para o Site de Documentação

Para executar a documentação localmente:

```bash
# Install Docsify
npm install -g docsify-cli

# Serve from repository root
docsify serve

# Access at http://localhost:3000
```

## Fluxo de Trabalho de Desenvolvimento

### Trabalhar com Notebooks de Lição

1. Navegue até ao diretório da lição (ex.: `2-Regression/1-Tools/`)
2. Abra o notebook Jupyter:
   ```bash
   jupyter notebook notebook.ipynb
   ```
3. Trabalhe no conteúdo e nos exercícios da lição
4. Consulte as soluções na pasta `solution/` se necessário

### Desenvolvimento em Python

- As lições utilizam bibliotecas padrão de ciência de dados em Python
- Notebooks Jupyter para aprendizagem interativa
- Código de solução disponível na pasta `solution/` de cada lição

### Desenvolvimento em R

- As lições em R estão no formato `.rmd` (R Markdown)
- Soluções localizadas em subdiretórios `solution/R/`
- Utilize RStudio ou Jupyter com kernel R para executar os notebooks em R

### Desenvolvimento da Aplicação de Questionários

```bash
cd quiz-app

# Start development server
npm run serve
# Access at http://localhost:8080

# Build for production
npm run build

# Lint and fix files
npm run lint
```

## Instruções de Teste

### Teste da Aplicação de Questionários

```bash
cd quiz-app

# Lint code
npm run lint

# Build to verify no errors
npm run build
```

**Nota**: Este é principalmente um repositório de currículo educacional. Não há testes automatizados para o conteúdo das lições. A validação é feita através de:
- Conclusão dos exercícios das lições
- Execução bem-sucedida das células dos notebooks
- Verificação dos resultados contra as soluções esperadas

## Diretrizes de Estilo de Código

### Código em Python
- Siga as diretrizes de estilo PEP 8
- Utilize nomes de variáveis claros e descritivos
- Inclua comentários para operações complexas
- Os notebooks Jupyter devem conter células markdown explicando os conceitos

### JavaScript/Vue.js (Aplicação de Questionários)
- Siga o guia de estilo Vue.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/ML-For-Beginners](https://github.com/microsoft/ML-For-Beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
