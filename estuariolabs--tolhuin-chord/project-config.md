---
trigger: always_on
description: Sintetizador de acordes diatónicos sobre ESP32-S3 + DAC PCM5102 (I2S).
---

# TOLHUIN Chord Synthesizer — Guía para el agente

## Visión
Sintetizador de acordes diatónicos sobre ESP32-S3 + DAC PCM5102 (I2S).
Motor de audio propio (sin AMY, sin librerías externas de síntesis).
Objetivo: acercarse a la calidad y funciones del HiChord.

## Hardware
| Señal | GPIO |
|-------|------|
| I2S BCK | 38 |
| I2S WS / LRCK | 39 |
| I2S DIN (→ PCM5102) | 40 |
| SCK del PCM5102 | GND |

- MCU: ESP32-S3 con PSRAM OPI
- DAC: PCM5102 (I2S estéreo, 16-bit)
- Sample rate: 44100 Hz, bloque: 256 muestras
- Core 0: tarea de audio (I2S); Core 1: UI/lógica

## Arquitectura de archivos
```
tolhuin_chord_v2/
  tolhuin_chord_v2.ino <- sketch principal (setup/loop + I2S)
  evloop.h/.cpp       <- looper de EVENTOS (1 pista de acordes, puro/testeable)
  icons.h             <- pixel-art del OLED (GENERADO por tools/gen_icons.py)
  config.h            <- constantes globales (SAMPLE_RATE, MAX_CHORD_NOTES, etc.)
  state.h             <- tipos compartidos (AppState, Mode, ColorZone, Voicing)
  harmony.h/.cpp      <- motor de armonía diatónica (compila en host y ESP32)
  dsp.h/.cpp          <- síntesis pura (OSC, ADSR, filtro, mix) — SIN hardware
  synth.h/.cpp        <- wrapper I2S + FreeRTOS que llama a dsp
  webcfg.h/.cpp       <- protocolo de config por USB (Serial '#...' -> JSON) + NVS
  webtool/            <- editor web estático (Web Serial): timbres, drums, flasheo
  test/
    run_tests.ps1     <- runner de host (clang++/g++)
    test_harmony.cpp  <- tests de armonía (deterministas)
    test_dsp.cpp      <- tests de audio (NaN/clip/RMS/Goertzel) [creado en T0.3]
  diag_i2s_sine/      <- diagnóstico de hardware I2S (no tocar)
```

## Comandos de build
```powershell
# arduino-cli vive en "..\arduino-cli.exe" (no está en PATH)
# Compilar firmware (SOLO compile, nunca --upload en modo agente)
& "..\arduino-cli.exe" compile `
    --fqbn "esp32:esp32:esp32s3:USBMode=default,CDCOnBoot=cdc,PSRAM=opi" .

# Correr tests de host (el script agrega g++ de WinLibs al PATH solo)
powershell -ExecutionPolicy Bypass -File test\run_tests.ps1
```

Toolchain de host: g++ 16.1.0 (WinLibs UCRT, vía winget) en
`%LOCALAPPDATA%\Microsoft\WinGet\Packages\BrechtSanders.WinLibs.POSIX.UCRT_*\mingw64\bin`.
Core ESP32 3.2.0 y libs Adafruit (GFX, SSD1306, BusIO, ADS1X15) ya instalados.

## REGLAS DURAS (jamás romper)

### Armonía
1. Todas las notas deben ser **diatónicas** a la tonalidad/modo activos.
2. **Ningún acorde dominante**: no puede coexistir una 3ra mayor con una 7ma menor (b7).
3. Sin **b9** (intervalo 13 sobre la raíz), sin **b2** (intervalo 1).
4. Cantidad de notas en [1, MAX_CHORD_NOTES].
5. Voice leading: registro MIDI 36–91. La CONDUCCIÓN es un parámetro
   INDEPENDIENTE del voicing/inversión (`VoiceLeadMode`, fila LEAD, campo
   `app.voiceLeadMode`, default `VL_SMOOTH`): `VL_SMOOTH` (histórico: mínimo
   movimiento + tonos comunes), `VL_PARALLEL` (bloques/posición fija, ignora el
   previo), `VL_CHORALE` y `VL_CONTRARY` (4 voces sin cruces, búsqueda acotada
   por costos). `harmonyVoiceLead()` = wrapper de `VL_SMOOTH`;
   `harmonyVoiceLeadMode()` selecciona. Se aplica ANTES de `harmonyApplyVoicing`
   y de la octava global; entero/determinista; el historial no se mezcla entre
   algoritmos. Web/serial: `#lead 0..3`, campo `lead` en `#state`.

### Arquitectura
- **No AMY**: el motor de síntesis es propio (`dsp.h/.cpp`). El sketch `diag_amy_min` existe sólo como referencia histórica.
- **dsp.h/.cpp** no debe incluir `ESP_I2S.h`, `freertos/`, ni ningún header de Arduino. Debe compilar con `g++` en la PC.
- Comentarios en **español**.
- Separación modular: armonía / DSP / hardware en capas independientes.

## Modos de escala implementados
- `MODE_IONIAN` (mayor): 0 2 4 5 7 9 11
- `MODE_AEOLIAN` (menor natural): 0 2 3 5 7 8 10

## Motor de audio (estado actual)
Núcleo DSP puro en `dsp.h/.cpp` (compila en host y ESP32). Cadena por muestra:
`voces (osc + ADSR + filtro SVF + LFO) + percusión -> mezcla -> [+ 4 capas del
looper] -> chorus -> reverb -> delay -> tremolo de salida -> L/R`.

Módulos (todos testeables en host, ver `test/test_dsp.cpp`):
- **Osciladores band-limited (PolyBLEP)**: saw y cuadrada sin alias en agudos.
  Paleta: BRASS (pulsos detuneados), EPIANO (FM Rhodes), STRINGS (3 saws),
  SINE, TRIANGLE, ORGAN (aditivo 4 drawbars), FLUTE (aditivo + aliento).
  ORGAN y FLUTE como **wavetable** (1 lookup/muestra) para bajar CPU.
- **`Adsr`**: envolvente A/D/S/R por timbre, sin clicks (retrigger legato).
- **`Svf`**: filtro pasa-bajos resonante de 2 polos (TPT) + envolvente de filtro.
- **`Lfo`**: vibrato (pitch) y trémolo (amplitud) con rate/depth por timbre.
- **`DelayStereo`** (L≠R, feedback, mezcla), **`Reverb`** (Freeverb-lite),
  **`Chorus`** (LFO de retardo corto, 2 tomas L/R), **tremolo de salida** sync BPM.
- **BODY**: capa de cuerpo/unísono por voz (detune fijo + drift + transitorio de
  ataque) que acerca los timbres a un sample, sin tocar los osciladores.
- **SUB-BAJO dedicado** (`synthSetBass`, `app.subBass`, on por defecto): la raíz
  del grado −1 octava como VOZ PROPIA con timbre sine, rastreada aparte del
  acorde (no se re-dispara si la raíz se mantiene). Es el "bass role" del
  HiChord: CALIBRADO por FFT contra un audio real (tools/render_ref.cpp +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [estuariolabs/tolhuin-chord](https://github.com/estuariolabs/tolhuin-chord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
