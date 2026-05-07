---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

This is a Visual Studio 2022 (v143 toolset) C++ project. Windows 10 SDK required.

```
# From Visual Studio
# Open AirPlay.sln, then:
Ctrl+B (Build Solution)
F5 (Run with debugging)
Ctrl+F5 (Run without debugging)

# From x64 Native Tools Command Prompt
msbuild AirPlay.sln /p:Configuration=Debug /p:Platform=x64
```

Output: `x64\Debug\AirPlayServer.exe`

**Startup Project**: AirPlayServer (set via right-click → "Set as Startup Project")

## Coding Rules

- When replacing or superseding code/files, always delete the old unused code/files. Never leave dead code or replaced files behind.

**Build dependency chain**: AirPlayLib (static lib) → airplay2dll (DLL) → AirPlayServer (exe). dnssd builds independently.

### Runtime DLLs Required

The following must be in the same directory as `AirPlayServer.exe` (copied by airplay2dll post-build step):
- `airplay2dll.dll`, `dnssd.dll`
- `SDL.dll` (SDL 1.2.15)
- `avcodec-58.dll`, `avutil-56.dll`, `swscale-5.dll` (FFmpeg 4.x)
- `msys-2.0.dll` (MSYS2 runtime, required by libplist)

### Build Caveats

- airplay2dll links `libplist.a` from a hardcoded MSYS2 path (`C:\msys64\mingw32\lib\gcc\i686-w64-mingw32\9.2.0`). Builds will fail without MSYS2 installed at that path.
- `legacy_stdio_definitions.lib` is linked for CRT compatibility.
- Win32 Release uses `/SAFESEH:NO` for the airplay2dll project.

## Architecture Overview

### Solution Structure (4 projects in AirPlay.sln)

| Project | Type | Language | Purpose |
|---------|------|----------|---------|
| **AirPlayLib** | Static lib (.lib) | C | Core AirPlay 2 protocol: RAOP, pairing, crypto, FDK-AAC decoding |
| **airplay2dll** | DLL | C++ | Wraps AirPlayLib + FFmpeg H.264 decoding, exports `fgServerStart/Stop/Scale` |
| **dnssd** | DLL | C | mDNS/Bonjour service discovery, exports 28 `DNSService*` functions via `dnssd.def` |
| **AirPlayServer** | Console exe | C++ | Windows GUI app using SDL 1.2 + ImGui software renderer |

### Entry Point

`AirPlayServer/AirPlayServer.cpp` → `WinMain()` (Windows subsystem). Initializes CRT leak detection, Winsock, gets hostname, then calls `CSDLPlayer::init()` which starts the AirPlay server and enters the SDL event loop.

### Threading Model

- **Main Thread**: SDL event loop, ImGui rendering, display presentation (`CSDLPlayer::loopEvents`)
- **Callback Thread**: Receives AirPlay frames, copies to source buffer (`outputVideo`)
- **Scaling Thread**: Background YUV scaling with double-buffered output (`ScalingThreadProc`)
- **Audio Thread**: SDL audio callback pulls from queue (`sdlAudioCallback`)

### Key Data Flow

```
Network → raop.c/airplay.c → FgAirplayChannel (FFmpeg H.264→YUV420P)
       → CAirServerCallback → CSDLPlayer::outputVideo (mutex-protected memcpy to m_srcYUV)
       → Scaling thread: sws_scale(YUV420P → BGRA) into m_scaledYUV double-buffer (lockless)
       → Main thread: memcpy BGRA to m_videoBuffer → SDL_BlitSurface → ImGui overlay → SDL_Flip
```

### DLL Boundary (airplay2dll)

**Exported API** (`Airplay2Head.h`):
```cpp
void* fgServerStart(const char serverName[128], unsigned int raopPort,
                    unsigned int airplayPort, IAirServerCallback* callback);
void fgServerStop(void* handle);
float fgServerScale(void* handle, float fRatio);
```

**Callback Interface** (`IAirServerCallback`):
- `connected(remoteName, remoteDeviceId)` / `disconnected(...)` — Connection lifecycle
- `outputVideo(SFgVideoFrame*)` — Decoded YUV420 frame (pitch[3] for Y/U/V planes)
- `outputAudio(SFgAudioFrame*)` — Decoded PCM samples
- `videoPlay(url, volume, startPos)` / `videoGetPlayInfo(...)` — URL-based video
- `setVolume(volume)` — Volume in dB (0.0 = max, -144.0 = mute)
- `log(level, msg)` — Syslog-level logging (0=EMERG through 7=DEBUG)

### Frame Structures (Airplay2Def.h)

```cpp
typedef struct SFgVideoFrame {
    unsigned long long pts;
    int isKey;
    unsigned int width, height;
    unsigned int pitch[3];       // Y, U, V plane pitches
    unsigned int dataLen[3];     // Y, U, V plane lengths
    unsigned int dataTotalLen;
    unsigned char* data;         // Packed YUV420 data
} SFgVideoFrame;

typedef struct SFgAudioFrame {
    unsigned long long pts;
    unsigned int sampleRate;
    unsigned short channels, bitsPerSample;
    unsigned int dataLen;
    unsigned char* data;
} SFgAudioFrame;
```

### Audio Pipeline

- SDL audio spec: `AUDIO_S16SYS`, typically 48kHz stereo, 1024 sample buffer
- AirPlay typically sends 44.1kHz → linear interpolation resamples to match system device rate
- System sample rate queried via WASAPI Core Audio API (`ole32.lib`)
- Audio queue: max 20 frames (~400ms buffer), playback starts after 3 frames buffered
- Dynamic limiter: threshold 0.5, ratio 0.6, 2ms attack, 100ms release

### Video Pipeline

- FFmpeg H.264 decoder: `AV_CODEC_FLAG_LOW_DELAY`, 4 threads (`FF_THREAD_SLICE`)
- Decoder initializes on first keyframe (not stream start) for faster startup
- YUV420 dimensions forced even via ceiling division to prevent chroma drift
- Scaling thread outputs BGRA directly (YUV420P → BGRA in one sws_scale call)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xenos1337/AirPlayServer](https://github.com/xenos1337/AirPlayServer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
