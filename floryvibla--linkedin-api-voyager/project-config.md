---
trigger: always_on
description: > Guia operacional para agentes de IA trabalhando no repositório `@florydev/linkedin-api-voyager`.
---

# AGENTS.md

> Guia operacional para agentes de IA trabalhando no repositório `@florydev/linkedin-api-voyager`.

## O que é este repositório

Este repo mantém uma biblioteca TypeScript para acessar endpoints internos do LinkedIn Web, conhecidos informalmente como **Voyager API**. Ela **não usa a API oficial do LinkedIn** e depende de uma sessão autenticada do navegador, reaproveitando cookies como `li_at` e `JSESSIONID`.

O pacote publicado no npm é `@florydev/linkedin-api-voyager`. A saída publicada é a pasta `lib/`, gerada a partir de `src/`.

## O que é LinkedIn Voyager

`Voyager` é o conjunto de endpoints internos usados pelo frontend web do LinkedIn. Na prática, este repo faz três coisas:

1. Inicializa um cliente HTTP autenticado com headers/cookies parecidos com o navegador.
2. Chama endpoints REST e GraphQL internos do LinkedIn.
3. Normaliza respostas cheias de `included`, URNs e estruturas aninhadas para objetos mais simples.

## Regras essenciais para qualquer agente

- Rode tudo em **Node.js**, nunca no browser.
- Sempre inicialize `Client({ JSESSIONID, li_at })` antes de qualquer chamada.
- Nunca commite cookies, `.env`, `linkedin_cookies.json` ou credenciais hardcoded.
- Nunca edite `lib/` manualmente. Edite `src/` e gere build.
- Nunca trate `src/teste.ts` como exemplo de produção: ele é um playground local e hoje contém credenciais hardcoded.
- Se adicionar nova feature pública, atualize `src/index.ts`, `README.md` e `skills/references/*.md`.
- Se mexer no `worker-service`, trate-o como um subprojeto local separado da lib principal.

## Comandos principais

- Instalar dependências raiz: `yarn`
- Build da lib: `yarn build`
- Playground local atual: `yarn dev`
- Build do worker: `yarn --cwd worker-service build`
- Rodar worker compilado: `yarn --cwd worker-service start`

## Como a arquitetura funciona

### 1. Núcleo HTTP

Arquivo central: `src/core/config.ts`

- `Client()` cria a instância global `apiInstance`.
- `fetchDataApi()` faz `GET` em `/voyager/api${endpoint}`.
- `fetchDataClient()` faz `GET` direto e retorna `location` quando o interesse é inspecionar redirect.

Sem `Client()`, o resto da lib quebra.

O diretório `src/core/` contém também `errors.ts` (classes de erro), `utils.ts` (parsers compartilhados) e `types.ts` (tipos compartilhados globais).

### 2. Camada de domínio

Cada módulo em `src/modules/<nome>/` encapsula uma parte do LinkedIn, com seu próprio `types.ts` e `index.ts` barrel:

- `src/modules/user/` -> perfis, about, experiências, contato, skills, educação, certificações, projetos, voluntariado, honras, cursos, organizações, publicações, patentes, testes/scores, idiomas, recomendações
- `src/modules/company/` -> empresa por slug + busca funcionários/ex-funcionários
- `src/modules/posts/` -> post, posts de usuário, comentários
- `src/modules/search/` -> busca geral e busca de pessoas
- `src/modules/messages/` -> inbox e mensagens
- `src/modules/network/` -> convites recebidos/enviados
- `src/modules/sse/` -> realtime/SSE

### 3. Parsing e normalização

Arquivo crítico: `src/core/utils.ts`

Este arquivo existe porque o LinkedIn devolve muitos dados em formato normalizado:

- entidades separadas em `included`
- referências em chaves como `*entityResult`
- imagens como `rootUrl + artifact`
- URNs como `urn:li:fsd_profile:...`

Helpers como `resolveReferences`, `extractDataWithReferences`, `extractFields`, `getIdFromUrn` e `normalizeRawOrganization` fazem esse trabalho pesado.

### 4. Tipagem

Não há mais um `types.ts` monolítico. A tipagem é distribuída:

- `src/core/types.ts` -> tipos compartilhados globais (Paging, URNs de imagem, etc)
- `src/modules/<dominio>/types.ts` -> tipos específicos do domínio

Se um endpoint novo entrar, coloque os tipos no `types.ts` do módulo correspondente. Se for compartilhado, coloque em `core/types.ts`.

### 5. Superfície pública

Arquivo: `src/index.ts`

Tudo que o pacote expõe publicamente precisa ser reexportado aqui. O barrel agregador reexporta de cada `src/modules/<nome>/index.ts`.

## Estrutura do repositório

### Raiz

- `package.json` -> pacote npm principal, build via `tsc`, publicação de `lib/**/*`
- `tsconfig.json` -> compila `src` para `lib`, exclui `lib`
- `README.md` -> documentação pública da biblioteca
- `AGENTS.md` -> este arquivo
- `yarn.lock` / `pnpm-lock.yaml` -> lockfiles
- `.gitignore` -> ignora `lib`, `.env`, `src/teste.ts`, `worker-service`, cookies e lockfile de npm

### `src/`

```
src/
├── index.ts                # barrel principal / exports públicos
├── teste.ts                # playground local (ignorado no git)
├── core/                   # compartilhado (não pertence a nenhum domínio)
│   ├── config.ts           # cliente HTTP, headers, cookies, fetch helpers
│   ├── errors.ts           # classes de erro
│   ├── utils.ts            # parsing, URNs, imagens, normalização
│   └── types.ts            # tipos compartilhados globais
└── modules/                # camada de domínio por assunto
    ├── user/               # APIs de perfil (15+ arquivos)
    │   ├── index.ts
    │   ├── types.ts
    │   └── user-*.ts
    ├── company/            # empresa + pessoas por empresa
    │   ├── index.ts
    │   ├── types.ts
    │   ├── company.ts
    │   └── company-people.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Floryvibla/linkedin-api-voyager](https://github.com/Floryvibla/linkedin-api-voyager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
