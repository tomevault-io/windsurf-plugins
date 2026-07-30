---
trigger: always_on
description: You have additional SKILLs documented in directories containing a "SKILL.md" file.
---

<skills>

You have additional SKILLs documented in directories containing a "SKILL.md" file.

These skills are:
 - hf-cli -> "skills/hf-cli/SKILL.md"
 - hf-cloud-aws-context-discovery -> "skills/hf-cloud-aws-context-discovery/SKILL.md"
 - hf-cloud-python-env-setup -> "skills/hf-cloud-python-env-setup/SKILL.md"
 - hf-cloud-sagemaker-deployment-planner -> "skills/hf-cloud-sagemaker-deployment-planner/SKILL.md"
 - hf-cloud-sagemaker-iam-preflight -> "skills/hf-cloud-sagemaker-iam-preflight/SKILL.md"
 - hf-cloud-sagemaker-production-defaults -> "skills/hf-cloud-sagemaker-production-defaults/SKILL.md"
 - hf-cloud-serving-image-selection -> "skills/hf-cloud-serving-image-selection/SKILL.md"
 - hf-mem -> "skills/hf-mem/SKILL.md"
 - huggingface-best -> "skills/huggingface-best/SKILL.md"
 - huggingface-community-evals -> "skills/huggingface-community-evals/SKILL.md"
 - huggingface-datasets -> "skills/huggingface-datasets/SKILL.md"
 - huggingface-gradio -> "skills/huggingface-gradio/SKILL.md"
 - huggingface-llm-trainer -> "skills/huggingface-llm-trainer/SKILL.md"
 - huggingface-local-models -> "skills/huggingface-local-models/SKILL.md"
 - huggingface-lora-space-builder -> "skills/huggingface-lora-space-builder/SKILL.md"
 - huggingface-paper-publisher -> "skills/huggingface-paper-publisher/SKILL.md"
 - huggingface-papers -> "skills/huggingface-papers/SKILL.md"
 - huggingface-spaces -> "skills/huggingface-spaces/SKILL.md"
 - huggingface-tool-builder -> "skills/huggingface-tool-builder/SKILL.md"
 - huggingface-trackio -> "skills/huggingface-trackio/SKILL.md"
 - huggingface-vision-trainer -> "skills/huggingface-vision-trainer/SKILL.md"
 - huggingface-zerogpu -> "skills/huggingface-zerogpu/SKILL.md"
 - train-sentence-transformers -> "skills/train-sentence-transformers/SKILL.md"
 - transformers-js -> "skills/transformers-js/SKILL.md"
 - trl-training -> "skills/trl-training/SKILL.md"

IMPORTANT: You MUST read the SKILL.md file whenever the description of the skills matches the user intent, or may help accomplish their task. 

<available_skills>

hf-cli: `Hugging Face Hub CLI (`hf`) for downloading, uploading, and managing models, datasets, spaces, buckets, repos, papers, jobs, and more on the Hugging Face Hub. Use when: handling authentication; managing local cache; managing Hugging Face Buckets; running or scheduling jobs on Hugging Face infrastructure; managing Hugging Face repos; discussions and pull requests; browsing models, datasets and spaces; reading, searching, or browsing academic papers; managing collections; querying datasets; configuring spaces; setting up webhooks; or deploying and managing HF Inference Endpoints. Make sure to use this skill whenever the user mentions 'hf', 'huggingface', 'Hugging Face', 'huggingface-cli', or 'hugging face cli', or wants to do anything related to the Hugging Face ecosystem and to AI and ML in general. Also use for cloud storage needs like training checkpoints, data pipelines, or agent traces. Use even if the user doesn't explicitly ask for a CLI command. Replaces the deprecated `huggingface-cli`.`
hf-cloud-aws-context-discovery: `Discover the user's local AWS context (active profile, region, account ID, caller identity) at the start of any AWS task. Use this skill before any other AWS work — deploying to SageMaker, creating resources, calling AWS APIs, or anything that touches an AWS account. Use it especially when the user has not specified a region or profile explicitly, when they say things like "use my AWS account", "deploy to AWS", "use my profile", or when about to make any AWS CLI or SDK call. Never guess the region or account ID — always use this skill to read it from the local configuration first.`
hf-cloud-python-env-setup: `Set up an isolated Python environment for SageMaker / AWS work, with the right Python version and current boto3. Use this skill whenever Python code will be executed for a SageMaker deployment, training job, or any AWS automation — including when about to run `pip install`, when about to invoke `boto3`, when creating or activating a virtualenv, or when the user asks to "set up the environment". Never use system Python and never `pip install` into it. Always isolate. This skill prevents the most common failure modes: wrong Python version, dependency conflicts, and stale SDKs.`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/skills](https://github.com/huggingface/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
