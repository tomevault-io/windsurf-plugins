---
trigger: always_on
description: Guia de integração para agentes de IA que trabalham neste repositório.
---

# AGENTS.md

Guia de integração para agentes de IA que trabalham neste repositório.

## Referência rápida

- **Nunca** editar versão manualmente → use sempre `packaging/release.sh X.Y.Z`.
- **Nunca** fazer commit ou push sem pedido explícito do usuário.
- Todo build roda dentro do distrobox `fedora` (o host é Silverblue imutável, sem toolchain).
- Antes de mexer no engine: `bash -n backend/open-couch-engine` (não há suíte de testes).
- Lógica de exibição/monitor vive no **engine bash** (`backend/`), não na GUI (`app/`).
- Se a mudança tocar em algo documentado aqui (build, versionamento, traduções, arquitetura), **atualize este arquivo na mesma mudança**.

```sh
# Build
distrobox enter fedora -- bash -c \
  "cmake -S app -B app-build -DCMAKE_BUILD_TYPE=Release -DINSTALL_ENGINE_BUNDLE=ON"
distrobox enter fedora -- bash -c "cmake --build app-build --parallel \$(nproc)"

# Checagem de sintaxe do engine
bash -n backend/open-couch-engine

# Release
packaging/release.sh X.Y.Z && git push origin main --tags
```

> Bloco acima é só referência rápida — detalhes completos em **Build**, **Versionamento** e **Armadilhas conhecidas** abaixo.

## Visão geral

Open Couch é um aplicativo Linux para KDE Plasma que alterna o layout de monitores entre a mesa e a TV da sala com um clique, lança o Steam Big Picture e restaura o layout do desktop automaticamente quando o jogo termina. Licença GPL-3.0-or-later. Repositório: `GustavoBelo/OpenCouch` (branch `main`).

O projeto é dividido em três partes:

| Caminho | Papel |
|---|---|
| `app/` | GUI em Qt6 + Kirigami (C++17 + QML). Ponte entre a UI e o engine. |
| `backend/` | O "engine": scripts bash que controlam os displays e monitoram o Steam. |
| `packaging/` | Scripts de release, Flatpak, instalador host e metadados AppStream. |

A GUI é apenas uma camada: toda a lógica de exibição vive no engine bash (`backend/open-couch-engine`), que é invocado pela aplicação via `QProcess`.

## Arquitetura

### app/ — GUI Qt6/QML

- `src/main.cpp` — bootstrap: instância única (QLocalServer), tradutores, engine QML, context properties (`backend`, `displaySettingsModel`, `appCleanupModel`, `appInfo`).
- `src/backend.{h,cpp}` — ponte QML↔engine. Expõe `Q_INVOKABLE`s para todas as ações (play, restore, status, logs, autostart, engine install). Roda o engine de forma síncrona (`runSync`) ou assíncrona (`runEngineAsync`).
- `src/engineclient.{h,cpp}` — constrói a linha de comando do engine (usa `flatpak-spawn --host` dentro de Flatpak), versão e instalação do engine empacotado em `~/.local/bin`.
- `src/configstore.{h,cpp}` — config (`config.env`), autostart (desktop entry / portal Background), `backgroundOnClose`, onboarding, e chaves de limpeza de apps (`CLOSE_APPS_ENABLED`, `CLOSE_APPS_WAIT_SECONDS`, `APPS_TO_CLOSE`).
- `src/displaysettingsmodel.{h,cpp}` — modelo de settings usado pela tela de configuração.
- `src/displaysettingsvalidator.{h,cpp}` — valida DESK_OUTPUT/TV_OUTPUT/scale/pos antes de salvar.
- `src/appcleanupmodel.{h,cpp}` — modelo de controle de recursos: lista de apps a fechar, tempo de espera e integração com `close-tracked-apps` do engine. Pontos-chave:
  - **Varredura nativa** de `.desktop` via `QStandardPaths`/`QDir`/`QFile`/`QDirIterator`, até depth 2, seguindo symlinks flatpak.
  - **Varredura de processos** via `/proc` + `/proc/<pid>/exe|comm|cmdline`, filtrando `PROTECTED_PROCESSES` e cruzando com os `.desktop` encontrados.
  - **Cache em memória por sessão** (`QMap` lower → displayName/icon).
  - **Carregamento assíncrono**: `QThread::create` + `installedApps`/`runningApps`/`loadingInstalled`/`loadingRunning` + `requestInstalledApplications`/`requestRunningApplications` + `BusyIndicator`.
  - Em Flatpak, usa `/run/host` ou `flatpak-spawn --host open-couch-engine` como fallback.
- `src/appinfomodel.{h,cpp}` — nome, versão e URL do script de instalação.
- `qml/` — `main.qml`, `SetupPage.qml`, `DashboardPage.qml`, `OnboardingSheet.qml`, `ChooseAppDialog.qml`, `RunningAppsDialog.qml` (Kirigami, `QtQuick.Controls`).
- `translations/` — catálogos Qt Linguist (`.ts`); `opencouch_en.ts` é o catálogo base.

### backend/ — engine

- `open-couch-engine` — script bash (com `set -euo pipefail`). Comandos core: `play`, `restore`, `status`, `outputs`, `check`, `version`, `watch`, `config-path`, `log`, `append-log`, `clear-log`, `log-history`, `print-history-log`, `export-history-log`, `export-log`, `close-tracked-apps`; legados `list-running`/`list-apps` mantidos só para CLI/host fallback.
- Dependências de host: `jq`, `kscreen-doctor`, `pgrep`; opcional: `wmctrl` (apenas para posicionar/fechar janelas X11 do Steam — não funciona em sessões Wayland puras).
- O `status` registra no log os componentes ausentes (obrigatórios como `ERROR`, opcionais como `WARNING`); o app usa `append-log` para persistir eventos próprios no arquivo (ex.: falha do watcher).
- **`EXIT_ON_ALL_CONTROLLERS_OFF`** (opção de config): quando habilitada, o modo sala encerra o Big Picture e restaura o desktop quando todos os controles são desligados.
  - Debounce de 10s antes de agir.
  - Exige mínimo de 1 minuto de uso de controle na sessão.
  - Detecção via `/dev/input/js*`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GustavoBelo/OpenCouch](https://github.com/GustavoBelo/OpenCouch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
