---
trigger: always_on
description: This document provides comprehensive instructions for AI agents, GitHub Copilot, and automated coding tools when working with WordPress plugin development. Follow these guidelines to ensure code quality, security, accessibility, and adherence to WordPress standards.
---

# WordPress Plugin Development Guidelines for AI Agents

This document provides comprehensive instructions for AI agents, GitHub Copilot, and automated coding tools when working with WordPress plugin development. Follow these guidelines to ensure code quality, security, accessibility, and adherence to WordPress standards.

---

## 1. WordPress Coding Standards

### 1.1 PHP Coding Standards
- Follow [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/)
- Use **Yoda conditions** for comparisons: `if ( true === $condition )`
- Use **tabs for indentation**, not spaces
- Opening braces `{` should be on the same line as the statement
- Use `elseif`, not `else if`
- Space after control structures: `if ( condition ) {`, `foreach ( $array as $item ) {`
- Use single quotes for strings unless variable interpolation or special characters are needed
- Always use full PHP opening tags `<?php`, never short tags `<?`
- Add trailing commas in multi-line arrays for better version control diffs

**Example:**
```php
<?php
if ( true === $is_active ) {
	foreach ( $items as $item ) {
		echo esc_html( $item['name'] );
	}
}
```

### 1.2 JavaScript Coding Standards
- Follow [WordPress JavaScript Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/)
- Use **tabs for indentation**
- Use `const` and `let`, avoid `var`
- Use camelCase for variable and function names
- Use single quotes for strings
- Add semicolons at the end of statements
- Use strict equality operators: `===` and `!==`
- Use jQuery wrapped in IIFE when working with WordPress: `( function( $ ) { ... } )( jQuery );`
- For modern React/Block development, follow ES6+ standards

**Example:**
```javascript
// jQuery example
( function( $ ) {
	'use strict';
	
	const handleClick = function( event ) {
		event.preventDefault();
		const $target = $( event.target );
		// Logic here
	};
	
	$( document ).ready( handleClick );
} )( jQuery );

// JavaScript ES6+ example
const initializeApp = () => {
	const elements = document.querySelectorAll( '.my-element' );
	
	elements.forEach( ( element ) => {
		element.addEventListener( 'click', ( event ) => {
			event.preventDefault();
			const { dataset } = event.currentTarget;
			console.log( dataset.value );
		} );
	} );
};

document.addEventListener( 'DOMContentLoaded', initializeApp );
```

### 1.3 CSS Coding Standards
- Follow [WordPress CSS Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/)
- Use **tabs for indentation**
- Use lowercase for selectors and properties
- Use shorthand properties where possible
- Add space after colons: `property: value;`
- Use single line for single declarations, multiple lines for multiple declarations
- Include vendor prefixes when necessary (use autoprefixer)
- Avoid ID selectors for styling; use classes instead

**Example:**
```css
.wp-block-custom {
	display: flex;
	align-items: center;
	padding: 1rem;
	background-color: #fff;
}
```

### 1.4 HTML Coding Standards
- Follow [WordPress HTML Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/html/)
- Use **tabs for indentation**
- Use lowercase for tags and attributes
- Always quote attribute values with double quotes
- Use semantic HTML5 elements
- Self-closing tags should have a space before `/>`
- Add ARIA attributes for accessibility

---

## 2. WordPress Security Best Practices

### 2.1 Data Sanitization, Validation, and Escaping

**CRITICAL: Always sanitize input and escape output**

#### Input Sanitization
When receiving user input, always sanitize:
- `sanitize_text_field()` - for text inputs
- `sanitize_email()` - for email addresses
- `sanitize_url()` - for URLs
- `sanitize_key()` - for keys/slugs
- `absint()` - for positive integers
- `wp_kses()` or `wp_kses_post()` - for HTML content
- `sanitize_textarea_field()` - for textarea content

**Example:**
```php
<?php
$username = sanitize_text_field( $_POST['username'] );
$email    = sanitize_email( $_POST['email'] );
$post_id  = absint( $_POST['post_id'] );
```

#### Output Escaping
When outputting data, always escape:
- `esc_html()` - for HTML content
- `esc_attr()` - for HTML attributes
- `esc_url()` - for URLs
- `esc_js()` - for JavaScript strings
- `esc_textarea()` - for textarea values
- `wp_kses_post()` - for post content with allowed HTML

**Example:**
```php
<div class="<?php echo esc_attr( $class_name ); ?>">
	<a href="<?php echo esc_url( $link ); ?>">
		<?php echo esc_html( $title ); ?>
	</a>
</div>
```

### 2.2 Nonce Verification
Always use nonces for form submissions and AJAX requests:

**Creating nonces:**
```php
wp_nonce_field( 'my_action_name', 'my_nonce_field' );
```

**Verifying nonces:**
```php
if ( ! isset( $_POST['my_nonce_field'] ) || ! wp_verify_nonce( $_POST['my_nonce_field'], 'my_action_name' ) ) {
	wp_die( 'Security check failed' );
}
```

**For AJAX:**
```javascript
$.ajax({
	url: ajaxurl,
	data: {
		action: 'my_ajax_action',
		nonce: myPlugin.nonce,
	}
});
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theaminulai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
