---
trigger: always_on
description: Exemplos em Python usados nas aulas de **engenharia de prompt** e **engenharia
---

# Contexto para sessões de IA neste repositório

Exemplos em Python usados nas aulas de **engenharia de prompt** e **engenharia
de contexto**. Não são de uma turma só: rodam em imersões, cursos e treinamentos
diferentes.

Cada exemplo tem seu próprio `CLAUDE.md` com as regras específicas dele — **leia
o do exemplo em que estiver trabalhando**. Este arquivo é só o que vale para os
quatro.

## O código é material de aula

Todo código aqui é lido em sala, projetado numa tela. Não é software de produção
e não deve ser tratado como tal.

**Regra de ouro — quando simplicidade e robustez colidirem, vence a
simplicidade.** É o inverso do default e é intencional. Cada abstração a mais é
uma coisa a mais para explicar antes de chegar ao ponto da aula.

A consequência prática é que **melhorias genuínas de engenharia são regressões
aqui** quando custam clareza: retry, cache, camada de serviço, injeção de
dependência, tratamento abrangente de erro, abstrações "para quando crescer".
Nenhuma entra sem pedido explícito.

## Cada exemplo ensina uma coisa — e só ela

| Pasta | Aula | Lição |
|---|---|---|
| `01-zero-e-few-shot` | prompt | o que dois exemplos no prompt mudam, e o que custam |
| `02-chain-of-thought` | prompt | raciocínio explícito como recurso e como custo |
| `03-contexto-de-conversa` | contexto | o histórico é um array que o seu código reenvia |
| `04-contexto-de-documento` | contexto | documento em runtime entrando na janela |

**Não antecipe o exemplo seguinte.** O recurso ausente num exemplo geralmente é
o assunto do próximo, e a ausência é o gancho.

Os exemplos funcionam **em pares** (`01`×`02`, `03`×`04`), e há um fio entre os
pares: o diagnóstico do `02` termina pedindo um dado que o documento do `04`
entrega. Mexer num lado exige reler o outro.

**Não faça um exemplo convergir para outro.** O `03` e o `04` compartilham o
miolo — `nova_conversa`, `mostrar`, o loop de render, o `append` e o `invoke`
com a lista inteira —, e é isso que faz um explicar o outro: reconhecer o mesmo
código dos dois lados é o que mostra que documento e histórico são a mesma
coisa. Mantenha esse miolo reconhecível. O que diverge de propósito é a barra
lateral, uma por lição: no `03` o checkbox que desliga o acúmulo do histórico,
no `04` o upload dos documentos.

## Uma tarefa por exemplo de prompt

O `01` e o `02` **não** compartilham a mesma tarefa, e isso é decisão do curso,
não descuido. Cada técnica roda na tarefa em que ela encaixa: documentação no
padrão do time no `01`, diagnóstico com hipóteses concorrentes no `02`.

**Dentro** do `01` a tarefa é uma só, e isso é obrigatório: zero-shot e
few-shot rodam o mesmo incidente, porque a comparação lado a lado só é honesta
se a única variável for o prompt.

O contraste entre os dois exemplos continua existindo — ele é **par a par** e
aparece no readout de custo, no mesmo formato nos dois. O `01` paga na entrada
(os exemplos entram no prompt toda chamada) e o `02` paga na saída (é onde a
deliberação acontece). Não unifique as tarefas para "melhorar a comparação".

## O modelo é Haiku, e isso é conteúdo

Os quatro exemplos rodam em **Haiku 4.5**
(`anthropic:claude-haiku-4-5-20251001`), não em Sonnet.

O motivo é pedagógico: um modelo com menos reasoning **amplia o efeito** que
cada aula quer mostrar. Em Sonnet o zero-shot já chega perto do
chain-of-thought sozinho e o contraste da aula de prompt fica sutil demais para
a tela; o mesmo vale para contexto — quanto menor o modelo, mais ele inventa sem
o documento, e mais o `04` se separa do `03`.

**Não "melhore" os exemplos subindo para Sonnet.** Isso apaga exatamente o que
as duas aulas existem para mostrar. O modelo continua vindo de `MODELO` no
`.env`, então rodar em Sonnet é um beat opcional na aula, não o default.

## Modelos e parâmetros

**Não use `temperature`** (nem `top_p`/`top_k`) em nenhum exemplo. Sonnet 5 e
Opus 4.7+ removeram os parâmetros de sampling e devolvem `400`.

## Os readouts são a interface da apresentação

O `01` e o `02` mostram tokens e tempo; o `03` e o `04` mostram o tamanho da
janela a cada volta. Isso não é observabilidade: é o que a turma vê acontecendo
na tela, e é o que transforma "a janela é finita" de afirmação em número.

São os mesmos três números no `01` e no `02` — tokens de entrada, tokens de
saída e tempo —, em `st.metric` acima da resposta nos dois. No `01` eles ficam
no topo de cada coluna, porque ali precisam estar alinhados entre os dois
painéis para serem comparáveis. Não tire um número de um sem tirar do outro.

No `03` e no `04` a janela aparece em `st.caption` embaixo de cada resposta do
assistente. Esse contador não sai dali: é a série inteira crescendo no scroll
que mostra a conta, não o último número sozinho.

O `03` e o `04` têm ainda uma barra lateral com o mesmo número contra o teto de
200k. Ela é acréscimo, não substituto: o contador por resposta mostra a série, o
medidor mostra a escala. O resto da barra diverge de propósito — no `03` é o
checkbox que desliga o acúmulo do histórico, no `04` é o upload dos documentos.
São os dois jeitos de encher a mesma janela, um por exemplo.

Mudança que reduza a legibilidade no projetor é regressão, não limpeza — mesmo
que deixe o código mais "limpo".

## Ambiente e pacotes — `uv`, sem exceção


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabricioveronez/prompt-e-contexto-na-pratica](https://github.com/fabricioveronez/prompt-e-contexto-na-pratica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
