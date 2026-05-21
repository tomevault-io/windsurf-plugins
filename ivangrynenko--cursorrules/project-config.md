---
trigger: always_on
description: This rule provides comprehensive guidance for AI assistants writing Behat tests for Drupal projects using the drevops/behat-steps package. It emphasizes reusing existing traits and steps rather than creating custom implementations. Contains the full STEPS.md reference embedded for easy access.
---


# AI Behat Test Writing Guide for Drupal Projects

## 🎯 Primary Directive

**ALWAYS prioritize using drevops/behat-steps traits and step definitions over writing custom steps.** The drevops/behat-steps package provides comprehensive test coverage for most Drupal testing scenarios.

## 📦 Essential Resources

Before writing ANY Behat test:
1. Check available steps in the [drevops/behat-steps STEPS.md](https://github.com/drevops/behat-steps/blob/main/STEPS.md) file or refer to the embedded reference below
2. Review trait source code in `vendor/drevops/behat-steps/src/` directory
3. Only create custom steps when absolutely necessary (functionality not covered by existing traits)

## 🔧 Setting Up FeatureContext

When creating or modifying FeatureContext.php, include the necessary traits from drevops/behat-steps. The traits are located in `vendor/drevops/behat-steps/src/`:

```php
<?php

namespace DrupalProject\Tests\Behat;

use Drupal\DrupalExtension\Context\DrupalContext;
// Generic traits from vendor/drevops/behat-steps/src/
use DrevOps\BehatSteps\CookieTrait;
use DrevOps\BehatSteps\DateTrait;
use DrevOps\BehatSteps\ElementTrait;
use DrevOps\BehatSteps\FieldTrait;
use DrevOps\BehatSteps\FileDownloadTrait;
use DrevOps\BehatSteps\KeyboardTrait;
use DrevOps\BehatSteps\LinkTrait;
use DrevOps\BehatSteps\PathTrait;
use DrevOps\BehatSteps\ResponseTrait;
use DrevOps\BehatSteps\WaitTrait;

// Drupal-specific traits from vendor/drevops/behat-steps/src/Drupal/
use DrevOps\BehatSteps\Drupal\BigPipeTrait;
use DrevOps\BehatSteps\Drupal\BlockTrait;
use DrevOps\BehatSteps\Drupal\ContentBlockTrait;
use DrevOps\BehatSteps\Drupal\ContentTrait;
use DrevOps\BehatSteps\Drupal\DraggableviewsTrait;
use DrevOps\BehatSteps\Drupal\EckTrait;
use DrevOps\BehatSteps\Drupal\EmailTrait;
use DrevOps\BehatSteps\Drupal\FieldTrait as DrupalFieldTrait;
use DrevOps\BehatSteps\Drupal\FileTrait;
use DrevOps\BehatSteps\Drupal\MediaTrait;
use DrevOps\BehatSteps\Drupal\MenuTrait;
use DrevOps\BehatSteps\Drupal\MetatagTrait;
use DrevOps\BehatSteps\Drupal\OverrideTrait;
use DrevOps\BehatSteps\Drupal\ParagraphsTrait;
use DrevOps\BehatSteps\Drupal\SearchApiTrait;
use DrevOps\BehatSteps\Drupal\TaxonomyTrait;
use DrevOps\BehatSteps\Drupal\TestmodeTrait;
use DrevOps\BehatSteps\Drupal\UserTrait;
use DrevOps\BehatSteps\Drupal\WatchdogTrait;

class FeatureContext extends DrupalContext {
    // Include only the traits you need for your tests
    // Generic traits
    use CookieTrait;
    use DateTrait;
    use ElementTrait;
    use FieldTrait;
    use FileDownloadTrait;
    use KeyboardTrait;
    use LinkTrait;
    use PathTrait;
    use ResponseTrait;
    use WaitTrait;
    
    // Drupal-specific traits
    use BlockTrait;
    use ContentTrait;
    use EmailTrait;
    use FileTrait;
    use MediaTrait;
    use TaxonomyTrait;
    use UserTrait;
    
    // Only add custom methods when drevops/behat-steps doesn't provide the functionality
}
```

## 🚫 When NOT to Create Custom Steps

Before creating ANY custom step, verify that drevops/behat-steps doesn't already provide it. Check the full reference below.

### Common Mistakes to Avoid:

1. **Creating custom user login steps**
   - ❌ Don't create: `Given I log in as an administrator`
   - ✅ Use UserTrait: `Given I am logged in as a user with the "administrator" role`

2. **Creating custom content creation steps**
   - ❌ Don't create: `Given I create an article titled :title`
   - ✅ Use ContentTrait: `Given "article" content:` with a table

3. **Creating custom field interaction steps**
   - ❌ Don't create: `When I fill in the body field with :text`
   - ✅ Use FieldTrait: `When I fill in "Body" with :text`

4. **Creating custom email verification steps**
   - ❌ Don't create: `Then I should receive an email`
   - ✅ Use EmailTrait: `Then an email is sent to :address`

5. **Creating custom element interaction steps**
   - ❌ Don't create: `When I click the submit button`
   - ✅ Use ElementTrait: `When I click on the element ".submit-button"`

## ✅ When to Create Custom Steps

Only create custom steps when:

1. **Business-specific logic** that wouldn't be reusable across projects
2. **Complex multi-step operations** that are repeated frequently in your tests
3. **Integration with third-party services** not covered by drevops/behat-steps
4. **Custom Drupal modules** with unique functionality

Example of a valid custom step:

```php
/**
 * @When I process the payment gateway response for order :order_id
 */
public function iProcessPaymentGatewayResponse($order_id) {
    // Custom implementation for your specific payment gateway
}
```

---

# Complete DrevOps Behat Steps Reference

The following is the complete reference from [drevops/behat-steps STEPS.md](https://github.com/drevops/behat-steps/blob/main/STEPS.md):

## Available steps

### Index of Generic steps

| Class | Description |
| --- | --- |
| [CookieTrait](#cookietrait) | Verify and inspect browser cookies. |
| [DateTrait](#datetrait) | Convert relative date expressions into timestamps or formatted dates. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
