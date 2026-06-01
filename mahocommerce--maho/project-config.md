---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Maho is an open-source ecommerce platform forked from OpenMage, designed for medium-to-small on-premise projects. It's based on the Magento 1 architecture but modernized with PHP 8.3+ support and contemporary development tools.

## Essential Commands

```bash
composer lint                      # Run all linters (cs-fixer, rector, phpstan)
composer lint:cs-fixer             # Code style only (dry-run)
composer lint:rector               # Rector only (dry-run)
composer lint:phpstan              # PHPStan only (level 6)
vendor/bin/php-cs-fixer fix        # Apply code style fixes (writes changes)
vendor/bin/rector -c .rector.php   # Apply rector fixes (writes changes)
./maho cache:flush                 # Flush all caches
composer test                        # Run all tests (Install → Backend → Frontend)
composer test -- --testsuite=Frontend # Run frontend tests only
composer test -- --testsuite=Backend  # Run backend tests only
composer test -- --testsuite=Install  # Run install tests only
./maho index:reindex:all           # Reindex all indexes
./maho db:query "QUERY"            # Execute a one-shot SQL query
```

## Architecture Overview

### Bootstrapping
```php
require 'vendor/autoload.php';
Mage::app();
```

### MVC Pattern
- **Models** (`Model/`): Business logic and data access
- **Views** (`Block/` and templates): Presentation layer
- **Controllers** (`controllers/`): Request handling

### Module Structure
```
app/code/core/Mage/[ModuleName]/
├── Block/          # View blocks
├── Helper/         # Helper classes
├── Model/          # Business logic
├── controllers/    # Controllers
├── etc/            # Configuration (config.xml, system.xml)
├── sql/            # Database migrations
└── data/           # Data install scripts
```

### Key Configuration Files
- `app/etc/local.xml`: Main configuration (DB, cache, etc.)
- `app/etc/config.xml`: Base configuration
- `app/etc/modules/*.xml`: Module declarations

### Theme Structure
```
app/design/
├── adminhtml/      # Admin panel themes
├── frontend/       # Frontend themes
└── install/        # Installer theme
```

### Database Access (Doctrine DBAL 4.4)
Replaces all Zend_Db components. Adapter: `Maho\Db\Adapter\AdapterInterface`. Query builder: `Maho\Db\Select` (wraps Doctrine QueryBuilder).

```php
$adapter = Mage::getSingleton('core/resource')->getConnection('core_read');
$select = $adapter->select()
    ->from(['p' => 'catalog_product'], ['entity_id', 'sku'])
    ->where('status = ?', 1)
    ->order('created_at DESC');

// Raw SQL expressions
$select->columns(['total' => new Maho\Db\Expr('COUNT(*)')]);

// Direct queries
$result = $adapter->fetchAll($select);
$adapter->insert('table_name', ['column' => 'value']);
$adapter->update('table_name', ['column' => 'new_value'], 'id = 1');
$adapter->delete('table_name', 'id = 1');
```

### Other Key Systems
- **Events**: `Mage::dispatchEvent('event_name', ['data' => $data])` - Observers defined via PHP attributes (see below)
- **Layout**: XML-based configuration with block hierarchy and template assignment
- **Sessions**: `Mage::getSingleton('customer/session')`, `'admin/session'`, `'checkout/session'`
- **Translations**: CSV files in `app/locale/[locale]/` - Use `$this->__('Text')` in code
- **Collections**: `Mage::getResourceModel('catalog/product_collection')->addAttributeToSelect('*')->addFieldToFilter('status', 1)`
- **Errors**: `Mage::throwException()` for user-facing errors, `Mage::log()` for logging

### Observers and Cron Jobs (PHP Attributes)
Observers and cron jobs are defined via `#[Maho\Config\Observer]` and `#[Maho\Config\CronJob]` PHP attributes on methods — **not** in XML. Run `composer dump-autoload` after any changes. See the attribute class docblocks in `lib/Maho/Config/` for all parameters.

```php
#[Maho\Config\Observer('catalog_product_save_after')]
public function handleEvent(\Maho\Event\Observer $observer) {}

#[Maho\Config\Observer('event_name', area: 'frontend')]
public function handleFrontendEvent(\Maho\Event\Observer $observer) {}

#[Maho\Config\CronJob('my_cron_job', schedule: '0 2 * * *')]
public function runJob(Mage_Cron_Model_Schedule $schedule) {}
```

- Prefer global area (default, omit `area:`) unless the observer must be restricted to a specific area
- Do **not** define observers or cron jobs in `config.xml`

### Routing (#[Route] attributes)
Routes are defined via the `#[Maho\Config\Route]` attribute (`lib/Maho/Config/Route.php`) on controller action methods — **not** in `<frontend><routers>` XML. The attribute is repeatable: stack multiple attributes on the same method for multiple paths or method lists. Run `composer dump-autoload` after any change; routes compile to `vendor/composer/maho_url_matcher.php`, `maho_url_generator.php`, and `maho_attributes.php`.

Parameters:
- `path` (required): URL pattern, e.g. `/catalog/product/view/{id}`
- `name`: route name for URL generation — auto-derived from `class::method` if omitted
- `methods`: HTTP method allow-list (e.g. `['GET', 'POST']`); empty = any
- `defaults`: default parameter values

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MahoCommerce/maho](https://github.com/MahoCommerce/maho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
