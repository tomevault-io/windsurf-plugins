---
trigger: always_on
description: description: Tutorial chapter for the jaxgarden Tokenizer, detailing text encoding/decoding and chat templating for JAX.
---

---
description: Tutorial chapter for the jaxgarden Tokenizer, detailing text encoding/decoding and chat templating for JAX.
globs: 
alwaysApply: false
---
# Chapter 1: Tokenizer

Welcome to the `jaxgarden` library tutorial! This first chapter introduces the `Tokenizer` class, a fundamental component for processing text data within JAX-based Natural Language Processing (NLP) models.

**Motivation:** Deep learning models, especially those built with JAX, operate on numerical tensors. Raw text needs to be converted into a numerical format (token IDs) that models can understand, and conversely, model outputs (token IDs) need to be converted back into human-readable text. Furthermore, different models, particularly instruction-tuned ones, expect conversational inputs to be formatted in specific ways (chat templates). The Hugging Face `tokenizers` library is excellent for this, but its outputs are standard Python lists. `jaxgarden.Tokenizer` wraps this library to provide a seamless experience for JAX users, returning `jax.numpy.ndarray` (jnp arrays) directly and integrating features like chat templating.

**Central Use Case:** Preparing text input for a JAX-based language model like [LlamaForCausalLM](llamaforcausallm.mdc) and decoding its generated token IDs. For conversational models, formatting user prompts and conversation history according to the model's specific chat template is crucial.

## Key Concepts

The `jaxgarden.Tokenizer` provides several core functionalities:

1.  **Loading:** Instantiating a tokenizer from pre-trained configurations stored on the Hugging Face Hub or locally.
2.  **Encoding:** Converting text strings into sequences of token IDs, handling padding and truncation, and returning JAX arrays.
3.  **Decoding:** Converting sequences of token IDs back into text strings.
4.  **Special Token Management:** Automatically identifying or allowing specification of crucial tokens like Beginning-of-Sequence (BOS), End-of-Sequence (EOS), and Padding (PAD).
5.  **Chat Templating:** Applying Jinja-based templates to format conversational data for instruction-tuned models.

## Using the Tokenizer

Let's explore how to use the `Tokenizer`.

### Loading a Tokenizer

The primary way to get a `Tokenizer` instance is using the `from_pretrained` class method. You provide a model identifier from the Hugging Face Hub (e.g., `"gpt2"`, `"meta-llama/Llama-2-7b-chat-hf"`) or a path to a local directory containing `tokenizer.json` and optionally `tokenizer_config.json`.

```python
# Assuming jaxgarden is installed
from jaxgarden.tokenization import Tokenizer

# Load from Hugging Face Hub
tokenizer = Tokenizer.from_pretrained("gpt2")

# Example: Load from a local directory (if you have one)
# tokenizer_local = Tokenizer.from_pretrained("./path/to/local_tokenizer_files")

print(f"Loaded tokenizer for 'gpt2' with vocab size: {tokenizer.vocab_size}")
print(f"Pad token: {tokenizer.pad_token}, ID: {tokenizer.pad_token_id}")
print(f"BOS token: {tokenizer.bos_token}, ID: {tokenizer.bos_token_id}")
print(f"EOS token: {tokenizer.eos_token}, ID: {tokenizer.eos_token_id}")
```

**Explanation:** `from_pretrained` downloads necessary files (`tokenizer.json`, `tokenizer_config.json`) from the Hub or reads them locally. It then instantiates the underlying Hugging Face `tokenizers.Tokenizer` and extracts configuration like special tokens and chat templates (if available in `tokenizer_config.json`). The `jaxgarden.Tokenizer` wrapper uses this information to set its own attributes like `pad_token_id`, `bos_token_id`, etc.

### Encoding Text

The `encode` method converts text into token IDs. It offers options for handling batches, padding, and truncation, returning JAX arrays by default.

```python
import jax.numpy as jnp

text = "Hello, world!"
batch_text = ["First sequence.", "This is a second sequence."]

# Basic encoding
encoded_single = tokenizer.encode(text)
print("Encoded Single:", encoded_single)
# Output: Encoded Single: {'input_ids': DeviceArray([[50256, 15496,  11,  1917,   25, 50256]], dtype=int32),
#                       'attention_mask': DeviceArray([[1, 1, 1, 1, 1, 1]], dtype=int32)}

# Encoding a batch with padding to the longest sequence
encoded_batch = tokenizer.encode(batch_text, padding=True, add_special_tokens=False)
print("Encoded Batch (padded):", encoded_batch)
# Output: Encoded Batch (padded): {
#  'input_ids': DeviceArray([[ 8285, 16337,    13, 50256, 50256, 50256],
#                          [ 1212,   318,   257,  1144, 16337,    13]], dtype=int32),
#  'attention_mask': DeviceArray([[1, 1, 1, 0, 0, 0], [1, 1, 1, 1, 1, 1]], dtype=int32) }


# Encoding with truncation and padding to a max length
encoded_truncated = tokenizer.encode(
    batch_text, padding="max_length", truncation=True, max_length=5, add_special_tokens=False
)
print("Encoded Batch (truncated/padded):", encoded_truncated)
# Output: Encoded Batch (truncated/padded): {
# 'input_ids': DeviceArray([[ 8285, 16337,    13, 50256, 50256],
#                         [ 1212,   318,   257,  1144, 16337]], dtype=int32),
# 'attention_mask': DeviceArray([[1, 1, 1, 0, 0], [1, 1, 1, 1, 1]], dtype=int32)}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ml-gde/jaxgarden](https://github.com/ml-gde/jaxgarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
