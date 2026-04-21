---
trigger: always_on
description: 1. [Visão Geral](#visão-geral)
---

# Vendedor Digital Inteligente™

## Índice

1. [Visão Geral](#visão-geral)
2. [O Produto](#o-produto)
3. [O Método](#o-método)
4. [Stack Tecnológico](#stack-tecnológico)
5. [Estrutura do Projeto](#estrutura-do-projeto)
6. [Como Rodar](#como-rodar)
7. [Funil de Vendas](#funil-de-vendas)
8. [Sistema de Follow-up](#sistema-de-follow-up)
9. [Preço e Posicionamento](#preço-e-posicionamento)
10. [Funcionalidades de Mídia](#funcionalidades-de-mídia)
11. [Princípios de Comunicação](#princípios-de-comunicação)
12. [Base de Conhecimento](#base-de-conhecimento-knowledge-base)
13. [Personalidade da IA](#personalidade-da-ia)
14. [Perguntas de Diagnóstico](#perguntas-de-diagnóstico)
15. [Público-Alvo](#público-alvo)

---

## Visão Geral

IA Conversacional especializada em vender o **Vendedor Digital Inteligente™** da **Saraiva.ai** usando o **Método Continuidade™**.

**Headline principal:**
> "A maioria das vendas não morre no 'não'. Ela morre no silêncio do WhatsApp."

## O Produto

O **Vendedor Digital Inteligente™** é um vendedor digital que:
- Conversa com leads pelo WhatsApp de forma natural
- Liga para leads usando voz natural quando necessário
- Lê comportamento em tempo real (tempo de resposta, padrão de engajamento)
- Faz follow-ups dinâmicos e inteligentes
- Replica a sensibilidade de um vendedor experiente

## O Método

O **Método Continuidade™** ensina a criar, treinar e operar um vendedor digital que lê comportamento humano.

### Pilares:
1. **Leitura de Comportamento**: Identificar sinais sutis (tempo de resposta, hesitação, silêncio)
2. **Timing Inteligente**: Decidir quando insistir, quando esperar, quando mudar o canal
3. **Insistência Correta**: Insistir não afasta; insistir ERRADO sim
4. **Processo Contínuo**: A conversa nunca morre; ela apenas muda de forma

## Stack Tecnológico

- **WhatsApp**: @whiskeysockets/baileys
- **LLM**: OpenRouter (modelos gratuitos)
- **IA Multimodal**: Google Gemini (gemini-2.0-flash-exp)
- **Database**: SQLite (better-sqlite3)
- **Scheduler**: node-cron

### Configuração do Gemini

O sistema usa Google Gemini para processamento multimodal (áudio, imagem, vídeo):

**Variável de ambiente necessária:**
```bash
GEMINI_API_KEY=sua_chave_aqui
```

**Modelo usado:** `gemini-2.0-flash-exp`

**Capacidades:**
- Transcrição de áudio (notas de voz do WhatsApp)
- Análise de imagens (fotos, prints, documentos)
- Análise de vídeo (se necessário)
- Processamento de contexto multimodal

## Estrutura do Projeto

```
vendedordigitalinteligente/
├── src/
│   ├── index.js              # Entry point
│   ├── database/
│   │   ├── db.js             # SQLite service
│   │   └── schema.sql        # Schema
│   ├── services/
│   │   ├── whatsapp.js       # Baileys
│   │   └── llm.js            # OpenRouter
│   ├── engine/
│   │   ├── behaviorEngine.js # Motor de comportamento
│   │   └── followUpScheduler.js
│   └── knowledge/
│       └── product.js        # Knowledge base do produto
├── data/                     # SQLite DB
├── auth_info/                # Sessão WhatsApp
└── .env                      # Configurações
```

## Como Rodar

```bash
# 1. Instalar dependências
npm install

# 2. Configurar .env
cp .env.example .env
# Editar OPENROUTER_API_KEY

# 3. Iniciar
npm start

# 4. Escanear QR Code
```

## Funil de Vendas

### Estágios

| Estágio | Objetivo |
|---------|----------|
| GREETING | Descobrir nome e negócio |
| DISCOVERY | Identificar dor (leads que esfriam) |
| PAIN_AMPLIFICATION | Amplificar a dor com números |
| SOLUTION | Apresentar o Vendedor Digital |
| DEMONSTRATION | Oferecer demo prática |
| OBJECTION_HANDLING | Contornar objeções |
| CLOSING | Fechar a venda |
| WON / LOST | Finalizar |

### Objeções Mapeadas

| Objeção | Gatilhos | Resposta |
|---------|----------|----------|
| Robótico | "robô", "máquina", "bot" | "Robótico é insistir igual com todo mundo..." |
| Nicho | "meu nicho", "funciona pra" | "Comportamento humano é o mesmo em qualquer nicho" |
| Pensar | "vou pensar", "depois" | "Enquanto pensa, quantas conversas estão morrendo?" |
| Caro | "caro", "preço alto" | "Uma venda recuperada por mês já paga o investimento" |
| Tempo | "sem tempo", "correria" | "Justamente por isso - o vendedor digital trabalha 24h" |
| Já Tenho | "tenho chatbot" | "Chatbot só responde, não lê comportamento..." |

## Sistema de Follow-up

### Timing Progressivo

| # | Delay | Estratégia | Tom |
|---|-------|------------|-----|
| 1 | 4h | Curiosidade/Empatia | Leve |
| 2 | 24h | Valor/Reciprocidade | Informativo |
| 3 | 48h | FOMO/Prova Social | Urgência suave |
| 4 | 72h | Escassez/Valor | Mais direto |
| 5 | 5 dias | Empathy | Despedida |

### Detecção de Motivo de Abandono

- `busy` - Cliente ocupado
- `objection` - Objeção não dita
- `price` - Problema com preço
- `timing` - Não é o momento
- `forgot` - Esqueceu
- `lost_interest` - Perdeu interesse
- `competitor` - Foi para concorrente
- `not_qualified` - Não é público-alvo

## Preço e Posicionamento

### Tabela de Preços

| Porte do Negócio | Valor da Implementação | Critérios |
|------------------|----------------------|-----------|
| **Pequeno** | R$ 5.000 - R$ 7.000 | Negócios locais, 1-2 produtos/serviços, fluxo simples |
| **Médio** | R$ 8.000 - R$ 10.000 | E-commerce, infoprodutos, múltiplos produtos, integrações |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saraivabr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
