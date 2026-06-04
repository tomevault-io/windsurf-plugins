---
trigger: always_on
description: Interactive astronomy image viewer and analysis toolkit for accessing real telescope data and virtual observatory archives.
---

# AstronomyImages

Interactive astronomy image viewer and analysis toolkit for accessing real telescope data and virtual observatory archives.

## Project Overview

This project provides Python-based tools for downloading, visualizing, and analyzing astronomical images from public sky surveys and virtual observatories. It includes both command-line scripts and interactive GUI applications for exploring the night sky.

## Key Features

- **Interactive Sky Map Viewer**: Click-based interface to explore any region of the sky with real telescope data
- **Multiple Survey Support**: Access to DSS2 (Red/Blue/IR), 2MASS (J/H/K bands), WISE, and other sky surveys
- **Real-Time Downloads**: Fetch FITS astronomical images via NASA SkyView API
- **Advanced Visualization**: Multiple color maps, intensity scaling, and analysis tools
- **FITS File Support**: Full support for astronomical FITS format with WCS (World Coordinate System)
- **Simulation Capabilities**: Generate simulated observations for educational purposes (Horsehead Nebula demo)

## Main Scripts

### Interactive Applications
- **`interactive_sky_map.py`**: Full-featured GUI with clickable all-sky map, constellation overlays, and telescope view
- **`astronomy_viewer_gui.py`**: Simplified viewer with object catalog and manual coordinate entry

### Demonstration & Examples
- **`horsehead_complete_demo.py`**: Complete workflow demonstration with simulated Horsehead Nebula observation
  - Physics-based simulation of dark nebulae
  - Optical depth calculations
  - Comprehensive 9-panel analysis plots
  - API documentation and examples

### Utility Scripts
- **`horsehead_skyview.py`** / **`horsehead_skyview_fixed.py`**: Direct SkyView API integration examples
- **`horsehead_alternative.py`**: Alternative implementation approaches

## Data Access APIs

### NASA SkyView Virtual Observatory
- **URL**: https://skyview.gsfc.nasa.gov
- **Authentication**: None required
- **Surveys**: DSS2, 2MASS, WISE, SDSS, and 30+ others
- **Format**: FITS images with WCS headers

### Las Cumbres Observatory (Educational Access)
- **URL**: https://observe.lco.global
- **Authentication**: Free API token for education
- **Capability**: Schedule real telescope observations worldwide
- **Details**: See `api_examples.json` for request format

## File Structure

```
AstronomyImages/
├── interactive_sky_map.py          # Main interactive viewer
├── astronomy_viewer_gui.py         # Simplified GUI
├── horsehead_complete_demo.py      # Full demo with simulation
├── horsehead_skyview*.py           # API integration examples
├── api_examples.json               # API documentation
├── *.fits                          # Downloaded astronomical images
├── *.png                           # Visualization outputs
└── horsehead_analysis.json         # Analysis results
```

## Dependencies

```python
# Core astronomy libraries
astropy              # FITS files, WCS, coordinates
numpy                # Numerical operations
matplotlib           # Visualization

# GUI (for interactive viewers)
tkinter              # Built-in Python GUI

# Network
requests             # API calls
```

## Usage Examples

### Interactive Sky Map
```bash
python interactive_sky_map.py
# Click anywhere on the sky map to select coordinates
# Adjust FOV, survey, and resolution
# Download and view in real-time
```

### Simple Viewer
```bash
python astronomy_viewer_gui.py
# Choose from catalog of famous objects
# Or enter manual coordinates
# Download FITS and save locally
```

### Run Horsehead Demo
```bash
python horsehead_complete_demo.py
# Generates simulated observation
# Creates comprehensive analysis plots
# Outputs FITS, PNG, and JSON files
```

## Technical Notes

- All coordinates use J2000 equatorial system (RA/Dec in degrees)
- FITS images include WCS headers for coordinate mapping
- Default pixel scale: 1 arcsec/pixel (configurable)
- Image scaling: Linear or percentile-based (1-99%)
- No API keys required for SkyView access

## Educational Applications

- Learn about astronomical coordinate systems
- Explore real telescope data interactively
- Understand FITS file format and WCS
- Practice image analysis techniques
- Visualize dark nebulae and emission regions

## Security & Privacy

- **No authentication credentials stored in code**
- All API tokens are placeholder examples only ("YOUR_API_TOKEN")
- Safe for public repository sharing
- Network requests use standard HTTPS

## Future Enhancements

- Add support for more sky surveys (Pan-STARRS, etc.)
- Implement advanced image processing (stacking, calibration)
- Integration with SIMBAD/NED for object identification
- Export to DS9 region files
- Multi-wavelength comparison views

## License & Data Usage Rights

### This Software (Python Code)
**MIT License** - Free to use, modify, and distribute for any purpose, including commercial applications.

Copyright (c) 2025 - This astronomy viewer toolkit


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mthiel74/AstronomyImages](https://github.com/mthiel74/AstronomyImages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
