---
trigger: always_on
description: > **Living Documentation for AI-Assisted Development**
---

# CLAUDE.md - MyVocaList Project Context

> **Living Documentation for AI-Assisted Development**
> Last Updated: December 23, 2025
> Version: 2.6

---

## 📱 Application Overview

**MyVocaList** is a comprehensive .NET MAUI 8.0 mobile application designed for intelligent karaoke queue management with advanced features and future social network capabilities.

### Core Purpose
Manage karaoke participant queues with intelligent round-based organization, allowing administrators to track participation/absence, reorder singers, and provide real-time queue status information.


## Detailed Objectives

MyVocaList is a .NET MAUI 8.0 application for managing participant queues in karaoke rounds. It allows managing 1 queue at a time, 
enabling the user to register participation/absence of each singer as they reach position 1 in the queue! When all singers who 
entered the queue have participated or been absent, the round is incremented (round 1, round 2, etc). It allows the user to end a 
round even if there are singers in the queue who haven't participated in the round! It also allows enabling the last closed queue 
as a workaround when the user accidentally ends a queue! It allows reverting the round to its last state, in case the user realizes 
there was a registration error or accidentally ended a round! It also allows moving singers to any position in the queue. It enables 
singers to register in the queue autonomously, where the queue administrator receives notifications for each new singer registration! 
It displays the estimated queue completion time, based on the number of singers still pending to sing in the current round. There are 
also 2 queue modes (mechanical karaoke and bandokê - artist/band performs the instrumental). 

The code is being developed by Claude AI, with Helder serving as the architect and auditor of the work, guiding the AI on the 
technical approaches to be adopted and continuously monitoring performance optimization opportunities.

Key technical achievements include: multilingual support (11 languages), robust anti-crash pthread_mutex system, component-based 
architecture with dependency injection, performance-optimized animations with hardware detection, and SQLite database with Entity 
Framework integration.

### Bandokê Queue Mode
Artists/bands can optionally register their song catalog. Song lyrics can be stored in the MyVocaList database or obtained via 
third-party APIs like Genius.com. The app allows the administrator to register or change the song a participant sang at any 
time (as long as the queue is still active). If a singer is going to sing a song not registered in the artist/band catalog, 
the administrator can register the song the singer will perform just before the performance. If there's an internet connection 
and the song lyrics are not available in local data, it will fetch the lyrics via third-party APIs.

### MVP Key Features (English-Only)
- **Queue Management**: One active queue at a time with round-based progression
- **Participation Tracking**: Admin registers singers and marks participation/absence when reaching position 1
- **Round System**: Automatic round increment when all participants complete
- **Flexibility**: End rounds prematurely or revert to last state
- **Admin-Managed Registration**: Each singer registered by admin using standalone device
- **Queue Modes**: Mechanical karaoke and Bandokê (live instrumental)
- **Time Estimation**: Display estimated completion time based on pending singers
- **Multi-language Infrastructure**: 6 languages supported (MVP focuses on English only)

### Future Features (Post-MVP)
- **Singer Autonomy**: Self-registration capability with admin notifications
- **Facial Recognition**: Quick registration for returning singers
- **Song Medleys**: The app will allow bands to register song medleys in the catalog. In this case, lyrics will notbe displayed on screen unless the medley has been previously registered by the band/musician.
- **Song History**: Personalized song suggestions based on past performances
- **Social Network Integration**: Singer profiles, followers, interactions
- **Live Competitions**: Real-time voting, scoring, leaderboards
- **Cloud Synchronization**: Multi-device support with cloud backend

## 🤖 MVP AI Features (AI Lite)

> **Philosophy:** Optional AI enhancements behind feature flags. Disabled by default.
> **Full Documentation:** `Docs/Guides/AI/` folder

### Quick Reference

| Priority | Feature | Time | Status |
|----------|---------|------|--------|
| 1 | Smart Wait Time Estimation | 1-2 wks | Planned |
| 2 | Song Recommendations | 2-3 wks | Planned |
| 3 | Smart Lyrics Search | 2-3 wks | Planned |

### Feature Flags (All disabled by default)
```csharp
// Services/AI/Configuration/AIFeatureFlags.cs
public static class AIFeatureFlags
{
    public static bool EnableAIFeatures => Preferences.Get("ai_features_enabled", false);
    public static bool EnableSmartWaitTime => Preferences.Get("ai_smart_wait_time", false);
    public static bool EnableSongRecommendations => Preferences.Get("ai_song_recommendations", false);
    public static bool EnableLyricsSearch => Preferences.Get("ai_lyrics_search", false);
}
```

### Architecture Overview

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heldercsousa/Legacy_MyVocaListMVP](https://github.com/heldercsousa/Legacy_MyVocaListMVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
