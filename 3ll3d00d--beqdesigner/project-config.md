---
trigger: always_on
description: provides as a prefix (4 of 8, or all 8), the *display* order JRiver's
---

# `model/jriver` — JRiver Media Center DSP config integration

This module reads and writes JRiver Media Center (JRMC) "DSP Studio" preset
files (`.dsp`) and MCWS-served DSP presets, translating between JRMC's own
on-disk/on-wire format and BEQDesigner's `Filter`/`FilterGraph` object model.
It is the only place in the codebase that understands the JRMC file format,
so it is the module to read/update whenever JRiver changes that format in a
new MC release.

Read this before touching parsing, encoding, or channel/format logic here —
the format has several non-obvious layers and a couple of sharp edges that
are easy to reintroduce a regression in.

## Module map

| File | Responsibility |
|---|---|
| `dsp.py` | `JRiverDSP` — top level orchestrator. Parses a whole `.dsp` config into one `FilterGraph` per enabled PEQ block, and serializes graphs back into config text. |
| `codec.py` | Low level, format-only helpers: XPath lookups into the `Preset` XML, the `(length:content)` micro-format used inside `<Value>` text, and splicing new filter text back into the original document. No knowledge of `Filter` classes. |
| `filter.py` | The `Filter` class hierarchy (one class per JRMC filter `Type` code), `ComplexFilter`/`Divider` protocol for grouping filters, `FilterGraph`, the crossover/bass-management synthesis engine (`MultiChannelSystem`, `MultiwayFilter`, `XO`/`MDSXO`), and `FilterOp` (in-process simulation of what JRMC's DSP engine would do to a signal). |
| `formats.py` | Channel name/index tables and `OutputFormat` (maps JRMC's "Output Channels"/"Output Padding Channels"/"Output Channel Layout" trio to a named speaker layout). |
| `routing.py` | `Matrix` — an editor-side input/way/output routing model used when the user designs a crossover in the UI. Independent of the XML format; feeds into `MultiChannelSystem`. |
| `render.py` | Graphviz `dot` rendering of a `FilterGraph`, for the UI's visual filter chain. Not part of the round-trip path. |
| `mcws.py` | `MediaServer` — talks to a running JRMC instance over the MCWS HTTP API (auth, zones, get/set DSP preset) and includes JRMC's own tolerant XML-equality check (`__compare_xml`) used after pushing a preset, as a live sanity check that JRMC accepted it unchanged. |
| `parser.py` | Adapters that turn *other* filter sources (MiniDSP exports, MSO exports) into JRMC `Filter` objects via `convert_filter_to_mc_dsp`. |

## The file format, from the outside in

A `.dsp` file is real XML, but only down to a point:

```
<Preset>
  <Key Name="Audio Settings">
    <Data><Name>Output Channels</Name><Value>6</Value></Data>
    <Data><Name>Output Padding Channels</Name><Value>0</Value></Data>
  </Key>
  <Key Name="Parametric Equalizer">
    <Data><Name>Enabled</Name><Value>1</Value></Data>
    <Data><Name>Filters</Name><Value>(1:1)(2:0)...</Value></Data>
  </Key>
  <Key Name="Parametric Equalizer 2"> ... </Key>
  <Key Name="DSP Studio">
    <Data><Name>Plugin Order</Name><Value>(...)(Parametric Equalizer)(...)</Value></Data>
  </Key>
</Preset>
```

Every setting lives at `/Preset/Key[@Name=X]/Data/Name[.=Y]/../Value`
(`codec.xpath_to_key_data_value`). There are up to **two independent PEQ
("Parametric Equalizer") blocks**, block 0 = `Parametric Equalizer`, block 1
= `Parametric Equalizer 2` (`codec.get_peq_key_name`). Only PEQ blocks with a
sibling `Enabled` `Data` set to `"1"` are active (`codec.get_peq_block_order`);
if both are active, the `DSP Studio`/`Plugin Order` value's token order
decides which one runs first (i.e. which one's *output* channels become the
other's *input* channels) — `JRiverDSP` builds one `FilterGraph` per active
block in that order, but each graph's `.stage` records the real block number
(0 or 1), not its position in the list. **Don't confuse graph list index
with `.stage`** — `config_txt()` writes back using `.stage`, and a reversed
plugin order is a real, if unusual, configuration.

Inside a PEQ block's `<Value>`, JRMC uses its **own length-prefixed
micro-format**, not further XML: a sequence of `(length:content)` tokens.
The first two tokens are a fixed header (`(1:1)(N:count)` where `count` is
the total number of filter entries that follow); everything after that is
one `(length:<XMLPH version="1.1">...</XMLPH>)` token per filter, where the
inner `<XMLPH>` blob is itself a small XML document of `<Item Name="K">V</Item>`
pairs — see `codec.filt_to_xml`/`codec.item_to_dicts`. So a single `<Value>`
text node is: real XML → custom length-prefixed text tokens → embedded XML
again. `codec.extract_filters`/`codec.include_filters_in_dsp` are the only
places that touch this middle layer.

Each `<Item>` dict has a `Type` code identifying a `Filter` subclass in
`filter.py` (`filter_classes_by_type`, keyed by `Filter.TYPE`):

| Type | Class | Notes |
|---|---|---|
| 1 | `LowPass` | order 1/2 native; `Slope` encodes order×6 |
| 2 | `HighPass` | as above |
| 3 | `Peak` | parametric EQ band |
| 4 | `Gain` | |
| 5 | `Mute` | |
| 6 | `Mix` | add/copy/move/swap/subtract between two channels |
| 7 | `Delay` | |
| 8 | `LinkwitzTransform` | |
| 9 | `Limiter` | |
| 10 | `LowShelf` | |
| 11 | `HighShelf` | |
| 12 | `Order` | channel reorder |
| 13 | `BitdepthSimulator` | |
| 14 | `SubwooferLimiter` | |
| 15 | `Polarity` | |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [3ll3d00d/beqdesigner](https://github.com/3ll3d00d/beqdesigner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
