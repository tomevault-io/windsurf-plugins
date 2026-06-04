---
trigger: always_on
description: Writing JavaScript inside `.js` files, or within the `{% javascript %}` or `{% script %}` or <script> </script> tags in `.liquid` files.
---

# JavaScript Standards

## General Principles

- **Zero external dependencies** - Use native browser APIs
- **Avoid mutation** - Use `const` over `let` unless necessary  
- **Use `for (const item of items)`** over `items.forEach()`
- **Add new lines before blocks** with `{` and `}`
- **Use vanilla Web Components** - Extend HTMLElement directly for custom elements

## Async Operations and Request Management

**Always use async/await over .then() chaining:**

```javascript
async renderSection(hasDifferentProductUrl, productUrl) {
  this.abortController?.abort();
  this.abortController = new AbortController();

  try {
    const response = await fetch(`${productUrl}?option_values=${this.selectedOptionValues}&section_id=${this.dataset.section}`, {
      signal: this.abortController.signal,
    });
    
    const responseText = await response.text();
    const html = new DOMParser().parseFromString(responseText, 'text/html');
    const variant = this.getSelectedVariant(html);
    
    if (hasDifferentProductUrl) {
      const productInfo = html.querySelector('product-info');
      this.replaceWith(productInfo);
      productInfo.updateURL(variant?.id);
    } else {
      this.updateMedia(variant?.featured_media?.id);
      this.updateURL(variant?.id);
      this.updateVariantInputs(variant?.id);
      this.updateSourceFromDestination(html, `price-${this.dataset.section}`);
    }
  } catch (error) {
    if (error.name === 'AbortError') {
      console.log('Fetch aborted by user');
    } else {
      console.error(error);
    }
  }
}
```

## Web Components Pattern

**Use vanilla Web Components with HTMLElement:**

```javascript
if (!customElements.get('product-info')) {
  class ProductInfo extends HTMLElement {
    // Property declarations with initialization
    abortController = undefined;
    swiper = null;

    constructor() {
      super();
      // Minimal constructor - defer setup to connectedCallback
    }

    setupEventListeners() {
      this.variantSelector?.addEventListener('change', this.onVariantChange.bind(this));
      this.quantitySelector.addEventListener('change', this.onQuantitySelectorEvent.bind(this));
      this.quantitySelector.querySelector('button[name="plus"]').addEventListener('click', this.onQuantitySelectorEvent.bind(this));
      this.quantitySelector.querySelector('button[name="minus"]').addEventListener('click', this.onQuantitySelectorEvent.bind(this));
      document.getElementById('swiper-script').addEventListener('load', this.initSwiper.bind(this));
      document.addEventListener('liquid-ajax-cart:request-end', this.onCartUpdate.bind(this));
    }

    connectedCallback() {
      this.setupEventListeners();
      if (typeof Swiper !== 'undefined') {
        this.initSwiper();
      }
    }

    disconnectedCallback() {
      this.abortController?.abort();
      this.swiper?.destroy();
    }

    // Getter methods for DOM element access
    get variantSelector() {
      return this.querySelector('variant-selector');
    }

    get quantitySelector() {
      return this.querySelector('quantity-selector');
    }

    // Event handlers with descriptive names
    onVariantChange(e) {
      const hasDifferentProductUrl = e.target?.dataset?.productUrl ? 
        (e.target?.dataset?.productUrl !== this.dataset.url) : false;
      const productUrl = e.target?.dataset?.productUrl || this.dataset.url;
      this.renderSection(hasDifferentProductUrl, productUrl);
    }

    // Public methods for external component communication
    updateMedia(variantFeaturedMediaId) {
      if (!variantFeaturedMediaId) return;
      var index = this.querySelector(`.swiper-slide[data-media-id="${variantFeaturedMediaId}"]`).dataset.mediaIndex;
      this.swiper?.slideTo(index);
    }

    // Arrow function for helper methods
    updateSourceFromDestination = (html, id) => {
      const source = html.getElementById(`${id}`);
      const destination = this.querySelector(`#${id}`);
      if (source && destination) {
        destination.innerHTML = source.innerHTML;
      }
    };
  }

  customElements.define('product-info', ProductInfo);
}
```

**HTML integration with Liquid templates:**

```liquid
<product-info
  data-url="{{ product.url}}"
  data-section="{{ section.id }}"
  class="color-{{ section.settings.color_scheme }} section-{{ section.id }}-padding"
>
  <!-- Component content with nested custom elements -->
  <variant-selector id="variant-selector-{{ section.id }}" data-picker-type="{{ block.settings.picker_type }}">
    <!-- Variant selection UI -->
  </variant-selector>
  
  <quantity-selector>
    <!-- Quantity controls -->
  </quantity-selector>
</product-info>

<!-- Load component script -->
<script src="{{ 'component-product-info.js' | asset_url }}" defer="defer"></script>
```

## Early Returns and Conditional Logic

**Use early returns over nested conditionals:**

```javascript
// Good
const processOrder = (order) => {
  if (!order) return;
  if (!order.items.length) return;
  if (order.status !== 'pending') return;

  // Process the order
  updateOrderStatus(order.id, 'processing');
  sendConfirmationEmail(order.email);
};

// Avoid
const processOrder = (order) => {
  if (order) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EcomExperts-io/Base](https://github.com/EcomExperts-io/Base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
