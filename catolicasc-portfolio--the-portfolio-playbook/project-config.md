---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que este repositório é

Repositório **somente de documentação** (PT-BR). Não há código, build, testes nem dependências. É o
regulamento das três disciplinas de Portfólio da Católica SC: define o que o aluno entrega, com que
critérios é avaliado e com que nota é aprovado. O público leitor são **alunos e professores
avaliadores** — o texto é normativo, não descritivo.

## Arquitetura: uma teia de regras, não uma árvore de arquivos

Os documentos se referenciam entre si e **repetem as mesmas regras em contextos diferentes**. É essa
duplicação intencional que define o trabalho aqui: alterar um número em um arquivo sem propagar para
os outros produz contradição normativa — o aluno e o avaliador passam a ler regras diferentes sobre o
mesmo fato. O histórico do repo é em boa parte a correção desse tipo de divergência.

`calendario.md` (raiz) é a **fonte única de datas** e traz o checklist cronológico do aluno. Nunca
introduza uma data nova em outro documento sem acrescentá-la lá — há script de verificação abaixo.

Os três documentos de disciplina estão na raiz e o `README.md` indexa tudo. Dois pontos que o `ls` não
conta: `directions/portfolio-directions-GERAL.md` traz a **tabela mestra** de tecnologias, processos e
temas (🔑 Obrigatório · ✅ Preferir · 🔍 Explorar · ⚠️ Evitar · 🚫 Não Usar), e `semesters/` guarda
planilhas de notas por turma — **gitignorado, nunca versionar**.

## Fonte única de verdade por regra

Antes de escrever qualquer número, confira aqui de onde ele vem. Se a mudança for numa dessas regras,
propague para **todos** os arquivos abaixo que a mencionam.

| Regra | Fonte canônica | Também aparece em |
|---|---|---|
| Nota do Demo Day: Likert 1–5 → 0–10 (**×2**); professores **90%** / comunidade **10%**; 1 casa decimal | `demoday/Avaliacao_Poster_DemoDay.md` § *Como a Likert vira nota* | `Portfolio.md` § Avaliação |
| 6 critérios de peso igual; **critério 5 (Ética/Autoria) é eliminatório**; piso da fórmula 2,0 | `demoday/Avaliacao_Poster_DemoDay.md` | `Portfolio.md` § Prova de Autoria |
| Nota da RFC: 0–10 em passos de **0,5**; aprovação a partir de **6,0**; faixa **6,0–7,0 = aprovada com correções obrigatórias** | `documentation/diretrizes-avaliacao-professores.md` § Anexo - Escala | `documentation/RFC/modelo-de-RFC.md` § 10 (ficha por avaliador + Consolidação) |
| Prazo de entrega **30/11/2026 (segunda-feira)**; Demo Day **10/12**, **15/12**, **16/12/2026** | `Portfolio.md` § Datas e Prazos | `PAC Extensionista VIII.md` § Prazos |
| **Seis participações obrigatórias**: 5 orientações (2 até **30/09/2026**, todas até 30/11) = gate de **habilitação**; 1 noite como visitante + avaliar os colegas = gate de **aprovação** | `Portfolio.md` § Requisitos de Aprovação | `Portfolio.md` § Requisitos de Entrega · `demoday/Avaliacao_Poster_DemoDay.md` § Pré-condição e § Participação da Comunidade · `demoday/guia_poster.md` § Participação |
| **Prova de autoria**: verifica autoria **e a cobertura de testes da linha**; admite **uma segunda tentativa** até **03/12/2026**; reprovar duas vezes = reprovado na disciplina, sem apresentar | `Portfolio.md` § Prova de Autoria | `demoday/Avaliacao_Poster_DemoDay.md` § Pré-condição e critério 3 · `demoday/guia_poster.md` |
| **Calendário do PAC VII é próprio** (7º período) e não compartilha datas com Portfólio/PAC VIII (8º período) | `PAC Extensionista VII.md` § 6. Prazos | — |
| Régua Destaque / Aprovado / Reprovado; aprovado exige nota ≥ **6,0** | cada `directions/portfolio-directions-*.md` § Régua de Avaliação | — |
| Núcleo comum de engenharia — **4 itens**: Wiki · CI/CD · análise estática · monitoramento/observabilidade | `directions/portfolio-directions-GERAL.md` | replicado sob `### Núcleo comum de engenharia` nas cinco directions |

**Testes não são iguais entre as linhas** — esta é a assimetria mais fácil de quebrar por engano:

| Linha | Meta obrigatória | TDD |
|---|---|---|
| **Web Apps** | **75% backend / 25% frontend** | 🔑 obrigatório |
| **Jogos Digitais** | **50%** da lógica de jogo (fora: engine, shaders, UI, assets) | ✅ preferir |
| **Mobile · IA · IoT** | **nenhuma** — testes unitários não se aplicam a essas tecnologias | ✅ preferir |

Em Mobile, IA e IoT os testes vivem em *O que é desejável atender*, com o instrumento próprio de cada uma
(instrumentados/integração · validação de modelo · rotinas críticas de firmware). Mobile volta a ter TDD
obrigatório **se o aluno escrever o próprio backend**. A meta é **gate conferido na prova de autoria**, não
item de nota — o critério 3 do Demo Day pontua a *qualidade* dos testes, e penaliza teste raso que só
persegue percentual. Cobertura **não** está no núcleo comum, justamente por não ser comum.

As cinco directions têm **as mesmas 9 seções nos mesmos níveis** (`## O que é obrigatório atender` com
`### Núcleo comum de engenharia` dentro · desejável · diferencial · deve ser evitado · não pode ter ·
temas a evitar · temas impedidos · `## Régua de Avaliação`). Há script de verificação abaixo.

**Exceções da linha de Jogos Digitais** — recorrem em vários documentos e são a origem mais comum de

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CatolicaSC-Portfolio/The-Portfolio-Playbook](https://github.com/CatolicaSC-Portfolio/The-Portfolio-Playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
