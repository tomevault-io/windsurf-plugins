---
trigger: always_on
description: Plataforma C++/Qt 6.11.0 (QML) para tracking comportamental em neurociência. O sistema monitora ratos em até 3 campos simultâneos (mosaico 2×2) usando modelo DeepLabCut exportado em ONNX, executando inferência **nativamente em C++** (sem subprocesso Python).
---

# MindTrace — Contexto para IA

## Visão Geral

Plataforma C++/Qt 6.11.0 (QML) para tracking comportamental em neurociência. O sistema monitora ratos em até 3 campos simultâneos (mosaico 2×2) usando modelo DeepLabCut exportado em ONNX, executando inferência **nativamente em C++** (sem subprocesso Python).

## Arquitetura

**Stack:** C++17, Qt 6.11.0, ONNX Runtime 1.24.4, QML
**Compatibilidade:** Windows 10/11 (64-bit), MSVC 14.4+ (VS 2022 ou superior)

### Fluxo de Tracking (Pipeline Nativo)

```
QMediaPlayer (headless)
  → QVideoSink::videoFrameChanged (DirectConnection, multimedia thread)
    → InferenceController::onVideoFrameChanged
      → frame.toImage() → convertToFormat(RGB888)
        → InferenceEngine::enqueueFrame (thread-safe, single-slot queue)
          → processJob() — 3 std::threads, uma por campo
            → inferCrop() — Ort::Session.Run() → locref → emit sinais
              → trackReceived / bodyReceived (QueuedConnection → QML)
```

**Pontos críticos da arquitetura:**
- **Headless player + displayPlayer separado**: O `QMediaPlayer` headless (C++) serve frames ao motor de inferência. O vídeo visível no QML é um segundo `MediaPlayer` independente — pode usar hardware acceleration.
- **3 sessões ONNX paralelas**: Uma `Ort::Session` por campo, cada uma rodando em `std::thread` próprio dentro de `processJob()`.
- **Single-slot queue**: `enqueueFrame()` descarta frame anterior pendente — sempre processa o mais recente, evitando backpressure.

## Componentes C++

| Arquivo | Responsabilidade |
|---|---|
| `src/tracking/inference_engine.h/cpp` | QThread dedicada. Cria 3 `Ort::Session`, processa crops em paralelo via `std::thread`, aplica locref sub-pixel |
| `src/tracking/inference_controller.h/cpp` | Orquestrador. `QVideoSink` recebe frames do `QMediaPlayer` headless via `videoFrameChanged`, alimenta InferenceEngine, emite sinais para QML |
| `src/manager/ExperimentManager.cpp/h` | Gestão de experimentos (CRUD, I/O JSON/CSV, `registry.json`) |
| `src/models/ExperimentTableModel.cpp/h` | Modelo de tabela para CSVs (lazy-loading) |
| `src/models/ArenaModel.cpp/h` | Engine de persistência das zonas e polígonos |
| `src/models/ArenaConfigModel.cpp/h` | Modelo de configuração da arena |
| `src/core/main.cpp` | Ponto de entrada e registro de tipos QML |

## QML

| Pasta / Arquivo | Responsabilidade |
|---|---|
| `qml/core/` | Navegação base (`main.qml`, `LandingScreen.qml`), componentes reutilizáveis (`GhostButton.qml`, `Toast.qml`) |
| `qml/shared/` | Funcionalidades comuns como `LiveRecording.qml` (Análise) e `SessionResultDialog.qml` (Dados pós-sessão) |
| `qml/nor/` | Fluxo do Reconhecimento de Objetos: `NORDashboard.qml` (Antigo `MainDashboard`), `ArenaSetup.qml`, `NORSetupScreen.qml` |

## Modelo ONNX

- **Arquivo:** `Network-MemoryLab-v2.onnx`
- **Input:** `[1, 240, 360, 3]` RGB float32 — sem mean subtraction (modelo já faz)
- **Output 0:** `[1, 30, 46, 2]` scoremap (heatmaps nose/body)
- **Output 1:** `[1, 30, 46, 4]` locref (sub-pixel offsets)
- **Stride:** 8.0
- **Locref stdev:** 7.2801
- **Sem mean subtraction** — o grafo já normaliza internamente

## GPU / ONNX Execution Providers

O `InferenceEngine` detecta o fabricante da GPU via **DXGI** e roteia para o provider ideal:

- **DXGI** (`IDXGIFactory1`) enumera os adaptadores e lê o `VendorId` (`0x10DE`=NVIDIA, `0x1002`=AMD, `0x8086`=Intel) — chamada única na inicialização, sem overhead durante inferência
- **NVIDIA → CUDA**: requer `onnxruntime-win-x64-gpu-1.24.4` + drivers CUDA. Se o build for o padrão (sem CUDA), cai automaticamente para DirectML
- **AMD/Intel → DirectML**: requer `onnxruntime-directml` binários + `DirectML.dll` (extraídos do NuGet) + DirectX 12 (Win10+)
- **CPU fallback**: ativa quando nenhum provider GPU está disponível
- **GPU ativo** (CUDA ou DML): `SetIntraOpNumThreads(1)` — provider gerencia paralelismo
- **CPU fallback**: `SetIntraOpNumThreads(4)` + `ORT_ENABLE_ALL` graph optimization
- **`dxgi.lib` linkado** para detecção de vendor. **`d3d12.lib` NÃO é linkado** — DX12/CUDA são internos ao `onnxruntime.dll`

> **Build ONNX Runtime:**
> - NVIDIA     → `setup_onnx.ps1 -GpuType CUDA` (Baixa do GitHub)
> - AMD/Intel → `setup_onnx.ps1 -GpuType DML` (Baixa do NuGet)
> O script automatiza o download e organização em `onnxruntime_sdk/`.

### Configuração GPU

```cpp
const GpuVendor vendor = detectGpuVendor(); // DXGI VendorId

if (vendor == GpuVendor::NVIDIA && try_add_cuda_provider(opts)) {
    opts.SetIntraOpNumThreads(1);
    emit infoMsg("Modo GPU: CUDA ativo (NVIDIA)");
} else if (try_add_dml_provider(opts)) {
    opts.SetIntraOpNumThreads(1);
    emit infoMsg(QString("Modo GPU: DirectML ativo (%1, DirectX 12)").arg(gpuName));
} else {
    opts.SetIntraOpNumThreads(4);
    opts.SetGraphOptimizationLevel(ORT_ENABLE_ALL);
    emit infoMsg("Modo CPU: GPU não disponível");
}
```

**Alternativa Python (`inference_processor.py`)**: script standalone que usa `onnxruntime` com `CPUExecutionProvider` apenas — serve como referência/gold standard, não integrado ao pipeline C++.

## Velocidade de Reprodução


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RodrigoOrvate) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
