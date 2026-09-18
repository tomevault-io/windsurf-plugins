---
trigger: always_on
description: Ditado inteligente para Windows e macOS: segura o atalho, fala, solta — o texto transcrito,
---

# Open Flow — instruções do projeto

Ditado inteligente para Windows e macOS: segura o atalho, fala, solta — o texto transcrito,
limpo e formatado entra onde o cursor estiver. Tauri 2 (Rust) + React/TS, com sidecar Python
opcional para STT local na GPU.

Arquitetura, decisões e histórico de bugs estão no [README.md](README.md) — não duplicar aqui.
Este arquivo é **versionado** e vale nas duas máquinas. O que é específico de uma delas vai em
`CLAUDE.local.md`, que fica fora do git.

## Duas máquinas, um repositório

O Macks trabalha alternando entre um **Windows** e um **macOS**, e o build de cada sistema só
sai na máquina dele. Isso é a origem da maior parte da confusão histórica do projeto, então:

- **Antes de qualquer coisa, `git fetch` e ver se a outra máquina deixou commit para trás.** É
  comum começar uma sessão uma release atrás.
- **Base de código única.** O código específico de SO vive em
  `app/src-tauri/src/platform/{windows,macos}.rs`, atrás de `#[cfg]`, com a mesma interface
  pública (ver `platform/mod.rs`). Mexeu em um, confira o par — o outro SO não compila aqui
  para avisar.
- **Uma correção em `lib.rs` vale para os dois sistemas.** Depois de lançar de um lado, o outro
  fica sem a correção até alguém buildar lá. Registrar isso em "Estado atual".
- **Ao fechar uma release, atualizar a seção "Estado atual" abaixo.** É por ela que o agente da
  outra máquina descobre o que ficou pendente do lado dele.

## Estado atual (2026-09-06)

- **Versão**: v0.1.15 no Windows, v0.1.14 no macOS. As correções da v0.1.15 são de `lib.rs`,
  comuns aos dois SOs — **buildar no Mac e anexar o DMG à tag v0.1.15**, que hoje só tem os
  instaladores Windows. O mesmo vale para o `.dmg` da v0.1.14, que já existe.
- **v0.1.15 trouxe**: (1) o overlay volta ao topo — `set_always_on_top(true)` sozinho não fazia
  nada, porque o tao só chama `SetWindowPos` quando a flag muda de valor e a janela já nasce
  `alwaysOnTop`; a correção é alternar `false`/`true`. (2) O fix do fallback do Gemini vindo do
  Mac (9b9ce95): só 404 rebaixa a sessão, reserva virou `gemini-flash-lite-latest`, teto por
  chamada de 30s para 4s com uma segunda tentativa.
- **Teto da reescrita voltou para 8s, sem retry (41839b1) — só no Mac por enquanto.** É
  mudança de `lib.rs`, vale nos dois SOs, mas o build saiu só aqui e não virou release: a tag
  v0.1.15 publicada NÃO a contém, e o Windows segue no teto de 4s. O Macks avalia que lá está
  rápido o bastante; se algum dia não estiver, o remédio é este commit. Motivo da mudança: os
  4s de 9b9ce95 foram calibrados com p95 de ~3,3s, e a remedição de 2026-09-06 deu p90 de
  7,30s — chamada saudável passou a morrer no teto, tentar de novo, morrer outra vez, e o
  ditado custava 8s para sair **sem reescrita** (78 casos no `history.jsonl`). Se a latência
  voltar a incomodar, remedir antes de mexer no número: `bench/bench_rewrite.py` com
  `--retry-apos` compara as estratégias, e o `history.jsonl` conta quantos ditados saíram com
  `raw == final`, que é o custo escondido de um teto curto.
- **Latência do Gemini é do modelo, não do SO.** Medido em 2026-09-06 no Mac, o
  `gemini-3.5-flash-lite` deu mediana 1,47s mas p90 7,30s e 18% de falha; o reserva
  `gemini-flash-lite-latest` tem 1,39s de mediana registrada. Como o modelo vive no
  `settings.json` de cada máquina (ver o item abaixo), as duas podem estar em modelos
  diferentes — daí uma parecer rápida e a outra não, com o mesmo código. Conferir a chave
  `gemini_model` antes de culpar a plataforma.
- **Pendente de confirmação em uso**: a assinatura do Whisper aparecia em ~3% dos ditados, então
  a prova de que o `strip_credit` resolveu vem com volume, não com sessão de teste.
- **Modelo do Gemini vive no `settings.json`, não no default.** Trocar o default só afeta
  settings novo: em cada máquina a chave `gemini_model` precisa ser trocada à mão, ou o app
  segue no modelo antigo. Toda troca de modelo futura precisa desse passo nos dois lados.
- **Fonte da verdade da versão**: `app/src-tauri/tauri.conf.json` (o `version` do
  `app/package.json` está em 0.1.0 e é ignorado — não é ele que nomeia os instaladores).

## Release

Versão nova: subir `version` em `tauri.conf.json` **e** em `Cargo.toml`, buildar, commit
incluindo o `Cargo.lock`, `git tag` e `gh release create` com os instaladores que a máquina
gerou. A outra máquina depois anexa o instalador dela **à tag existente**, com
`gh release upload` — sem criar release nova.

**NUNCA APONTAR OS LINKS DE DOWNLOAD DO README PARA `/releases/latest` NEM PARA UMA TAG
ESPECÍFICA.** Os dois links de instalador do README vão para `../../releases` — a lista inteira
— e ficam assim. `/releases/latest` é UMA release só, a do último build feito: depois de um
build Windows ela não tem `.dmg`, depois de um build Mac não tem `.exe`. Foi isso que cruzou os
links a cada release e fez cada máquina "consertar" apontando para a sua própria tag, quebrando
o outro lado. **NÃO EXISTE URL DO GITHUB QUE RESOLVA PARA "a release mais recente que tem
.dmg"** — o único link que nunca mente é a lista. Só volte a usar `/latest` se toda release
passar a sair com os dois instaladores juntos.

## Ao trabalhar no código


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mackswendhell/open-flow](https://github.com/mackswendhell/open-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
