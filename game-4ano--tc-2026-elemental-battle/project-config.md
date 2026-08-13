---
trigger: always_on
description: Projeto: **Elemental Battle** (TC-2026-ELEMENTAL-BATTLE)
---

# CLAUDE.md — Regras de Operação do Claude Code

Projeto: **Elemental Battle** (TC-2026-ELEMENTAL-BATTLE)
Disciplina: Tópicos em Computação · Python + Pygame
Rode o jogo com: `python -m meu_jogo.main`

Este arquivo é lido no início de toda sessão. Siga-o à risca.

---

## O QUE É O JOGO

RPG por turnos estilo Pokémon: overworld de exploração (mundo aberto com câmera que
segue o jogador) + batalhas por turno contra 4 chefes elementais. O jogador entra em
portais no campo de treino para lutar; vencer todos os 4 conclui o jogo.

Sistemas já implementados e **funcionando** (não quebrar):
- Áudio (`AudioManager`) — música e SFX gerados
- Pontuação (`ScoreSystem`) — combo, multiplicadores de tempo/HP, highscore
- Save (`SaveSystem`) — highscore em JSON
- Menu de batalha com 4 ações: **Atacar, Especial, Defender, Curar**
- IA Smart dos bosses
- Cenas: MenuScene, CampoDeTreinoScene, BattleScene, GameOverScene, VictoryScene
- Sistema de XP/nível em `character.py` (`gain_xp`, `level_up`)

---

## COMO EU TRABALHO COM VOCÊ (fluxo obrigatório)

Eu (Artur) desenho os prompts na chat do Claude e te entrego prompts em markdown com
**fases numeradas**. Sua execução segue estas regras sempre:

1. **Uma fase/tarefa por vez.** Ao terminar: resumo curto do que mudou + como testar.
   **PARE e aguarde meu OK** antes da fase seguinte. Nunca faça bulk de várias fases.
2. **Fase 0 de auditoria** (somente leitura) quando o prompt pedir: mapeie a estrutura
   real antes de editar qualquer coisa. Confirme comigo antes de tocar no código.
3. **Edições cirúrgicas com `str_replace`.** Nunca reescreva um arquivo inteiro se dá
   para editar um trecho. Não releia arquivos que já leu na mesma sessão.
4. **Respostas curtas.** Sem despejar arquivos inteiros na saída; mostre só os diffs
   relevantes e explique em poucas linhas.
5. Se um pedido conflitar com estas regras ou com a arquitetura, **pare e me pergunte**
   antes de improvisar.

---

## RESTRIÇÕES DA DISCIPLINA (valem nota — inegociáveis)

- **Orientação a objetos com herança/polimorfismo.** O professor já criticou
  fragmentação de lógica e "God-files"; arquitetura limpa é critério avaliado.
  Ao adicionar personagens, prefira uma hierarquia adequada (`Character` → `Player`/
  `Enemy`/`Boss`) a inchar uma classe só.
- **Sem novas dependências.** Apenas Python stdlib + Pygame (Box2D já está autorizado
  no projeto; qualquer outra lib precisa de aprovação do professor). Não instale nada.
- **Física correta com dt.** Posição += velocidade × dt (nunca posição += velocidade).
  Toda animação e movimento usam o `dt` do frame.
- **Vetores agrupados.** Use `pygame.Vector2` para posição/velocidade — nunca x e y
  soltos.
- **Ponto flutuante** para posição/velocidade; converta para pixel só ao renderizar.
- **Eu preciso saber explicar cada linha.** Gere código que eu entenda; comente as
  partes não óbvias. Nada de "mágica" que eu não consiga defender num questionamento.

---

## ARQUITETURA E ONDE CADA COISA VIVE

```
meu_jogo/
  main.py            → só inicializa GameManager + cena inicial. Sem lógica de jogo.
  core/              → motor: game_manager, scene_manager, game_state, config,
                       battle, game, map, elements  (+ progression, se criado)
  cenas/             → telas/renderização (menu, campo_de_treino, battle_scene, ...)
  entidades/         → character, acoes, ai  (estado + regras, não desenham na tela)
  data/              → characters_data, maps_data  (SÓ dados, sem lógica complexa)
  midia/sprites/     → sprite_factory (pixel art), animated_sprite (animações)
  utils/             → helpers (matemática, desenho, cores, debug)
  testes/            → protótipos e scripts; não fazem parte do jogo final
```

Regras de camada:
- `main.py` simples. Nada de batalha/XP/mapas aqui.
- `core/` controla funcionamento; **não** coloca sprites nem dados de inimigos aqui.
- `cenas/` desenham; **não** implementam regras de dano/XP.
- `entidades/` guardam estado e regras; **não** desenham direto na tela.
- `data/` só definições prontas.
- `midia/` só código de geração de arte/áudio; assets não viram lógica.

---

## CONVENÇÕES DE CÓDIGO

- **Comentários e mensagens em português.**
- **Constantes de tuning só em `core/config.py`.** Nada de números mágicos espalhados
  (tamanho da janela, FPS, XP por nível, incrementos por nível, multiplicadores de
  boss, etc.). Dados puros de personagem/mapa ficam em `data/`.
- **Reusar antes de reconstruir.** Antes de criar um sistema novo, verifique se já
  existe. Exemplos reais deste projeto:
  - XP/nível já existe em `character.py` → wire nele, não crie paralelo.
  - Cura (`HealAction`) já escala com `max_hp` → não duplicar essa lógica.
- **Sprites novos seguem o padrão da factory:** matriz 16×16, palette dict, caracteres
  de 1 letra, `'.'` = transparente. Não invente outro padrão — o professor espera
  consistência.
- **Tiles/sprites animados** usam `pygame.time.get_ticks()` para ciclos, ex:
  `phase = (pygame.time.get_ticks() % 1000) / 1000.0`.
- **Fallback preservado:** sprites/tiles antigos continuam funcionando se um novo
  falhar. Nada de remover o caminho antigo sem substituto testado.

---

## TRABALHO ATIVO: BUG DA MORTE + PROGRESSÃO + ESCALONAMENTO DE BOSSES


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Game-4ano/TC-2026-Elemental-Battle](https://github.com/Game-4ano/TC-2026-Elemental-Battle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
