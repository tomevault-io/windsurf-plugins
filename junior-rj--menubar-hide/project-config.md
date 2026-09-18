---
trigger: always_on
description: App macOS pra ocultar ícones da menu bar quando ela lota: um separador expande/recolhe os ícones escondidos, com opção de mostrar num painel abaixo da menu bar (resolve o notch).
---

# menubar-hide

## O que é
App macOS pra ocultar ícones da menu bar quando ela lota: um separador expande/recolhe os ícones escondidos, com opção de mostrar num painel abaixo da menu bar (resolve o notch).

## Escopo
- App novo em SwiftUI, macOS 14+ (subiu de 13 por causa do ScreenCaptureKit do painel)
- Esconder/mostrar ícones da menu bar com um clique no botão + / − (separador é #)
- Painel abaixo da menu bar com os ícones escondidos (modo alternável no clique-direito)
- Atalho de teclado ⌃⌥H pra alternar; iniciar com o sistema
- Arranjo dos ícones persistente: memoriza a posição de TODOS os ícones e reescreve no launch
- Espaçamento da menu bar ajustável pelo menu (NSStatusItemSpacing/NSStatusItemSelectionPadding)

## Contexto
- Referência de técnica lateral: https://github.com/dwarvesf/hidden (Hidden Bar, MIT). Só referência de estudo, o código aqui é novo
- Referência de técnica do painel: https://github.com/jordanbaird/Ice (Ice Bar, GPL-3.0). Só estudo de técnica, NUNCA copiar código (licença incompatível)
- Truque lateral (macOS 26 e anteriores): NSStatusItem separador com length 10000 empurra os ícones à esquerda dele pra fora da tela. Nunca usar isVisible (perde a posição do autosave). No macOS 27 o mecanismo mudou: ver "Pegadinhas do macOS 27"
- Truque do painel: CGWindowList acha os itens fora da tela, ScreenCaptureKit captura as imagens, NSPanel mostra abaixo da menu bar, clique é encaminhado via CGEvent (expande temporário + re-colapsa)
- Permissões do painel: Gravação de Tela (capturar) e Acessibilidade (encaminhar clique). Gravação de Tela exige relançar o app após conceder
- Autostart via SMAppService.mainApp
- Projeto via XcodeGen (project.yml fonte de verdade, .xcodeproj gitignored, rodar `xcodegen` após mudar)
- Repo: https://github.com/junior-rj/menubar-hide (público desde 2026-07-13, MIT, releases com DMG assinado e notarizado)

## Arquivos importantes
- project.yml — definição do projeto (rodar xcodegen após mudar)
- MenubarHide/StatusBarController.swift — botão +/− + separador #, toggle, menu, launch at login, integração do painel
- MenubarHide/HotkeyManager.swift — hotkey global ⌃⌥H via Carbon
- MenubarHide/MenuBarItemScanner.swift — descoberta dos itens escondidos via CGWindowList
- MenubarHide/ItemCapturer.swift — captura via ScreenCaptureKit
- MenubarHide/HiddenItemsPanel.swift — NSPanel + SwiftUI com os ícones
- MenubarHide/ClickForwarder.swift — clique sintético via CGEvent
- MenubarHide/MenuBarArrangement.swift — snapshot e restauração das posições de todos os ícones via CFPreferences
- MenubarHide/MenuBarSpacing.swift — leitura e escrita das duas chaves globais de espaçamento
- MenubarHide/Localizable.xcstrings — String Catalog com toda a UI em EN e pt-BR (InfoPlist.xcstrings traduz o copyright do painel About)
- Release pela skill global `release-macos` (desde 05/09/2026; o wrapper `scripts/release.sh` saiu). Um comando faz build fora do repo, Developer ID, DMG, notarização, staple, instala em /Applications e publica. Config inferida do repo, sem override. Saída `build/MenubarHide-X.Y.Z.dmg` + `build/export/MenubarHide-stapled.app`; perfil de notary `sparrow-notary` do keychain

## Regras específicas
- Pegadinhas do macOS 27 (v1.6.1; a barra foi reescrita e quebrou o hide, não regredir):
  - A menu bar virou UMA janela "Menubar" do MenuBarAgent (layer 24). `CGWindowListCopyWindowInfo` devolve ZERO janelas no layer de status (25), então scanner, painel (captura + clique) e Icon Arrangement ficam sem base. Tudo isso é gated por `MenuBarItemScanner.isPerItemWindowModelAvailable` (major < 27); `StatusBarController.legacyModel` carrega o valor. Painel e Arranjo ficam desabilitados no menu no 27; `toggle()` ignora `showInPanel` (age lateral)
  - Um NSStatusItem cujo comprimento chega ao CLIFF da tela é DESCARTADO (o 26 clampava ~5016). Os 10000 do separador matam o item e nada é empurrado. Abaixo do cliff a barra não desliza: ela transborda pela ESQUERDA pro menu de overflow nativo (`«`). Cliff medido: metade da largura em tela sem notch; em tela COM notch (1512, área útil 663.5) o corte fica entre 520 e 530, ou seja ~0.79 da área à direita do notch, independente do app frontmost. Por isso a referência é `auxiliaryTopRightArea?.width` quando há notch, senão `frame.width`
  - Unidade = `max(200, floor(menorCliff - 64))` (`CollapsedGeometry.unitLength`), porque UM comprimento vale pra cópia da barra em toda tela e o cliff é por tela. Pra cobrir a tela mais larga sem passar do cliff da mais estreita, 6 spacers de length 0 inflam junto com o separador (`activeSpacers` = mínimo necessário; spacer sobrando transborda sozinho, vira linha em branco no `«`, inofensivo). Contagem FIXA: nome novo só visto num launch posterior cairia à esquerda do bloco

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junior-rj/menubar-hide](https://github.com/junior-rj/menubar-hide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
