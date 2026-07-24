---
trigger: always_on
description: Gemini can generate and edit images conversationally using specialized image models known as **Nano Banana** (Gemini 2.5 Flash Image) and **Nano Banana Pro** (Gemini 3 Pro Image Preview).
---


# Google AI Gemini Image Generation

Gemini can generate and edit images conversationally using specialized image models known as **Nano Banana** (Gemini 2.5 Flash Image) and **Nano Banana Pro** (Gemini 3 Pro Image Preview).

## Table of Contents

- [Overview](#overview)
- [Models Available](#models-available)
- [GoogleAiGeminiImageModel](#googleaigeminiimagemodel)
    - [Basic Usage](#basic-usage)
    - [Configuration](#configuration)
- [Image Generation](#image-generation)
    - [Text-to-Image](#text-to-image)
    - [Aspect Ratios](#aspect-ratios)
    - [Image Sizes](#image-sizes)
- [Image Editing](#image-editing)
    - [Adding and Removing Elements](#adding-and-removing-elements)
    - [Style Transfer](#style-transfer)
    - [Inpainting](#inpainting)
- [Batch Image Generation](#batch-image-generation)
- [Limitations](#limitations)
- [Resources](#resources)

## Overview

Gemini's native image generation capabilities allow you to:

- **Text-to-Image**: Generate high-quality images from text descriptions
- **Image Editing**: Add, remove, or modify elements in existing images
- **Style Transfer**: Apply artistic styles to images
- **Iterative Refinement**: Conversationally refine images over multiple turns
- **High-Fidelity Text Rendering**: Generate images with legible, well-placed text

All generated images include a [SynthID watermark](https://ai.google.dev/responsible/docs/safeguards/synthid).

## Models Available

| Model | Description | Max Resolution | Max Input Images |
|-------|-------------|----------------|------------------|
| `gemini-2.5-flash-image` | Fast, efficient image generation (Nano Banana) | 1024px | 3 |
| `gemini-3-pro-image-preview` | Advanced features, thinking mode, Google Search grounding (Nano Banana Pro) | 4K | 14 |

## GoogleAiGeminiImageModel

### Basic Usage

```java
ImageModel imageModel = GoogleAiGeminiImageModel.builder()
    .apiKey(System.getenv("GOOGLE_AI_GEMINI_API_KEY"))
    .modelName("gemini-2.5-flash-image")
    .build();

Response<Image> response = imageModel.generate(
    "A nano banana dish in a fancy restaurant with a Gemini theme"
);

// Save the generated image
Image image = response.content();
byte[] imageBytes = Base64.getDecoder().decode(image.base64Data());
Files.write(Paths.get("nano-banana.png"), imageBytes);
```

### Configuration

```java
ImageModel imageModel = GoogleAiGeminiImageModel.builder()
    .apiKey(System.getenv("GOOGLE_AI_GEMINI_API_KEY"))
    .modelName("gemini-3-pro-image-preview")
    .aspectRatio("16:9")              // Output aspect ratio
    .imageSize("2K")                   // Resolution (Gemini 3 Pro only)
    .timeout(Duration.ofSeconds(120))
    .maxRetries(3)
    .logRequestsAndResponses(true)
    .safetySettings(...)               // Content safety settings
    .build();
```

## Image Generation

### Text-to-Image

Generate images from descriptive text prompts:

```java
ImageModel imageModel = GoogleAiGeminiImageModel.builder()
    .apiKey(System.getenv("GOOGLE_AI_GEMINI_API_KEY"))
    .modelName("gemini-2.5-flash-image")
    .build();

// Photorealistic style
Response<Image> photo = imageModel.generate("""
    A photorealistic close-up portrait of an elderly Japanese ceramicist
    with deep wrinkles and a warm smile, inspecting a tea bowl.
    Soft golden hour light, 85mm portrait lens, shallow depth of field.
    """);

// Stylized illustration
Response<Image> sticker = imageModel.generate("""
    A kawaii-style sticker of a happy red panda wearing a bamboo hat,
    munching on a leaf. Bold outlines, cel-shading, vibrant colors,
    white background.
    """);

// Logo design
Response<Image> logo = imageModel.generate("""
    A modern, minimalist logo for 'The Daily Grind' coffee shop.
    Clean, bold sans-serif font. Black and white. Circular design
    with a clever coffee bean element.
    """);
```

### Aspect Ratios

Supported aspect ratios for both models:

| Aspect Ratio | Use Case |
|--------------|----------|
| `1:1` | Square, social media posts |
| `2:3`, `3:2` | Portrait/landscape photos |
| `3:4`, `4:3` | Standard photos |
| `4:5`, `5:4` | Instagram posts |
| `9:16`, `16:9` | Stories, YouTube thumbnails |
| `21:9` | Cinematic, ultrawide |

```java
ImageModel imageModel = GoogleAiGeminiImageModel.builder()
    .apiKey(System.getenv("GOOGLE_AI_GEMINI_API_KEY"))
    .modelName("gemini-2.5-flash-image")
    .aspectRatio("16:9")  // Widescreen format
    .build();
```

### Image Sizes

**Gemini 3 Pro Image Preview** supports higher resolutions:

| Size | Description |
|------|-------------|
| `1K` | Default resolution |
| `2K` | Higher resolution |
| `4K` | Maximum resolution |

```java
ImageModel imageModel = GoogleAiGeminiImageModel.builder()
    .apiKey(System.getenv("GOOGLE_AI_GEMINI_API_KEY"))
    .modelName("gemini-3-pro-image-preview")
    .aspectRatio("1:1")
    .imageSize("4K")  // High resolution output
    .build();
```

## Image Editing

### Adding and Removing Elements

Edit existing images by providing them alongside text prompts:

```java
ImageModel imageModel = GoogleAiGeminiImageModel.builder()
    .apiKey(System.getenv("GOOGLE_AI_GEMINI_API_KEY"))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langchain4j/langchain4j](https://github.com/langchain4j/langchain4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
