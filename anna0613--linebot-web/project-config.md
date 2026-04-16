---
trigger: always_on
description: LINE DevelopersLINE Developers
---

LINE DevelopersLINE Developers
Developers
About
News
Products
Documentation
FAQ
Glossary

More



Log in to Console
Home Documentation Messaging API Flex Message layout
API reference
Development guidelines
Quickstart
Guides
Webhooks
Rich menus
LINE Bot Designer
Flex Message
Send Flex Messages
Flex Message elements
Flex Message layout
Create a Flex Message including a video
Tutorial - Using Flex Message Simulator
Flex Message Simulator
(opens new window)
Concepts
Reference
Resources
#Flex Message layout
You can build complex Flex Messages layouts based on the specification for CSS Flexible Box (CSS flexbox) (opens new window). The flex container and flex items in CSS flexbox correspond to the box component and the Flex Message components, respectively.

Learn how to compose a layout of your Flex Message. For more information about the JSON schema, see Flex Message in the Messaging API reference.

#Box component orientation
Box components have two orientations, landscape and portrait. Boxes in landscape mode are called horizontal boxes and those in portrait mode are called vertical boxes. The orientation determines the main axis and cross axis of a box. The main axis is parallel to the orientation. A horizontal box's main axis is horizontal and a vertical box's main axis is vertical. The cross axis is perpendicular to the main axis. The main axis determines how a box's child components are arranged. For more information, see Child component arrangement with free space.

You need to specify the orientation in the layout property of the box component. In addition to horizontal boxes and vertical boxes, baseline boxes are available.

Box	layout property	Main axis	Cross axis	Child components placed
Horizontal box	horizontal	Horizontal	Vertical	Horizontally
Vertical box	vertical	Vertical	Horizontal	Vertically
Baseline box	baseline	Horizontal	Vertical	Horizontally
For more information, see Child components in baseline boxes.
#Child components in baseline boxes
Baseline boxes behave in the same way as horizontal boxes. However, the behavior of baseline boxes differs from that of horizontal boxes in the following ways:

#Vertical alignment position
Components in a baseline box are vertically aligned by the same baseline. This means all the child components use the same baseline, regardless of the font size. The baseline of an icon component is the bottom of the icon image.



#Unavailable properties
You can't use the properties gravity and offsetBottom in child components of a baseline box component.

#Available child components
A box's layout property determines which components you can use as child components of a box component:

Baseline box	Horizontal box
Vertical box
Box	❌	✅
Button	❌	✅
Image	❌	✅
Icon	✅	❌
Text	✅	✅
Span
(Using as a child of a text component is okay)	❌	❌
Separator	❌	✅
Filler (deprecated)	✅	✅
✅: You can use this component in the box ❌: You can't use the component in the box

#Component size
If a component's position property is set to relative, the component's width and height are determined by the flex property of the component.

Width allocation in a horizontal box
Height allocation in a vertical box
Box width
Maximum width of a box
Box height
Maximum height of a box
Image size
Icon, text, and span size
Size of other components
Automatically shrink fonts to fit
Scaling to size according to the font size setting
#Width allocation in a horizontal box
Child components in a horizontal box that have the flex property set to 1 or greater share the parent box's width with sibling components. The default value of the flex property is 1. The proportion of width each child component gets is determined by the value of the child component's flex property against the sum of the values of the flex properties.

Suppose a horizontal box has two components, one with the flex property set to 2 and the other to 3. Then the available width (the width of the horizontal box) is divided at a ratio of two to three and allocated to each component.



The JSON definition of this Flex Message example is as follows.

{
  "type": "bubble",
  "body": {
    "type": "box",
    "layout": "horizontal",
    "contents": [
      {
        "type": "text",
        "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
        "wrap": true,
        "color": "#ff0000",
        "flex": 2
      },
      {
        "type": "text",
        "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
        "wrap": true,
        "color": "#0000ff",
        "flex": 3
      }
    ]
  }
}
If a component's flex property is 0, the component occupies the width required to show all of the component's content, within the box's width. However, the part that goes outside the box's width isn't displayed.

For example, suppose we have a horizontal box with three child components, each with its flex property set to 0, 2, and 3, respectively. The first component has the flex property set to 0, so the component takes up the width to fit its text, "Hello". Then the rest of the available width is shared between the remaining two components, in the ratio of 2 to 3, as illustrated below.




<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Anna0613) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
