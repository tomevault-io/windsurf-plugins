---
trigger: always_on
description: Este ficheiro é lido no arranque de qualquer thread que mexa neste projeto.
---

# Spit (VoiceFlow) — Instruções para agentes

Este ficheiro é lido no arranque de qualquer thread que mexa neste projeto.
**Lê-o por completo antes de editar código.**

## Nome

O projeto chama-se **Spit** (user-facing). O bundle identifier é `app.getspit`.
O repo ainda se chama `VoiceFlow` por razões históricas — ignora essa inconsistência.

## O que é

App macOS menu-bar de ditado por voz com:
- Hotkey global (por defeito Globe 🌐) para iniciar/parar ditado
- Transcrição via Whisper (proxy Groq/OpenIA para trial/pro, BYOK OpenAI/Groq, ou local)
- Injecção de texto no app em foco (AX API, fallback clipboard+⌘V)
- TTS (read selection) na mesma hotkey com comportamento "smart"
- Live preview de palavras via `SFSpeechRecognizer` durante a gravação
- Pausa/retoma media (Spotify, Apple Music, etc.) durante ditado ou leitura caso esteja a reproduzir algo ao teclar o hotkey

Para visão geral de arquitectura → `ARCHITECTURE.md`.
Para spec funcional → `SPEC.md` e `SPEC-AUTH.md`.
Para histórico de bugs corrigidos → `CHANGELOG.md`.

> **Regra de auditoria**: quando o utilizador pedir para "rever / auditar /
> verificar divergências" entre código e documentação, ler **todos** os
> documentos listados acima (CLAUDE.md, CHANGELOG.md, ARCHITECTURE.md,
> SPEC.md, SPEC-AUTH.md, `.claude/rules/*.md`). Nunca restringir a um
> subset sem instrução explícita do utilizador. Cada um cobre uma camada
> diferente (regras para agentes, lições de bugs, arquitectura interna,
> spec funcional de produto, autenticação) e uma auditoria só é completa
> quando cobre todas.

## Build de desenvolvimento vs produção — SEPARADAS (2026-07-07)

A build de **Debug** é uma app distinta da de produção, para poderem coexistir
e ser testadas em simultâneo sem conflito:

| | Debug ("Spit Dev") | Release (produção) |
|---|---|---|
| Bundle ID | `app.getspit.dev` | `app.getspit` |
| Ícone na barra | martelo 🔨 | waveform |
| Hotkey default | ⌥ Option direito (keyCode 61) | Globe 🌐 (keyCode 63) |
| Auto-relaunch (LaunchAgent) | **desligado** | ligado |
| Container/definições/log | isolados em `app.getspit.dev` | `app.getspit` |

Isto resolveu o bug em que ter as duas instaladas com o mesmo bundle ID fazia
o guard de instância matar uma e as duas competirem pelo mesmo CGEventTap do
Globe → travamentos. **Nunca reverter o bundle ID da Debug para `app.getspit`.**
A proibição de mudar o bundle ID refere-se só à **produção** (Keychain/licenças).

## Protocolo de rebuild (OBRIGATÓRIO após qualquer edit)

Usa o slash command `/rebuild` — é o caminho canónico. Corresponde a:

```bash
# Mata SÓ a Dev (por path), nunca a produção do utilizador
pkill -f "DerivedData/VoiceFlow-hfayfoyiaxzwzjdermhtiguqxtnn/Build/Products/Debug/Spit Dev.app" 2>/dev/null
cd "/Users/rafaellopes/Library/CloudStorage/GoogleDrive-rafa@rafamail.com/Meu Drive/Empreendedorismo/Spit"
xcodebuild -project VoiceFlow.xcodeproj -scheme VoiceFlow -configuration Debug -destination 'platform=macOS' build
open "$HOME/Library/Developer/Xcode/DerivedData/VoiceFlow-hfayfoyiaxzwzjdermhtiguqxtnn/Build/Products/Debug/Spit Dev.app"
```

Não esperar que o utilizador peça — é automático.
Avise o usuário depois e feito para que ele possa testar.

## Paths críticos

| Artefacto | Path |
|---|---|
| Código fonte | `/Users/rafaellopes/Library/CloudStorage/GoogleDrive-rafa@rafamail.com/Meu Drive/Empreendedorismo/Spit/VoiceFlow/` |
| DerivedData app (Dev) | `~/Library/Developer/Xcode/DerivedData/VoiceFlow-hfayfoyiaxzwzjdermhtiguqxtnn/Build/Products/Debug/Spit Dev.app` |
| **Debug log (Dev, runtime)** | `~/Library/Containers/app.getspit.dev/Data/Library/Logs/Spit/spit-debug.log` |
| Debug log (produção) | `~/Library/Containers/app.getspit/Data/Library/Logs/Spit/spit-debug.log` |
| Crash reports | `~/Library/Logs/DiagnosticReports/Spit-*.ips` |
| Settings Dev (UserDefaults) | `~/Library/Containers/app.getspit.dev/Data/Library/Preferences/app.getspit.dev.plist` |
| Keychain service | `app.getspit` — chaves `byok.openai`, `byok.groq`, JWT de licença |

**Regra de ouro de debugging:** `tail -200` do `spit-debug.log` antes de propor qualquer fix.
Nunca diagnosticar por intuição — `FileLogger.swift` escreve síncrono com fsync.

## Regras scoped (lidas automaticamente por `paths:`)

Regras específicas de áreas do código estão em `.claude/rules/*.md` com
`paths:` no frontmatter. Só são carregadas quando editas ficheiros dessa
área. Conteúdo actual:

| Ficheiro | Cobre |
|---|---|
| `.claude/rules/audio-pipeline.md` | `AudioRecorder`, `SystemAudioManager`, `LiveSpeechRecognizer` |
| `.claude/rules/hotkey-and-input.md` | `HotkeyManager`, `TextInjector`, `FocusDetector` |
| `.claude/rules/dictation-controller.md` | `DictationController`, `HUDCoordinator` |

Se vais editar um destes ficheiros, a regra correspondente já estará no teu
contexto. Se vais editar código que **interage** com estes módulos mas não
pertence a eles, abre o ficheiro de regras manualmente antes de propor mudanças.

## Como diagnosticar bugs (método, não intuição)

1. **Ler os últimos 100-200 linhas do `spit-debug.log`** — o ciclo completo
   aparece lá (`startDictation called` → `media paused` → `LiveSpeechRecognizer
   started` → `stopDictation called` → `transcribe OK` → `injected via …`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Draxo-io/spit](https://github.com/Draxo-io/spit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
