---
trigger: always_on
description: Você está na **raiz** do monorepo Remote Pi. Esta pasta é exclusivamente para **planejamento**.
---

# Remote Pi — Orquestrador

Você está na **raiz** do monorepo Remote Pi. Esta pasta é exclusivamente para **planejamento**.

## O que fazer aqui

- Ler e escrever em `plan/NN-<slug>.md` (ex: `plan/03-protocol.md`)
- Discutir arquitetura, decisões de produto, trade-offs
- Refinar planos existentes baseado em feedback
- Indicar qual subprojeto recebe a próxima implementação

## O que NÃO fazer aqui

- Não editar código em `app/`, `pi-extension/`, `relay/`, `site/`
- Não rodar comandos de build/test dos subprojetos a partir daqui
- Para implementar algo, despache via `cmux send` pro pane do subprojeto
  alvo (ver seção [Panes deste workspace cmux](#panes-deste-workspace-cmux)
  abaixo). Só peça pro usuário abrir terminal novo se o pane sumiu.

## Estrutura

Veja [README.md](./README.md) para visão geral e [plan/](./plan/) para os planos.

## Decisões já tomadas

Antes de propor mudança de direção (arquitetura, pareamento, escopo, UI, segurança),
leia [`plan/00-decisions.md`](./plan/00-decisions.md). Esse arquivo lista decisões
fechadas em conversa exploratória e **não devem ser revisitadas sem evidência forte**.

Se ainda assim quiser revisitar, abra discussão explícita — não mude silenciosamente.

## Convenções de planos

- Numeração sequencial: `01-bootstrap.md`, `02-ai-orchestration.md`, ...
- Cada plano tem: Contexto, Estrutura esperada, Passos com critério de aceite, DoD, Próximos planos
- Planos descrevem **o que** + **como verificar**, não o código completo
- Pseudocódigo ou comandos exatos são bem-vindos; implementação real fica no subprojeto

## Quando promover um plano a implementação

Quando o plano tem aceite do usuário e os passos estão concretos o suficiente
para um agente executar, abra Claude no subprojeto alvo e passe o plano como
contexto. O agente daquele subprojeto seguirá sua própria persona.

## Scouts disponíveis

Para fotografar o estado de qualquer subprojeto antes de planejar, invoque os
subagents Scout em paralelo via `Task` — eles são read-only e reportam em
formato fixo:

- `scout-app` — Flutter (`app/`)
- `scout-pi-extension` — Node/TS (`pi-extension/`)
- `scout-relay` — Rust (`relay/`)
- `scout-site` — NextJS (`site/`)

Dispare múltiplos numa única mensagem para rodar em paralelo. Cada reporte
volta com Stack & versões, Dependências, Estrutura, Saúde (lint/build/testes)
e Smells detectados.

## Panes deste workspace cmux

Este workspace ("Remote PI") tem 4 panes dedicados — um por subprojeto — e este
Orquestrador. Cada pane já tem um `claude` rodando em sessão própria. **Use os
panes existentes em vez de pedir pro usuário abrir terminal novo.**

| Pane (título) | Subprojeto (cwd) |
|---|---|
| `App` | `app/` |
| `Relay` | `relay/` |
| `Extension` | `pi-extension/` |
| `Site` | `site/` |
| `Orquestrador` (você) | raiz do monorepo |

> **Nunca hardcode surface IDs nesta documentação.** Eles mudam a cada
> bootstrap dos panes. Sempre resolva por título via `cmux tree`.

### Descobrir o surface ID por título

```bash
# helper: imprime o surface:N do pane com título <Nome>
surface_of() {
  cmux tree | awk -v t="$1" '
    $0 ~ "\""t"\"" {
      for (i = 1; i <= NF; i++) if ($i ~ /^surface:/) { print $i; exit }
    }
  '
}

surface_of Extension   # imprime o surface:N atual
```

### Despachar tarefa pra um pane (modo orquestrado)

**Sempre** use o wrapper `scripts/cmux-dispatch.sh`. Ele resolve o surface
pelo título, injeta `[ORCH:<task-id>]`, e envia + Enter num call:

```bash
scripts/cmux-dispatch.sh Extension 03-ts-codec "Implemente passo 3 do plan/03-protocol.md"
```

Por que o wrapper existe: o gatilho `[ORCH:<task-id>]` é o que faz cada
agente entrar em modo orquestrado (ler `.orchestration/INSTRUCTIONS.md`,
respeitar cwd-only, não comitar). Sem o marker, o agente responde em modo
solo. Mandar `cmux send` direto pra um pane de agente é fácil de errar
(esqueci o marker em conversas anteriores e o user cobrou). **Use o wrapper.**

Quando NÃO usar o wrapper:
- Conversa exploratória direta ("qual sua função?", "o que você vê em X?")
- Debug, comando shell, retomar claude — modo solo é apropriado
- Nesses casos `cmux send --surface "$(surface_of <Nome>)" -- "<texto>"` +
  `cmux send-key --surface "$(surface_of <Nome>)" enter` (Enter separado
  porque `\n` vira newline multilinha no prompt do claude, não submit)

### Aguardar o worker terminar (hook `agent.hook.Stop`)

Pré-requisito: o pane alvo precisa ter sido lançado via `cmux claude-teams`
(o wrapper que injeta os hooks Claude Code). O `cmux-bootstrap-agents.sh`
já faz isso desde 2026-05-24. Panes lançados com `claude` puro **não emitem
hooks** — orquestrador fica refém de "feito" digitado pelo humano.

Pra checar se um pane está no formato certo, rode no orquestrador:

```bash
cmux events --category agent --limit 1 --no-heartbeat --no-ack &
PID=$!; sleep 2; kill $PID 2>/dev/null
```

Se voltar silêncio em 2s sem nenhum evento, os panes estão em modo solo —
ofereça rodar `scripts/cmux-close-agents.sh` + `scripts/cmux-bootstrap-agents.sh
--resume` pra reativar com hooks.

**Forma preferida** — dispatch com `--wait` bloqueia até o worker emitir Stop:

```bash
scripts/cmux-dispatch.sh --wait Extension 25-wave-x "..."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacobaraujo7/remote_pi](https://github.com/jacobaraujo7/remote_pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
