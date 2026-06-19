---
trigger: always_on
description: Análise forense de processos judiciais brasileiros. Extrai, classifica e cruza peças processuais, detecta contradições, calcula prazos CPC e gera relatórios de risco com exportação para Obsidian. Use quando mencionar: processo judicial, petição, sentença, acórdão, contradições, prazos, timeline, análise forense, Obsidian.
---


# JuriScan

## Quick Run

Quando o usuário invocar `/juriscan <caminho_do_pdf>` ou pedir para analisar um processo, execute TODO o pipeline abaixo em sequência, sem parar para perguntar. O PDF pode estar em qualquer diretório — o usuário passa o caminho.

Se o usuário não passar um caminho, pergunte: "Qual o caminho do PDF do processo?"

**CWD check:** Se `pwd` apontar para `*/.claude/skills/juriscan*` (ou seja, a sessão do Claude foi iniciada dentro do diretório da própria skill), avise o usuário:

> "Você iniciou o Claude Code dentro do diretório da skill (`~/.claude/skills/juriscan`). Skills são globais — o normal é rodar o Claude na pasta do seu projeto, onde estão os PDFs. Saia desta sessão, faça `cd` para a pasta do processo e rode `claude` de novo. Posso continuar aqui mesmo assim se você passar o caminho absoluto do PDF."

Prossiga apenas se o usuário insistir ou fornecer caminho absoluto.

**Fluxo completo automático:**

1. Resolver `SKILL_DIR` e garantir dependências (Setup)
2. Criar diretório de análise ao lado do PDF: `<pdf_dir>/juriscan-output/`
3. Resolver o **modo de execução** (ver "Modos de execução" abaixo)
4. Executar o pipeline correspondente ao modo escolhido
5. No final, informar ao usuário:
   - Resumo executivo do processo (3-5 frases)
   - Quantas peças encontradas
   - Contradições detectadas (quantidade e as mais graves)
   - Prazos calculados (se houver)
   - Nível de risco
   - Onde estão os arquivos de saída
   - Se quiser Obsidian: "O vault está em `<output>/obsidian/` — abra essa pasta no Obsidian como vault"
   - run_id do audit trail (`.juriscan/audit/<run_id>.jsonl`) quando em modo agents

---

## Modos de execução

O juriscan suporta dois pipelines. O modo é resolvido a partir dos argumentos:

| Invocação | Modo | Status |
|---|---|---|
| `/juriscan --selftest` | Selftest | Ativo (ver seção Selftest) |
| `/juriscan --pipeline=legacy <pdf>` | **Legacy** (default até Phase 6) | Ativo — pipeline determinístico descrito em "Step-by-Step Pipeline" |
| `/juriscan --pipeline=agents <pdf>` | **Agents** (opt-in durante Phases 2–5) | Em construção — ver "Agents Pipeline" |
| `/juriscan <pdf>` (sem flag) | Legacy (por enquanto) | Flip do default acontece na Phase 6 Step 6.4 |

**Parsing dos argumentos:** quando o usuário invocar `/juriscan`, o primeiro token não-flag é o caminho do PDF. Flags reconhecidas: `--selftest`, `--pipeline=legacy`, `--pipeline=agents`. Qualquer flag desconhecida → avisar e cair em legacy.

**Gate de pré-requisitos (modo agents):** antes de seguir o modo agents, confirme que:

1. O arquivo `.claude/agents/juriscan-echo.md` existe em `$SKILL_DIR/.claude/agents/` (proxy para "subagents foram instalados").
2. `python3 "$SKILL_DIR/scripts/agent_io.py" validate --agent echo --input "$SKILL_DIR/tests/fixtures/agent_io/echo_valid.json"` retorna exit 0 (proxy para "schemas + jsonschema ok").

Se algum check falhar, **não** prossiga em modo agents. Informe o usuário o check que falhou e sugira `./install.sh` + `/juriscan --selftest`.

**Manifesto (ambos os modos):** no início de qualquer análise (não-selftest), crie/atualize `<output>/manifest.json`:

```bash
RUN_ID="$(python3 "$SKILL_DIR/scripts/agent_io.py" new-run --root "$OUTPUT_DIR/.juriscan/audit")"
python3 -c "
import json, os, time
manifest = {
    'run_id': '$RUN_ID',
    'pipeline_mode': '$PIPELINE_MODE',  # 'legacy' | 'agents'
    'pdf_path': os.path.abspath('$PDF_PATH'),
    'started_at': time.time(),
    'skill_dir': '$SKILL_DIR',
}
open('$OUTPUT_DIR/manifest.json','w').write(json.dumps(manifest, indent=2))
"
```

Nota: por ora `.juriscan/audit/` vive dentro de `<output>/` (não na raiz do projeto) para evitar poluir o cwd do usuário. Depois do Phase 6 podemos discutir se faz sentido centralizar.

---

## Agents Pipeline (modo `--pipeline=agents`)

Sequência quando o modo agents está selecionado. Cada passo é `[Python]` (script determinístico) ou `[Task]` (invocação de subagent via Task tool). Passos marcados `[PENDING: Phase N]` ainda não têm subagent real e abortam a pipeline com mensagem clara até serem implementados.

```
1. [Python]  scripts/extract_pdf.py              -> raw_text.txt + page_map.json
                                                    (Phase 2.x, stub usa extract_and_chunk.py)
2. [Task]    juriscan-segmenter                  -> /tmp/$RUN_ID-segmenter.json
3. [Python]  scripts/agent_io.py validate --agent segmenter ...
4. [Python]  scripts/persist_chunks.py           -> chunks/*.txt + index.json
5. [Task×N]  juriscan-parser (paralelo)          -> /tmp/$RUN_ID-parser-NN.json por chunk
6. [Python]  scripts/agent_io.py validate (N×)
7. [Python]  scripts/enrich_deterministic.py     -> pieces enriquecidas (normalizações)
8. [Task×3]  juriscan-advogado-autor, juriscan-advogado-reu, juriscan-auditor-processual (paralelo)
             [PENDING: Phase 3]
9. [Task]    juriscan-verificador                 [PENDING: Phase 4]
10.[Task]    juriscan-sintetizador                [PENDING: Phase 3]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunobracaioli/juriscan](https://github.com/brunobracaioli/juriscan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
