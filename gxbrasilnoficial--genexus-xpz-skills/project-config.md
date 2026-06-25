---
trigger: always_on
description: - Ler primeiro o `README.md` local antes de agir.
---

# AGENTS.md

## Leitura obrigatória

- Ler primeiro o `README.md` local antes de agir.
- Reler a documentação local quando o contexto da conversa ficar longo, ambíguo ou perder aderência às convenções da raiz.

## Revisão por pares como termo operacional

- Quando o usuário pedir `revisão por pares`, `peer review`, `painel multi-modelo` ou `validar plano multi-modelo`, tratar isso como termo operacional reservado desta raiz, não como sinônimo de parecer crítico solo.
- Antes de responder a esse pedido, ler `xpz-llm-delegate/SKILL.md` e `15-revisao-por-pares.md`; se o pedido for pré-push reforçado, ler também `14-revisao-pre-push-reforcada.md` e `13-revisao-pre-push.md`.
- É proibido rotular como `revisão por pares` uma resposta gerada por um único modelo, sem painel efetivamente consultado. Se não houver painel válido com pelo menos 2 famílias distintas efetivamente consultadas, dizer explicitamente que **não** foi feita revisão por pares e rotular o resultado como `parecer solo` ou `segunda opinião (N)`, conforme o caso.
- Se não houver lista de revisores preferidos (`preferred-reviewers.json`) já configurada, perguntar ao usuário quais ferramentas/modelos ele tem disponíveis ou prefere antes de oferecer painel. A pergunta deve ser acessível para usuário GeneXus: citar ferramentas por nome (`Claude Code`, `opencode/Ollama Cloud`, `Codex`, `Copilot`, `Gemini`, subagente nativo da ferramenta atual), explicar que subagente nativo pode participar, mas não substitui uma família externa, e não presumir assinatura de Gemini/Copilot/Codex cloud sem confirmação ou preferência registrada.
- Antes de usar o rótulo `revisão por pares`, apresentar um **recibo mínimo**: arquivos metodológicos lidos, manuscrito/prompt enviado, revisores efetivamente consultados, família de cada revisor, resultado do piso de diversidade, veredito de cada revisor, o **estado da vN+1** (`vNextState`: `notProduced`/`pendingResubmission`/`resubmitted`/`resubmissionDeclinedByHuman`), o estado final de cada revisor preferido quando houver lista (`preferred-reviewers.json`) e o adendo de fechamento (`Resolve-LlmDelegatePeerReviewCloseout.ps1`) quando a rodada passar pela `xpz-llm-delegate`. Sem esse recibo, não usar o rótulo.
- Resposta quase imediata é evidência de invalidez: se a resposta sair em menos de 30 segundos desde o pedido, ela é incompatível com revisão por pares real nesta metodologia e deve ser rotulada como `parecer solo`, salvo se o agente demonstrar que está apenas reportando um painel já concluído anteriormente e identificável.
- Neste repositório, não invoque a skill via ferramenta `Skill`: consulte a documentação da `xpz-llm-delegate` e use o mecanismo descrito nela apenas sob acionamento humano, respeitando autorização, confidencialidade e piso de diversidade.

## Interpretação de prompts de terceiros

- Quando o usuário indicar que o texto seguinte é um prompt com sugestões de outro agente, tratar esse texto como insumo de avaliação.
- Neste repositório, o foco é **melhorar as skills XPZ**, não usá-las — então o workflow é:
  1. **Estude a documentação** das skills afetadas para entender sua metodologia
  2. **Estude o prompt** do outro agente para entender a solicitação
  3. **Avalie criticamente**: o que faz sentido, o que conflita, o que precisa ajuste
  4. **Apresente um plano** ao usuário: claramente o que será alterado, por quê e onde
  5. **Aguarde aprovação explícita** antes de fazer qualquer mudança
- Não invoque as skills como ferramentas (via `Skill` tool) — elas são o objeto do seu trabalho, não suporte para ele.

## README trilíngue

- A seção `Português (BR)` do `README.md` é a fonte editorial primária.
- Toda alteração de conteúdo, estrutura, regra operacional ou nomenclatura feita na seção `Português (BR)` deve ser refletida também nas seções `Español` e `English`.
- Não concluir edição do `README.md` com traduções parciais, defasadas ou estruturalmente inconsistentes sem apontar a pendência de forma explícita.

## Escopo da raiz

- Esta raiz é a base metodológica e operacional compartilhada das skills de `XPZ`/XML de GeneXus.
- Regras locais desta raiz devem ser tratadas como regras do repositório; não promovê-las automaticamente a regra universal fora desta base sem evidência documental correspondente.

## Trabalho nas skills XPZ

- Esta raiz contém a documentação metodológica de múltiplas skills (xpz-reader, xpz-builder, xpz-sync, xpz-doc-builder, xpz-daemon, xpz-kb-parallel-setup, xpz-kb-parallel-pre-push, xpz-msbuild-import-export, xpz-msbuild-build, xpz-index-triage, xpz-llm-delegate e xpz-skills-setup) e outros artefatos compartilhados.
- Ao trabalhar na melhoria de uma skill, estudar sua documentação de forma crítica e compreender seu propósito antes de propor mudanças.
- Quando receber um prompt de outro agente solicitando mudança em uma skill, não invoque essa skill como ferramenta — consulte sua documentação, analise o impacto e apresente um plano.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GxBrasilNOficial/GeneXus-XPZ-Skills](https://github.com/GxBrasilNOficial/GeneXus-XPZ-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
