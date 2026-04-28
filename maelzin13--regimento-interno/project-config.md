---
trigger: always_on
description: TITLE: Implementing Individual Test Cases with Descriptive Names
---

TITLE: Implementing Individual Test Cases with Descriptive Names
DESCRIPTION: Demonstrates how to create individual test cases with descriptive names that clearly state the expected behavior being tested.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/core/testing/best-practices.md#2025-04-16_snippet_3

LANGUAGE: jsx
CODE:
```
 // src/components/button/test/basic/button.e2e.ts

import { configs, test } from '@utils/test/playwright';

configs().forEach(({ title }) => {
  test.describe(title('button: disabled state'), () => {
    test('should not have any visual regressions', async ({ page }) => {
      ...
    });
  });
});
```

----------------------------------------

TITLE: Implementing Accessible Checkbox with VoiceOver Support in TSX
DESCRIPTION: TSX implementation for a checkbox that works properly with VoiceOver. Sets required ARIA attributes on both host and input elements for proper accessibility.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_18

LANGUAGE: tsx
CODE:
```
render() {
  const { checked, disabled } = this;

  return (
    <Host
      aria-checked={`${checked}`}
      aria-hidden={disabled ? 'true' : null}
      role="checkbox"
    >
      <input
        type="checkbox"
      />
      ...
    </Host>
  );
}
```

----------------------------------------

TITLE: Defining the Ionic Searchbar API
DESCRIPTION: Comprehensive documentation of the ion-searchbar component API including properties, methods, events, and CSS custom properties. The searchbar provides functionality for user input with features like autocomplete, debounce, and clear/cancel buttons.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/core/api.txt#2025-04-16_snippet_33

LANGUAGE: typescript
CODE:
```
ion-searchbar,prop,autocomplete,"name" | "email" | "tel" | "url" | "on" | "off" | "honorific-prefix" | "given-name" | "additional-name" | "family-name" | "honorific-suffix" | "nickname" | "username" | "new-password" | "current-password" | "one-time-code" | "organization-title" | "organization" | "street-address" | "address-line1" | "address-line2" | "address-line3" | "address-level4" | "address-level3" | "address-level2" | "address-level1" | "country" | "country-name" | "postal-code" | "cc-name" | "cc-given-name" | "cc-additional-name" | "cc-family-name" | "cc-number" | "cc-exp" | "cc-exp-month" | "cc-exp-year" | "cc-csc" | "cc-type" | "transaction-currency" | "transaction-amount" | "language" | "bday" | "bday-day" | "bday-month" | "bday-year" | "sex" | "tel-country-code" | "tel-national" | "tel-area-code" | "tel-local" | "tel-extension" | "impp" | "photo",'off',false,false
```

----------------------------------------

TITLE: CSS Differences: Scoped vs Shadow DOM in Ionic Components
DESCRIPTION: This snippet illustrates the CSS differences when converting from scoped to shadow DOM. It covers targeting host elements, slotted children, and host-context scenarios.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_26

LANGUAGE: css
CODE:
```
/* IN SCOPED */
:host(.ion-color)::slotted(ion-segment-button)

/* IN SHADOW*/
:host(.ion-color) ::slotted(ion-segment-button)
```

LANGUAGE: css
CODE:
```
/* IN SCOPED */
:host-context(ion-toolbar.ion-color):not(.ion-color) {

/* IN SHADOW */
:host-context(ion-toolbar.ion-color):host(:not(.ion-color))  {
```

LANGUAGE: css
CODE:
```
/* IN SCOPED */
:host-context(ion-toolbar:not(.ion-color)):not(.ion-color)::slotted(ion-segment-button) {

/* IN SHADOW*/
:host-context(ion-toolbar:not(.ion-color)):host(:not(.ion-color)) ::slotted(ion-segment-button) {
```

----------------------------------------

TITLE: Implementing Accessible Labels for Form Controls in TSX
DESCRIPTION: TSX implementation showing how to properly handle labels for form controls, inheriting ARIA attributes from host elements for better accessibility.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/docs/component-guide.md#2025-04-16_snippet_20

LANGUAGE: tsx
CODE:
```
import { Prop } from '@stencil/core';
import { inheritAttributes } from '@utils/helpers';
import type { Attributes } from '@utils/helpers';

...

private inheritedAttributes: Attributes = {};

@Prop() labelText?: string;

componentWillLoad() {
  this.inheritedAttributes = inheritAttributes(this.el, ['aria-label']);
}

render() {
  return (
    <Host>
      <label>
        {this.labelText}
        <input type="checkbox" {...this.inheritedAttributes} />
      </label>
    </Host>
  )
}
```

----------------------------------------

TITLE: Initializing Ionic React Project with Capacitor
DESCRIPTION: Commands to initialize a new Ionic React project and enable Capacitor integration. These commands set up the basic project structure and prepare it for native app development.
SOURCE: https://github.com/ionic-team/ionic-framework/blob/main/packages/react/README.md#2025-04-16_snippet_0

LANGUAGE: sh
CODE:
```
ionic init "My React App" --type=react
ionic integrations enable capacitor
```

----------------------------------------

TITLE: Using Ionic Overlay Controllers with Custom Elements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Maelzin13) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
