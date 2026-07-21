---
trigger: always_on
description: This document contains detailed explanations, code examples, and implementation guidance for all Inertia Rails best practices.
---

# Inertia Rails Best Practices - Complete Reference

This document contains detailed explanations, code examples, and implementation guidance for all Inertia Rails best practices.

---

## 1. Server-Side Setup & Configuration (CRITICAL)

These rules establish the foundation for all Inertia functionality. Proper setup ensures reliable operation and maintainability.

---

### setup-01: Use the Rails generator for initial setup

**Impact:** CRITICAL - Ensures correct configuration and avoids common setup errors

**Problem:** Manual setup can miss important configuration steps, leading to subtle bugs.

**Solution:** Use the built-in Rails generator for consistent setup:

```bash
# Add the gem
bundle add inertia_rails

# Run the generator
bin/rails generate inertia:install
```

The generator handles:
- Vite Rails detection and installation
- TypeScript configuration
- Frontend framework selection (React, Vue, or Svelte)
- Tailwind CSS integration (optional)
- Example controller and view files
- Application configuration

**When manual setup is needed:**

```erb
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csp_meta_tag %>
    <%= inertia_ssr_head %>
    <%= vite_client_tag %>
    <%= vite_javascript_tag 'application' %>
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

---

### setup-02: Configure asset versioning for cache busting

**Impact:** CRITICAL - Ensures users receive updated assets after deployments

**Problem:** Without version tracking, users may see stale JavaScript after deployments.

**Solution:** Configure version in your initializer:

```ruby
# config/initializers/inertia_rails.rb
InertiaRails.configure do |config|
  # Using ViteRuby digest (recommended)
  config.version = -> { ViteRuby.digest }

  # Or using a custom version string
  # config.version = Rails.application.config.assets_version

  # Or using Git commit hash
  # config.version = -> { `git rev-parse HEAD`.strip }
end
```

**How it works:** When the version changes, Inertia triggers a full page visit instead of an XHR request, ensuring fresh assets are loaded.

---

### setup-03: Set up proper layout inheritance

**Impact:** HIGH - Enables flexible layout management across controllers

**Problem:** All Inertia pages using the same layout limits design flexibility.

**Solution:** Configure default layout and override per-controller:

```ruby
# config/initializers/inertia_rails.rb
InertiaRails.configure do |config|
  config.layout = 'application'  # default
end

# app/controllers/admin/base_controller.rb
class Admin::BaseController < ApplicationController
  layout 'admin'
end

# Or use inertia_config for controller-specific settings
class MarketingController < ApplicationController
  inertia_config(layout: 'marketing')
end
```

---

### setup-04: Configure flash keys appropriately

**Impact:** MEDIUM - Ensures proper flash message delivery to frontend

**Problem:** Custom flash keys may not be passed to the frontend by default.

**Solution:** Configure allowed flash keys:

```ruby
# config/initializers/inertia_rails.rb
InertiaRails.configure do |config|
  # Default: %i[notice alert]
  config.flash_keys = %i[notice alert success error warning info]
end
```

For custom flash data beyond allowlisted keys:

```ruby
# Controller
flash.inertia[:custom_data] = { message: 'Success!', type: 'toast' }

# Or for current request only
flash.inertia.now[:custom_data] = { message: 'Success!' }
```

---

### setup-05: Use environment variables for configuration

**Impact:** MEDIUM - Enables deployment flexibility without code changes

**Problem:** Hardcoded configuration limits deployment options.

**Solution:** Inertia Rails supports `INERTIA_` prefixed environment variables:

```bash
# .env or deployment config
INERTIA_SSR_ENABLED=true
INERTIA_SSR_URL=http://localhost:13714
INERTIA_ENCRYPT_HISTORY=true
INERTIA_DEEP_MERGE_SHARED_DATA=true
```

Boolean values must be exactly `"true"` or `"false"` (case-sensitive).

---

### setup-06: Set up default render behavior thoughtfully

**Impact:** MEDIUM - Reduces boilerplate while maintaining explicitness

**Problem:** Overly implicit rendering can make code harder to understand.

```ruby
# config/initializers/inertia_rails.rb
InertiaRails.configure do |config|
  # Enable convention-based rendering
  config.default_render = true

  # Customize component path resolution
  config.component_path_resolver = ->(path:, action:) do
    "#{path.camelize}/#{action.camelize}"
  end
end
```

**Incorrect - implicit rendering without clear conventions:**
```ruby
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
    # What component gets rendered? Unclear.
  end
end
```

**Correct - explicit rendering:**
```ruby
class UsersController < ApplicationController
  def show
    user = User.find(params[:id])
    render inertia: { user: user.as_json(only: [:id, :name, :email]) }
  end
end
```

---

## 2. Props & Data Management (CRITICAL)

Proper props management is crucial for performance and security. These rules can provide 2-5× performance improvements.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cole-robertson/inertia-rails-skills](https://github.com/cole-robertson/inertia-rails-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
