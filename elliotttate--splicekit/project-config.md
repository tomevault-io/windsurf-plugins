---
trigger: always_on
description: Generates word-by-word highlighted, animated caption titles as FCPXML. The pipeline:
---

# SpliceKit - Programmatic Final Cut Pro Control

SpliceKit is an ObjC dylib injected into FCP's process. It exposes all 78,000+ ObjC classes
via a JSON-RPC server on TCP 127.0.0.1:9876. Everything is fully programmatic -- no AppleScript,
no UI automation, no menu clicks.

## NEVER Use Keyboard Simulation or AppleScript

All FCP actions MUST go through direct ObjC calls. Never use any of these:
- `SpliceKit_simulateKeyPress()` or synthetic NSEvent key events
- `CGEvent` keyboard/mouse posting
- `osascript` / `NSAppleScript` / AppleScript `tell application`
- Accessibility APIs (`AXUIElement`) for clicking or typing
- Frame-stepping loops (`nextFrame` x N) when `seekToTime` exists

**Instead, always use one of these patterns (in priority order):**
1. **Direct method call** on the timeline/sequence/clip object via `objc_msgSend`
2. **Responder chain** via `[NSApp sendAction:NSSelectorFromString(@"selector:") to:nil from:nil]`
3. **Batch bridge endpoint** (e.g. `timeline.addMarkers`) for operations on multiple items
4. **Menu execute** via `menu.execute` for actions only accessible through menus

If an action doesn't have a direct ObjC selector yet, find one using `explore_class` /
`search_methods` on the relevant class (FFAnchoredTimelineModule, FFAnchoredSequence, etc.)
rather than simulating the keyboard shortcut.

## Quick Start

```
1. bridge_status()                    -- verify connection
2. get_timeline_clips()               -- see timeline contents
3. timeline_action("blade")           -- edit
4. verify_action("after blade")       -- confirm
```

## CRITICAL: Must Know Before Editing

### Opening a Project
If `get_timeline_clips()` returns an error about "no sequence", load a project:
```python
# Navigate: library -> sequences -> find one with content -> load it
libs = call_method_with_args("FFLibraryDocument", "copyActiveLibraries", "[]", true, true)
libs_handle = json.loads(libs)["handle"]
lib = call_method_with_args(libs_handle, "objectAtIndex:", '[{"type":"int","value":0}]', false, true)
lib_handle = json.loads(lib)["handle"]
seqs = call_method_with_args(lib_handle, "_deepLoadedSequences", "[]", false, true)
seqs_handle = json.loads(seqs)["handle"]
allSeqs = call_method_with_args(seqs_handle, "allObjects", "[]", false, true)
# For each sequence result, extract its "handle" before calling hasContainedItems
# Example: seq_handle = json.loads(seq)["handle"]
# Check each: call_method_with_args(seq_handle, "hasContainedItems", "[]", false)
# Load: get NSApp -> delegate -> activeEditorContainer -> loadEditorForSequence:
```

### Select Before Acting
Color correction, retiming, titles, and effects require a selected clip:
```
playback_action("goToStart")              # position
playback_action("nextFrame") x N          # navigate
timeline_action("selectClipAtPlayhead")   # select
timeline_action("addColorBoard")          # now apply
```

### Playhead Positioning
- 1 frame = ~0.042s at 24fps, ~0.033s at 30fps
- Use `nextFrame` with repeat count for precise positioning
- `batch_timeline_actions` is fastest for multi-step sequences
- Always go to a known position (goToStart) before stepping

### Undo After Mistakes
```
timeline_action("undo")   # undoes last edit, returns action name
timeline_action("redo")   # redoes it
```
Undo routes through FCP's FFUndoManager (not the responder chain).

### Timeline Data Model (Spine)
FCP stores items in: `sequence -> primaryObject (FFAnchoredCollection) -> containedItems`
- `FFAnchoredMediaComponent` = video/audio clips
- `FFAnchoredTransition` = transitions (Cross Dissolve, etc.)
- `get_timeline_clips()` handles this automatically

## All Timeline Actions

| Category | Actions |
|----------|---------|
| Blade | blade, bladeAll |
| Markers | addMarker, addTodoMarker, addChapterMarker, deleteMarker, nextMarker, previousMarker, deleteMarkersInSelection |
| Transitions | addTransition |
| Navigation | nextEdit, previousEdit, selectClipAtPlayhead, selectToPlayhead |
| Selection | selectAll, deselectAll |
| Edit | delete, cut, copy, paste, undo, redo, pasteAsConnected, replaceWithGap, copyTimecode |
| Edit Modes | connectToPrimaryStoryline, insertEdit, appendEdit, overwriteEdit |
| Effects | pasteEffects, pasteAttributes, removeAttributes, copyAttributes, removeEffects |
| Insert | insertGap, insertPlaceholder, addAdjustmentClip |
| Trim | trimToPlayhead, extendEditToPlayhead, trimStart, trimEnd, joinClips, nudgeLeft, nudgeRight, nudgeUp, nudgeDown |
| Color | addColorBoard, addColorWheels, addColorCurves, addColorAdjustment, addHueSaturation, addEnhanceLightAndColor, balanceColor, matchColor, addMagneticMask, smartConform |
| Volume | adjustVolumeUp, adjustVolumeDown |
| Audio | expandAudio, expandAudioComponents, addChannelEQ, enhanceAudio, matchAudio, detachAudio |
| Titles | addBasicTitle, addBasicLowerThird |
| Speed | retimeNormal, retimeFast2x/4x/8x/20x, retimeSlow50/25/10, retimeReverse, retimeHold, freezeFrame, retimeBladeSpeed, retimeSpeedRampToZero, retimeSpeedRampFromZero |
| Keyframes | addKeyframe, deleteKeyframes, nextKeyframe, previousKeyframe |
| Rating | favorite, reject, unrate |
| Range | setRangeStart, setRangeEnd, clearRange, setClipRange |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elliotttate/SpliceKit](https://github.com/elliotttate/SpliceKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
