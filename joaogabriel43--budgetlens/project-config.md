---
trigger: always_on
description: > Versão: 1.1.0 | Última atualização: 2026-04-06
---

# CLAUDE.md — BudgetLens
> Versão: 1.1.0 | Última atualização: 2026-04-06

---

## 🏗️ Visão Geral do Projeto

**BudgetLens** é um analisador de extrato bancário com IA. O usuário faz upload de um PDF ou CSV do seu extrato, o sistema categoriza as transações automaticamente via LLM, identifica padrões de comportamento financeiro e gera insights que aplicativos bancários tradicionais não oferecem.

### Objetivo de negócio
- Parsing robusto de extratos de diferentes bancos (cada banco tem formato próprio)
- Categorização automática via LLM com cache inteligente (evitar chamadas redundantes)
- Sistema de aprendizado: quando o usuário corrige uma categoria, o sistema aprende e aplica em transações futuras similares
- Geração de insights contextuais em PT-BR

### Stack
| Camada | Tecnologia | Versão |
|---|---|---|
| Linguagem | Java | 17 |
| Framework principal | Spring Boot | 3.3.x |
| Pipeline de dados | Spring Batch | 5.x |
| Persistência | Spring Data JPA + Hibernate | 6.x |
| Banco de dados | PostgreSQL | 16 |
| Migrations | Flyway | 10.x |
| Segurança | Spring Security + JWT (jjwt) | 0.12.x |
| HTTP Client (LLM) | RestClient (Spring 6 nativo) | — |
| Build | Maven | 3.9.x |
| Containerização | Docker Compose | — |
| Frontend | Angular + Angular Material | 17 |
| Testes | JUnit 5 + Mockito + Testcontainers | — |

### Arquitetura
Clean Architecture com 4 camadas:
- `domain` — modelos de domínio e interfaces de repositório (sem dependências externas)
- `application` — serviços de aplicação e DTOs
- `infrastructure` — implementações: JPA, batch, parsers, AI client
- `presentation` — controllers REST

---

## 📁 Estrutura de Pastas

```
budgetlens/
├── backend/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/budgetlens/
│   │   │   │   ├── domain/
│   │   │   │   │   ├── model/              # Entidades de domínio puras (sem anotações JPA)
│   │   │   │   │   │   ├── Transaction.java
│   │   │   │   │   │   ├── Category.java
│   │   │   │   │   │   ├── Statement.java
│   │   │   │   │   │   ├── UserCorrection.java
│   │   │   │   │   │   └── Insight.java
│   │   │   │   │   └── port/               # Interfaces de repositório (contrato)
│   │   │   │   │       ├── TransactionRepository.java
│   │   │   │   │       ├── CategoryRepository.java
│   │   │   │   │       ├── StatementRepository.java
│   │   │   │   │       └── UserCorrectionRepository.java
│   │   │   │   ├── application/
│   │   │   │   │   ├── service/            # Lógica de negócio
│   │   │   │   │   │   ├── StatementParserService.java
│   │   │   │   │   │   ├── CategorizationService.java
│   │   │   │   │   │   ├── InsightGeneratorService.java
│   │   │   │   │   │   ├── UserCorrectionService.java
│   │   │   │   │   │   └── StatementSummaryService.java
│   │   │   │   │   └── dto/                # Request/Response DTOs (fronteira da API)
│   │   │   │   ├── infrastructure/
│   │   │   │   │   ├── batch/              # Spring Batch: Jobs, Steps, Readers, Writers
│   │   │   │   │   │   ├── CategorizationJob.java
│   │   │   │   │   │   ├── CategorizationProcessor.java
│   │   │   │   │   │   └── InsightJobListener.java
│   │   │   │   │   ├── ai/                 # Integração com LLM
│   │   │   │   │   │   ├── OpenAIClient.java
│   │   │   │   │   │   ├── PromptBuilder.java
│   │   │   │   │   │   └── CategorizationCache.java
│   │   │   │   │   ├── parser/             # Parsers de extrato (padrão Strategy)
│   │   │   │   │   │   ├── StatementParser.java       (interface)
│   │   │   │   │   │   ├── NubankCsvParser.java
│   │   │   │   │   │   ├── GenericBankCsvParser.java
│   │   │   │   │   │   └── ParserFactory.java
│   │   │   │   │   └── persistence/        # Implementações JPA + entidades
│   │   │   │   │       ├── entity/         # Entidades JPA (anotações aqui, não no domínio)
│   │   │   │   │       ├── TransactionCategoryJpaRepository.java  # findAllByStatementId, deleteAllByStatementId
│   │   │   │   │       ├── TransactionJpaRepository.java          # findAllByStatementId(Pageable), filtro por categoria
│   │   │   │   │       └── UserCorrectionJpaRepository.java       # LIKE pattern%, ORDER BY comprimento desc
│   │   │   │   └── presentation/
│   │   │   │       └── controller/
│   │   │   │           ├── AuthController.java          # POST /api/auth/register, /login
│   │   │   │           ├── StatementController.java     # GET /api/statements, /{id}/summary, /insights, /transactions
│   │   │   │           ├── TransactionController.java   # PATCH /api/transactions/{id}/category
│   │   │   │           └── CategoryController.java      # GET /api/categories
│   │   │   └── resources/
│   │   │       ├── application.yml
│   │   │       ├── application-dev.yml
│   │   │       └── db/migration/           # Scripts Flyway (V1__..., V2__..., etc.)
│   │   └── test/
│   │       ├── java/com/budgetlens/        # Espelha estrutura de main/
│   │       └── resources/
│   │           └── extratos/               # Arquivos CSV/PDF de teste
│   └── pom.xml
├── frontend/
│   └── src/app/
│       ├── features/
│       │   ├── upload/
│       │   ├── dashboard/
│       │   └── insights/
│       └── shared/
├── docker-compose.yml
└── CLAUDE.md
```

---

## ⚙️ Configurações do Ambiente

### Pré-requisitos

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joaogabriel43/budgetlens](https://github.com/joaogabriel43/budgetlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
