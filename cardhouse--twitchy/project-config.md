---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== .ai/structure rules ===

# Directory Structure & Conventions

## Application Structure

```
app/
├── Actions/                    # Single-responsibility action classes
│   └── ClearChannel.php
├── Console/Commands/           # Artisan commands (auto-registered)
│   ├── TwitchRelayCommand.php  # IRC connection to Twitch
│   └── TwitchyRunCommand.php   # Main orchestration command
├── Events/                     # Broadcastable events
│   ├── MessageReceived.php
│   ├── MessagePromoted.php
│   └── MessageDemoted.php
├── Http/Controllers/
│   ├── ChatHookController.php  # Webhook endpoints for external integration
│   ├── OverlayController.php   # Overlay display and API
│   └── NewChatRequest.php      # Form request for validation
├── Livewire/                   # Livewire components
│   └── Overlay/
│       └── ToastDisplay.php    # Main overlay toast component
├── Messages/                   # Message handling domain
│   ├── Actions/                # Message-specific actions
│   │   ├── CreateMessage.php
│   │   ├── PromoteMessage.php
│   │   ├── DemoteMessage.php
│   │   ├── ClearMessages.php
│   │   └── ClearPromotedMessages.php
│   ├── Contracts/              # Interfaces
│   │   └── Store.php
│   ├── Processors/             # Message processing logic
│   │   └── TwitchMessageProcessor.php
│   └── Stores/                 # Data persistence strategies
│       ├── CacheStore.php
│       ├── ChatMessageStore.php
│       └── PromotedMessageStore.php
├── Models/                     # Eloquent models
│   ├── Chatroom.php
│   ├── Message.php             # Base message model
│   └── Messages/               # Message type hierarchy
│       ├── ChatMessage.php
│       ├── PingMessage.php
│       ├── PrivateMessage.php
│       └── UnknownMessage.php
└── Services/
    └── OverlayService.php      # Core overlay management service

resources/views/
├── control/                    # Control panel views
│   └── index.blade.php
├── layouts/                    # Layout files
│   ├── app.blade.php
│   └── overlay.blade.php       # Transparent layout for OBS
├── livewire/                   # Livewire component views
│   ├── control/
│   │   ├── chat-feed.blade.php
│   │   └── toast-preview.blade.php
│   └── overlay/
│       └── toast-display.blade.php
└── overlay/                    # Overlay-specific views
    └── show.blade.php
```


=== .ai/architecture rules ===

# Application Architecture

## Two-Window System

**1. Overlay Page (OBS Browser Source)**
- URL: `/overlay/{key}` (e.g., `/overlay/local`)
- Transparent background for OBS integration
- Displays single toast of selected chat message
- Supports URL parameters for customization: `?theme=dark&fontScale=1.2&safeMargin=50&animation=slide-up`
- Auto-dismiss (timed) or persistent (manual clear) modes

**2. Control Panel (Management Interface)**
- URL: `/control`
- Live chat feed streaming from configured Twitch channel
- Search/filter capabilities
- "Activate" button to promote message to overlay
- "Clear" button to remove current toast
- Real-time preview of current overlay state

## Data Flow Architecture

```
[Twitch IRC] → [Artisan Command: twitch:relay] → [Database + Cache] → [Reverb Broadcasting]
                                                                              ↓
                                                        [Control Panel] ← → [Overlay Page]
```

## Real-time Channels

- `chat.messages` (public): Control Panel subscribes to receive new chat messages
- `overlay.{key}` (public): Overlay subscribes to receive toast show/hide events
- Message flow: IRC → Store → Broadcast → UI components

## Key Conventions

**Message Architecture**
- Use the Action pattern for message operations: `CreateMessage`, `PromoteMessage`, `DemoteMessage`
- Store pattern for data persistence: `ChatMessageStore`, `PromotedMessageStore`
- Processor pattern for parsing external message formats: `TwitchMessageProcessor`
- Message types inherit from base `Message` model

**Cache-Based State Management**
- Messages stored in cache with overlay-specific keys
- 30-second TTL for automatic cleanup
- Use `cache()->pull()` for single-consumption events
- Polling interval: 2 seconds for real-time feel

**Events & Broadcasting**
- All events implement `ShouldBroadcast`
- Event naming: `MessageReceived`, `MessagePromoted`, `MessageDemoted`
- Channel naming: `chat.messages`, `overlay.{key}`
- Always include necessary payload data in event properties


=== .ai/obs rules ===

# OBS Integration

## Browser Source Setup

1. **Add Browser Source** in OBS
2. **URL**: `http://your-domain.test/overlay/local?theme=dark&fontScale=1.0`
3. **Width**: 1920, **Height**: 1080
4. **Custom CSS**: Leave blank (transparency handled in app)
5. **Refresh browser when scene becomes active**: Optional

## URL Parameters

- `theme`: `dark` or `light` (default: dark)
- `fontScale`: 0.5 to 3.0 (default: 1.0)
- `animation`: `slide-up`, `slide-down`, `slide-left`, `slide-right`, `fade`, `zoom` (default: slide-up)
- `safeMargin`: 0 to 100 pixels from edges (default: 24)

## Example URLs

```
# Dark theme with larger text
http://your-domain.test/overlay/local?theme=dark&fontScale=1.2

# Light theme with slide-down animation
http://your-domain.test/overlay/local?theme=light&animation=slide-down


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cardhouse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
