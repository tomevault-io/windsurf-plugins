---
trigger: always_on
description: Guidelines for writing Pipelex pipelines
---

# Guide to write or edit pipelines using the Pipelex language in .plx files

- Always first write your "plan" in natural language, then transcribe it in pipelex.
- You should ALWAYS RUN the terminal command `make validate` when you are writing or editing a `.plx` file. It will ensure the pipe is runnable. If not, iterate.
- Please use POSIX standard for files. (empty lines, no trailing whitespaces, etc.)

## Pipeline File Naming
- Files must be `.plx` for pipelines (Always add an empty line at the end of the file, and do not add trailing whitespaces to PLX files at all)
- Files must be `.py` for code defining the data structures
- Use descriptive names in `snake_case`

## Pipeline File Outline
A pipeline file has three main sections:
1. Domain statement
2. Concept definitions
3. Pipe definitions

### Domain Statement
```plx
domain = "domain_name"
description = "Description of the domain" # Optional
```
Note: The domain name usually matches the plx filename for single-file domains. For multi-file domains, use the subdirectory name.

### Concept Definitions

Concepts represent ideas and semantic entities in your pipeline. They define what something *is*, not how it's structured.

```plx
[concept]
ConceptName = "Description of the concept"
```

**Naming Rules:**
- Use PascalCase for concept names
- Never use plurals (no "Stories", use "Story") - lists are handled implicitly by Pipelex
- Avoid circumstantial adjectives (no "LargeText", use "Text") - focus on the essence of what the concept represents
- Don't redefine native concepts (Text, Image, PDF, TextAndImages, Number, Page)

**Native Concepts:**
Pipelex provides built-in native concepts: `Text`, `Image`, `PDF`, `TextAndImages`, `Number`, `Page`. Use these directly or refine them when appropriate.

**Refining Native Concepts:**
To create a concept that specializes a native concept without adding fields:

```plx
[concept.Landscape]
description = "A scenic outdoor photograph"
refines = "Image"
```

For details on how to structure concepts with fields, see the "Structuring Models" section below.

### Pipe Definitions

## Pipe Base Definition

```plx
[pipe.your_pipe_name]
type = "PipeLLM"
description = "A description of what your pipe does"
inputs = { input_1 = "ConceptName1", input_2 = "ConceptName2" }
output = "ConceptName"
```

The pipes will all have at least this base definition. 
- `inputs`: Dictionary of key being the variable used in the prompts, and the value being the ConceptName. It should ALSO LIST THE INPUTS OF THE INTERMEDIATE STEPS (if PipeSequence) or of the conditional pipes (if PipeCondition).
So If you have this error:
`StaticValidationError: missing_input_variable • domain='expense_validator' • pipe='validate_expense' • 
variable='['invoice']'``
That means that the pipe validate_expense is missing the input `invoice` because one of the subpipe is needing it.

NEVER WRITE THE INPUTS BY BREAKING THE LINE LIKE THIS:
<NEVER DO THIS>
```plx
inputs = {
    input_1 = "ConceptName1",
    input_2 = "ConceptName2"
}
```
</NEVER DO THIS>

- `output`: The name of the concept to output. The `ConceptName` should have the same name as the python class if you want structured output:

### Input Multiplicity

By default, inputs expect a single item. Use bracket notation to specify multiple items:

```plx
# Single item (default)
inputs = { document = "Text" }

# Variable list - indeterminate number of items
inputs = { documents = "Text[]" }

# Fixed count - exactly N items
inputs = { comparison_items = "Image[2]" }
```

**Key points:**
- No brackets = single item (default behavior)
- Use `[]` for lists of unknown length
- Use `[N]` (where N is an integer) when operation requires exact count (e.g., comparing 2 items)

## Structuring Models

Once you've defined your concepts semantically (see "Concept Definitions" above), you need to specify their structure if they have fields.

### Three Ways to Structure Concepts

**1. No Structure Needed**

If a concept only refines a native concept without adding fields, use the TOML table syntax shown in "Concept Definitions" above. No structure section is needed.

**2. Inline Structure Definition (RECOMMENDED for most cases)**

For concepts with structured fields, define them inline using TOML syntax:

```plx
[concept.Invoice]
description = "A commercial document issued by a seller to a buyer"

[concept.Invoice.structure]
invoice_number = "The unique invoice identifier"
issue_date = { type = "date", description = "The date the invoice was issued", required = true }
total_amount = { type = "number", description = "The total invoice amount", required = true }
vendor_name = "The name of the vendor"
line_items = { type = "list", item_type = "text", description = "List of items", required = false }
```

**Supported inline field types:** `text`, `integer`, `boolean`, `number`, `date`, `list`, `dict`

**Field properties:** `type`, `description`, `required` (default: true), `default_value`, `choices`, `item_type` (for lists), `key_type` and `value_type` (for dicts)

**Simple syntax** (creates required text field):
```plx
field_name = "Field description"
```

**Detailed syntax** (with explicit properties):
```plx
field_name = { type = "text", description = "Field description", required = false, default_value = "default" }
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pipelex/cocode](https://github.com/Pipelex/cocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
