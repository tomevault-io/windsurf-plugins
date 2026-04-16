---
trigger: always_on
description: This is a WordPress plugin for business listings with Gutenberg blocks, WhatsApp integration, multisite sub-site creation, and advanced search functionality.
---

# Happy Business Listing Plugin - Cursor Rules

## Project Overview
This is a WordPress plugin for business listings with Gutenberg blocks, WhatsApp integration, multisite sub-site creation, and advanced search functionality.

## Core Architecture
- **Main Plugin File**: `happy-business-listing.php` - Entry point and initialization
- **Includes Directory**: Core functionality organized by feature
- **Assets Directory**: Frontend CSS, JS, and images
- **Templates Directory**: Custom WordPress templates
- **Tests Directory**: Unit, integration, and e2e tests

## Coding Standards

### PHP Standards
- Follow WordPress Coding Standards (WPCS)
- Use WordPress naming conventions (snake_case for functions, camelCase for variables)
- Prefix all functions with `hbl_` to avoid conflicts
- Use proper WordPress hooks and filters
- Implement proper nonce verification and capability checks
- Use `wp_kses()` for sanitizing HTML output
- Use `esc_attr()`, `esc_html()`, `esc_url()` for escaping

### JavaScript Standards
- Use ES6+ features when possible
- Follow WordPress JavaScript standards
- Use `wp_localize_script()` for PHP-JS data passing
- Implement proper error handling and user feedback
- Use WordPress REST API for AJAX requests

### CSS Standards
- Use BEM methodology for class naming
- Follow WordPress CSS standards
- Implement responsive design principles
- Use CSS custom properties for theming
- Ensure accessibility compliance (WCAG 2.1)

## File Organization

### Includes Directory Structure
```
includes/
├── helpers.php              # Utility functions
├── custom-post-types.php    # Post type registration
├── acf-fields.php          # ACF field definitions
├── user-registration.php   # User management
├── site-creation.php       # Multisite functionality
├── whatsapp-integration.php # WhatsApp API integration
├── form-shortcode.php      # Form shortcodes
├── settings.php            # Admin settings
├── templates.php           # Template functions
├── search-and-filters.php  # Search functionality
├── gutenberg-blocks.php    # Gutenberg blocks
├── permalinks.php          # URL structure
└── security.php            # Security measures
```

### Assets Directory Structure
```
assets/
├── css/
│   ├── blocks-editor.css   # Gutenberg editor styles
│   ├── blocks-style.css    # Frontend block styles
│   ├── forms.css          # Form styles
│   └── templates.css      # Template styles
├── js/
│   ├── blocks.js          # Gutenberg block logic
│   ├── blocks-editor.js   # Editor-specific JS
│   ├── forms.js           # Form handling
│   └── live-search.js     # AJAX search
└── img/                   # Images and icons
```

## Development Guidelines

### Security First
- Always validate and sanitize user input
- Use WordPress nonces for form submissions
- Implement proper capability checks
- Log security events when enabled
- Use prepared statements for database queries
- Implement rate limiting for API calls

### Error Handling
- Use `WP_Error` for error responses
- Implement proper logging with `hbl_log_security_event()`
- Provide user-friendly error messages
- Handle edge cases gracefully
- Use try-catch blocks for external API calls

### Performance
- Use WordPress transients for caching
- Implement lazy loading for images
- Optimize database queries
- Use `wp_enqueue_script()` properly
- Minimize HTTP requests

### Accessibility
- Use semantic HTML elements
- Implement proper ARIA labels
- Ensure keyboard navigation
- Provide alt text for images
- Test with screen readers

## Feature Implementation Patterns

### Custom Post Types
- Register in `custom-post-types.php`
- Use proper labels and capabilities
- Implement custom meta boxes
- Add proper rewrite rules

### Gutenberg Blocks
- Register in `gutenberg-blocks.php`
- Use `register_block_type()` with render callback
- Implement proper attributes
- Provide editor and frontend styles
- Use `wp_localize_script()` for data

### WhatsApp Integration
- Support both Twilio and WhatsApp Business API
- Implement rate limiting
- Use proper error handling
- Log all message attempts
- Validate phone numbers

### Multisite Features
- Check `is_multisite()` before operations
- Use `switch_to_blog()` and `restore_current_blog()`
- Handle site creation errors gracefully
- Implement proper user role assignment

## Testing Guidelines

### Unit Tests
- Test individual functions in isolation
- Mock WordPress functions when needed
- Use `WP_UnitTestCase` as base class
- Test both success and failure scenarios
- Cover edge cases and error conditions

### Integration Tests
- Test complete workflows
- Test database interactions
- Test WordPress hooks and filters
- Test user interactions
- Test API integrations

### E2E Tests
- Test complete user journeys
- Test frontend functionality
- Test admin workflows
- Test responsive design
- Test accessibility features

## Documentation Standards

### Code Documentation
- Use PHPDoc blocks for all functions
- Document parameters and return values
- Include usage examples
- Document hooks and filters
- Explain complex logic

### User Documentation
- Maintain comprehensive README
- Include installation instructions
- Provide usage examples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HappyPress) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
