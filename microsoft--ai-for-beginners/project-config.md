---
trigger: always_on
description: AI for Beginners é um currículo abrangente de 12 semanas e 24 aulas que cobre os fundamentos da Inteligência Artificial. Este repositório educacional inclui lições práticas usando Jupyter Notebooks, questionários e laboratórios práticos. O currículo aborda:
---

# AGENTS.md

## Visão Geral do Projeto

AI for Beginners é um currículo abrangente de 12 semanas e 24 aulas que cobre os fundamentos da Inteligência Artificial. Este repositório educacional inclui lições práticas usando Jupyter Notebooks, questionários e laboratórios práticos. O currículo aborda:

- IA Simbólica com Representação de Conhecimento e Sistemas Especialistas
- Redes Neurais e Aprendizagem Profunda com TensorFlow e PyTorch
- Técnicas e arquiteturas de Visão Computacional
- Processamento de Linguagem Natural (NLP), incluindo transformers e BERT
- Tópicos especializados: Algoritmos Genéticos, Aprendizagem por Reforço, Sistemas Multiagentes
- Ética em IA e princípios de IA Responsável

**Principais Tecnologias:** Python 3, Jupyter Notebooks, TensorFlow, PyTorch, Keras, OpenCV, Vue.js (para aplicação de questionários)

**Arquitetura:** Repositório de conteúdo educacional com Jupyter Notebooks organizados por áreas temáticas, complementado por uma aplicação de questionários baseada em Vue.js e suporte extensivo a múltiplos idiomas.

## Comandos de Configuração

### Ambiente de Desenvolvimento Principal (Python/Jupyter)

O currículo foi projetado para ser executado com Python e Jupyter Notebooks. A abordagem recomendada é usar miniconda:

```bash
# Clone the repository
git clone https://github.com/microsoft/ai-for-beginners
cd ai-for-beginners

# Create and activate conda environment
conda env create --name ai4beg --file environment.yml
conda activate ai4beg

# Start Jupyter Notebook
jupyter notebook
# OR
jupyter lab
```

### Alternativa: Usar devcontainer

```bash
# Open in VS Code and select "Reopen in Container" when prompted
# The devcontainer will automatically set up the environment
```

### Configuração da Aplicação de Questionários

A aplicação de questionários é uma aplicação Vue.js separada localizada em `etc/quiz-app/`:

```bash
cd etc/quiz-app
npm install
npm run serve  # Development server
npm run build  # Production build
npm run lint   # Lint and fix files
```

## Fluxo de Trabalho de Desenvolvimento

### Trabalhar com Jupyter Notebooks

1. **Desenvolvimento Local:**
   - Ativar o ambiente conda: `conda activate ai4beg`
   - Iniciar o Jupyter: `jupyter notebook` ou `jupyter lab`
   - Navegar pelas pastas das lições e abrir os ficheiros `.ipynb`
   - Executar as células interativamente para acompanhar as lições

2. **VS Code com Extensão Python:**
   - Abrir o repositório no VS Code
   - Instalar a extensão Python
   - O VS Code deteta automaticamente e utiliza o ambiente conda
   - Abrir ficheiros `.ipynb` diretamente no VS Code

3. **Desenvolvimento na Nuvem:**
   - **GitHub Codespaces:** Clique em "Code" → "Codespaces" → "Create codespace on main"
   - **Binder:** Use o emblema Binder no README para iniciar no navegador
   - Nota: O Binder tem recursos limitados e algumas restrições de acesso à web

### Suporte a GPU para Lições Avançadas

As lições posteriores beneficiam significativamente da aceleração por GPU:

- **Azure Data Science VM:** Utilize VMs da série NC com suporte a GPU
- **Azure Machine Learning:** Utilize funcionalidades de notebooks com computação GPU
- **Google Colab:** Carregue os notebooks individualmente (tem suporte gratuito a GPU)

### Desenvolvimento da Aplicação de Questionários

```bash
cd etc/quiz-app
npm run serve  # Hot-reload development server at http://localhost:8080
```

## Instruções de Teste

Este é um repositório educacional focado em conteúdo de aprendizagem, em vez de testes de software. Não há um conjunto de testes tradicional.

### Abordagens de Validação:

1. **Jupyter Notebooks:** Execute as células sequencialmente para verificar se os exemplos de código funcionam
2. **Teste da Aplicação de Questionários:** Teste manual através do servidor de desenvolvimento
3. **Validação de Traduções:** Verifique o conteúdo traduzido na pasta `translations/`
4. **Linting da Aplicação de Questionários:** `npm run lint` em `etc/quiz-app/`

### Executar Exemplos de Código:

```bash
# Activate environment first
conda activate ai4beg

# Run Python scripts directly
python lessons/4-ComputerVision/07-ConvNets/pytorchcv.py

# Or execute notebooks
jupyter notebook lessons/3-NeuralNetworks/03-Perceptron/Perceptron.ipynb
```

## Estilo de Código

### Estilo de Código Python

- Convenções padrão de Python para código educacional
- Código claro e legível, priorizando o aprendizado em vez da otimização
- Comentários explicando conceitos-chave
- Compatível com Jupyter Notebook: as células devem ser autossuficientes sempre que possível
- Não há requisitos rigorosos de linting para o conteúdo das lições

### JavaScript/Vue.js (Aplicação de Questionários)

- Configuração ESLint em `etc/quiz-app/package.json`
- Execute `npm run lint` para verificar e corrigir automaticamente problemas
- Convenções do Vue 2.x
- Arquitetura baseada em componentes

### Organização de Ficheiros

```
lessons/
  ├── 0-course-setup/          # Setup instructions
  ├── 1-Intro/                 # Introduction to AI
  ├── 2-Symbolic/              # Symbolic AI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/AI-For-Beginners](https://github.com/microsoft/AI-For-Beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
