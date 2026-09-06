---
trigger: always_on
description: Before anything, read this file and all relevant .md files - AI_REFLECTIONS.md, README.md, TRUTHS.md, AI_LOGS.md to understand what this is about. If some do not exist, skip. Also read the guidelines_writing_papers/README.md .
---

# Guidelines for the LaTex article writing

Before anything, read this file and all relevant .md files - AI_REFLECTIONS.md, README.md, TRUTHS.md, AI_LOGS.md to understand what this is about. If some do not exist, skip. Also read the guidelines_writing_papers/README.md .

These are the guidelines for how the LaTex manuscript should be written. The LaTex manuscript worked on is a scientific paper to be published in peer reviewed literature, and should follow the corresponding writing style. The human in the loop / person behind the prompts (me) is a scientific expert who is knowledgeable about the field. If you disagree with the human (me) about the science / content of the paper, you should start discussing with the human until you are sure we understand each other. Feel free to push back on me, but if I say I am sure / I insist on a scientific point after discussing together, trust me, I know what I talk about when I say so after a discussion.

The goal here is that you, the AI, write a manuscript that follows what I, the human, want. I know the manuscript I want to write and what it should contain, but writing is time consuming so you should expedite the scientific writing for me, following my instructions. I am an expert with PhD level background in physics / applied mathematics / computer science / machine learning / electronics / oceanography, take this into account in our discussions - be as smart as you can, you dont need to explain me simple things if I dont ask you.

The main task is to help going from bullet-point-like summary to proper text. Make sure that the bullet point materials are turned into consistent paragraphs, good quality text, that there are transitions between these, etc. Also double check that the content is correct and makes sense and come with comments and suggestions if relevant.

If looking at a manuscript for which there are both bullet points in the .md file and some latex main.tex files, be aware that the user may have edited the latex file directly by hand. So, if you are asked to update the manuscript either following bullet point updates, or on-the-fly, make sure to preserve both the content of the bullet points and of the latex file and to respect and extend both. If you find some important point are present in the latex .tex file but not in the .md bullet points file, you can update the bullet points file as relevant. Generally, try to keep all .md files up to date when you perform changes.

## General organization of the repository

- The manuscript comes in several consecutive main versions. Each of these is in an own folder with `v`, such as `v1`, `v2`, etc. For example `v2` starts with a copy of `v1`, with further changes gradually applied. Only start a new version if the user asks for it, otherwise edit the highest version. Each version is self contained - for example, copy all the figures (`figs` folder) and the `compile.sh` script.
- The root of the repository contains the general instructions.
- By default when asked to do some work, do so in the latest `v` folder (the one with the highest version name).

## General organization within a LaTex version folder

- The figures should all be in a `figs` folder, flat folder.
- The whole `.tex` content is in a single `main.tex` file.
- The whole references set is in a `references.bib` file.

## Guidelines for the LaTex writing

- Have equations on their own line follow the LaTex pattern (this is just to illustrate with an example; see the `begin` kind used and the `\label`):

```latex
\begin{equation}
    \omega^2 = \frac{gk + Bk^5}{\coth(kH)}.
    \label{eqn:reduced_disprel}
\end{equation}
```

- Have equations or formula within a line be part of the text and inserted through `$` at the start and end (this is just to illustrate with an example):

```latex
The phase shift of the cross-spectral density between sensors 1 and 2, $\Delta \phi_{1, 2}(f) = \mathrm{phase}(P_{1,2}(f))$, is calculated at the frequencies for which coherence about the threshold is observed.
```

- Have figures follow the pattern (notice the `begin` kind used, `label`, `caption`, and `includegraphics` patterns, this is just to illustrate with an example):

```latex
\begin{figure}
    \centering
    \includegraphics[width=0.49\linewidth]{figs/t15_PSD_Tempelfjorden_2015_segment_2.png}
    \includegraphics[width=0.49\linewidth]{figs/t15_PSD_Tempelfjorden_2015_segment_16.png}
    \caption{PSDs obtained from the 3 VN100 IMUs that are part of the T15 dataset. Left: PSDs from the segment number 2. A clear high-frequency peak is observed in IMUs VN1002 and VN1007 (farther in the ice), but not in VN1005 (closest to the open water). The high-frequency peak is relatively broad. Right: PSDs from the segment number 16. No high-frequency peak is observed.}
    \label{fig:t15_psd}
\end{figure}
```

- Use `citet` and `citep` as relevant (this is just to illustrate with an example):

```latex

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jerabaul29/2026_template_paper_agentic_ai](https://github.com/jerabaul29/2026_template_paper_agentic_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
