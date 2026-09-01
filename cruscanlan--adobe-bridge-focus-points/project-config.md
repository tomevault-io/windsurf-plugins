---
trigger: always_on
description: A CEP plugin for Adobe Bridge that reads the camera's autofocus point(s) from
---

# Bridge Focus Points

A CEP plugin for Adobe Bridge that reads the camera's autofocus point(s) from
maker-note metadata and draws them as an overlay on the selected photo. The
Bridge equivalent of digiKam's native display and the Focus-Points Lightroom
plugin.

Full design rationale lives in [project.md](project.md). This file is the
working reference for how the code is actually put together.

## The one idea that makes this tractable

**We do not parse maker notes.** That problem is solved twice already and we
delegate to both:

- **exiftool** (bundled, shelled out) does all metadata extraction.
- **The Focus-Points Lr plugin** holds the per-manufacturer geometry that turns
  exiftool output into pixel coordinates. We port that, manufacturer by
  manufacturer (Lua -> JS). digiKam (`reference/digikam-focuspoint-metadataengine/`)
  is a secondary cross-reference only — it uses Exiv2, not exiftool, so its
  extraction layer does not transfer.

## Architecture (decided — do not re-evaluate)

- **CEP**, not UXP (Bridge has no UXP). Host id `KBRG`. Bridge 16 ships CEP 12.
- **Node.js enabled** in the panel (`--enable-nodejs --mixed-context` in the
  manifest) so we can `child_process` to exiftool. CEP 12 ships Node 17.7.1.
- **Bundled exiftool**, shelled out. Never reimplement.
- **HTML `<img>` + SVG overlay** for rendering (resolution-independent; drops
  the Lr plugin's ImageMagick dependency).
- **Windows** target (Cru runs Adobe in a Windows VFIO VM). Mac later.

### Data flow

```
Bridge selection
  -> host/index.jsx (ExtendScript)  : reports selected file path; pings panel on change
  -> client/main.js (Node/JS)       : spawns exiftool, picks delegate, renders
  -> vendor/exiftool                : returns maker-note / AF tags as JSON
  -> client/delegates/<mfr>.js      : computes AF box(es) in image-pixel coords
  -> SVG overlay                    : draws the box(es) on the preview
```

ExtendScript (ES3) is the *only* thing that can talk to Bridge but is feeble, so
`host/index.jsx` stays tiny: it answers "what file is selected?" and dispatches a
`CSXSEvent` (`com.cru.bridgefocuspoints.selectionChanged`) when the selection
changes. Everything else is in the Node/JS panel. The two talk via
`CSInterface.evalScript()` and CSXS events.

## Layout

```
CSXS/manifest.xml          extension config, host KBRG, Node enable
.debug                     remote-debug port 8088
host/index.jsx             ExtendScript: selection -> path (+ change event)
client/
  index.html               panel shell
  main.js                  Node/JS: exiftool -> delegate -> SVG overlay
  styles.css
  lib/
    CSInterface.js         Adobe CEP 12 library (vendored)
    exiftool.js            spawn wrapper: readTags() / extractPreview()
  delegates/
    fujifilm.js            ported Fuji geometry (getAfPoints)
    dji.js                 DJI: preview + flight info, no AF data (none exists)
vendor/exiftool/           bundled Windows exiftool 13.55 (complete standalone)
tools/                     dev-only: ZXPSignCmd.exe + cert.p12 + sign-install.sh
reference/                 read-only porting sources (Lr plugin, digiKam)
fixtures/                  test RAFs; expected/ holds rendered AF-box oracles
```

## How the Fuji geometry works (the ported core)

Ported from `reference/focuspoints.lrplugin/FujifilmDelegates.lua`
(`getAfPoints`) into `client/delegates/fujifilm.js`. For the X-series / GFX:

- `FocusPixel` is a single `(x, y)` in the coordinate system of the **embedded
  JPEG**, whose dimensions are reported by `ExifImageWidth`/`ExifImageHeight`.
- The transform is a pure scale to the displayed image:
  `xScale = displayW / ExifImageWidth`, same for y. The box is a square of side
  `min(displayW, displayH) * 0.04` (the Lr "medium" default) centred on the
  scaled point.
- **We render the embedded preview**, extracted with exiftool. On the X-H2 that
  preview is exactly `ExifImageWidth x ExifImageHeight` (4416x2944 on the test
  bodies), so `xScale = yScale = 1.0` and `FocusPixel` maps directly. The SVG
  overlay uses `viewBox="0 0 imgW imgH"` so coordinates are in image-pixel space
  regardless of on-screen zoom.
- Faces (`FacesDetected`/`FacesPositions`), subject detection
  (`FaceElementPositions`, exiftool >= 12.44) and tele-converter crop
  (`CropSize`/`CropTopLeft`, >= 12.82) use the same scale and are already ported.
  None of the current fixtures exercise them.

**OOC caveat:** the `FocusPixel`/`ExifImageWidth` reference is lost or corrupted
on RAF->DNG conversion. Feed straight-out-of-camera `.RAF` (or the OOC JPEG).
`fujifilm.makerNotesFound()` rejects DNGs and files missing `InternalSerialNumber`.

### Tag access convention

exiftool is invoked with `-json -s`, so tag keys are short PascalCase
(`FocusPixel`, `ExifImageWidth`, `FacesDetected`, `InternalSerialNumber`).
Delegates read tags by those names. `FocusPixel` comes back as the string
`"2515 1164"`; dimensions come back numeric.

## Orientation handling (done — and NOT in the delegate)

AF coordinates are relative to the camera's native (unrotated) orientation.
Rotated/portrait shots are handled, but **not** the way the Lr plugin does it.
Because we draw a live SVG overlay (not Mogrify-baked pixels), we don't rotate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CruScanlan/adobe-bridge-focus-points](https://github.com/CruScanlan/adobe-bridge-focus-points) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
