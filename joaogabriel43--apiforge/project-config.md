---
trigger: always_on
description: * **Concluído**: Maio de 2026
---

# CLAUDE.md - APIForge Memória Persistente

* **Version**: 1.0.0
* **Concluído**: Maio de 2026
* **Status**: Concluído & Consolidado

Este documento serve como memória persistente e guia definitivo de desenvolvimento para o projeto **APIForge**. Ele estabelece as regras arquiteturais, decisões técnicas (ADRs), padrões de projeto, definições da stack e o resumo executivo para fins de portfólio.

---

## 🚀 Visão Geral do Projeto
O **APIForge** é um gerador inteligente de APIs REST a partir de schemas SQL. O sistema lê, analisa e interpreta definições de tabelas SQL (DDLs) e gera automaticamente estruturas de APIs completas, modulares, padronizadas e prontas para execução sob o framework Spring Boot.

---

## 🏛️ Estrutura de Diretórios (Clean Architecture)

A estrutura de pacotes principal reside em `src/main/java/com/apiforge/` e reflete estritamente as camadas da **Clean Architecture**:

```
com.apiforge/
│
├── ApiForgeApplication.java    # Classe de bootstrapping do Spring Boot (Root)
│
├── domain/                     # Camada de Domínio (Pureza de Negócios)
│   ├── model/                  # Entidades de negócio e Objetos de Valor
│   ├── exception/              # Exceções de regras de negócio
│   └── repository/             # Interfaces (Ports) de persistência e serviços externos
│
├── application/                # Camada de Aplicação (Casos de Uso)
│   ├── usecase/                # Implementação dos fluxos de casos de uso
│   ├── dto/                    # Modelos de transferência interna (Commands/Queries)
│   └── exception/              # Exceções específicas da aplicação
│
├── infrastructure/             # Camada de Infraestrutura (Detalhes Técnicos)
│   ├── config/                 # Beans de configuração (Spring, P6Spy, etc.)
│   ├── db/                     # Entidades JPA, Repositórios Spring Data e Flyway Migrations
│   └── adapter/                # Implementações concretas de serviços externos / persistência
│
└── presentation/               # Camada de Apresentação (Interface Externa)
    ├── controller/             # Controllers REST do Spring MVC exposing endpoints
    ├── dto/                    # DTOs de Request e Response da API
    ├── mapper/                 # Conversores entre DTOs de Apresentação e DTOs de Casos de Uso
    └── exception/              # Handler global de exceções e mapeador de HTTP Status
```

### Estrutura do Frontend (Angular Playground)

O ecossistema client reside na pasta `frontend/src/` e segue uma arquitetura modular orientada a componentes reativos:

```
frontend/src/
│
├── main.ts                     # Ponto de entrada bootstrap do Angular standalone
├── styles.css                  # Folha de estilos central contendo diretivas Tailwind CSS
│
└── app/
    ├── app.config.ts           # Configurações globais (Monaco, HTTP Client, Roteador)
    ├── app.component.ts        # Controlador reativo principal (form validation, SSE events, memory leak preventions)
    ├── app.component.html      # Layout grid com split File Tree e Code Viewer
    │
    └── core/                   # Núcleo da aplicação
        ├── models/             # Tipagens e interfaces TypeScript (GenerationOptions, SSE events)
        └── services/           # Serviços de comunicação HTTP/SSE reativos
```

---

## 📋 Padrões do Projeto

1. **Clean Architecture Estrita**:
   * O **Domínio** é o centro e não tem conhecimento de frameworks, bancos de dados ou qualquer detalhe técnico externo (sem anotações Spring ou JPA).
   * A **Aplicação** gerencia os casos de uso e depende puramente do domínio e de interfaces (Ports).
   * A **Infraestrutura** implementa os adaptadores concretos e lida com frameworks diretamente.
   * A **Apresentação** expõe endpoints REST e converte os payloads para chamadas de aplicação.
2. **TDD (Test-Driven Development)**:
   * Escreva os testes unitários e de propriedades antes ou em paralelo às regras de negócio.
   * Utilize `jqwik` para descobrir falhas de borda por meio de testes baseados em propriedades.
3. **YAGNI (You Aren't Gonna Need It)**:
   * Evite implementar código antecipadamente sem um caso de uso claro associado. Sem "futurismo" de código.
4. **Tratamento de Exceções**:
   * Exceções do domínio ou aplicação não podem vazar diretamente HTTP Status para fora. O `presentation` captura as exceções e mapeia em DTOs estruturados padrão RFC 7807 (Problem Details).

---

## 📦 Dependências Críticas

| Dependência | Versão | Justificativa |
| :--- | :--- | :--- |
| **Java** | `21` | Versão LTS moderna com suporte a Pattern Matching, Records estáveis e melhorias de performance. |
| **Spring Boot** | `3.2.11` | Framework corporativo robusto para bootstrap, injeção de dependência e suporte REST. |
| **JSQLParser** | `4.8` | Biblioteca de parsing de SQL que lê e interpreta DDLs de forma precisa para a extração do schema. |
| **P6Spy Starter** | `1.9.1` | Interceptador JDBC para profiling de SQL queries em tempo de execução, vital para detectar problemas de performance como N+1. |
| **jqwik** | `1.8.5` | Framework de Property-based Testing integrado ao JUnit 5 para validações matemáticas e de integridade dos DDLs analisados. |
| **Flyway** | `3.2` parent | Ferramenta de versionamento e migração de banco de dados PostgreSQL estruturado e resiliente. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joaogabriel43/APIForge](https://github.com/joaogabriel43/APIForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
