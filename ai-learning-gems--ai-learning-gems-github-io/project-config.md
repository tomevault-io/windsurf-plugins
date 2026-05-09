---
trigger: always_on
description: Curated list of high-quality technical blogs for textbook research
---


# High-Quality Technical Blogs & Sources

This is a **curated registry** of blogs and websites that produce textbook-quality technical content. When running the `/research-textbook-chapter` or `/write-textbook-chapter` workflows, the agent SHOULD search these sources for relevant material on the topic being researched.

## How to Use This Registry

### Attribution (MANDATORY)

These blogs contain **original content** by independent researchers and writers. When using their material in textbook chapters, you MUST:

1. **Cite the author and post** in the section's collapsible source header table
2. **Attribute figures and diagrams** in image captions: `Source: [Author Name], "[Post Title]" ([year]). [URL]`
3. **Quote, don't paraphrase** — when using a specific explanation or framing from a blog post, use a blockquote with attribution rather than rewording it without credit
4. **Never present blog content as original** — if an explanation or worked example is inspired by or adapted from a blog post, say so explicitly (e.g., "The following derivation is adapted from Gundersen's excellent treatment in [post title]")

### During Research (MANDATORY)

When researching a topic, **spend at least 2-3 searches specifically targeting these blogs**:

```
site:lilianweng.github.io {TOPIC}
site:colah.github.io {TOPIC}
site:cameronrwolfe.substack.com {TOPIC}
site:magazine.sebastianraschka.com {TOPIC}
site:distill.pub {TOPIC}
site:gregorygundersen.com {TOPIC}
```

Pick the blogs most relevant to the topic's domain (see the category tags below).

### During Source Downloading

Use the extraction method listed for each site. The `authenticated_extract.py` script handles most sites. Key notes:
- **Substack** sites auto-detect the `article` CSS selector
- **GitHub Pages** blogs usually work with default settings (no `-s` needed)
- Some sites need custom CSS selectors — see the "Extraction" field for each entry

### Adding New Blogs to This Registry

When you discover a blog of exceptional quality during research, add it to this file following the template at the bottom. You MUST:
1. Read at least 3 posts from the blog to assess quality and depth
2. Test extraction with `authenticated_extract.py` to determine the best CSS selector
3. Categorize the blog using the depth tags defined below
4. List 2-3 notable posts with URLs
5. Add the entry in the correct category section

---

## Depth Tags

| Tag | Meaning | Example Authors |
|-----|---------|----------------|
| `deep-technical-with-math` | Rigorous derivations, proofs, equations | Lilian Weng, Gregory Gundersen, Francis Bach |
| `intuition-and-visualization` | Visual explanations, interactive demos | Jay Alammar, Chris Olah, Distill.pub |
| `paper-explainer` | Accessible breakdowns of recent papers | Cameron Wolfe, Sebastian Raschka, Sebastian Ruder |
| `production-systems` | MLOps, system design, deployment | Chip Huyen, Eugene Yan, Simon Willison |
| `safety-alignment` | AI safety, interpretability, alignment | Neel Nanda, LessWrong, Alex Irpan |
| `pure-math` | Research-level mathematics | Terence Tao, Timothy Gowers |
| `mixed` | Combination of multiple depth levels | Nathan Lambert, Jason Wei |

---

## The Registry

### Deep Technical with Math

#### Lilian Weng — lilianweng.github.io
- **Depth**: `deep-technical-with-math` (20-45 min reads, comprehensive bibliographies, derivations)
- **Focus**: LLMs, RL, diffusion models, hallucination, reward hacking, prompt engineering, meta-learning
- **Platform**: GitHub Pages / Hugo
- **Auth**: None
- **Frequency**: ~4-5 posts/year
- **Extraction**: `python scripts/authenticated_extract.py "URL"` (default settings work well; 35K+ chars per post)
- **Notable posts**:
  - "Attention? Attention!" — canonical attention mechanism survey
  - "LLM Powered Autonomous Agents"
  - "Reward Hacking in Reinforcement Learning"
- **Search**: `site:lilianweng.github.io {TOPIC}`

#### Gregory Gundersen — gregorygundersen.com/blog
- **Depth**: `deep-technical-with-math` (thorough derivations, proofs, careful exposition)
- **Focus**: Probability & statistics, linear algebra, information theory, ML theory, some quantitative finance
- **Platform**: GitHub Pages / Jekyll
- **Auth**: None
- **Frequency**: ~8-12 posts/year
- **Extraction**: `python scripts/authenticated_extract.py "URL"` (may need `-s "main"` or `-s ".post-body"` — test per post; some posts have non-standard DOM)
- **Notable posts**:
  - "Understanding Moments"
  - "The Gauss-Markov Theorem"
  - "Expectation Maximization"
- **Search**: `site:gregorygundersen.com {TOPIC}`

#### Francis Bach — francisbach.com
- **Depth**: `deep-technical-with-math` / `pure-math` (rigorous proofs, aimed at researchers)
- **Focus**: Optimization theory, kernel methods, convergence analysis, neural network theory
- **Platform**: WordPress
- **Auth**: None
- **Frequency**: ~Monthly (targets first Monday)
- **Extraction**: `python scripts/authenticated_extract.py "URL" -s "article, .entry-content"`
- **Notable posts**:
  - "Scaling Laws of Optimization"
  - "Gradient Descent for Wide Two-Layer Neural Networks"
- **Search**: `site:francisbach.com {TOPIC}`

#### Sander Dieleman — sander.ai
- **Depth**: `deep-technical-with-math` (60-min deep dives; multiple formal framings)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Learning-Gems/AI-Learning-Gems.github.io](https://github.com/AI-Learning-Gems/AI-Learning-Gems.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
