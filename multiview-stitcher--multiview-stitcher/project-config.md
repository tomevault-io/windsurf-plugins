---
trigger: always_on
description: - be simple and intuitive
---

# Web app interface design

The interface should
- be simple and intuitive
- be responsive (should work on different screen sizes)

## Upper panel

- left: multiview stitcher logo / app name
- center: progress bar
- right:
  - "Log" button to show log
  - spin box for number of workers, by default 3
  - Viewer controls help button
    - zoom in/out (requires ctrl)
    - pan
  - github link to multiview-stitcher repo
  - "About" button
    - neuroglancer, ome-zarr, pyodide

## Center panel

- neuroglancer viewer
- for yx data, the layout should be 'xy'
- for zyx data, the layout should be 4 panels
- don't show an open layer panel, but don't make it inaccessible
- don't show the layer bar - neuroglancer's horizontal list of layers across
  the top of the viewer. The left panel already lists them, and that row grows
  with the number of tiles
- don't show the tool palette
- layers can be highlighted over their borders as shown - the image's own edges,
  so a tile that has been turned is outlined at its angle rather than by the
  upright box around it - in two strengths matching the layers list: *very
  subtle* (just highlighted) and *subtle* (highlighted as selected)
  - the layer under the pointer is very subtly highlighted, in the viewer and
    in the layers list, whichever of the two the pointer is over
  - selected layers are subtly highlighted
  - clicking a layer that is uniquely under the pointer selects it; ctrl / cmd
    adds it to or removes it from the selection, as in the layers list
  - a hidden layer is not highlighted, pointed at, picked or dragged: the
    visibility toggle takes a layer out of the way entirely. It stays selected
    while hidden, and comes back as it was

## Left panel

Data visualization and control panel

- Data drop zone
  - drag and drop OME-Zarrs (top level ome-zarr or folder containing multiple ome-zarrs)
  - drag and drop mosaic .czi files; every tile of the file becomes a view
  - click to open file dialog
- List of loaded msims + fused preview (if available)
  - elements on each msim:
    - Remove
    - Short info (shape per dim, number of res levels)
    - Visibility toggle
    - layers can be selected by clicking on them, and the selected layer is subtly highlighted. Several layers can be selected at the same time by holding down ctrl while clicking on them. there's a small button to select / unselect all layers
- Coordinate system selection
  - dropdown menu to select transform_key to show in neuroglancer viewer
- If a time dimension is present, add a time slider to select the timepoint to show in neuroglancer viewer
- Display options
  - list of channels
    - "Positional colors": checkbox to color channels based on their position (see vis_utils.py for details), fused image is unaffected by this option
    - for each channel:
      - visibility toggle
      - contrast limits (min, max)
        - double range slider
        - in the same line / height: min / max text fields

## Right panel

Data manipulation and computation panel

Different tabs for different types of operations:

- Interactive tile placement
  - "New transform_key" button to create a new transform_key, with text for user to enter name of new transform_key (by default, the new transform_key is created as a copy of the currently selected transform_key)
  - Section "Manual placement":
    - Checkbox: "Activate manual placement". In this case, the user can modify transform_keys by dragging the mouse in the viewer
      - drag tiles manually when holding down option/alt (translation)
      - drag tiles manually when holding down ctrl + option/alt (rotation)
      - only the orthogonal views can be used for manual placement, the oblique view is disabled. in each orthogonal view, the user can drag tiles in the plane of the view, and the tile will be moved in 3D space accordingly. for rotation, a given tile is rotated around its center in 2D (that panel's projection plane)
      - After a drag is completed, the transform_key is updated, and the user can see the updated transform_key in the neuroglancer viewer. The layer moves in real time, but the transform_key is only updated after the drag is completed
      - Which layer is being dragged? If it's unique because of the mouse position, then that layer is being dragged. If multiple layers are under the mouse position, the currently selected layer is being dragged; if none is selected, the last layer in the list (the topmost) is. If no layer is under the mouse position, then no layer is being dragged. Layers are selected by clicking on them in the layers list in the left panel, or on the tile itself in the viewer

      - additional options for manual placement:
        - a small list of checkboxes for each channel: only for the checked channels the manual transformation is applied. this works together with the idea that transform_keys can be defined per channel (e.g. `xr.concat(params_c, dim='c').assign_coords({'c': channels})`). By default all channels are ticked. If all channels are treated in the same way, the parameters don't need to have a channel dimension

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [multiview-stitcher/multiview-stitcher](https://github.com/multiview-stitcher/multiview-stitcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
