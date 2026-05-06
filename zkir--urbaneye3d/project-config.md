---
trigger: always_on
description: *   **JAVA version:** use JAVA 11
---

# Urban Eye 3D – JOSM 3D Viewer Plugin

## Operation instructions
*   **JAVA version:** use JAVA 11
*   **Definition of Done:** A task is considered DONE only when:
    * `mvn package` completes successfully without any errors.
    * Successful execution of manual test confirmed by the human.
    * Unit test is created or at least proposed.
    * GEMINI.md file is updated, including (but not limiting to) the following sections: *Recent Accomplishments*, *Learnings*, and if necessary, *Next Steps*.  
    * [features.md](docs/features.md) is reviewed and updated if necessary.
*   **Do not suggest git commits**. Git commits in this project are allowed for protein-based developers only.
*   **JOSM source code** can be found in d:\UrbanEye3D\ext_sources\josm_source
*   Use UrbanEye3dPlugin.debugMsg() for debug messages instead of System.out.println().
*   The `GroundPlaneTest` autotest is not stable and should be run several times in case of failure. 

## Goals

* Create a JOSM plugin that displays loaded buildings (including `building:part=*`) and other objects in a separate 3D window, making creation and editing of 3d building in OSM easier.
* Make it possible to generate more realistic 3D buildings based on OSM data, including windows, cornices, doors, entrances and  building passages.

## Next Steps

### Musts for the Next Release 

* None currently?
    
#### JOSM patches to monitor

* None currently   
    
### Nice to have in the Next Release     

* None currently   

### Feature candidates

1. **Support windows/facades**
    * Buildings with windows are nice.  This feature is present in osm2world, so we also want it. 
    * There is a tag in osm for windows: [window=*](https://wiki.openstreetmap.org/wiki/Key:window).
    * We want to implement "facade" feature similar to X-plane one. https://developer.x-plane.com/article/facade-creation
    * We already have some sample facades: https://github.com/Zkir/VFR_LANDMARKS_3D_RU/blob/master/Facades

2. **Support objects from pre-made meshes**
    * `highway=street_light`    
    * `amenity=bench` 

3. **Increase resolution for GroundTile/MapCSS style**.
    * Some kind of smart scaling is required, for the nearest tiles only, because it will create huge performance impact otherwise.
    
4. **Support forests**
    * Since we have trees now, it would be nice to render them on `natural=wood` and `landuse=forest`
    * We already have a plan for it: [NATURAL-WOOD.md](docs/dev/NATURAL-WOOD.md)
    * Could be tricky, because proper implementation require subtraction of roads.

5. **Support chimney/frustum**
    * F4 displays chimneys (`man_made=chimney`), we currently do not. To make chimneys look realistic, we need to support 'shape=frustum', like we already support 'shape=hyperboloid'. probably explicit shape=prism should be supported too.
    
6. **Validate overlapping 3D walls**
    * Coplanar walls cause flickering and look bad. 
    * Validator check should be added to find such walls.   
    * This check could be based on finding walls which share common ways/nodes   

7.  **Display MapCSS-based ground-plane regardless of selected satellite layers**    
    * Currently, 2D Ground Plane only shows up if satellite imagery is disabled. It's not realy convinient ad counter-intuitive. One may want to see sattelite in 2D window and MapCss based layer in 3D windows. 
	* Solution can be to introduce a new switch in plugin Preferences and a keyboard shortcut to quickly turn satellite imagery on and of, if selected.

### Ideas for the Further Development

See: [IDEAS.md](docs/dev/IDEAS.md)


## Recent Accomplishments

### Earlier
See [Devblog](DEVBLOG.md)


## Architecture and Key Concepts

This section combines high-level architectural overview with key lessons learned during development.

### Code Structure

```
src
├── main
│   └── java
│       └── ru
│           └── zkir
│               ├── customtms              // Module for working with satellite imagery (TMS).
│               │   └── ...                // Contains the implementation for tile loading and caching,
│               │                          // as well as the definition of imagery providers. 
│               │                           
│               ├── easytext               // Internationalization (i18n) module, implemented purely in Java.
│               │   └── ...                // Replaces external utilities for parsing PO/POT files
│               │                          // and compiling binary LANG files for JOSM.
│               │
│               └── urbaneye3d                   // Main module for the "UrbanEye3D" JOSM plugin.
│                   ├── UrbanEye3dPlugin.java    // Main plugin class, entry point.
│                   ├── DialogWindow3D.java      // Dockable window for displaying the 3D scene.
│                   └── ...                      // All other files :) 
│
└── test
    └── java
        └── ru
            └── zkir
                ├── customtms              // Tests for the TMS engine. 
                │   └── ...                //More or less independent from main plugin funtionality 
                │
                └── urbaneye3d
                    ├── RoofGeneratorTopologyTest.java     // Tests the topology of generated 3D roof models.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zkir/UrbanEye3D](https://github.com/Zkir/UrbanEye3D) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
