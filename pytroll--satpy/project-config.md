---
trigger: always_on
description: Satpy is a [Pytroll](https://pytroll.github.io/) library for reading, manipulating, and writing data
---

# Satpy — Agent Guide

Satpy is a [Pytroll](https://pytroll.github.io/) library for reading, manipulating, and writing data
from remote-sensing earth-observing satellite instruments.
Satpy is an abstraction layer over `pyresample`, `pyspectral`, `trollimage`, `pycoast`, and
`python-geotiepoints` libraries.

If something below is not clear, consult the Sphinx documentation in `doc/source/`; the
"Where to read more" table at the bottom points at the most relevant pages.

## Core concepts

Satpy uses a high-level `Scene` object (`satpy/scene.py`) to wrap the functionality of the other parts
of Satpy. The Scene is both a container for the data being worked with and the interface to act on
that data. Every component of Satpy is technically optional: the component classes can be imported
and called directly, and users can skip anything their task doesn't need. A `Scene` is not required,
it is only the easiest way to tie the components together.

Data processed by Satpy may be referred to as a dataset, a product, channel, or a band.
Data is always an `xarray.DataArray` wrapping a **dask array**. Dims should be `y` and `x`.
A third `bands` dimension is common for representing image bands (ex. R, G, B). Other dimensions
(ex. time) are also possible. In rare cases 1D arrays are used and may only have a `y` dimension.
Metadata lives in `.attrs`; `area`, `start_time`, `end_time`, `units`, `standard_name`, and `sensor`
are expected to be present.
A `DataID` (`satpy/dataset/dataid.py`) object is used as the identifier for each product.
A user may use a `DataQuery` (`satpy/dataset/dataid.py`) to access a product of a specific `DataID`
as shorthand for the full `DataID` or when not all parts of the ID are known.
Default ID keys for imager bands: `name`, `wavelength`, `resolution`, `calibration`, `modifiers`.

### Geographic data

Geolocation objects are stored in the `.attrs["area"]` field. The types of geographic data Satpy
deals with:

- 2D projected data: Data mapped to a projected grid of pixels. Dimensions are `y` and `x`
  and are most often in units of "meters" or "degrees". Geolocation is usually defined by
  an `AreaDefinition` object from `pyresample` using a Coordinate Reference System (CRS) to define
  the projection, width and height to define the shape in pixels, and extents to define the outer
  edges of the area in projection units.
- 2D swath data: Data recorded from a polar-orbiting satellite where the `y` dimension usually is the
  along-track direction and `x` is the cross-track direction. Geolocation is usually defined by
  a 2D longitude and 2D latitude array contained in a `SwathDefinition` object from `pyresample`.
- 1D data: Sometimes 1-dimensional data is processed that represents an unstructured version of one
  of the above 2 cases or some other case not well represented by the existing structures.

### High-level components

The main components wrapped by the `Scene` and used in a typical user workflow are:

- **Readers**: Given input data files (typically on-disk) determine which files can be read and make
  the contents available to the user as `xarray.DataArray` objects identified by a `DataID`. Readers
  use a collection of file handlers (one per input file) to do the actual reading of the files.
- **Compositors**: Combine multiple datasets to create a new dataset. It is common for a composite's
  `standard_name` to be used to link it to an enhancement configuration.
- **Modifiers**: Transform or correct a single band. Generally modifiers retain the overall meaning
  of the original data, but have adjusted some aspect of it (ex. removing atmospheric effects).
- **Writers**: Write data to another (typically on-disk) format. Writers also use "enhancements" to scale
  data to be written.

All high-level components are a Python class plus a YAML config. YAML under `satpy/etc/` points at
classes with `!!python/name:` and is loaded with PyYAML's `UnsafeLoader` (which is why `.bandit`
skips B506). Satpy typically finds a YAML configuration and then loads the associated
Python object with information from the YAML file. The YAML is what makes a component
*discoverable*: a class with no YAML entry can still be imported and called directly, but nothing
config-driven (`Scene.load`, `available_dataset_names`, the `DependencyTree`) will ever find it.

When composites or modifiers have dependencies the `DependencyTree` (`satpy/dependency_tree.py`)
is used to resolve which dataset should be used and where it should come from
(ex. another composite versus a loaded reader).

### Low-level components

- **Resamplers**: Remap data from one geolocation to another (swath → area, area → area, …). Usually
  referenced by a short string name rather than instantiated directly.
- **Enhancements**: Normalize (0-1) or colorize data (ex. single band -> RGB) before data is written
  to an output format or visualized (ex. matplotlib plot).

### Typical workflow

`Scene(filenames, reader=)` → `load([...])` → `resample(area)` → `save_datasets()`

Internally that is **readers** → **compositors** / **modifiers** (ordered by the `DependencyTree`) →
**resampling** → **enhancements** → **writers**.

## MultiScene


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pytroll/satpy](https://github.com/pytroll/satpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
