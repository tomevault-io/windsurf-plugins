---
trigger: always_on
description: You are an expert full-stack developer working on Spleeter Web, a music source separation web application.
---

# AGENTS.md

You are an expert full-stack developer working on Spleeter Web, a music source separation web application.

## Project Overview

Spleeter Web isolates or removes vocal, accompaniment, bass, drums, guitar, and piano components from songs. Users upload audio files or import from YouTube, then the backend separates the audio into stems using ML models.

**Tech Stack:**
- **Backend:** Django, Django REST Framework, Celery, Redis
- **Frontend:** React 16 with TypeScript, Webpack 4, Bootstrap 4
- **ML Models:** Spleeter (TensorFlow), Demucs v4 (PyTorch), BS-RoFormer (PyTorch)
- **Audio:** Tone.js for browser playback, FFmpeg for processing
- **Infrastructure:** Docker, Docker Compose, Nginx, SQLite

## File Structure

```
├── api/                   # Django backend API
│   ├── models.py          # Database models (SourceTrack, StaticMix, DynamicMix)
│   ├── views.py           # API endpoints
│   ├── tasks.py           # Celery tasks for separation & YouTube import
│   ├── serializers.py     # DRF serializers
│   ├── separators/        # ML model wrappers
│   │   ├── spleeter_separator.py
│   │   ├── demucs_separator.py
│   │   └── bs_roformer_separator.py
│   └── migrations/        # Database migrations
├── frontend/              # React frontend
│   ├── src/
│   │   ├── components/    # React components
│   │   │   ├── Mixer/     # Audio mixer (MixerPlayer.tsx is core)
│   │   │   ├── SongTable/ # Track listing and forms
│   │   │   └── Home/      # Homepage components
│   │   ├── models/        # TypeScript interfaces
│   │   └── Constants.tsx  # App-wide constants
│   └── templates/         # Django templates
├── django_react/          # Django project settings
│   ├── settings.py        # Base settings (non-Docker)
│   ├── settings_docker.py # Docker settings
│   └── urls.py            # URL routing
├── media/                 # User uploads and separated tracks
│   ├── uploads/           # Uploaded source files
│   └── separate/          # Separated stem files
├── config/                # Model configuration files
└── pretrained_models/     # Downloaded ML models (gitignored)
```

## Code Style

### Python
- Follow Django conventions for models, views, and serializers
- Celery tasks go in `api/tasks.py`
- Separator implementations go in `api/separators/` with a common interface

### TypeScript/React
- Use TypeScript interfaces in `frontend/src/models/`
- Components are class-based (legacy codebase), not functional/hooks
- State management via component state, not Redux
- Use Bootstrap 4 classes for styling
- Prettier and ESLint configured (see `frontend/.eslintrc.js`)

### Frontend Architecture
- **Entry point:** `frontend/src/index.tsx` renders the main `App` component
- **Routing:** React Router in `App.tsx` with routes for Home, Mixer, etc.
- **API calls:** Use `fetch()` with endpoints defined in `Constants.tsx`
- **Audio playback:** Tone.js library (`Tone.Player`, `Tone.Channel`, `Tone.Destination`)

### Key Frontend Files
| File | Purpose |
|------|---------|
| `components/Mixer/MixerPlayer.tsx` | Core audio mixer with playback, solo/mute, volume controls |
| `components/SongTable/Form/StaticMixModalForm.tsx` | Form for creating static mixes (parts to keep) |
| `components/SongTable/Form/DynamicMixModalForm.tsx` | Form for creating dynamic mixes |
| `components/SongTable/Form/SeparatorFormGroup.tsx` | Separator model selection dropdown |
| `components/SongTable/Modal/StaticMixModal.tsx` | Modal + state for static mix creation, sends API request |
| `components/SongTable/Modal/DynamicMixModal.tsx` | Modal + state for dynamic mix creation, sends API request |
| `components/SongTable/MixTable.tsx` | Table showing mixes with status, badges, actions |
| `components/Home/MusicPlayer.tsx` | Audio player with badge display for current track |
| `models/Separator.ts` | TypeScript types for separator models |
| `models/MusicParts.ts` | Maps of stem keys to display names |
| `models/PartId.ts` | Union type of valid stem identifiers |
| `Constants.tsx` | API endpoints, URLs, app-wide constants |

### Adding UI for New Stems
When adding a new stem type (e.g., guitar, piano):

1. **Type definitions:**
   - Add to `PartId` type in `models/PartId.ts`
   - Add URL field to interfaces in `models/DynamicMix.ts` and `models/StaticMix.ts`

2. **Badge component:**
   - Create `components/Mixer/Badges/<Stem>Badge.tsx`
   - Export from `components/Badges.tsx` barrel file
   - Use consistent color scheme (vocals=green, drums=yellow, bass=red, guitar=brown, piano=purple)

3. **MixerPlayer updates:**
   - Add to `VolumeLevels`, `MuteStatus`, `SoloStatus` interfaces
   - Add state initialization in constructor
   - Add `has<Stem>()` method to check if stem exists
   - Add keyboard shortcut in `onKeyPress()` handler
   - Add to `isNoneSoloed()` check (critical for solo/mute logic)
   - Add `Tone.Player` and `Tone.Channel` for audio playback
   - Update `componentWillUnmount()` cleanup

4. **Forms and Modals:**
   - Add new `MusicPartMap` in `models/MusicParts.ts`
   - Update `StaticMixModalForm.tsx` to select correct map based on separator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeffreyCA/spleeter-web](https://github.com/JeffreyCA/spleeter-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
