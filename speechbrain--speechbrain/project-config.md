---
trigger: always_on
description: SpeechBrain is an open-source PyTorch toolkit for conversational AI (speech recognition, speaker verification, speech enhancement, separation, TTS, spoken language understanding, and more) known for its ease of use and flexibility. It is Apache 2.0 licensed.
---

# AGENTS.md

SpeechBrain is an open-source PyTorch toolkit for conversational AI (speech recognition, speaker verification, speech enhancement, separation, TTS, spoken language understanding, and more) known for its ease of use and flexibility. It is Apache 2.0 licensed.

## Writing code

- **Minimal try/except**: let errors propagate — silent failures hide bugs. Only catch exceptions for intentional fault tolerance (retries, robustness).
- **Targeted comments**: don't explain your work process or reference old code. Use targeted comments sparingly to clarify ambiguous logic.
- **Zen of Python**: remember the Zen of Python when writing code.
```
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

## Project structure

```
speechbrain/          # Core library (importable as `import speechbrain`)
  core.py             # Brain class — the central training/eval orchestrator
  dataio/             # Data loading, batching, samplers, dataset objects
  nnet/               # Neural network building blocks (RNN, CNN, attention, transformers, losses, etc.)
  lobes/              # Higher-level model components (feature extractors, encoders, full models like CRDNN, wav2vec2)
  decoders/           # CTC, seq2seq beam search, transducer decoders
  processing/         # Signal processing (features, augmentation, multi-mic)
  utils/              # Checkpointing, distributed training, metrics, logging, profiling
  inference/          # Pretrained model interfaces (EncoderClassifier, EncoderDecoderASR, etc.)
  integrations/       # Optional heavy-dependency integrations (Transformers, Whisper, etc.)
  lm/                 # Language model utilities
recipes/              # Training scripts organized as recipes/{dataset}/{task}/{model}/
  {dataset}/{task}/
    {model}/          # Recipe implementation for a specific model/configuration
      train.py        # Training script (subclasses Brain)
      hparams/        # HyperPyYAML config files (train.yaml, etc.)
      extra_requirements.txt  # Recipe-specific pip dependencies (if any)
      README.md       # Results, how to run, pretrained model links
templates/            # Minimal working examples to bootstrap new recipes
tests/
  unittests/          # Unit tests for core library
  integration/        # Integration tests (small end-to-end training runs)
docs/                 # Documentation
  tutorials/          # Jupyter notebooks integrated into ReadTheDocs
tools/                # Maintenance scripts (tutorial cell updater, etc.)
```

## Architecture concepts

### The Brain class (`speechbrain.core.Brain`)

Brain is the central abstraction for all training and evaluation. Every recipe subclasses it and overrides the following methods:

- `compute_forward(batch, stage)` — forward pass, returns predictions
- `compute_objectives(predictions, batch, stage)` — computes loss, logs metrics

The `stage` argument is a `Stage` enum: `TRAIN`, `VALID`, or `TEST` which defines the current stage of the training loop. Brain handles the training loop, checkpointing, distributed training (DDP), gradient accumulation, mixed precision, and logging.

```python
class ASR(sb.Brain):
    def compute_forward(self, batch, stage):
        wavs, lens = batch.sig  # (batch, time), (batch,) relative lengths
        feats = self.hparams.compute_features(wavs)
        feats = self.modules.encoder(feats)
        return self.modules.decoder(feats)

    def compute_objectives(self, predictions, batch, stage):
        tokens, token_lens = batch.tokens
        loss = self.hparams.ctc_cost(predictions, tokens, lens, token_lens)
        if stage != sb.Stage.TRAIN:
            self.cer_metric.append(batch.id, predictions, tokens)
        return loss
```

Key lifecycle methods you can override: `on_stage_start`, `on_stage_end`, `on_fit_batch_end`, `fit_batch`, `evaluate_batch`, `init_optimizers`.

### HyperPyYAML (the YAML config system)

SpeechBrain uses HyperPyYAML, an extended YAML syntax maintained by SpeechBrain at https://github.com/speechbrain/HyperPyYAML. This is NOT plain YAML — it is a declarative system that can instantiate Python objects, resolve references, and perform simple arithmetic. Understanding it is essential.

Key tags:
- `!new:module.ClassName` — instantiates a Python object. Indented keys become constructor kwargs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [speechbrain/speechbrain](https://github.com/speechbrain/speechbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
