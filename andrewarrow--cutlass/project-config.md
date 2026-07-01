---
trigger: always_on
description: **There are extensive validation checks in `fcpxml_lib/validation/`. Never bypass these. Better to let an error stop generation because validation failed than to ever produce invalid FCPXML.**
---

# Project Context for AI Assistance - Python FCPXML Library

## 🚨 CRITICAL: use ../FCPXMLv1_13.dtd for DTD validation 🚨 ##

## 🚨 CRITICAL: ALL CODE MUST USE VALIDATION 🚨
**There are extensive validation checks in `fcpxml_lib/validation/`. Never bypass these. Better to let an error stop generation because validation failed than to ever produce invalid FCPXML.**

## 🚨 CRITICAL: CHANGE CODE NOT XML 🚨
**NEVER EVER only change problem XML in an XML file, always change the code that generates it too**

## 🚨 CRITICAL: NO XML STRING TEMPLATES 🚨
**NEVER EVER generate XML from hardcoded string templates with f-strings or % formatting, use dataclasses**

❌ BAD: `xml = f"<video ref=\"{video_ref}\">{content}</video>"`
❌ BAD: `"<asset-clip ref=\"%s\" name=\"%s\"/>" % (ref, name)`
✅ GOOD: `Video(ref=video_ref, duration=duration)` → `serialize_to_xml()`

**All FCPXML generation MUST use the dataclasses in `fcpxml_lib/models/elements.py`.**

## 🚨 CRITICAL: Conform-Rate Elements Must Include srcFrameRate 🚨

**The #1 cause of FCP import warnings: Missing srcFrameRate attribute on conform-rate elements**

### ✅ CORRECT conform-rate Structure:
```xml
<clip offset="0s" name="VideoClip" duration="240240/24000s" format="r2" tcFormat="NDF">
    <conform-rate scaleEnabled="0" srcFrameRate="24"/>
    <adjust-transform><!-- keyframe animations --></adjust-transform>
    <video ref="r3" offset="0s" duration="240240/24000s"/>
</clip>
```

### ❌ INCORRECT conform-rate (causes FCP warnings):
```xml
<clip offset="0s" name="VideoClip" duration="240240/24000s" format="r2" tcFormat="NDF">
    <conform-rate scaleEnabled="0"/>  <!-- Missing srcFrameRate! -->
    <adjust-transform><!-- keyframe animations --></adjust-transform>
    <video ref="r3" offset="0s" duration="240240/24000s"/>
</clip>
```

### 🚨 FCP Import Error Without srcFrameRate:
```
Encountered an unexpected value. (conform-rate: /fcpxml[1]/library[1]/event[1]/project[1]/sequence[1]/spine[1]/clip[1])
```

**CRITICAL: Always include srcFrameRate attribute matching media frame rate (24, 25, 29.97, 30, etc.)**

## 🚨 CRITICAL: Multi-Lane Video Audio Implementation 🚨

**The #1 cause of silent videos: Missing audio elements and asset properties**

### ✅ CORRECT Audio Implementation for Complex Clips:

For multi-lane video effects with audio, you need **BOTH** asset-level audio properties AND timeline audio elements:

```xml
<!-- 1. Assets MUST have audio properties -->
<asset id="r2" name="video" hasVideo="1" hasAudio="1" audioSources="1" audioChannels="2" audioRate="48000">
    <media-rep kind="original-media" src="file:///path/to/video.mov"/>
</asset>

<!-- 2. Timeline MUST have both video AND audio elements -->
<clip offset="0s" name="Video Clip" duration="240240/24000s" format="r3" tcFormat="NDF">
    <conform-rate scaleEnabled="0" srcFrameRate="24"/>
    <adjust-transform><!-- keyframe animations --></adjust-transform>
    <video ref="r2" offset="0s" duration="240240/24000s"/>
    <audio ref="r2" offset="0s" duration="240240/24000s" role="dialogue"/>
</clip>
```

### ❌ INCORRECT Audio Implementation (silent in FCP):

```xml
<!-- Missing audio properties on asset -->
<asset id="r2" name="video" hasVideo="1" videoSources="1">  <!-- NO hasAudio! -->
    <media-rep kind="original-media" src="file:///path/to/video.mov"/>
</asset>

<!-- Missing audio element on timeline -->
<clip offset="0s" name="Video Clip" duration="240240/24000s" format="r3" tcFormat="NDF">
    <conform-rate scaleEnabled="0" srcFrameRate="24"/>
    <video ref="r2" offset="0s" duration="240240/24000s"/>  <!-- Only video, no audio! -->
</clip>
```

### 🚨 CRITICAL Audio Implementation Rules:

1. **Asset Audio Properties Required**: Assets MUST have `hasAudio="1"`, `audioSources="1"`, `audioChannels="2"`, `audioRate="48000"`
2. **Timeline Audio Elements Required**: Complex clips need separate `<audio ref="...">` elements alongside `<video ref="...">` elements
3. **DTD Compliance**: Use `role="dialogue"` on `<audio>` elements, NOT `audioRole` on `<clip>` elements
4. **Both Required**: You need BOTH asset properties AND timeline elements for audio to work

### ✅ Implementation Pattern:

```python
# 1. Create assets with audio properties when include_audio=True
asset, format_obj = create_media_asset(
    video_path, asset_id, format_id, include_audio=True
)

# 2. Add both video and audio elements to timeline
clip_elements = [
    {"type": "video", "ref": asset_id, "duration": duration},
    {"type": "audio", "ref": asset_id, "duration": duration, "role": "dialogue"}
]
```

### 🚨 Common Audio Failures:

- **Asset without audio properties**: `hasAudio` missing → silent video
- **Timeline without audio elements**: Only `<video>` elements → silent video  
- **Wrong DTD attributes**: `audioRole` on clips → validation failure
- **Missing role attribute**: Audio elements without `role="dialogue"` → routing issues

## 🚨 CRITICAL: Images vs Videos Architecture 🚨

**The #2 cause of crashes: Using wrong element types for images vs videos**

### ✅ IMAGES (PNG/JPG files):
```xml
<!-- Asset: duration="0s" (timeless) -->
<asset id="r2" duration="0s" hasVideo="1" format="r3" videoSources="1"/>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewarrow/cutlass](https://github.com/andrewarrow/cutlass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
