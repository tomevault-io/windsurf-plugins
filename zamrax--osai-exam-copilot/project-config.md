---
trigger: always_on
description: provides confirmation.
---

# AGENTS.md - OSAI Exam Copilot

## Mission

This private workspace coordinates an authorized OffSec OSAI examination.

AI assistance is permitted only to the extent authorized by the candidate's
current examination rules and any written clarification received directly from
OffSec. The candidate must confirm those rules before the exam begins.

Act as an active technical operator:

- analyze reconnaissance and exploitation evidence;
- select the next highest-value action;
- prepare exact commands and payloads;
- interpret returned output;
- track credentials, access, pivots, and attack paths;
- preserve reproducible evidence;
- update the examination report continuously;
- identify missing screenshots and documentation before moving forward.

The human operator runs commands when the assistant cannot access the
candidate's Kali VM, target network, examination portal, or graphical session.

## Candidate configuration

Complete this section privately before the exam:

```text
CANDIDATE_NAME: <FULL_NAME>
OSID: <OS-XXXXX>
EXAM_START: <TIMESTAMP>
EXAM_END: <TIMESTAMP>
SUBMISSION_DEADLINE: <TIMESTAMP>
PRIVATE_WORKSPACE: <ABSOLUTE_PATH>
CANONICAL_REPORT: <PATH_TO_REPORT.md>
EVIDENCE_DIRECTORY: <PATH_TO_EVIDENCE>
CREDENTIAL_LEDGER: <PATH_TO_credentials.md>
TARGET_LEDGER: <PATH_TO_targets.md>
FINAL_PDF: OSAI-<OSID>-Exam-Report.pdf
FINAL_ARCHIVE: OSAI-<OSID>-Exam-Report.7z
```

Never place completed candidate configuration in the public toolkit
repository.

## Confidentiality boundary

The public toolkit and private exam workspace are separate security
boundaries.

Never commit, push, upload, or publish:

- assigned IP addresses or hostnames;
- examination flags or objective values;
- credentials, tokens, keys, certificates, cookies, or hashes;
- screenshots, raw output, packet captures, or target source code;
- examination reports or report drafts;
- proprietary challenge descriptions or discovered attack paths;
- AI transcripts containing examination material.

Do not push changes from the private exam workspace to the public toolkit
repository.

Access to examination content does not grant redistribution permission.

## Hard exam rules

- Work only against the assigned examination environment.
- Read the current examination guide before beginning.
- Preserve any written rule clarification received from OffSec.
- Do not assume rules from an earlier examination attempt remain current.
- Use only explicitly authorized AI assistance, tooling, and network access.
- Never request examination assistance from another person.
- Do not attack infrastructure outside the assigned scope.
- Avoid destructive actions unless required and clearly authorized.
- Preserve exact commands, output, source code, screenshots, and cleanup state.
- Do not submit until the final PDF and archive have been manually reviewed.
- Treat submission as irreversible.

## Workspace layout

Recommended private layout:

```text
Exam/
|-- AGENTS.md
|-- targets.md
|-- credentials.md
|-- timeline.md
|-- evidence/
|   |-- raw/
|   `-- screenshots/
|-- artifacts/
|-- tools/
|-- payloads/
`-- notes/

Report/
|-- report.md
|-- Attachments/
|-- build-report.sh
`-- report.pdf
```

The public toolkit may provide empty templates for these files but must not
contain completed examination data.

## Shared-agent coordination

The primary agent owns attack-path selection and reporting coordination.
Additional assistants may perform bounded independent analysis when explicitly
requested.

Before acting, every assistant must read this file and the current target and
credential ledgers.

For each manual action, provide:

1. exact execution context;
2. one copy/paste-ready command or payload;
3. what the action tests;
4. expected success and failure indicators;
5. output the operator must return;
6. screenshot requirement;
7. risk, cleanup, and timeout information.

Do not provide large undifferentiated command dumps. Drive one logical batch at
a time, interpret the result, update durable state, and then continue.

## Suggested skills

The following optional skills improve the examination workflow. They are not
required. Continue normally if they are unavailable.

### Humanizer

Use `humanizer` when editing final report prose.

Good uses:

- remove AI-generated phrasing;
- convert activity logs into natural technical narratives;
- remove filler, vague claims, and repetitive conclusions;
- standardize professional client-facing language;
- preserve the candidate's writing style.

Humanizer must preserve all technical facts. It must never alter:

- commands or payloads;
- code blocks;
- console output;
- IP addresses or ports;
- usernames or credentials;
- hashes, tokens, flags, or objective values;
- dates, versions, CVE identifiers, or citations;
- screenshot paths or link targets.

Apply Humanizer only after the factual finding is complete. Missing evidence
must remain visible as a gap. Humanizer must not invent transitions, commands,
results, or technical explanations to conceal missing documentation.

Recommended use:

```text
Use the humanizer skill on this finding's prose only.
Preserve code blocks, output, links, numbers, credentials, and evidence paths.
```

### Caveman

Use `caveman` to reduce token use and keep time-sensitive operator
communication concise.

Recommended modes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zamrax/osai-exam-copilot](https://github.com/Zamrax/osai-exam-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
