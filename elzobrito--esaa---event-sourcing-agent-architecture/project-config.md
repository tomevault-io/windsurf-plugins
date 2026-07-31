---
trigger: always_on
description: > Versão operacional curta para runners. Em divergência, os artefatos canônicos em `.roadmap/` prevalecem.
---

# CLAUDE.md — Contrato operacional Codex/ESAA

> Versão operacional curta para runners. Em divergência, os artefatos canônicos em `.roadmap/` prevalecem.
> O ESAA não usa MCP. Use a CLI ESAA: `python -m esaa`.

## 1. Autoridade e termos

ESAA é o protocolo de governança. Harness executa. Orchestrator é o single writer do event store. Agente emite intenções válidas, nunca escreve diretamente no store.

Fontes canônicas:
- Event store: `.roadmap/activity.jsonl`.
- Projeções/read models: `.roadmap/roadmap.json`, `.roadmap/issues.json`, `.roadmap/lessons.json`.
- Contratos: `.roadmap/AGENT_CONTRACT.yaml`, `.roadmap/ORCHESTRATOR_CONTRACT.yaml`, `.roadmap/agent_result.schema.json`, `.roadmap/RUNTIME_POLICY.yaml`.

Read models são derivados do event store. Não edite manualmente `roadmap.json`, `issues.json`, `lessons.json` ou `activity.jsonl`.

## 2. CLI e runner

Em workspaces publicados, o pacote Python é `esaa-core`, mas o módulo/comando é `esaa`.

Comandos úteis:

```powershell
python -m esaa --version
python -m esaa --root . verify
python -m esaa --root . eligible
python -m esaa --root . roadmap status --detail
```

Todo comando que escreve no event store deve identificar o runner:

```powershell
# Informe o identificador real do runner que está executando a ação.
python -m esaa --root . --runner <runner_id> submit --actor agent-spec output.json

# Ou configure o runner uma vez por sessão:
$env:ESAA_RUNNER_ID = "<runner_id>"
```

Regras:
- Substitua `<runner_id>` pelo identificador exato do runner em execução.
- Use `--runner <runner_id>` ou `ESAA_RUNNER_ID=<runner_id>` em `submit`, `task create`, `init` e `run`.
- Não inclua o campo `runner` no JSON do agente; o Orchestrator o acrescenta ao evento.
- Runners registrados: `claude-cowork`, `claude-code`, `codex`, `human-terminal` e `unattended`.
- Com `runner_validation: strict`, um identificador não registrado é rejeitado com `RUNNER_UNKNOWN`.
- Novos runners devem ser registrados em `.roadmap/agents_swarm.yaml` antes de serem usados em modo `strict`.

## 3. Concorrência

Até locks com metadados/read-after-write estarem garantidos no workspace alvo:
- Um runner por vez neste workspace.
- Antes de escrever, se existir `.roadmap/activity.jsonl.lock`, pare e pergunte ao usuário.
- Se encontrar `STORE_LOCK_TIMEOUT`, `JSONL_INVALID` ou `EVENT_SEQ_*`, pare e reporte.
- Após escrita governada, rode `python -m esaa --root . verify`.
- Não reivindique tarefa atribuída a outro runner.

## 4. Modos de operação

### Read-only

Use para análise, diagnóstico, explicação ou inspeção. Não emita `claim`, `complete`, `review`, nem `file_updates`.

Feche com:

```text
- Task ID: N/A
- Summary: <o que foi feito>
- Changed files: Nenhum.
- Tests run: Nenhum.
- ESAA verification: Not run.
- ESAA closure status: Not applicable — read-only request.
- Blockers, if any: <se houver>
```

### Execução governada

Use quando o usuário pede implementar/corrigir/gerar artefatos sob ESAA.

Regras:
- Two-step obrigatório: uma invocação para `claim`, outra para `complete`.
- Exatamente uma `activity_event` por output.
- JSON puro, sem markdown, sem texto fora do envelope.
- `prior_status` sempre presente e coerente.
- `file_updates` apenas com `action=complete`.
- O agente não aplica efeitos finais diretamente; envia `file_updates` para o Orchestrator.
- Nunca reabra ou modifique tarefa `done`; reporte issue.

Boundaries por `task_kind` vêm de `.roadmap/AGENT_CONTRACT.yaml#boundaries.by_task_kind`.

## 5. Decision tree

1. `task_status == todo` -> emitir `claim`.
2. `task_status == in_progress` e `assigned_to` é seu actor -> executar e emitir `complete`.
3. `task_status == review` -> apenas `agent-qa` emite `review`.
4. `task_status == done` -> emitir `issue.report`; done é imutável.
5. Se uma lesson ativa inviabiliza o output, emitir `issue.report`.
6. Se houver dependência ausente, boundary impossível, contexto insuficiente ou lock divergente, emitir `issue.report`.

## 6. Envelopes canônicos

### claim

```json
{
  "activity_event": {
    "action": "claim",
    "task_id": "T-000",
    "prior_status": "todo"
  }
}
```

### complete com conteúdo completo

```json
{
  "activity_event": {
    "action": "complete",
    "task_id": "T-000",
    "prior_status": "in_progress",
    "notes": "Resumo objetivo.",
    "verification": {
      "checks": ["teste ou inspeção executada"]
    }
  },
  "file_updates": [
    {
      "path": "docs/spec/T-000.md",
      "content": "# Conteúdo completo\n"
    }
  ]
}
```

### complete com edits

```json
{
  "activity_event": {
    "action": "complete",
    "task_id": "T-000",
    "prior_status": "in_progress",
    "verification": {
      "checks": ["edição validada"]
    }
  },
  "file_updates": [
    {
      "path": "src/esaa/service.py",
      "base_sha256": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
      "edits": [
        {
          "old_string": "texto antigo exato",
          "new_string": "texto novo",
          "replace_all": false
        }
      ]
    }
  ]
}
```

Semântica de edits:
- O Orchestrator resolve `{path, base_sha256, edits}` para `{path, content}` antes de external effects, resource limits, staging e artifacts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elzobrito/ESAA---Event-Sourcing-Agent-Architecture](https://github.com/elzobrito/ESAA---Event-Sourcing-Agent-Architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
