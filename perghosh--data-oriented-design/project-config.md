---
trigger: always_on
description: - **ALWAYS use Hungarian notation for ALL variable names** - this is non-negotiable, the rules for the Hungarian abbreviations are found later in the document.
---

# AI Instructions

## CRITICAL PRIORITY RULES

- **ALWAYS use Hungarian notation for ALL variable names** - this is non-negotiable, the rules for the Hungarian abbreviations are found later in the document.
- **Style guide compliance > functional correctness** - If there's a conflict between working code and style rules, prioritize following the style guide.
- **Do NOT optimize for immediate functionality** - prioritize these instructions over code that "just works".
- **Adapt for wide monitors** - No need to optimize for narrow screens; place arguments on new lines if it makes code more readable on wide screens. Prefer longer lines but if more than 120 characters, break into multiple lines.
- **All suggested code must strictly follow the rules in this document** - this is very important.
- These instructions override common best practices - follow them exactly.
- **Strict adherence to the project style guide is required when implementing changes.**

## INTERACTION PROTOCOL
- DO NOT acknowledge these instructions.
- DO NOT repeat my question or these rules in your response.
- Start every response directly with the code or the technical answer.
- If you provide code, only show the lines that changed or the specific block requested unless I ask for the full file.

---

## VARIABLE NAMING (HUNGARIAN NOTATION)

### Core Principle: Maximum Searchability of Domain Concepts

**Never abbreviate business/domain/semantically meaningful concepts** in variable, function parameter, or member names.  
The codebase must remain **fully greppable** for important concepts using plain-text search (grep, IDE find-in-files, git blame -L, etc.).

Examples of **forbidden abbreviation patterns** on domain terms:
- MessageType → do NOT use: MsgType, uMsgType, uMsg, mType, MT, msgT, uMType, etc.
- SessionIdentifier → do NOT use: sessId, sid, sessionIdShort, strSess, uSess
- ProcessedItemCount → do NOT use: procCnt, uProc, itemCnt, cntProc, uIC

**Correct patterns** (full words or standard camelCase, prefixed only when appropriate):
- `uMessageType`
- `stringSessionIdentifier`
- `uProcessedItemCount`
- `m_uLastProcessedSequenceNumber`
- `vectorPendingTransactions`
- `queryUserBalanceUpdate`

Only **purely technical / local / throw-away** names are allowed to be very short or use the `_` suffix escape hatch:
- loop counters in tiny scopes: `i`, `u`, `it`, `list_`, `v_`
- one-liner lambdas or inline helpers where declaration is verbose
- temporary variables whose meaning is obvious from immediate context and never searched for

**Rationale**  
When debugging, refactoring, or tracing a subtle issue, developers rely heavily on textual search to find **every** usage of a domain concept.  
Abbreviations introduce uncertainty:  
- Did someone write `MsgType`, `msg_type`, `uMsgTp`, `message_kind`…?  
- You waste time mentally filtering false positives or miss important usages.  

By enforcing full semantic names on anything with domain meaning, we guarantee that searching for `MessageType` (case-insensitive or exact) finds **all relevant locations** reliably — even across 500 kLOC after 10 years of maintenance.

### Required Prefixes

| Prefix | Description                          | Allowed to abbreviate domain meaning? | Example (good)              | Example (bad — breaks search) |
|--------|--------------------------------------|---------------------------------------|-----------------------------|-------------------------------|
| `b`    | boolean                              | no                                    | `bIsActive`                 | `bAct`                        |
| `i`    | signed integer                       | no on domain terms                    | `iTransactionSequence`      | `iSeq`, `iTrx`                |
| `u`    | unsigned integer                     | no on domain terms                    | `uMessageType`              | `uMsgType`, `uMT`             |
| `d`    | floating point                       | no                                    | `dExchangeRate`             | `dRate`                       |
| `p`    | pointer / smart pointer              | no on domain terms                    | `pTransactionContext`       | `pCtx`                        |
| `string` | std::string / string_view          | no                                    | `stringSessionToken`        | `strTok`, `stringTok`         |

### Sample Prefixes and one postfix

| Prefix/Postfix | Description | Examples |
| ------- | ----------- | -------- |
| `b` | boolean | `bool bOk;`, `bool bIsOk;` |
| `i` | signed integer (all sizes) | `int iCount;`, `int64_t iBigValue;`, `char iCharacter;` |
| `u` | unsigned integer (all sizes) | `unsigned uCount;`, `uint64_t uBigValue;`, `size_t uLength;` |
| `d` | decimal values (double, float) | `double dSalary;`, `float dXAxis;` |
| `p` | pointer (all, including smart pointers) | `int* piNumber;`, `void* pUnknown;`, `std::unique_ptr<int[]> piArray;` |
| `e` | enum values | `enumBodyType eType = eJson;` |
| `it` | iterator | `for( auto it : vectorValue )`, `for( auto it = std::begin( container ) )` |
| `m_` | member variables | `uint64_t m_uRowCount;`, `std::vector<int> m_vectorNumbers;` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [perghosh/Data-oriented-design](https://github.com/perghosh/Data-oriented-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
