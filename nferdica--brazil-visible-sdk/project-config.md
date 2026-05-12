---
trigger: always_on
description: > Este documento e o briefing completo para implementacao do SDK. Qualquer sessao Claude Code neste repositorio deve comecar lendo este arquivo.
---

# Brazil Visible SDK — AGENTS.md

> Este documento e o briefing completo para implementacao do SDK. Qualquer sessao Claude Code neste repositorio deve comecar lendo este arquivo.

## Visao Geral

SDK TypeScript unificado para acesso a 93+ fontes de dados publicos brasileiros. Oferece uma interface programatica unica sobre APIs REST, downloads CSV, FTP e portais do governo federal.

**Proposta de valor**: Ninguem unificou o acesso a dados publicos brasileiros num unico pacote. Existem SDKs isolados por orgao (python-bcb, pysus, sidra), mas nenhum em TypeScript/JavaScript que cubra o ecossistema inteiro. Este SDK resolve isso — e alcanca o maior ecossistema de desenvolvedores do mundo.

**Repositorio irmao**: [Brazil Visible](https://github.com/nferdica/brazil-visible) — catalogo de documentacao com 92 APIs mapeadas em frontmatter YAML estruturado (url_base, formato_dados, campos_chave, tipo_acesso, autenticacao). Esse frontmatter e a base de configuracao do SDK.

---

## Publico-Alvo

1. **Jornalistas de dados** — Abraji, Agencia Publica, Fiquem Sabendo. Querem cruzar bases sem perder horas com boilerplate.
2. **Pesquisadores academicos** — ciencia politica, economia, saude publica. Scripts rapidos em Node/Deno/Bun.
3. **Desenvolvedores civicos** — civic hackers. Querem construir ferramentas de fiscalizacao com tipagem forte e ecossistema npm.
4. **Desenvolvedores fullstack** — ja usam TypeScript no front e no back, agora podem acessar dados publicos nativamente.

---

## Arquitetura

### Principios

- **Uma interface, muitas fontes**: `import { ibge, bcb } from '@brazilvisible/sdk'` — independente se a fonte e REST, CSV ou FTP
- **Typed-first**: Toda resposta retorna arrays tipados `T[]` com interfaces completas para cada fonte
- **Zero config para 80% dos casos**: 80% das APIs nao exigem autenticacao
- **Zero deps HTTP**: Usa `fetch` nativo (Node >=18) — sem axios, got ou undici no core
- **Fail-fast com mensagens claras**: Se uma API esta fora do ar, erro descritivo com link para o health check
- **Tree-shakeable**: Importar `ibge` nao puxa codigo de `datasus`

### Estrutura do Pacote

```
src/
  index.ts                # Re-exports publicos (ibge, bcb, cgu, etc.)
  client.ts               # Cliente HTTP base (fetch, retries, rate limiting, user-agent)
  types.ts                # Tipos compartilhados (BVResponse, Pagination, etc.)
  cache.ts                # Cache local opcional (respostas, downloads)
  download.ts             # Utilitarios para download + descompressao (ZIP, GZ)
  parsers.ts              # Parsers de formato (CSV, JSON, XML)
  errors.ts               # Hierarquia de excecoes (BVError, SourceOfflineError, AuthRequiredError)
  config.ts               # Configuracao global (API keys, timeouts)
  sources/
    index.ts              # Re-exports de todos os sources
    base.ts               # Interface abstrata Source
    ibge.ts               # IBGE (Sidra, Agregados, Censos)
    bcb.ts                # Banco Central (SGS series, IFData)
    cgu.ts                # CGU Portal da Transparencia (CEIS, CNEP, CEPIM, contratos, servidores, emendas, viagens)
    receita.ts            # Receita Federal (CNPJ, QSA, Estabelecimentos, Simples)
    tse.ts                # TSE (candidaturas, resultados, prestacao de contas, bens, filiados, boletins, eleitorado)
    tesouro.ts            # Tesouro Nacional (SICONFI, SIAFI, SIOP)
    inep.ts               # INEP/Educacao (ENEM, Censo Escolar, Censo Superior, FNDE)
    datasus.ts            # DATASUS (TabNet, CNES, SIH, SIM, SINAN, SINASC)
    cnj.ts                # CNJ (DataJud, BNMP, SisbaJud, Justica em Numeros)
    ambiente.ts           # Meio Ambiente (PRODES, DETER, CAR, focos calor, IBAMA, UC, recursos hidricos)
    trabalho.ts           # Trabalho (RAIS, CAGED)
    previdencia.ts        # Previdencia (INSS, PREVIC)
    mercado.ts            # Mercado Financeiro (CVM DFP/ITR, CVM Administradores, CVM Fatos Relevantes, B3)
    ipea.ts               # IPEA (IpeaData)
    transportes.ts        # Transportes (ANAC, PRF, DNIT, ANTT, DENATRAN)
    reguladoras.ts        # Agencias Reguladoras (ANATEL, ANEEL, ANP, ANVISA)
    geo.ts                # Dados Geoespaciais (IBGE Geociencias, CPRM, INCRA, INDE, INPE)
    diarios.ts            # Diarios Oficiais (DOU, DOEs estaduais)
    governamentais.ts     # APIs Governamentais (CADIN, SIAPE, SIORG)
    seguranca.ts          # Seguranca Publica (SINESP)
    outros.ts             # Outros (ANS, ANTAQ, ANCINE)
    portais.ts            # Portais Centrais (Portal Dados Abertos, Base dos Dados, Tesouro Transparente, Portal Transparencia)
tests/
  sources/
    ibge.test.ts
    bcb.test.ts
    cgu.test.ts
    ...
  client.test.ts
  parsers.test.ts
README.md                 # Documentacao com exemplos de uso
```

### Interface Source (base abstrata)

Toda fonte implementa esta interface:

```typescript
import type { BVClient } from "../client";

export interface SourceConfig {
  client?: BVClient;
}

export abstract class Source {
  protected client: BVClient;

  constructor(config?: SourceConfig) {
    this.client = config?.client ?? getDefaultClient();
  }

  /** Nome legivel da fonte (ex: 'IBGE Agregados'). */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nferdica/brazil-visible-sdk](https://github.com/nferdica/brazil-visible-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
