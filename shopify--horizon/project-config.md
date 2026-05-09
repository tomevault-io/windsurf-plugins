---
trigger: always_on
description: Writing JavaScript inside `.js` files, or within the `{% javascript %}` or `{% script %}` tags in `.liquid` files.
---

# JavaScript Standards

## General Principles

- **Zero external dependencies** - Use native browser APIs
- **Avoid mutation** - Use `const` over `let` unless necessary
- **Use `for (const item of items)`** over `items.forEach()`
- **Add new lines before blocks** with `{` and `}`
- **Use the component framework** - See [the framework code](mdc:assets/component.js) and the [component documentation](mdc:codex/component-framework.md)

## Async/Await Syntax

**Always use async/await over .then() chaining:**

```javascript
const fetchProducts = async () => {
  try {
    const response = await fetch('/products.json');
    const data = await response.json();
    return data.products;
  } catch (error) {
    console.error('Failed to fetch products:', error);
    return [];
  }
};

## Web Components Pattern

**Initialize JavaScript components using the Component framework:**

```javascript
import { Component } from '@theme/component';

/**
 * @typedef {Object} ProductCardRefs
 * @property {HTMLButtonElement} addButton - Add to cart button
 * @property {HTMLElement} priceDisplay - Price display element
 * @property {HTMLImageElement} [productImage] - Optional product image
 */

/**
 * @extends {Component<ProductCardRefs>}
 */
class ProductCard extends Component {
  constructor() {
    super();
    this.cache = new Map();
  }

  connectedCallback() {
    super.connectedCallback();
    this.#initializeCard();
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    this.#cleanup();
  }

  // Public method for external use
  updatePrice(newPrice) {
    if (!this.refs.priceDisplay) return;
    this.refs.priceDisplay.textContent = newPrice;
  }

  // Event handler for add to cart button
  async handleAddToCart(event) {
    event.preventDefault();

    const productId = this.cache.get('productId');
    this.refs.addButton.disabled = true;
    this.refs.addButton.textContent = 'Adding...';

    try {
      await addToCart(productId);
      this.refs.addButton.textContent = 'Added!';

      // Dispatch custom event for cart updates
      this.dispatchEvent(new CustomEvent('cart:item-added', {
        detail: { productId },
        bubbles: true
      }));
    } catch (error) {
      this.refs.addButton.textContent = 'Try again';
      console.error('Add to cart error:', error);
    } finally {
      setTimeout(() => {
        this.refs.addButton.disabled = false;
        this.refs.addButton.textContent = 'Add to cart';
      }, 2000);
    }
  }

  // Private method requiring instance access
  #initializeCard() {
    const productId = this.dataset.productId;
    this.cache.set('productId', productId);
  }

  #cleanup() {
    this.cache.clear();
  }
}

// Module-scoped utility - no instance access needed
const addToCart = async (productId) => {
  const formData = new FormData();
  formData.append('id', productId);
  formData.append('quantity', 1);

  try {
    const response = await fetch('/cart/add.js', {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      throw new Error('Failed to add to cart');
    }

    const cartData = await response.json();
    return cartData;
  } catch (error) {
    console.error('Add to cart error:', error);
    throw error;
  }
};

customElements.define('product-card', ProductCard);
```

**HTML usage with the Component framework:**

```liquid
<product-card data-product-id="{{ product.id }}">
  <img ref="productImage" src="{{ product.featured_image | image_url }}" alt="{{ product.title }}">
  <h3>{{ product.title }}</h3>
  <div ref="priceDisplay" class="product-card__price">{{ product.price | money }}</div>
  <button ref="addButton" on:click="/handleAddToCart" data-add-to-cart>
    Add to cart
  </button>
</product-card>
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
    if (order.items.length) {
      if (order.status === 'pending') {
        updateOrderStatus(order.id, 'processing');
        sendConfirmationEmail(order.email);
      }
    }
  }
};
```

**Optional chaining guidelines:**

```javascript
// Multiple chains - use early return
const updateButton = (product) => {
  const button = product.querySelector('[data-ref="button"]');
  if (!button) return;

  button.disabled = false;
  button.textContent = 'Add to cart';
};

// Single chain is fine
const updateButton = (product) => {
  const button = product.querySelector('[data-ref="button"]');
  button?.enable();
};
```

## Simplification Patterns

**Ternary operators for simple conditions:**
```javascript
const buttonText = isLoading ? 'Loading...' : 'Add to cart';
element.textContent = buttonText;
```

**One-liner conditionals:**
```javascript
if (isOutOfStock) return;
```

**Return boolean comparisons directly:**
```javascript
const isAvailable = product.available && product.price > 0;
return isAvailable;
```

## Event-Driven Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/horizon](https://github.com/Shopify/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
