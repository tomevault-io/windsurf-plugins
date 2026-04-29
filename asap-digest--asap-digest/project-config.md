---
trigger: always_on
description: This protocol establishes standardized patterns for implementing AJAX handlers in WordPress plugins. Following these standards ensures security, maintainability, consistency, and proper error handling across all AJAX interactions. Inconsistent AJAX implementations can lead to security vulnerabilities, debugging difficulties, and code fragmentation.
---

# WordPress AJAX Handler Standardization Protocol v1.0

## 1. Purpose and Importance

This protocol establishes standardized patterns for implementing AJAX handlers in WordPress plugins. Following these standards ensures security, maintainability, consistency, and proper error handling across all AJAX interactions. Inconsistent AJAX implementations can lead to security vulnerabilities, debugging difficulties, and code fragmentation.

## 2. Core AJAX Handler Structure

### 2.1 Handler Registration

```php
/**
 * Register all AJAX handlers for the plugin.
 *
 * @since 1.0.0
 * @return void
 */
function your_plugin_register_ajax_handlers() {
    // Admin-only AJAX handlers
    add_action('wp_ajax_your_plugin_action_name', 'your_plugin_handle_action_name');
    
    // AJAX handlers available to non-logged-in users (use sparingly and with caution)
    add_action('wp_ajax_nopriv_your_plugin_public_action', 'your_plugin_handle_public_action');
    
    // Handlers available to both logged-in and non-logged-in users
    add_action('wp_ajax_your_plugin_public_action', 'your_plugin_handle_public_action');
}
add_action('init', 'your_plugin_register_ajax_handlers');
```

### 2.2 Handler Implementation

```php
/**
 * Handle the your_plugin_action_name AJAX request.
 *
 * @since 1.0.0
 * @return void Dies with JSON response.
 */
function your_plugin_handle_action_name() {
    // 1. Security check: Verify user capabilities
    if (!current_user_can('required_capability')) {
        wp_send_json_error([
            'message' => __('You do not have permission to perform this action.', 'your-plugin'),
            'code'    => 'insufficient_permissions'
        ], 403);
    }
    
    // 2. Security check: Verify nonce
    if (!isset($_POST['nonce']) || !wp_verify_nonce($_POST['nonce'], 'your_plugin_action_nonce')) {
        wp_send_json_error([
            'message' => __('Security verification failed.', 'your-plugin'),
            'code'    => 'invalid_nonce'
        ], 400);
    }
    
    // 3. Parameter validation
    $required_params = ['param1', 'param2'];
    foreach ($required_params as $param) {
        if (!isset($_POST[$param])) {
            wp_send_json_error([
                'message' => sprintf(__('Missing required parameter: %s', 'your-plugin'), $param),
                'code'    => 'missing_parameter'
            ], 400);
        }
    }
    
    // 4. Sanitize input data
    $param1 = sanitize_text_field($_POST['param1']);
    $param2 = absint($_POST['param2']);
    
    // 5. Process the request
    try {
        $result = your_plugin_process_action($param1, $param2);
        
        // 6. Return success response
        wp_send_json_success([
            'message' => __('Action completed successfully.', 'your-plugin'),
            'data'    => $result
        ]);
    } catch (Exception $e) {
        // 7. Handle and log errors
        error_log('AJAX Error: ' . $e->getMessage());
        
        wp_send_json_error([
            'message' => __('An error occurred while processing your request.', 'your-plugin'),
            'code'    => 'processing_error',
            'details' => WP_DEBUG ? $e->getMessage() : null
        ], 500);
    }
}
```

## 3. Frontend AJAX Implementation

### 3.1 Enqueuing Scripts

```php
/**
 * Enqueue scripts and localize AJAX data.
 *
 * @since 1.0.0
 * @return void
 */
function your_plugin_enqueue_scripts() {
    wp_enqueue_script(
        'your-plugin-ajax',
        YOUR_PLUGIN_URL . 'assets/js/ajax.js',
        ['jquery'],
        YOUR_PLUGIN_VERSION,
        true
    );
    
    // Pass AJAX URL and nonces to JavaScript
    wp_localize_script('your-plugin-ajax', 'yourPluginAjax', [
        'ajaxUrl'   => admin_url('admin-ajax.php'),
        'nonces'    => [
            'actionName' => wp_create_nonce('your_plugin_action_nonce'),
            'otherAction' => wp_create_nonce('your_plugin_other_action_nonce')
        ],
        'i18n'      => [
            'error' => __('An error occurred.', 'your-plugin'),
            'success' => __('Action completed successfully.', 'your-plugin')
        ]
    ]);
}
add_action('wp_enqueue_scripts', 'your_plugin_enqueue_scripts');
```

### 3.2 JavaScript AJAX Implementation

```javascript
/**
 * Handle AJAX requests in JavaScript.
 */
(function($) {
    'use strict';
    
    /**
     * Perform an AJAX action.
     *
     * @param {string} action The AJAX action name without prefix.
     * @param {Object} data The data to send with the request.
     * @param {Function} callback Callback function on success.
     * @param {Function} errorCallback Callback function on error.
     */
    function doAjaxAction(action, data, callback, errorCallback) {
        // Ensure nonce is included if available
        const nonceName = action + 'Name'; // Match the nonce name pattern
        const nonce = yourPluginAjax.nonces[nonceName];
        
        // Prepare request data
        const requestData = {
            action: 'your_plugin_' + action,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ASAP-Digest) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
