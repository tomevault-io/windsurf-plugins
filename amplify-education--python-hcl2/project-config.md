---
trigger: always_on
description: Forward:  HCL2 Text → [PostLexer] → Lark Parse Tree → LarkElement Tree → Python Dict/JSON
---

# HCL2 Parser — CLAUDE.md

## Pipeline

```
Forward:  HCL2 Text → [PostLexer] → Lark Parse Tree → LarkElement Tree → Python Dict/JSON
Reverse:  Python Dict/JSON → LarkElement Tree → Lark Tree → HCL2 Text
Direct:   HCL2 Text → [PostLexer] → Lark Parse Tree → LarkElement Tree → Lark Tree → HCL2 Text
```

The **Direct** pipeline (`parse_to_tree` → `transform` → `to_lark` → `reconstruct`) skips serialization to dict, so all IR nodes (including `NewLineOrCommentRule` nodes for whitespace/comments) directly influence the reconstructed output. Any information discarded before the IR is lost in this pipeline.

## Module Map

| Module | Role |
|---|---|
| `hcl2/hcl2.lark` | Lark grammar definition |
| `hcl2/api.py` | Public API (`load/loads/dump/dumps` + intermediate stages) |
| `hcl2/postlexer.py` | Token stream transforms between lexer and parser |
| `hcl2/parser.py` | Lark parser factory with caching |
| `hcl2/transformer.py` | Lark parse tree → LarkElement tree |
| `hcl2/deserializer.py` | Python dict → LarkElement tree |
| `hcl2/formatter.py` | Whitespace alignment and spacing on LarkElement trees |
| `hcl2/reconstructor.py` | LarkElement tree → HCL2 text via Lark |
| `hcl2/builder.py` | Programmatic HCL document construction |
| `hcl2/walk.py` | Generic tree-walking primitives for the LarkElement IR tree |
| `hcl2/utils.py` | `SerializationOptions`, `SerializationContext`, string helpers |
| `hcl2/const.py` | Constants: `IS_BLOCK`, `COMMENTS_KEY`, `INLINE_COMMENTS_KEY` |
| `cli/helpers.py` | File/directory/stdin conversion helpers |
| `cli/hcl_to_json.py` | `hcl2tojson` entry point |
| `cli/json_to_hcl.py` | `jsontohcl2` entry point |
| `cli/hq.py` | `hq` CLI entry point — query dispatch, formatting, optional operator |
| `hcl2/query/__init__.py` | Public query API exports |
| `hcl2/query/_base.py` | `NodeView` base class, view registry, `view_for()` factory |
| `hcl2/query/body.py` | `DocumentView`, `BodyView` facades for top-level and body queries |
| `hcl2/query/blocks.py` | `BlockView` facade for block queries |
| `hcl2/query/attributes.py` | `AttributeView` facade for attribute queries |
| `hcl2/query/containers.py` | `TupleView`, `ObjectView` facades for container queries |
| `hcl2/query/expressions.py` | `ConditionalView` facade for conditional expressions |
| `hcl2/query/functions.py` | `FunctionCallView` facade for function call queries |
| `hcl2/query/for_exprs.py` | `ForTupleView`, `ForObjectView` facades for for-expressions |
| `hcl2/query/path.py` | Structural path parser (`PathSegment`, `parse_path`, `[select()]`, `type:name`) |
| `hcl2/query/resolver.py` | Path resolver — segment-by-segment with label depth, type filter |
| `hcl2/query/pipeline.py` | Pipe operator — `split_pipeline`, `classify_stage`, `execute_pipeline` |
| `hcl2/query/builtins.py` | Built-in transforms: `keys`, `values`, `length` |
| `hcl2/query/diff.py` | Structural diff between two HCL documents |
| `hcl2/query/predicate.py` | `select()` predicate tokenizer, recursive descent parser, evaluator |
| `hcl2/query/safe_eval.py` | AST-validated Python expression eval for hybrid/eval modes |
| `hcl2/query/introspect.py` | `--describe` and `--schema` output generation |

`hcl2/__main__.py` is a thin wrapper that imports `cli.hcl_to_json:main`.

### Rules (one class per grammar rule)

| File | Domain |
|---|---|
| `rules/abstract.py` | `LarkElement`, `LarkRule`, `LarkToken` base classes |
| `rules/tokens.py` | `StringToken` (cached factory), `StaticStringToken`, punctuation constants |
| `rules/base.py` | `StartRule`, `BodyRule`, `BlockRule`, `AttributeRule` |
| `rules/containers.py` | `TupleRule`, `ObjectRule`, `ObjectElemRule`, `ObjectElemKeyRule` |
| `rules/expressions.py` | `ExprTermRule`, `BinaryOpRule`, `UnaryOpRule`, `ConditionalRule` |
| `rules/literal_rules.py` | `IntLitRule`, `FloatLitRule`, `IdentifierRule`, `KeywordRule` |
| `rules/strings.py` | `StringRule`, `InterpolationRule`, `HeredocTemplateRule`, `TemplateStringRule` |
| `rules/functions.py` | `FunctionCallRule`, `ArgumentsRule` |
| `rules/indexing.py` | `GetAttrRule`, `SqbIndexRule`, splat rules |
| `rules/for_expressions.py` | `ForTupleExprRule`, `ForObjectExprRule`, `ForIntroRule`, `ForCondRule` |
| `rules/directives.py` | `TemplateIfRule`, `TemplateForRule`, and flat directive start/end rules |
| `rules/whitespace.py` | `NewLineOrCommentRule`, `InlineCommentMixIn` |

## Public API (`api.py`)

Follows the `json` module convention. All option parameters are keyword-only.

- `load/loads` — HCL2 text → Python dict
- `dump/dumps` — Python dict → HCL2 text
- `query` — HCL2 text/file → `DocumentView` for structured queries
- Intermediate stages: `parse/parses`, `parse_to_tree/parses_to_tree`, `transform`, `serialize`, `from_dict`, `from_json`, `reconstruct`

### Option Dataclasses

**`SerializationOptions`** (LarkElement → dict):
`with_comments`, `with_meta`, `wrap_objects`, `wrap_tuples`, `explicit_blocks`, `preserve_heredocs`, `force_operation_parentheses`, `preserve_scientific_notation`, `strip_string_quotes`

**`DeserializerOptions`** (dict → LarkElement):
`heredocs_to_strings`, `strings_to_heredocs`, `object_elements_colon`, `object_elements_trailing_comma`

**`FormatterOptions`** (whitespace/alignment):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amplify-education/python-hcl2](https://github.com/amplify-education/python-hcl2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
