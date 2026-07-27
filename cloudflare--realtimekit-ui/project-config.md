---
trigger: always_on
description: One directory per component, named `rtk-<name>/`. Each contains `rtk-<name>.tsx` (class + JSX), `rtk-<name>.css` (scoped Shadow DOM styles), optional `*.spec.tsx` tests, and a `usage/` subdirectory with framework example markdown.
---

# packages/core/src/components — 136 Stencil Web Components

One directory per component, named `rtk-<name>/`. Each contains `rtk-<name>.tsx` (class + JSX), `rtk-<name>.css` (scoped Shadow DOM styles), optional `*.spec.tsx` tests, and a `usage/` subdirectory with framework example markdown.

## COMPONENT CATEGORIES

| Category                  | Components                                                                                                                                                                  | Sub-namespace                      |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| Meeting shell             | `rtk-meeting`, `rtk-ui-provider`, `rtk-dialog-manager`                                                                                                                      | —                                  |
| Video grid / layout       | `rtk-grid`, `rtk-simple-grid`, `rtk-mixed-grid`, `rtk-spotlight-grid`, `rtk-audio-grid`, `rtk-grid-pagination`                                                              | —                                  |
| Participant tile          | `rtk-participant-tile`, `rtk-participant`, `rtk-screenshare-view`, `rtk-audio-tile`, `rtk-audio-visualizer`                                                                 | —                                  |
| Chat                      | 17 components                                                                                                                                                               | `rtk-chat-*`                       |
| AI / transcription        | `rtk-ai`, `rtk-ai-toggle`, `rtk-ai-transcriptions`                                                                                                                          | `rtk-ai-*`                         |
| Participants panel        | 8 components                                                                                                                                                                | `rtk-participants-*`               |
| Control bar & toggles     | ~20 components                                                                                                                                                              | `rtk-*-toggle`                     |
| Breakout rooms            | `rtk-breakout-rooms-manager`, `rtk-breakout-room-manager`, `rtk-breakout-room-participants`, `rtk-breakout-room-toggle`                                                     | `rtk-breakout-*`                   |
| Settings / device pickers | `rtk-settings`, `rtk-settings-audio`, `rtk-settings-video`, `rtk-camera-selector`, `rtk-microphone-selector`, `rtk-speaker-selector`                                        | `rtk-settings-*`, `rtk-*-selector` |
| Polls                     | `rtk-polls`, `rtk-polls-toggle`, `rtk-poll`, `rtk-poll-form`                                                                                                                | `rtk-poll*`                        |
| Debugger                  | 6 components                                                                                                                                                                | `rtk-debugger-*`                   |
| Meeting state screens     | `rtk-setup-screen`, `rtk-idle-screen`, `rtk-ended-screen`, `rtk-waiting-screen`, `rtk-permissions-message`, `rtk-participant-setup`                                         | —                                  |
| Notifications / status    | `rtk-notifications`, `rtk-notification`, `rtk-network-indicator`, `rtk-recording-indicator`, `rtk-livestream-indicator`, `rtk-spotlight-indicator`                          | —                                  |
| UI primitives             | `rtk-button`, `rtk-icon`, `rtk-avatar`, `rtk-spinner`, `rtk-tooltip`, `rtk-dialog`, `rtk-menu`/`rtk-menu-item`/`rtk-menu-list`, `rtk-switch`, `rtk-tab-bar`, `rtk-name-tag` | —                                  |
| Livestream / media        | `rtk-livestream-player`, `rtk-livestream-toggle`, `rtk-viewer-count`, `rtk-image-viewer`                                                                                    | —                                  |

**Naming patterns:**

- `rtk-*-toggle` — every panel/feature has a dedicated toggle button.
- `rtk-*-view` suffix — presentational-only variant (no logic, just rendering): e.g., `rtk-text-message-view` vs. `rtk-text-message`.
- `rtk-*-ui` suffix — "dumb" UI layer that pairs with a smart container: e.g., `rtk-chat-selector-ui` + `rtk-chat-selector`.

## STANDARD COMPONENT ANATOMY

```ts
@Component({
  tag: 'rtk-foo',
  styleUrl: 'rtk-foo.css',
  shadow: true,             // always; some use { delegatesFocus: true }
})
export class RtkFoo {
  // 1. Private arrow-function handlers (never .bind())
  private onTrack = (track: MediaStreamTrack) => { ... };

  // 2. @Element() — host reference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/realtimekit-ui](https://github.com/cloudflare/realtimekit-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
