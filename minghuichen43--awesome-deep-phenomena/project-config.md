---
trigger: always_on
description: This repository is a curated Awesome list for papers and resources about deep learning phenomena. The main file for Codex-assisted updates is:
---

# AGENTS.md

## Project Context

This repository is a curated Awesome list for papers and resources about deep learning phenomena. The main file for Codex-assisted updates is:

- `README.md`: the primary curated paper list shown on GitHub.

Preserve the current Markdown style, table of contents, badges, image links, related-resource section, and contribution text unless the user explicitly asks to edit them.

## Default Task: Add an arXiv Paper

When the user gives an arXiv abstract link and asks to use this project's agent instructions, add the paper end-to-end unless it is already present.

### 1. Normalize the arXiv ID

- Accept `https://arxiv.org/abs/<id>`, `http://arxiv.org/abs/<id>`, and arXiv PDF links.
- Canonicalize the list URL to `https://arxiv.org/abs/<base-id>`.
- Strip version suffixes such as `v2` from the list URL unless the user explicitly asks to track a specific version.
- Prefer modern IDs like `2507.16795`; handle legacy IDs only when needed.

### 2. Fetch Metadata Reproducibly

Use the arXiv Atom API as the primary source, not ad hoc scraping of the HTML page. A reliable read-only helper is:

```bash
python3 - "$ARXIV_ID" <<'PY'
import json
import re
import sys
import time
import urllib.parse
import urllib.request
import xml.etree.ElementTree as ET
from urllib.error import HTTPError, URLError

raw = sys.argv[1]
match = re.search(r'([a-z-]+/)?\d{4}\.\d{4,5}(?:v\d+)?|[a-z-]+(?:\.[A-Z]{2})?/\d{7}(?:v\d+)?', raw)
if not match:
    raise SystemExit(f"Could not find an arXiv id in: {raw}")

paper_id = re.sub(r'v\d+$', '', match.group(0))
url = 'https://export.arxiv.org/api/query?' + urllib.parse.urlencode({'id_list': paper_id})
request = urllib.request.Request(
    url,
    headers={'User-Agent': 'awesome-deep-phenomena-agent/1.0'},
)
for attempt in range(2):
    try:
        with urllib.request.urlopen(request, timeout=30) as response:
            data = response.read()
        break
    except HTTPError as exc:
        if exc.code == 429 and attempt == 0:
            time.sleep(10)
            continue
        raise SystemExit(f"arXiv API request failed with HTTP {exc.code}: {exc.reason}") from exc
    except URLError as exc:
        raise SystemExit(f"arXiv API request failed: {exc.reason}") from exc

ns = {'atom': 'http://www.w3.org/2005/Atom', 'arxiv': 'http://arxiv.org/schemas/atom'}
root = ET.fromstring(data)
entry = root.find('atom:entry', ns)
if entry is None:
    raise SystemExit(f"No arXiv API entry found for: {paper_id}")

def text(path):
    node = entry.find(path, ns)
    return '' if node is None or node.text is None else ' '.join(node.text.split())

authors = [text_node.text.strip() for text_node in entry.findall('atom:author/atom:name', ns)]
primary = entry.find('arxiv:primary_category', ns)
categories = [node.attrib.get('term', '') for node in entry.findall('atom:category', ns)]

print(json.dumps({
    'id': paper_id,
    'url': f'https://arxiv.org/abs/{paper_id}',
    'title': text('atom:title'),
    'authors': authors,
    'published': text('atom:published'),
    'updated': text('atom:updated'),
    'primary_category': '' if primary is None else primary.attrib.get('term', ''),
    'categories': categories,
    'abstract': text('atom:summary'),
}, indent=2, ensure_ascii=False))
PY
```

If the API is unavailable, use the arXiv abstract page as a fallback and say so in the final response.

### 3. Check for Duplicates

Before editing, search `README.md` for the canonical arXiv ID:

```bash
rg -n "arxiv.org/abs/<ARXIV_ID>|<ARXIV_ID>" README.md
```

If the paper already exists, do not add a duplicate. If the user asked for an update, edit the existing entry in place.

### 4. Choose Exactly One Primary Category

Use the title, abstract, arXiv categories, and nearby existing entries. Prefer the most specific phenomenon category over `Others`.

Allowed paper categories:

- `Empirical Study`: broad empirical analyses of deep learning behavior, generalization, memorization, representation, scaling, fine-tuning, pruning, optimization, hallucination, or benchmarking that do not fit a more specific phenomenon category.
- `Neural Collapse`: neural collapse, class-means geometry, simplex ETF structure, terminal-phase training behavior, or related classification geometry.
- `Deep Double Descent`: double descent, benign overfitting, overparameterization peaks, interpolation thresholds, and related generalization curves.
- `Lottery Ticket Hypothesis`: lottery tickets, pruning at initialization, sparse subnetworks, rewinding, sparse training, and related pruning phenomena.
- `Emergence and Phase Transitions`: emergent abilities, in-context learning transitions, grokking, scaling-law transitions, phase changes, sharp capability jumps, and related dynamics.
- `Interactions with Neuroscience`: links between deep learning and neuroscience, brain alignment, neural data, cognitive models, or biologically inspired phenomena.
- `Information Bottleneck`: information bottleneck, compression, mutual information, representation compression, and information-theoretic explanations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MinghuiChen43/awesome-deep-phenomena](https://github.com/MinghuiChen43/awesome-deep-phenomena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
