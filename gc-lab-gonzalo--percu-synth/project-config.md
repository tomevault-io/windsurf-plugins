---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

```
percusynth/
├── firmwares/                      # Hand-written Arduino sketches
│   ├── drum_machine_basic/         #   16-step drum machine + real-time synth
│   ├── drum_ruido/                 #   Noisy-timbre drum machine with a CLEAN output chain: random 32-step patterns, 5 kits, tap tempo, half-time, fill, pot beat-repeat
│   ├── drum_poder/                 #   Percussion-only drum machine with a big low end (12 hand-written odd-time grooves) + MIDI Clock out the DIN-5
│   ├── synth_basico/               #   5-voice polyphonic synth
│   ├── trance_imu/                 #   Polyphonic trance sequencer (IMU → filter)
│   ├── trance_imu_leds/            #   trance_imu + 6 on-board SMD LEDs as visualizer
│   ├── pads_imu/                   #   Deep ambient pads: 5 buttons play sustained chords (no sequencer, stereo)
│   ├── pads_imu_leds/              #   pads_imu + 6 on-board SMD LEDs as visualizer (panel palette + arp running dot)
│   ├── cancion_aleatoria_leds/     #   Autonomous random-song machine (Play/Stop + 3 volume pots): generative key/mode/progression/tempo + random monophonic melody + random synthesized percussion + 6 LEDs
│   ├── paisajes_relax_leds/        #   Relaxing soundscape machine: 10 combinable synthesized layers (wind/sea/chimes/drops+echo/…) + 2 WS2812 rings of 30 LEDs
│   ├── cyber_kit/                  #   Cyber texture/FX/lead sequencer (no drums): 4 sound banks tuned to 10 scales, 4 pot panels, PERC/SEQ modes with full transport (play/stop/reverse/beat-repeat/speed/chaos), POTS/IMU filter
│   ├── asistente_ia/               #   Voice assistant (Whisper → GPT-4o-mini → TTS); the simplest of the AI family — read it first
│   ├── asistente_naga/             #   Same assistant over NagaAI (api.naga.ac): one key for STT/chat/TTS, free models, WAV-header-driven playback
│   ├── sampler_ia/                 #   Voice-prompted AI sampler: Whisper → GPT → ElevenLabs SFX → PSRAM slot, 3 trigger buttons + 16-step sequencer
│   ├── asistente_musical/          #   Voice chat with GPT over a never-stopping generative pad (voice high-pass on a pot + sidechain ducking; audio on core 1, network on core 0)
│   ├── oscilador_ia/               #   The AI sample IS the oscillator: ElevenLabs note @A2 → pitch-detect → tuned keyboard/arps/infinite drones/generative seq (10 scales)
│   ├── compositor_ia/              #   Ask for a song out loud: Whisper → GPT returns a compact song JSON → the cancion_aleatoria engine plays it (real swing, sections, 5 bass patterns); JSON can also be pasted over Serial with no WiFi
│   ├── oscilador_escalas/          #   Drone: 4 pots = 4 oscillators quantized to a scale (port of Proto-Synth v2's Oscilador_4_escalas without Mozzi) — unison+sub, 10 scales on one button, IMU X → resonant LPF, tempo gate + tap tempo, tape delay
│   ├── oscilador_escalas_clock/    #   oscilador_escalas + MIDI Clock out the DIN-5 (24 PPQN at the tap tempo; Start/Stop follow the intermittence, Stop+Start on every tap) + the 6 LEDs blink with the gate + the module's RGB LED shows the sounding note's colour
│   ├── bajo_8_pasos/               #   8-step saw bass sequencer in A minor pentatonic: panels A/B = step notes, C = vol/speed/decay/cutoff, BTN5 octave (made with the participants of the open workshop at Hive)
│   ├── bajo_8_pasos_fsr_clock/     #   bajo_8_pasos + FSR on EXT1 sweeping the filter + MIDI Clock out the DIN-5
│   ├── industrial_matriz/          #   Kick + powerful bass base with a parallel industrial sequencer (always in phase) + continuous oscillator on the menu's POT4 + visuals on a 32×8 WS2812 matrix
│   ├── secuenciador_melodico/      #   (by Nicolás Martínez) 16-step mono sequencer in the 7 Greek modes + Web Serial webapp; needs ESP32Synth
│   ├── secuenciador_melodico_cromatico/ # (by Nicolás Martínez) 16-step polyphonic sequencer over 12 chromatic notes + Web Serial webapp; needs ESP32Synth
│   ├── la_partida/                 #   Plays "La partida" (Víctor Jara) from its 3 MIDI tracks: filtered lead / detuned saw+square bass an octave down / electronic pluck arp, sample-accurate timing, panels per track
│   ├── espacio_modular/            #   MONOPHONIC film-ambience machine (6-osc bank + continuous tonic drone): 24 curated 4-bar THEMES over a per-mode non-functional progression, one chord per 4-bar cycle, fixed-time ping-pong delay + reverb; single panel (5 buttons, 4 pots), Play rolls the mode, BTN2 rolls the theme, IMU X → cutoff
│   ├── impact_chimes/              #   Floor-impact chimes (accelerometer triggers scale notes)
│   ├── laser_chimes/               #   Same chimes triggered by breaking a laser beam (LDR on EXT1/GPIO 3)
│   ├── voz_fx/                     #   Real-time voice processor: INMP441 → 11 effects (vocoder, autotune, robot, pitch, harmonizer, tape echo, cathedral…) → DAC
│   ├── grabador_campo/             #   Field recorder to microSD: live gain/filter/EQ/FX on the mic + an untouched raw track, freeze

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GC-Lab-Gonzalo/Percu-Synth](https://github.com/GC-Lab-Gonzalo/Percu-Synth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
