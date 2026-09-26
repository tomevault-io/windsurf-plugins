---
trigger: always_on
description: Agents are particularly useful when you need to validate hypotheses or run many experiments in parallel. However, they are less effective when forced to create and sift through thousands of lines of configuration files and training scripts. The Training Gym solves this with an intuitive API, a CLI for maximum observability into the run status, and skills that teach agents best practices such as smoking runs and tactics for debugging.
---


# Agent-driven training

Agents are particularly useful when you need to validate hypotheses or run many experiments in parallel. However, they are less effective when forced to create and sift through thousands of lines of configuration files and training scripts. The Training Gym solves this with an intuitive API, a CLI for maximum observability into the run status, and skills that teach agents best practices such as smoking runs and tactics for debugging.

This guide demonstrates how to effectively use agents with the Gym by getting Claude to post-train a model of its choosing to respond only in [rhyme](https://open.spotify.com/episode/5txYOHA44zWiSgNK623Epp).

## Set up

First, we'll install the `training-gym` CLI:

```bash
pip install -q git+https://github.com/modal-projects/training-gym.git@main
training-gym --help
```

Then, we'll install the provided skills into our current project:

```bash
training-gym skills install
```

The main skill agents should use is `agent-driven-training`, which lays out the RL training lifecycle:

- Ask before making choices that change model behavior or GPU cost.
- Catch dataset and reward bugs locally before they waste GPU time.
- Scale up only after smoke runs indicate the training pipeline is healthy.
- Inspect actual model outputs to verify that higher rewards induce the intended behavior.
- Investigate suspicious reward trends to prevent [reward hacking](https://en.wikipedia.org/wiki/Reward_hacking).

To learn more about the CLI and the provided skills, see the [reference page](https://gym.modal.dev/reference/cli).

## Let it cook

Here's the example prompt:

```txt
can you post-train a model to rhyme in its output
```

We leave it ambiguous to demonstrate that when empowered with the right tools and skills, agents are capable of making sensible choices. Here, it chose to train [Qwen3-4B](https://huggingface.co/Qwen/Qwen3-4B) on prompts taken from [tatsu-lab/alpaca](https://huggingface.co/datasets/tatsu-lab/alpaca).

Since it is just writing Python code, we can easily inspect what it wrote. First, it loaded the dataset:

```python
from modal_training_gym import HuggingFaceDataset

SYSTEM_PROMPT = (
    "You are a poet who answers every question in rhyme. Answer the question "
    "correctly and completely, but write the entire answer as verse: at least "
    "four lines, one clause per line, with line endings that rhyme in couplets "
    "(AABB). Do not write any prose, preamble, or explanation outside the verse."
)


rhyme_dataset = HuggingFaceDataset(
    hf_repo="tatsu-lab/alpaca",
    hf_split=f"train[:512]",
    input_column="instruction",
    output_column="output",
    input_format="text",
    system_prompt=SYSTEM_PROMPT,
)
```

Next, it defined the reward function. Here, we care about the model's ability to both rhyme and answer the user's question. As our [intro tutorial](https://gym.modal.dev/tutorials/rl_basics) shows, NLTK’s [CMU Pronouncing Dictionary](https://github.com/prosegrinder/python-cmudict) is a useful library for measuring the former.

<details>
<summary>What's going on here</summary>

The reward function finds phonemes from each line's last stressed vowel onward and compares line endings under both the AABB and ABAB rhyme schemes. After some initial testing, the agent found two exploits the model took advantage of:

- Words rhyme with themselves, so the model repeated the last word of the sentence.
- One-word lines are easy to write and rhyme, so the model found that being concise was better than trying its best.

Luckily, these are simple problems that can be detected, and the agent implemented anti-gaming measures accordingly.

</details>

```python
import re

_CMUDICT: dict = {}
_VOWELS = ("A", "E", "I", "O", "U")


def _cmudict() -> dict:
    if not _CMUDICT:
        import nltk
        from nltk.corpus import cmudict

        nltk.download("cmudict", quiet=True)
        _CMUDICT.update(cmudict.dict())
    return _CMUDICT


def _strip_thinking(text: str) -> str:
    """Drop a ``<think>`` block and any stray markdown bullets/numbering."""
    text = re.sub(r"<think>.*?</think>", "", text, flags=re.DOTALL)
    text = re.sub(r"</?think>", "", text)
    return text.strip()


def _lines(text: str) -> list[str]:
    return [line.strip() for line in _strip_thinking(text).split("\n") if line.strip()]


def _end_word(line: str) -> str:
    words = re.findall(r"[a-zA-Z']+", line)
    return words[-1].lower().strip("'") if words else ""


def rhyme_tail(word: str) -> tuple:
    """Phonemes from the last stressed vowel onward, stress markers removed.

    Falls back to the last three letters for words the dictionary doesn't know
    (names, coinages), which is a decent orthographic proxy.
    """
    if not word:
        return ()
    phones = _cmudict().get(word)
    if not phones:
        return ("~", word[-3:])
    seq = phones[0]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modal-projects/training-gym](https://github.com/modal-projects/training-gym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
