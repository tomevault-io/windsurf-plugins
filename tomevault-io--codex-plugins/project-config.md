---
trigger: always_on
description: - **PEP8 & Odoo Standards:**
---


### 1\. Code Style & Formatting

- **PEP8 & Odoo Standards:**
  - Enforce PEP8 for all Python code.
  - Adopt Odoo's recommended code styles for module structures, naming conventions, and documentation.
- **Multi-language Formatting:**
  - Apply consistent formatting for XML, QWeb templates, and CSS to align with Odoo's UI guidelines.

---

### 2\. Module Structure & Best Practices

- **Mandatory Files & Structure:**
  - Ensure every module includes essential files (e.g., `__manifest__.py`, `__init__.py`, subdirectories for models, views, controllers).
- **Separation of Concerns:**
  - Keep business logic, data models, views, and controllers distinct.
  - Validate module dependencies and use Odoo's ORM patterns effectively.
- **Template & Scaffolding:**
  - Include commands to scaffold new modules using customizable templates that reflect Odoo 18's best practices.

---

### 3\. Linting & Static Analysis

- **Customized Linters:**
  - Integrate Python linters (like flake8 and pylint) with rules specifically tuned for Odoo projects.
- **Static Code Checks:**
  - Automatically flag deviations from Odoo's module guidelines and coding patterns.

---

### 4\. Testing & Debugging

- **Integrated Testing Frameworks:**
  - Support running unit and integration tests via Odoo's built-in testing framework.
  - Automatically trigger tests on file changes or pre-commit actions.
- **Robust Debugging Tools:**
  - Configure breakpoints, step-through debugging, and live log monitoring tailored for Odoo server mode.
  - Enable hot-reloading and module-specific debugging for rapid iteration.

---

### 5\. Auto-completion & IntelliSense

- **Odoo-Aware Suggestions:**
  - Enhance auto-completion for Odoo-specific API calls, models, and ORM methods.
  - Utilize dynamic code analysis to recommend improvements based on Odoo development patterns.
- **Smart Imports:**
  - Automatically suggest and manage imports for frequently used Odoo libraries and modules.

---

### 6\. Deployment & Version Control Integration

- **Git Integration:**
  - Integrate seamlessly with Git for versioning, branching, and merge conflict resolution.
  - Set up automated pre-commit hooks to run tests, linting, and module integrity checks.
- **CI/CD Pipelines:**
  - Provide support for continuous integration pipelines tailored to Odoo module deployment and server restarts.

---

### 7\. Security & Error Handling

- **Best Practice Enforcement:**
  - Implement checks to ensure safe coding practices in user input, ORM queries, and API interactions.
  - Integrate guidelines for comprehensive error handling, logging, and exception management.
- **Static Security Analysis:**
  - Run static analysis to detect potential security vulnerabilities common in Odoo development.

---

### 8\. Custom Cursor Commands for Odoo Development

- **Module Management Commands:**
  - Create custom commands to scaffold new modules, update module lists, and manage server configurations.
- **Database & Migration Tools:**
  - Incorporate one-click tools for database migration management, upgrade scripts, and seamless Odoo server restarts.

---

### 9\. Documentation & Code Comments

- **Inline Documentation:**
  - Enforce comprehensive inline comments and module-level docstrings that follow Odoo documentation standards.
- **Auto-Documentation Tools:**
  - Utilize tools to auto-generate documentation for models, fields, and business logic with direct links to official Odoo documentation.

---

### 10\. Environment Management & Odoo-Specific Configurations

- **Multi-Environment Support:**
  - Manage different development environments (local, staging, production) with clear configuration profiles.
- **Dynamic Settings:**
  - Auto-detect and adjust to changes in Odoo 18 settings (e.g., database connections, port settings, logging levels).
- **Real-Time Collaboration:**

  - Optionally enable real-time collaboration features to facilitate team coding, code reviews, and shared debugging sessions.

  # Odoo 18 Technical Guide

## 1. XML View Changes

### 1.1 Tree to List Tag Change

The `<tree>` tag has been renamed to `<list>` in all views.

**Before:**

```xml
<tree>
    <field name="name"/>
</tree>
```

**After:**

```xml
<list>
    <field name="name"/>
</list>
```

### 1.2 Simplified Conditional Attributes

Odoo 18 simplifies the use of conditional attributes by replacing `attrs` and `states` with direct attributes.

**Single Condition:**

```xml
<!-- Before -->
<field name="field_name" attrs="{'invisible': [('condition_field', '=', False)]}"/>

<!-- After -->
<field name="field_name" invisible="not condition_field"/>
```

**Multiple Conditions with OR:**

```xml
<!-- Before -->
<field name="field_name" attrs="{'invisible': ['|', ('state', '=', 'done'), ('type', '=', 'internal')]}"/>

<!-- After -->
<field name="field_name" invisible="state == 'done' or type == 'internal'"/>
```

**Multiple Conditions with AND:**

```xml
<!-- Before -->
<field name="field_name" attrs="{'readonly': [('state', '=', 'approved'), ('user_id', '!=', user.id)]}"/>

<!-- After -->
<field name="field_name" readonly="state == 'approved' and user_id != user.id"/>
```

### 1.3 Button States

Button states attribute has been replaced with invisible conditions:

```xml
<!-- Before -->
<button string="Submit" states="draft"/>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/codex-plugins](https://github.com/tomevault-io/codex-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
