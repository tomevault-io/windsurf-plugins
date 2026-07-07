---
trigger: always_on
description: Este arquivo é contexto persistente para trabalho técnico em `/game` e `/tools`.
---

# AZOTH: BREACH — Contexto para Sessões de Implementação

Este arquivo é contexto persistente para trabalho técnico em `/game` e `/tools`.
Ele não é fonte de cânone — apenas orienta sessões futuras a encontrar rapidamente
a documentação que já manda em qualquer decisão de implementação.

## Precedência documental

`/docs` é a fonte de verdade e tem precedência sobre qualquer instrução de sessão
ou suposição feita aqui. Em caso de divergência, os documentos vencem. Se um
conflito exigir alterar `/docs`, siga as regras de versionamento/changelog do
Documento 00.3 e sinalize explicitamente — não resolva em silêncio.

Ordem de leitura obrigatória antes de qualquer trabalho técnico:

1. `docs/00_constituicao/AZOTH_00.3_Estrutura_Mestre_da_Documentacao_v2.40.md` — hierarquia de precedência e regras de versionamento
2. `docs/06_tecnologia/README.md` — papel da série de tecnologia
3. `docs/06_tecnologia/AZOTH_06.0_Estrategia_Tecnica_e_Stack_Inicial_v0.2.md` — stack e princípios técnicos obrigatórios
4. `docs/06_tecnologia/AZOTH_06.1_Arquitetura_Inicial_da_Build_e_Estrutura_de_Pastas_v0.1.md` — arquitetura da build e estrutura de pastas
5. `docs/06_tecnologia/AZOTH_06.2_Pacote_de_Arranque_do_M1_v0.2.md` — tarefas técnicas `TK-M1-xxx` e critérios de aceite
6. `docs/05_produto/AZOTH_05.11_Backlog_Executavel_da_Vertical_Slice_v0.1.md` — backlog `BL-xxx`
7. `docs/05_produto/AZOTH_05.12_Milestones_de_Implementacao_da_Vertical_Slice_v0.2.md` — milestones `M1`–`M6`

## Stack (fechada — não reabrir sem gatilho documental)

- **Engine:** Godot 4.x, ramo estável (instalado: 4.7.stable)
- **Linguagem:** GDScript
- **Formato:** 2D, retrato vertical
- **Plataforma prioritária:** iOS (Android é secundário/futuro)
- **Desktop:** apenas debug/iteração local
- **Execução:** local-first até o fechamento de `M3` — sem backend, conta ou sync em nuvem
- **Dados:** JSON (conteúdo e save local), JSONL (telemetria mínima local)

Reabrir a stack só é permitido pelos gatilhos listados na seção 7 do Documento 06.0.

## Estrutura de pastas

```text
/game
  project.godot
  icon.svg
  /assets
    /audio /sprites /ui /vfx /temp
  /data
    /codex /ets /loadouts /operations /regions /rewards /state_templates
  /scenes
    /app /hub /loadout /operations /reports /research /shared
  /scripts
    /core
    /features
      /codex /hub /loadout /operations /reports /research /rewards
    /services
    /state
  /tests
/tools
  /content
  /exports
```

Cada pasta reflete uma camada da arquitetura definida em `06.1`:
aplicação (`core`), estado (`state`), conteúdo (`data`), features (`scenes` +
`scripts/features`), serviços (`services`). Placeholders devem ser substituíveis
sem reescrever a arquitetura base (princípio 2.5 do Documento 06.1).

## Configuração de projeto atual

- `run/main_scene`: `res://scenes/app/app_shell.tscn` — App Shell real
  (`TK-M1-004`). Faz roteamento básico via um container `ScreenHost`.
  Qualquer tela filha pode emitir `navigate_to(scene_path, context)` para
  pedir navegação com contexto (ex.: `operation_id`) — o App Shell conecta
  esse sinal automaticamente e chama `set_context(context)` na tela seguinte
  antes dela entrar na árvore, se o método existir (padrão reaproveitável
  por Loadout, entrada em operação etc.).
- Hub (`TK-M1-007`) embute um `OperationBoard` (`scenes/hub/operation_board.tscn`)
  que lista as operações de `data/operations/*.json` via `OperationLoader`
  (`scripts/services/operation_loader.gd`), diferenciando disponível/bloqueada
  a partir de `SliceState.is_operation_unlocked`. Selecionar uma operação
  disponível navega para o Briefing repassando `operation_id`. Acesso a Codex
  e Mesa de Pesquisa ainda não existem no Hub — entram em tarefas de
  milestones posteriores (`M3`).
- Briefing (`TK-M1-008`, `scenes/operations/briefing_screen.tscn`) recebe o
  `operation_id` via `set_context`, busca a operação com `OperationLoader` e
  exibe nome, sublocal, objetivo, risco e recompensa prevista (`rewards_guaranteed`
  formatado). O botão "Seguir" navega para `scenes/loadout/loadout_screen.tscn`
  repassando o mesmo `operation_id`.
- Dados de ET e loadout (`TK-M1-009`) em `data/ets/*.json` (4 ETs-base,
  `role` transcrito de `05.3` §"Biblioteca Base de ETs") e `data/loadouts/*.json`
  (3 presets — Contenção/Agressão/Investigação, `ets` e `description_short`
  de `05.9` §6.3-6.4). `upgrade_path_basic` por ET é decisão de implementação:
  associei cada ET à "melhoria preferencial" do build onde ela aparece
  primeiro listada como central (Selagem Parcial → Contenção, Decomposição
  Dirigida → Agressão, Análise de Vestígio → Investigação); Cristalização
  Controlada não tem build primária nos documentos (é secundária em
  Contenção e Agressão) — tratada como utilidade cross-build. Loaders em
  `scripts/services/et_loader.gd` e `loadout_loader.gd`, ambos delegando a
  `json_directory_loader.gd` (extraído de `operation_loader.gd` para evitar
  triplicar a mesma leitura de diretório).
- Loadout real (`TK-M1-010`, `scenes/loadout/loadout_screen.tscn`) exibe o
  preset ativo (nome, papel/foco tático, ETs com nome+papel) e permite
  trocar entre os três presets via botões dinâmicos — a troca atualiza
  `SliceState.active_loadout` imediatamente (sem estado local paralelo,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itsmeluan/azoth-breach](https://github.com/itsmeluan/azoth-breach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
