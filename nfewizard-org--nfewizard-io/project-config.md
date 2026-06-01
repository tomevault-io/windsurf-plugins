---
trigger: always_on
description: Instruções obrigatórias para agentes de IA que atuem neste monorepo. **Leia primeiro
---


# Copilot Instructions — NFeWizard-io

Instruções obrigatórias para agentes de IA que atuem neste monorepo. **Leia primeiro
[.github/nfewizard-context.md](.github/nfewizard-context.md)** para contexto funcional e
arquitetural. Este arquivo define **regras de engenharia, arquitetura e segurança**
que devem ser seguidas em qualquer alteração de código, documentação ou configuração.

---

## 1. Princípios gerais

1. **Idioma**: responda ao usuário em **português do Brasil**. Código, identificadores, mensagens de log e commits permanecem em português ou inglês conforme o padrão já existente no arquivo tocado — **nunca** misture idiomas dentro do mesmo símbolo.
2. **Mínima intervenção**: faça apenas o que foi pedido. Não refatore, não renomeie, não "melhore" código fora do escopo da tarefa.
3. **Retrocompatibilidade é lei**: a API pública do pacote `nfewizard-io` (classe `NFeWizard` e métodos `NFE_*`/`NFCE_*`/`CTE_*`) **não pode quebrar** sem Changeset major + entrada em [BREAKING_CHANGES.md](BREAKING_CHANGES.md).
4. **Leia antes de editar**: nunca altere arquivos sem antes inspecioná-los; nunca invente assinaturas, tipos ou nomes de pacotes.
5. **Use os tipos existentes**: importe sempre de `@nfewizard/types/shared` ou `@nfewizard/types/nfe` — não duplique definições.

---

## 2. Regras de arquitetura

### 2.1 Respeite as fronteiras dos pacotes
- Grafo permitido: `types → shared → {nfewizard-io, nfce, nfse, cte, danfe}`.
- **Proibido** import circular ou de "irmão" (ex.: `nfce` importando `nfewizard-io`).
- Cross-package **sempre** via aliases `@nfewizard/types`, `@nfewizard/shared`, `@nfewizard/danfe`, etc. **Nunca** caminhos relativos `../../packages/...`.

### 2.2 Estrutura de uma nova operação de webservice
Ao adicionar uma operação fiscal, replique o padrão existente:

1. `packages/<pkg>/src/<dominio>/services/<NomeOperacao>/<NomeOperacao>Service.ts`
   - Classe que estende implicitamente o protocolo de service: monta XML, resolve URL, define `contentType`, faz a chamada via `BaseNFE.callWebService`.
2. `packages/<pkg>/src/<dominio>/operations/<NomeOperacao>/<NomeOperacao>.ts`
   - Classe com método público `Exec(...)` que orquestra: validação de input → chamada do service → parsing → retorno tipado.
3. Tipo de payload e tipo de retorno em `packages/types/src/<dominio>/`.
4. Método na facade (`NFeWizard`, `NFCEWizard`, etc.) **e** na interface `*Impl` correspondente.
5. Atualizar `packages/shared/src/config/*ServicosUrl.json` se houver nova URL/UF.
6. Adicionar testes em `src/__tests__/` (unit + factory).
7. Criar Changeset (`pnpm changeset`) descrevendo o impacto.

### 2.3 Padrões obrigatórios
- **Facade** (`NFeWizard*`) só delega para o `*Service`. Não coloque lógica de negócio na facade.
- **Service** estende `BaseNFE` (NF-e/NFC-e/CT-e) ou `BaseNFSe` (NFS-e). **Não** crie nova hierarquia paralela.
- **Template Method**: implemente `gerarXml()` na subclasse; **nunca** sobrescreva `callWebService()` sem motivo documentado.
- **Builder**: construa XMLs apenas via `XmlBuilder`. Nunca concatene strings XML manualmente.
- **Strategy de validação**: respeite `useForSchemaValidation`. Não force JDK em código novo.
- **Proxy de validação**: novos métodos da facade são **automaticamente** interceptados por `NFeWizardService` para checagem de `Environment.isLoaded`. Não burle esse Proxy.
- **DI por construtor**: novos services recebem `(environment, utility, xmlBuilder, axios, saveFiles, gerarConsulta)` — não crie `new` de dependências dentro do service.

### 2.4 Convenções de import
- Imports de arquivos locais **sempre com extensão `.js`** (mesmo em `.ts`), por exigência do ESM Node.
- Cabeçalho de licença GPL-3.0 obrigatório no topo de todo `.ts` novo (copie de qualquer arquivo existente).

---

## 3. Padrões de código

- **TypeScript estrito**: sem `any` implícito; quando precisar de `any`, justifique em comentário curto.
- **Nomenclatura**:
  - Classes/Operations: `PascalCase` com prefixo de domínio (`NFE`, `NFCE`, `NFSe`, `CTE`).
  - Métodos públicos da facade: `<DOMINIO>_<Operacao>` (ex.: `NFE_ConsultaProtocolo`).
  - Interfaces sufixadas com `Impl` (`NFeWizardImpl`, `SaveFilesImpl`).
  - Arquivos de tipos puros: nome casa com a entidade (`NFEAutorizacao.ts`, `EmailConfig.ts`).
- **Funções pequenas e coesas**: uma operação por método público; helpers privados quando algo passar de ~40 linhas.
- **Sem dead code**: não deixe imports não usados, variáveis comentadas ou `console.log` esquecidos.
- **Sem comentários óbvios**: comente apenas o "porquê", nunca o "o quê".
- **Ordenação**: imports externos → imports `@nfewizard/*` → imports relativos.

---

## 4. Erros, logs e observabilidade

- **Toda operação pública** deve estar dentro de `try/catch`:
  ```ts
  try {
    // ...
    return response;
  } catch (error: any) {
    logger.error('', error, { context: 'NFE_NomeMetodo' });
    throw new Error(`NFE_NomeMetodo: ${error.message}`);
  }
  ```
- **Nunca use `console.log`/`console.error` em código de produção** — use `logger` (winston) com `context` semântico. As únicas exceções são os `console.log` já existentes na facade `NFeWizardService` que imprimem `xMotivo` para DX (mantenha o estilo se editar essas funções).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nfewizard-org/nfewizard-io](https://github.com/nfewizard-org/nfewizard-io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
