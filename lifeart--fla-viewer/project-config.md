---
trigger: always_on
description: FLA files (Adobe Animate/Flash Professional) are ZIP archives containing XML files in the XFL (XML-based FLA) format.
---

# FLA Viewer - Agent Documentation

## FLA/XFL Format Specification

FLA files (Adobe Animate/Flash Professional) are ZIP archives containing XML files in the XFL (XML-based FLA) format.

### Archive Structure

```
.fla (ZIP archive)
├── DOMDocument.xml          # Main document definition
├── PublishSettings.xml      # Export/publish configuration
├── MobileSettings.xml       # Mobile platform settings
├── META-INF/
│   └── metadata.xml         # XMP metadata (creation date, tools, etc.)
├── LIBRARY/                  # Symbol definitions
│   ├── Symbol1.xml
│   ├── Symbol2.xml
│   └── ...
└── bin/
    └── SymDepend.cache      # Symbol dependency cache
```

### DOMDocument.xml Structure

```xml
<DOMDocument xmlns="http://ns.adobe.com/xfl/2008/"
    width="1920"
    height="1080"
    frameRate="25"
    backgroundColor="#BEEEFD"
    currentTimeline="1">

    <folders>...</folders>           <!-- Library folders -->
    <media>...</media>               <!-- External media (video, audio) -->
    <symbols>                        <!-- Symbol library references -->
        <Include href="Symbol.xml" itemID="xxx" lastModified="timestamp"/>
    </symbols>
    <timelines>                      <!-- Main stage timeline -->
        <DOMTimeline name="Scene 1">
            <layers>...</layers>
        </DOMTimeline>
    </timelines>
</DOMDocument>
```

### Layer Structure

```xml
<DOMLayer name="LayerName"
    color="#FF4F4F"           <!-- Layer color in timeline UI -->
    visible="true"            <!-- Layer visibility -->
    locked="false"            <!-- Layer lock state -->
    layerType="normal"        <!-- normal | guide | folder | mask | masked -->
    parentLayerIndex="2">     <!-- Parent folder index (if in folder) -->

    <frames>
        <DOMFrame index="0" duration="10" keyMode="9728">
            <elements>...</elements>
        </DOMFrame>
    </frames>
</DOMLayer>
```

### Frame Types

#### Static Frame
```xml
<DOMFrame index="0" duration="5" keyMode="9728">
    <elements>
        <DOMSymbolInstance libraryItemName="Symbol" symbolType="graphic">
            <matrix><Matrix tx="100" ty="200"/></matrix>
        </DOMSymbolInstance>
    </elements>
</DOMFrame>
```

#### Motion Tween Frame
```xml
<DOMFrame index="0" duration="10"
    tweenType="motion"
    keyMode="22017"
    acceleration="-50">        <!-- -100 to 100: negative=ease-in, positive=ease-out -->

    <tweens>
        <Ease target="all" intensity="-50"/>
        <!-- OR custom bezier easing -->
        <CustomEase target="all">
            <Point x="0" y="0"/>
            <Point x="0.333" y="0.396"/>
            <Point x="0.667" y="0.729"/>
            <Point x="1" y="1"/>
        </CustomEase>
    </tweens>
    <elements>...</elements>
</DOMFrame>
```

### Matrix Transform

2D affine transformation matrix: `[a c tx; b d ty; 0 0 1]`

```xml
<Matrix
    a="1.0"    <!-- scale X (default: 1) -->
    b="0.0"    <!-- skew Y (default: 0) -->
    c="0.0"    <!-- skew X (default: 0) -->
    d="1.0"    <!-- scale Y (default: 1) -->
    tx="100"   <!-- translate X (default: 0) -->
    ty="200"   <!-- translate Y (default: 0) -->
/>
```

### Symbol Instance

```xml
<DOMSymbolInstance
    libraryItemName="SymbolName"
    symbolType="graphic"           <!-- graphic | movieclip | button -->
    loop="loop"                    <!-- loop | play once | single frame -->
    firstFrame="0"                 <!-- Starting frame for nested timeline -->
    centerPoint3DX="100"           <!-- 3D center point for transforms -->
    centerPoint3DY="200">

    <matrix><Matrix .../></matrix>
    <transformationPoint><Point x="0" y="0"/></transformationPoint>
    <color>                        <!-- Optional color transform -->
        <Color alphaMultiplier="0.5"/>
    </color>
</DOMSymbolInstance>
```

### Camera Layer (Ramka Pattern)

Camera movement in FLA files is often simulated using a "ramka" (frame) layer:

```xml
<DOMLayer name="ramka" color="#9933CC" locked="true">
    <frames>
        <DOMFrame index="0" duration="10" tweenType="motion" keyMode="22017">
            <elements>
                <DOMSymbolInstance libraryItemName="Ramka" symbolType="graphic">
                    <matrix>
                        <!-- Camera position/zoom: scale for zoom, tx/ty for pan -->
                        <Matrix a="1.0" d="1.0" tx="100" ty="50"/>
                    </matrix>
                </DOMSymbolInstance>
            </elements>
        </DOMFrame>
    </frames>
</DOMLayer>
```

The camera layer contains a symbol that represents the viewport. Detection criteria (ALL must be met):
1. **Layer name** indicates camera: `ramka`, `camera`, `cam`, `viewport`, or contains "camera"/"viewport"
2. **Layer is non-rendering**: `layerType="guide"` OR `visible="false"` OR `outline="true"`
3. **Single symbol**: Layer contains exactly one symbol instance in its first frame
4. **Centered transformation point**: Symbol's transformation point is near document center
   - Uses **per-axis tolerances**: 15% of width for X, 15% of height for Y

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lifeart/fla-viewer](https://github.com/lifeart/fla-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
