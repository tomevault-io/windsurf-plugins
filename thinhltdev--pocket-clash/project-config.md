---
trigger: always_on
description: > Đây là file context master. Claude Code đọc file này đầu tiên mỗi session để hiểu toàn bộ dự án.
---

# Pocket Clash — Project Context for Claude Code

> Đây là file context master. Claude Code đọc file này đầu tiên mỗi session để hiểu toàn bộ dự án.
> Cập nhật file này khi có quyết định lớn hoặc thay đổi architecture.

---

## 1. Project Overview

**Tên game:** Pocket Clash (tạm thời — có thể đổi)
**Thể loại:** Mobile Auto-Battler Card Game (như Hearthstone Battlegrounds, Storybook Brawl)
**Platform:** iOS + Android, **OFFLINE-FIRST**
**Engine:** Unity 6 LTS (2D), C#
**Target launch:** Q4 2026 (6 tháng từ tháng 5/2026)
**Team:** Solo developer + freelance support khi cần

---

## 2. Core Gameplay (Quick Summary)

- 1 vs 7 đối thủ async (snapshot từ player pool cùng rank, không phải realtime PvP)
- Mỗi player có hero HP 30
- 7-8 round, mỗi round: Shop Phase (30s mua/đặt thẻ) → Combat Phase (auto-battle 15-30s)
- Board 3x2 (6 slot: 3 frontline, 3 backline)
- Shop có 5 slot, refresh free đầu round, reroll tốn 1 gold
- Match ~8-10 phút
- Mục tiêu: là người cuối cùng còn sống (HP > 0)

**3-Star System:** 3 thẻ giống nhau → ghép thành thẻ 2-sao (1.5x power). 3 thẻ 2-sao → 3-sao (2.5x power + ability bonus).

**Synergy:** 6 traits chính (Warrior, Mage, Beast, Undead, Holy, Rogue). Sở hữu 2/4/6 unit cùng trait → bonus toàn team.

---

## 3. Content Scope (Launch Day)

- **50 unit cards** (T1:13, T2:10, T3:9, T4:7, T5:6, T6:5)
- **8 playable heroes** với hero power riêng
- **6 traits** với synergy bonus 3 bậc
- **12 ability keywords** (Battlecry, Deathrattle, Taunt, Divine Shield, Poison, Heal, Buff, Debuff, Summon, Reborn, Windfury, Reflect)

**Single source of truth:** File `Pocket_Clash_CardDB_v1.xlsx` (sẽ export sang JSON để Unity load).

---

## 4. Technical Architecture

### Engine & Tools
- **Unity 6 LTS** (chọn version cụ thể: 6000.0.x)
- **Visual Studio 2022** hoặc **Rider** cho C# (recommended Rider nếu có budget)
- **Git + Git LFS** (cho asset binary)
- **GitHub Private Repo**

### Project Folder Structure
```
/Assets
  /Scripts
    /Core              # Game logic core
      CombatSystem.cs
      ShopManager.cs
      EconomyManager.cs
      GameStateManager.cs
    /Data              # ScriptableObject definitions
      CardData.cs
      HeroData.cs
      TraitData.cs
      AbilityData.cs
    /UI                # UI controllers
      /Screens
      /Popups
      /Components
    /AI                # Bot AI for async opponents
      BotAI.cs
      BotDifficulty.cs
    /Meta              # Meta-game systems
      BattlePassManager.cs
      RankedManager.cs
      ProgressionManager.cs
    /Services          # Third-party integrations
      AnalyticsService.cs
      IAPService.cs
      AdsService.cs
      RemoteConfigService.cs
  /Resources
    /Cards             # JSON card database
    /Heroes
    /Audio
  /Art
    /Sprites
    /Animations
    /UI
  /Prefabs
  /Scenes
    MainMenu.unity
    Match.unity
    Tutorial.unity
```

### Third-Party SDKs
- **Firebase**: Analytics, Remote Config, Crashlytics (free tier)
- **AppLovin MAX**: Ad mediation (rewarded, interstitial, banner)
- **Unity IAP**: In-app purchase
- **GameAnalytics**: Backup analytics
- **DOTween**: Animation library (free)
- **Newtonsoft.Json**: JSON parsing

### Async PvP Implementation
Không có server realtime. Cơ chế:
1. Mỗi match, save "snapshot" board cuối mỗi round vào Firebase Firestore
2. Khi cần đối thủ, fetch snapshot từ pool cùng rank
3. Simulate combat **local** trên máy player
4. Early launch: dùng AI bot snapshot (80% bot, 20% real) → grow dần

---

## 5. Design Pillars (Đừng quên)

1. **Strategic Depth, Casual Friction** — quyết định ý nghĩa, UI tối giản
2. **Short Session, Long Progression** — 8-10 phút/match, meta progression hàng tháng
3. **Watch & React, Not Click & Click** — auto-combat, 80% suy nghĩ, 20% xem
4. **Fair F2P, Ethical Monetization** — KHÔNG bán power, chỉ cosmetic + convenience + content

---

## 6. Monetization Stack (Ads-Heavy Hybrid)

### Rewarded Video Ads (60-70% revenue)
- x2 reward sau match
- Free chest mỗi 4h (6 lần/ngày)
- Reroll shop free (mỗi round)
- +1 battle pass tier (1 lần/ngày)
- x2 daily login
- Hero shard pack (1 lần/ngày)

### Interstitial Ads
- Sau mỗi 3 matches
- Skip cho user mua Remove Ads ($3.99)
- KHÔNG hiển thị 24h đầu tiên (boost D1 retention)

### IAP Catalog
- Starter Pack $0.99 (one-time)
- Remove Ads $3.99
- Battle Pass $4.99 / Bundle $9.99
- Gem Pack S/M/L: $1.99 / $4.99 / $9.99
- Mega Pack $19.99
- Skin Bundle $4.99

---

## 7. Coding Conventions

### C# Style
- **PascalCase** cho class, method, public properties
- **camelCase** cho local variables, private fields (prefix `_` cho private fields)
- **UPPER_SNAKE_CASE** cho const
- **XML doc comments** cho public APIs

### Architecture Principles
- **ScriptableObject-driven data** (cards, heroes, abilities — không hardcode trong C# class)
- **Event-driven combat** (UnityEvent hoặc custom event bus, tránh tight coupling)
- **Deterministic combat** (same input → same output, dùng seed cho RNG)
- **State machine** cho match flow (Lobby → Shop → Combat → Result → NextRound)
- **MVC** cho UI (View không tự ý gọi logic, chỉ raise event)

### File Naming
- 1 file = 1 class chính
- Tên file = tên class (CardData.cs chứa class CardData)
- Folder organize theo feature, không theo type


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thinhltdev/pocket-clash](https://github.com/thinhltdev/pocket-clash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
