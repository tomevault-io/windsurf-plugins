---
trigger: always_on
description: Este documento define os princípios de engenharia de software, padrões de arquitetura e o fluxo de trabalho obrigatório para agentes de IA que desenvolvem no repositório **Monky**.
---

# Diretrizes e Fluxo de Trabalho para Agentes de IA

Este documento define os princípios de engenharia de software, padrões de arquitetura e o fluxo de trabalho obrigatório para agentes de IA que desenvolvem no repositório **Monky**.

---

## 🏛️ Princípios de Engenharia de Software & Arquitetura

Todo código produzido deve seguir as melhores práticas da indústria, com foco em **clareza, robustez, performance e manutenibilidade**, evitando complexidade desnecessária (_sem overengineering_).

### 1. Diretrizes Técnicas por Área do Projeto

#### 🔹 Electron, IPC & Segurança

- **Isolamento Total:** Mantenha `contextIsolation: true` e `nodeIntegration: false`. Nunca exponha módulos inteiros do Node.js ou Electron diretamente no `window` via Preload.
- **Tipagem Estrita de IPC:** Utilize sempre os contratos centralizados em `packages/shared/src/ipc.ts`. Proibido utilizar strings literais soltas ou tipos `any` em chamadas `ipcMain.handle` / `ipcRenderer.invoke`.
- **Prevenção de Memory Leaks no IPC:** Remova listeners e evite registrar handlers IPC repetidamente durante o ciclo de vida da aplicação.
- **Sanitização no Main Process:** Valide todos os payloads recebidos do Renderer antes de executar operações de I/O, rede, arquivos ou navegação externa (`shell.openExternal`).

#### 🔹 WebRTC & Processamento de Áudio/Vídeo

- **Gerenciamento de Ciclo de Vida:** Sempre execute o teardown completo de `RTCPeerConnection` e interrompa trilhas de mídia (`MediaStreamTrack.stop()`) ao encerrar chamadas, desativar câmeras ou mutar microfones.
- **Web Audio API Cleanup:** Feche e desconecte instâncias de `AudioContext`, `MediaStreamAudioSourceNode`, `GainNode` e `AudioWorkletNode` para evitar consumo fantasma de CPU e memória.
- **Resiliência de Rede:** Trate _race conditions_ em trocas de ofertas/respostas SDP e candidatos ICE, garantindo reconexão graciosa em instabilidades de rede.

#### 🔹 Módulos Nativos C++ / Node-API (`@monky/screen-audio`)

- **Memory & Thread Safety:** Garanta a liberação correta de buffers e recursos de áudio do sistema operacional (WASAPI no Windows, CoreAudio no macOS). Nunca trave a thread principal do Node.js — utilize `napi_threadsafe_function` para callbacks assíncronos.
- **Resiliência a Falhas:** Isole erros de captura de áudio com tratamento de exceções robusto para que falhas em dispositivos de som não causem crash no processo Main.

#### 🔹 Renderer Vanilla TypeScript & DOM

- **Limpeza de Event Listeners:** Sempre remova listeners vinculados a `window`, `document`, elementos do DOM ou ao `EventBus` quando uma view, modal ou componente for desmontado/fechado.
- **Gerenciamento de Estado Previsível:** Concentre estados nas Stores dedicadas (`chatStore`, `voiceStore`, `serverStore`, `settingsStore`, `connectionStore`), aplicando mutações claras e sem dependências circulares.
- **Performance de Renderização:** Evite reflows e repaints desnecessários no DOM; prefira mutações cirúrgicas a recriações massivas de HTML.

#### 🔹 Backend / Servidor (Clean Architecture Pragmática)

- **Separação de Responsabilidades:** Respeite a divisão entre `domain` (regras puras), `application` (casos de uso) e `infrastructure` (SQLite, WebSocket, rede).
- **Persistência Segura:** No SQLite, utilize transações para operações em lote e prepare queries parametrizadas para evitar corrupção e injeção de dados.
- **WebSocket Resiliente:** Mantenha rotinas de heartbeat (ping/pong) para identificar conexões zumbis e limpar recursos associados.

#### 🔹 Rigor em TypeScript

- **Sem `any`:** Não utilize `any` nem asserções forçadas (`as unknown as Type`). Modele tipos e interfaces no `@monky/shared` sempre que forem compartilhados.
- **Tratamento Seguro de Nulos/Erros:** Trate explicitamente `null`, `undefined` e exceções com blocos `try/catch` e tipos discriminados.

---

### 2. Pragmatismo & Anti-Overengineering

- **KISS (Keep It Simple, Stupid):** Prefira sempre a solução mais simples e legível que resolva o problema com eficácia.
- **YAGNI (You Aren't Gonna Need It):** Não crie abstrações prematuras, camadas intermediárias vazias, fábricas ou padrões complexos para funcionalidades que não existem no escopo atual.
- **Legibilidade em Primeiro Lugar:** Código limpo é autoexplicativo, bem nomeado e com comentários focados no _porquê_ das decisões não óbvias.

---

## 👨‍🏫 Didática & Comunicação com o Desenvolvedor

A comunicação do agente deve ser sempre **clara, didática, transparente e respeitosa**:

1. **Explique o Raciocínio:** Não se limite a entregar código pronto. Explique de forma didática o que causava o problema anterior e por que a solução adotada é a ideal.
2. **Ensine o Conceito:** Ao corrigir bugs ou aplicar refatorações complexas (ex.: memory leaks, gerenciamento de eventos, concorrência WebRTC), contextualize a causa raiz e a boa prática envolvida.
3. **Transparência em Decisões:** Caso precise escolher entre diferentes abordagens, exponha brevemente os trade-offs (prós e contras) para o desenvolvedor.

---

## 🔍 Rigor ao Afirmar e ao Documentar


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MonkyOrg/Monky](https://github.com/MonkyOrg/Monky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
