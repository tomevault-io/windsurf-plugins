---
trigger: always_on
description: - create a front end user interface for the SotoHP project
---

# frontend user interface guidelines

## goals

- create a front end user interface for the SotoHP project
    - follow the available features in the OPENAPI REST API specification provided by this project
- overall look & feel
    - application title is `Sotohp`
    - create a SVG square logo inspired by the photos mosaic thematic
        - to be used as the favicon.svg
        - put the logo on the left of the application title
- organize features through tabs
- tasks related to tabs to implement
    - [x] image/media visualization tab named `Viewer`
        - maximize the space for the image
        - add a set of control buttons (on the right of the screen)
            - basic buttons (only display icons for all basic buttons, text when the mouse goes over the button) :
                - first
                - previous
                - next
                - last
                - random
                - fullscreen
                - play/pause (slideshow using the policy described by the choice button)
            - starred toggle button (update the media accordingly) :
                - use yellow star (⭐) for starred
                - use empty star (☆) for not starred
                - (should be placed just before the random button)
            - edit button which popup an edit form when clicked (update the media accordingly) :
                - using regular text field for the `description`
                - for the `shootDateTime` timestamp field use a date picker
                - for keywords use colored labels with `x` buttons to easily remove any keyword, enter some text to change it into a colored label
                - for the `userDefinedLocation` use a map with a marker to select/update the location
                - (should be placed just after the button labelled `last`)
            - slideshow choices button : slide show delay choices to control how the slideshow should behave, seconds delay and what to do which is either take the next photo or a random new one
                - 5s random
                - 5s next
                - 10s random
                - 10s next
                - 20s random
                - 20s next
                - 30s random
                - 30s next
        - display image information (on the right of the screen below the buttons)
            - date
            - keywords
            - first event name
            - hasLocation indicator
                - green pin logo : when the location is known in the Original (true GPS position)
                - orange pin logo : when the location is deducted or user defined
                - red pin logo when : when the location is unknown
        - add mouse/keyboard shortcuts
            - Mouse behavior on the image
                - double-click on the image to toggle the full screen mode
                - single-click on the left zone of the image (1/4) to navigate to the previous image
                - single-click on the right to of the image (1/4) navigate to the next image
                - single-click on the "middle" to navigate to a new random image
            - Support some keyboards to navigate through images :
                - key `Home` : go to the first image
                - key `End` : go to the last one
                - key `PageDown`: go to the next one
                - key `PageUp`: got to the previous one
        - when the image is in full screen mode
            - use a background semi-transparent color for the added text to make it always visible
            - show the event information on the top left of the image
            - show the starred and hasLocation indicators
        - a Click on the has location indicator when location is known whould switch to the world (openstreetmap) zoomable tab
            - with the zoom level set to the location of the photo
        - remember the currently displayed image in the viewer on page reload
    - [x] world (openstreetmap) zoomable tab
        - use zoom-in and zoom-out to see the photo distribution all other the world
        - use a clustered approach as thousands of photos can be displayed
            - filter media that has a known location ignore all others
        - when a photo is selected, popup an information zone with
            - photo information (date, first event name, starred indicator)
            - the photo itself in a reduced format
            - a button to switch back to the visualization zone
    - [ ] events management tab
        - list events using tiles, hide event id, show the timestamp
            - sorted by the timestamp in reverse order (newest first)
            - include media normalized image inside each event tile (use the new API which allow to get media access key from an originalId)
            - when click on a tile, display the event associated originalId/mediaAccessKey in the image viewer
            - add a small image zooming effect when mouse goes hover tiles
                - when leaving a zoomed event, add a quick zoom out effect
            - add before the timestamp the small green location icon when event location is known, use the same icon has for the viewer tab
        - take care to performance as it may requires a lot of queries :

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dacr/sotohp](https://github.com/dacr/sotohp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
