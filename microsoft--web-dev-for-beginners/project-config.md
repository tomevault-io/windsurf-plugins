---
trigger: always_on
description: Este é um repositório educacional para ensinar fundamentos de desenvolvimento web a iniciantes. O currículo é um curso abrangente de 12 semanas desenvolvido pelos Microsoft Cloud Advocates, com 24 aulas práticas abordando JavaScript, CSS e HTML.
---

# AGENTS.md

## Visão Geral do Projeto

Este é um repositório educacional para ensinar fundamentos de desenvolvimento web a iniciantes. O currículo é um curso abrangente de 12 semanas desenvolvido pelos Microsoft Cloud Advocates, com 24 aulas práticas abordando JavaScript, CSS e HTML.

### Componentes Principais

- **Conteúdo Educacional**: 24 aulas estruturadas organizadas em módulos baseados em projetos  
- **Projetos Práticos**: Terrarium, Jogo de Digitação, Extensão de Navegador, Jogo Espacial, App Bancário, Editor de Código e Assistente de Chat de IA  
- **Quizzes Interativos**: 48 quizzes com 3 perguntas cada (avaliações pré e pós-lesson)  
- **Suporte Multilíngue**: Traduções automáticas para mais de 50 idiomas via GitHub Actions  
- **Tecnologias**: HTML, CSS, JavaScript, Vue.js 3, Vite, Node.js, Express, Python (para projetos de IA)  

### Arquitetura

- Repositório educacional com estrutura baseada em lições  
- Cada pasta de lição contém README, exemplos de código e soluções  
- Projetos independentes em diretórios separados (quiz-app, vários projetos de lição)  
- Sistema de tradução usando GitHub Actions (co-op-translator)  
- Documentação servida via Docsify e disponível em PDF  

## Comandos de Configuração

Este repositório é principalmente para consumo de conteúdo educacional. Para trabalhar com projetos específicos:

### Configuração Principal do Repositório

```bash
git clone https://github.com/microsoft/Web-Dev-For-Beginners.git
cd Web-Dev-For-Beginners
```

### Configuração do Quiz App (Vue 3 + Vite)

```bash
cd quiz-app
npm install
npm run dev        # Iniciar servidor de desenvolvimento
npm run build      # Construir para produção
npm run lint       # Executar ESLint
```

### Projeto API Bancária (Node.js + Express)

```bash
cd 7-bank-project/api
npm install
npm start          # Iniciar servidor API
npm run lint       # Executar ESLint
npm run format     # Formatar com Prettier
```

### Projetos de Extensão de Navegador

```bash
cd 5-browser-extension/solution
npm install
# Siga as instruções específicas do navegador para carregar extensões
```

### Projetos de Jogo Espacial

```bash
cd 6-space-game/solution
npm install
# Abra index.html no navegador ou use o Live Server
```

### Projeto de Chat (Backend Python)

```bash
cd 9-chat-project/solution/backend/python
pip install openai
# Defina a variável de ambiente GITHUB_TOKEN
python api.py
```

## Fluxo de Desenvolvimento

### Para Contribuidores de Conteúdo

1. **Fork o repositório** para sua conta no GitHub  
2. **Clone seu fork** localmente  
3. **Crie uma nova branch** para suas alterações  
4. Faça mudanças no conteúdo das lições ou exemplos de código  
5. Teste quaisquer alterações de código nos diretórios dos projetos relevantes  
6. Envie pull requests seguindo as diretrizes de contribuição  

### Para Aprendizes

1. Faça fork ou clone do repositório  
2. Navegue sequencialmente pelos diretórios das lições  
3. Leia os arquivos README de cada lição  
4. Complete os quizzes pré-lição em https://ff-quizzes.netlify.app/web/  
5. Trabalhe nos exemplos de código nas pastas da lição  
6. Complete as tarefas e desafios  
7. Faça os quizzes pós-lição  

### Desenvolvimento ao Vivo

- **Documentação**: Rode `docsify serve` na raiz (porta 3000)  
- **Quiz App**: Rode `npm run dev` no diretório quiz-app  
- **Projetos**: Use a extensão Live Server do VS Code para projetos HTML  
- **Projetos API**: Rode `npm start` nos diretórios das APIs respectivas  

## Instruções de Teste

### Teste do Quiz App

```bash
cd quiz-app
npm run lint       # Verificar problemas de estilo de código
npm run build      # Verificar se a compilação foi bem-sucedida
```

### Teste da API Bancária

```bash
cd 7-bank-project/api
npm run lint       # Verificar problemas de estilo de código
node server.js     # Verificar se o servidor inicia sem erros
```

### Abordagem Geral de Teste

- Este é um repositório educacional sem testes automatizados abrangentes  
- Teste manual foca em:  
  - Exemplos de código executando sem erros  
  - Links na documentação funcionando corretamente  
  - Builds dos projetos completando com sucesso  
  - Exemplos seguindo as melhores práticas  

### Checagens Pré-envio

- Execute `npm run lint` em diretórios com package.json  
- Verifique se os links markdown são válidos  
- Teste exemplos de código no navegador ou Node.js  
- Verifique que as traduções mantêm a estrutura adequada  

## Diretrizes de Estilo de Código

### JavaScript

- Use sintaxe moderna ES6+  
- Siga as configurações padrão do ESLint fornecidas nos projetos  
- Use nomes significativos para variáveis e funções para clareza educacional  
- Adicione comentários explicando conceitos para os aprendizes  
- Formate usando Prettier quando configurado  

### HTML/CSS

- Elementos semânticos HTML5  
- Princípios de design responsivo  
- Convenções claras de nomenclatura de classes  
- Comentários explicando técnicas CSS para aprendizes  

### Python

- Diretrizes de estilo PEP 8  
- Exemplos de código claros e educacionais  
- Dicas de tipo onde úteis para aprendizagem  


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Web-Dev-For-Beginners](https://github.com/microsoft/Web-Dev-For-Beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
