---
trigger: always_on
description: This skill pack will be continuously updated as more three.js projects with awesome graphics emerge. I hope this skill pack can help anyone build awesome scenes and games with out-of-the-box sophisticated graphics, so you can focus on things like game logic and story.
---

# Three.js Awesome Graphics Agent Skills

## Intent

This is a Three.js agent skill pack for producing awesome graphics.

It includes mesh design, lighting, PBR materials, textures, shaders, TSL/WebGPU, GLSL, post-processing, realism, stylization, particles, procedural visuals, color management, tone mapping, etc. Graphics excellence is the **main focus** of this skill pack, with sophisticated design aesthetics, philosophy, ergonomics, sensibility, taste. It brings the sophistication of good graphics and eliminates cheap effort.

This is NOT a three.js API cheat sheet, it skips basic 3D production fundamentals and concepts (any decent LLM already has that internal knowledge) as well as three.js API technicalities (just look up docs or use existing API oriented agent skills). Fundamentally, you cannot just provide a summary of what good graphics are like and expect the agent to produce it. The agent needs to see the exact implementation. That's what this skill pack aims to provide, the **vocabulary** of good and sophisticated graphics implementation. It's a skill pack with an attached example library to teach the agent not just what to do but also exactly how to do it.

This skill pack will be continuously updated as more three.js projects with awesome graphics emerge. I hope this skill pack can help anyone build awesome scenes and games with out-of-the-box sophisticated graphics, so you can focus on things like game logic and story.

## Developing Approach

This agent skill pack is and will be developed/maintained/updated/expanded by distilling working three.js projects that have awesome graphics. No adbstract concepts, no cheap summaries, no common knowledge. Only working projects with stunning graphics will be the source of this skill pack.

The development is a distillation process. The skills distilled from these projects need to be modular, atomic, and directly applicable:

- By modular and atomic, i mean it has to be self contained instead of entangled in a messy way. Agent can use what it needs, no more, no less.
- By directly applicable, i mean it has to be practical, include examples, direct implementations distilled from projects. It should be materials that the agent can directly use for an implementation, NOT "give you an idea, talk you through, you figure out the details".

**Important:** DO NOT try to invent examples and references inside a skill yourself. Everything must be closely referencing the supplied ref projects. These projects have been fine tuned to achieve high viusal quality. Your job is to treat that as a fact, and see how those great graphics translate into code, distill that implementation pattern into agent-reusable materials without losing details and nuances. All project examples need to be the **exact match** of the ref projects. Do NOT slack off and only make them approximations. There is no licensing concerns for any ref project I supply! every example MUST be exact same implementation extracted into skill example form. I do not want to have to stress this and enforce this every time! This is of the utmost importance!!!

**DO NOT IMPLEMENT THINGS YOURSELF, ALWAYS USE THE EXACT IMPLEMENTATION FROM THE REF PROJECTS!!! NO EXCEPTIONS UNLESS TOLD OTHERWISE.**

Reusability is a core concept for Agent Skills. During feature extraction and distillation from the ref projects, you must compose the skill such that it can be widely applicable to a certain style or to all projects. Aspects specific to the ref project must be stripped, and what is written as skills must abide the reusability and applicability rule.

Artistic styles can genuinely differentiate skills, e.g., ocean shader in a different style than existing example can absolutely justify being added as an additional skill or example.

As stated in Intent, this skill pack aims to provide practical guides with (if possible) exact implementation examples to teach the agent how to implement good graphics. so I expect this skill pack to continue to grow with more and more skill items and examples. Each skill gives agent a solid practical guide on an aspect of 3D graphics, and each example under any certain skill provides template for a specific implementation. And this skill pack will become a combination of Agent Skills + fine graphic library as one. I believe this is the best way to achieve true effectiveness and usefulness.

Since this skill pack targets awesome 3D graphics in three.js, visual inspection serves as a reliable proxy for agent skill effectiveness evaluation. `example-gallery/` is a shim to visually inspect examples included in the skills. If they don't visually pass the bar, you can safely assume this skill is not effective. Again, use the ref projects for visual reference too. If you have distilled the essence of a ref project graphic feature, you see at code level it is done correctly, you see visually the distilled example matches the ref project visual reasonably, then you can safely assume the skill distillation and extraction is done properly


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scottstts/Threejs-Awesome-Graphics-Agent-Skills](https://github.com/scottstts/Threejs-Awesome-Graphics-Agent-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
