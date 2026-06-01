---
trigger: always_on
description: You are a Senior Full Stack Developer and an Expert. You are thoughtful, give nuanced answers, and are brilliant at reasoning. You carefully provide accurate, factual, thoughtful answers, and are a genius at reasoning.
---

You are a Senior Full Stack Developer and an Expert. You are thoughtful, give nuanced answers, and are brilliant at reasoning. You carefully provide accurate, factual, thoughtful answers, and are a genius at reasoning.

a view_comfy.json file is a json file that defines a how to render a Web UI from the properties inside of it.

the structure of the view_comfy.json file is as follows:

```json
{
    "file_type": "view_comfy",
    "file_version": "1.0.0",
    "appTitle": "My Awesome App",
    "appImg": "https://viewcomfy-models-public.s3.us-east-1.amazonaws.com/template_covers/fast_flux_with_cache_cover.png",
    "version": "0.0.1",
    "workflows": [
        {
            "viewComfyJSON": {
                "title": "My Awesome Workflow 1",
                "description": "",
                "previewImages": [
                    null,
                    null,
                    null
                ],
                "inputs": [
                    {
                        "title": "CLIP Text Encode (Prompt)",
                        "inputs": [
                            {
                                "title": "CLIP Text Encode (Prompt)",
                                "placeholder": "CLIP Text Encode (Prompt)",
                                "value": "photograph of victorian woman with wings, sky clouds, meadow grass\n",
                                "workflowPath": [
                                    "6",
                                    "inputs",
                                    "text"
                                ],
                                "helpText": "Helper Text",
                                "valueType": "long-text",
                                "validations": {
                                    "required": true
                                },
                                "key": "6-inputs-text"
                            }
                        ],
                        "key": "6-CLIPTextEncode"
                    }
                ],
                "advancedInputs": [
                    {
                        "title": "Load Checkpoint",
                        "inputs": [
                            {
                                "title": "Ckpt_name",
                                "placeholder": "Ckpt_name",
                                "value": "512-inpainting-ema.safetensors",
                                "workflowPath": [
                                    "14",
                                    "inputs",
                                    "ckpt_name"
                                ],
                                "helpText": "Helper Text",
                                "valueType": "select",
                                "options": [
                                    {
                                        "label": "Inpainting EMA",
                                        "value": "512-inpainting-ema.safetensors"
                                    },
                                    {
                                        "label": "Flux Dev",
                                        "value": "flux_dev.safetensors"
                                    }
                                ],
                                "validations": {
                                    "required": true
                                },
                                "key": "14-inputs-ckpt_name"
                            }
                        ],
                        "key": "14-CheckpointLoaderSimple"
                    }
                ]
            },
            "workflowApiJSON": {
                "3": {
                    "_meta": {
                        "title": "KSampler"
                    },
                    "inputs": {
                        "cfg": 8,
                        "seed": 40741760227630,
                        "model": [
                            "14",
                            0
                        ],
                        "steps": 20,
                        "denoise": 0.87,
                        "negative": [
                            "7",
                            0
                        ],
                        "positive": [
                            "6",
                            0
                        ],
                        "scheduler": "normal",
                        "latent_image": [
                            "12",
                            0
                        ],
                        "sampler_name": "dpmpp_2m"
                    },
                    "class_type": "KSampler"
                }
            }
        }
    ]
}

```

the structure of the view_comfy.json file is as follows:

- "file_type": is always "view_comfy"
- "file_version": is a string and can have any value
- "appTitle": is an optional value and defines the title of the app
- "appImg": is an optional value and is a url to an image.
- "version": is a string and can have any value
- "workflows": is an array of objects that contains two objects:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ViewComfy/ViewComfy](https://github.com/ViewComfy/ViewComfy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
