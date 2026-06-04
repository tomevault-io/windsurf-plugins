---
trigger: always_on
description: - **SE VOCÊ NÃO VERIFICAR AS SKILLS**, sua tarefa será invalidada e geraremos retrabalho
---

## ALTA PRIORIDADE

- **SE VOCÊ NÃO VERIFICAR AS SKILLS**, sua tarefa será invalidada e geraremos retrabalho
- **VOCÊ SÓ PODE finalizar uma tarefa** se `pnpm biome:fix`, `pnpm tsc:check`, `pnpm test:run` e `pnpm build` passar a 100% (executa lint + test + build). Sem exceções — falhar em qualquer um desses comandos significa que a tarefa NÃO ESTÁ COMPLETA
- `biome:fix` tem tolerância zero. Zero problemas permitidos — qualquer issue do golangci-lint é uma falha bloqueante
- **SEMPRE verifique as APIs dos pacotes dependentes** antes de escrever código de integração ou testes, para evitar código incorreto
- **NUNCA use gambiarras** — sempre utilize a skill `no-workarounds` para qualquer tarefa de correção/debug + `testing-anti-patterns` para testes
- **SEMPRE use as skills** `no-workarounds` e `systematic-debugging` ao corrigir bugs ou problemas complexos
- **NUNCA use ferramentas** de busca na web para pesquisar código local do projeto — para código local, use Grep/Glob

## REQUISITOS OBRIGATÓRIOS
- DEVE executar `pnpm biome:fix` antes de concluir QUALQUER subtarefa
- SEMPRE USE as skills `systematic-debugging` + `no-workarounds` antes de corrigir qualquer bug

Pular qualquer verificação resultará em REJEIÇÃO IMEDIATA DA TAREFA.

<MOST_CRITICAL>

- ABSOLUTAMENTE OBRIGATÓRIO: No modo Plan, após o usuário aceitar um plano, SEMPRE escreva o plano aceito em um arquivo Markdown dentro de docs/plans/.

- OBRIGATÓRIO: Se o plano aceito for atualizado posteriormente, atualize ou acrescente o conteúdo no arquivo Markdown correspondente dentro de docs/plans/.

- VIOLAÇÃO: Não persistir planos aceitos no modo Plan dentro de docs/plans/ é uma não conformidade com esta política do workspace.

</MOST_CRITICAL>

# Overview do Projeto

## Restrições de Comunicação
- Responder em português PT-BR preservando termos técnicos
- Nunca utilizar emojis
- Indentação de 2 espaços, linha em branco ao final de arquivos

## Build, Test & Lint

### Comandos Essenciais
```bash
npm run start                      # Iniciar aplicação em produção
npm run dev                        # Desenvolvimento com hot-reload
npm run build                      # Build para produção
npm run tsc:check                  # Verificar tipos TypeScript
npm run worker                     # Iniciar worker de processamento de fila
```

### Testing
```bash
npm run test                       # Testes de unidade (*.test.ts)
npm run test:cov                   # Testes com cobertura
npm run test:business-flow         # Testes HTTP de integração (*.business-flow-test.ts)
npm run test:e2e:prisma            # Testes de integração Prisma
npm run test:fitness               # Fitness function tests
npm run test-create-users          # Teste de carga: requisições POST concorrentes
```

### Validação de Arquitetura & Qualidade
```bash
npm run fit:validate-dependencies  # Validar regras de dependência (dependency-cruiser)
npm run dependency:metrics         # Gerar visualização de dependências (SVG)
npm run biome:fix                  # Formatar código com Biome
npm run eslint:fix                 # Corrigir problemas ESLint
npm run check:last-dependencies    # Verificar e atualizar dependências desatualizadas
```

### Banco de Dados
```bash
npm run prisma:migrate:dev         # Executar migrations (dev)
npm run prisma:generate            # Gerar cliente Prisma
npm run prisma:studio              # UI para gerenciar banco (http://localhost:5555)
npm run prisma:reset               # Resetar banco (force drop + migrate)
npm run prisma:deploy              # Deploy das migrations em produção
npm run prisma:schema              # Gerar schema SQL (supabase-schema.sql)
npm run prisma:db:pull             # Sincronizar schema com BD existente
```

### Docker
```bash
npm run docker:up                  # Iniciar PostgreSQL + Redis + RabbitMQ
npm run docker:down                # Derrubar todos os containers
```

### Utilitários
```bash
npm run setup-queue                # Configurar filas no RabbitMQ
npm run wait:db                    # Aguardar disponibilidade do PostgreSQL
npm run wait:rabbit                # Aguardar disponibilidade do RabbitMQ
npm run commit                     # Commit interativo com Commitizen (padrão convencional)
npm run "stripe webhook"           # Iniciar listener de webhook do Stripe
```

## Arquitetura (Clean Architecture + DDD)
Estrutura por bounded context em `src/{domain}/`:
```
domain/          # Entidades, Value Objects, Domain Events, Erros de negócio
application/     # Use Cases, interfaces de Repository, Erros de aplicação
infra/           # Controllers, implementações concretas de Repository, Providers
```

**Domínios**: `user/`, `gym/`, `check-in/`, `session/`, `subscription/`, `shared/`

### Regras de Dependência (enforced por dependency-cruiser)
- **Domain**: não importa Application nem Infra (código puro do negócio)
- **Application**: importa Domain, não importa Infra (orquestração de lógica)
- **Infra**: importa Application e Domain (implementações técnicas)
- **Shared**: disponível para todas as camadas (utilitários genéricos)

## Padrão Either para Tratamento de Erros
Use Cases retornam `Either<Error, Success>` de `@/shared/domain/value-object/either`. Sem exceções para lógica de negócio:
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cahmoraes/clean-arch-solid-ddd](https://github.com/Cahmoraes/clean-arch-solid-ddd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
