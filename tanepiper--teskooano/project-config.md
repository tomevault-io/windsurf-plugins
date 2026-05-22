---
trigger: always_on
description: For fine-tuning parameters, use addBinding() of the pane to add components. Tweakpane provides suitable components for bound values.
---

Bindings
For fine-tuning parameters, use addBinding() of the pane to add components. Tweakpane provides suitable components for bound values.

Number

For number parameters, Tweakpane provides a text input by default.

const PARAMS = {
  speed: 0.5,
};

const pane = new Pane();
pane.addBinding(PARAMS, 'speed');

Range
You can specify a range of number by min and max. If you specify both of them, slider control will be created.

const PARAMS = {
  speed: 50,
};

const pane = new Pane();
pane.addBinding(PARAMS, 'speed', {
  min: 0,
  max: 100,
});

Step
step constraints step of changes.

const PARAMS = {
  speed: 0.5,
  count: 10,
};

const pane = new Pane();
pane.addBinding(PARAMS, 'speed', {
  step: 0.1,
});
pane.addBinding(PARAMS, 'count', {
  step: 10,
  min: 0,
  max: 100,
});

Number list
If you want to choose a value from presets, use options.

const PARAMS = {
  quality: 0,
};

const pane = new Pane();
pane.addBinding(PARAMS, 'quality', {
  options: {
    low: 0,
    medium: 50,
    high: 100,
  },
});

Formatter
You can use a custom number formatter with format.

const PARAMS = {
  k: 0,
};

const pane = new Pane();
pane.addBinding(PARAMS, 'k', {
  format: (v) => v.toFixed(6),
});

String
For string parameters, text input will be provided by default.

const PARAMS = {
  message: 'hello, world',
};

const pane = new Pane();
pane.addBinding(PARAMS, 'message');

String list
Same as for number properties, options provides a list component.

const PARAMS = {
  theme: '',
};

const pane = new Pane();
pane.addBinding(PARAMS, 'theme', {
  options: {
    none: '',
    dark: 'dark-theme.json',
    light: 'light-theme.json',
  },
});

Boolean
For boolean parameters, checkbox field component will be provided.

const PARAMS = {
  hidden: true,
};

const pane = new Pane();
pane.addBinding(PARAMS, 'hidden');


Color
For object parameters that have components key r, g, and b (and optional a), text field with a color swatch will be provided. You can choose a color from a color picker by clicking the swatch.

const PARAMS = {
  background: {r: 255, g: 0, b: 55},
  tint: {r: 0, g: 255, b: 214, a: 0.5},
};

const pane = new Pane();
pane.addBinding(PARAMS, 'background');
pane.addBinding(PARAMS, 'tint');

Passing {color: {type: 'float'}} will change the maximum value of color components to 1.0. It may be useful for some cases (e.g. shader colors).

const PARAMS = {
  overlay: {r: 1, g: 0, b: 0.33},
};

const pane = new Pane();
pane.addBinding(PARAMS, 'overlay', {
  color: {type: 'float'},
});

This field will also be provided for string parameters that can be parsed as a color.

const PARAMS = {
  primary: '#f05',
  secondary: 'rgb(0, 255, 214)',
};

const pane = new Pane();
pane.addBinding(PARAMS, 'primary');
pane.addBinding(PARAMS, 'secondary');
)
If you want to regard a hex number (like 0x0088ff) as a color, specify {view: 'color'} option, and {color: {alpha: true}} to add an alpha component.

const PARAMS = {
  background: 0xff0055,
  tint: 0x00ffd644,
};

const pane = new Pane();
pane.addBinding(PARAMS, 'background', {
  view: 'color',
});
pane.addBinding(PARAMS, 'tint', {
  view: 'color',
  color: {alpha: true},
});

Or, if you want to force a color-like string to be a string input, pass view: 'text' option.

const PARAMS = {
  hex: '#0088ff',
};

const pane = new Pane();
pane.addBinding(PARAMS, 'hex', {
  view: 'text',
});

picker can change the layout of the picker.

const PARAMS = {
  key: '#ff0055ff',
};

const pane = new Pane();
pane.addBinding(PARAMS, 'key', {
  picker: 'inline',
  expanded: true,
});

Point 2D
For object parameters that have number properties x and y, text fields and a picker will be provided.

const PARAMS = {
  offset: {x: 50, y: 25},
};

const pane = new Pane();
pane.addBinding(PARAMS, 'offset');

Each dimension can be constrained with step, min and max parameters just like a numeric input.

const PARAMS = {
  offset: {x: 20, y: 30},
};

const pane = new Pane();
pane.addBinding(PARAMS, 'offset', {
  x: {step: 20},
  y: {min: 0, max: 100},
});

{inverted: true} inverts Y-axis.

const PARAMS = {
  offset: {x: 50, y: 50},
};

const pane = new Pane();
pane.addBinding(PARAMS, 'offset', {
  y: {inverted: true},
});

picker can change the layout of the picker.

const PARAMS = {
  offset: {x: 50, y: 50},
};

const pane = new Pane();
pane.addBinding(PARAMS, 'offset', {
  picker: 'inline',
  expanded: true,
});

Point 3D/4D
Tweakpane also has a support for 3D and 4D vector object. You can constrain each axis same as Point 2D.

const PARAMS = {
  source: {x: 0, y: 0, z: 0},
  camera: {x: 0, y: 20, z: -10},
  color: {x: 0, y: 0, z: 0, w: 1},
};

// 3d
const pane = new Pane();
pane.addBinding(PARAMS, 'source');
pane.addBinding(PARAMS, 'camera', {
  y: {step: 10},
  z: {max: 0},
});

// 4d
pane.addBinding(PARAMS, 'color', {
  x: {min: 0, max: 1},
  y: {min: 0, max: 1},
  z: {min: 0, max: 1},
  w: {min: 0, max: 1},
});

To monitor primitive value changes, use addBinding() with the option {readonly: true}.

const pane = new Pane();
pane.addBinding(PARAMS, 'wave', {
  readonly: true,
});

Multiline
multiline option provides a multiline log component. rows can change the display height.

const pane = new Pane();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanepiper/teskooano](https://github.com/tanepiper/teskooano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
