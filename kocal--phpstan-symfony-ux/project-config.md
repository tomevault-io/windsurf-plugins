---
trigger: always_on
description: Reference guide for AI agents working on this PHPStan rules project for Symfony UX.
---

# AI Agent Instructions - PHPStan Symfony UX

Reference guide for AI agents working on this PHPStan rules project for Symfony UX.

## Project Overview

This project provides custom PHPStan rules to improve static analysis of Symfony UX applications, specifically for:

- **TwigComponent**: Twig components (classes annotated with `#[AsTwigComponent]`)
- **LiveComponent**: Live components (classes annotated with `#[AsLiveComponent]`)

> [!IMPORTANT]
> TwigComponent rules also apply to LiveComponents since a LiveComponent is an extension of a TwigComponent. Use `AttributeFinder::findAnyAttribute()` to target both attributes.

## Project Structure

```
src/
├── NodeAnalyzer/
│   └── AttributeFinder.php          # Utility to find PHP attributes
└── Rules/
    ├── TwigComponent/                # Rules for TwigComponent (also apply to LiveComponents)
    │   ├── ClassMustBeFinalRule.php
    │   ├── ForbiddenAttributesPropertyRule.php
    │   └── ...
    └── LiveComponent/                # LiveComponent-specific rules
        ├── LivePropHydrationMethodsRule.php
        └── ...

tests/
└── Rules/
    ├── TwigComponent/
    │   └── ClassMustBeFinalRule/
    │       ├── ClassMustBeFinalRuleTest.php
    │       ├── config/
    │       │   └── configured_rule.neon
    │       └── Fixture/
    │           ├── InvalidNonFinalTwigComponent.php
    │           ├── ValidTwigComponent.php
    │           └── NotAComponent.php
    └── LiveComponent/
        └── ...
```

## Available Commands

| Command | Description |
|---------|-------------|
| `symfony composer qa-fix` | Runs cs-fix + phpstan + tests (run before each commit) |
| `symfony composer phpstan` | PHPStan analysis of the project |
| `symfony composer test` | Runs PHPUnit tests |
| `symfony composer cs` | Checks code style |
| `symfony composer cs-fix` | Automatically fixes code style |

## Creating a New Rule

### Step 1: Create the Rule Class

Location: `src/Rules/<Package>/<RuleName>Rule.php`

```php
<?php

declare(strict_types=1);

namespace Kocal\PHPStanSymfonyUX\Rules\TwigComponent;

use Kocal\PHPStanSymfonyUX\NodeAnalyzer\AttributeFinder;
use PhpParser\Node;
use PhpParser\Node\Stmt\Class_;
use PHPStan\Analyser\Scope;
use PHPStan\Rules\Rule;
use PHPStan\Rules\RuleErrorBuilder;
use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\TwigComponent\Attribute\AsTwigComponent;

/**
 * @implements Rule<Class_>
 */
final class ExampleRule implements Rule
{
    public function getNodeType(): string
    {
        return Class_::class;
    }

    public function processNode(Node $node, Scope $scope): array
    {
        // 1. Check for attribute presence (early return)
        if (! AttributeFinder::findAnyAttribute($node, [AsTwigComponent::class, AsLiveComponent::class])) {
            return [];
        }

        // 2. Validation logic
        if ($violationDetected) {
            return [
                RuleErrorBuilder::message('Clear description of the problem.')
                    ->identifier('symfonyUX.twigComponent.exampleRule')
                    ->line($node->getLine())
                    ->tip('Suggestion to fix the problem.')
                    ->build(),
            ];
        }

        return [];
    }
}
```

### Step 2: Create Tests

Required structure:
```
tests/Rules/TwigComponent/ExampleRule/
├── ExampleRuleTest.php
├── config/
│   └── configured_rule.neon
└── Fixture/
    ├── InvalidTwigComponent.php
    ├── InvalidLiveComponent.php
    ├── ValidTwigComponent.php
    ├── ValidLiveComponent.php
    └── NotAComponent.php
```

#### Test File (`ExampleRuleTest.php`)

```php
<?php

declare(strict_types=1);

namespace Kocal\PHPStanSymfonyUX\Tests\Rules\TwigComponent\ExampleRule;

use Kocal\PHPStanSymfonyUX\Rules\TwigComponent\ExampleRule;
use PHPStan\Rules\Rule;
use PHPStan\Testing\RuleTestCase;

/**
 * @extends RuleTestCase<ExampleRule>
 */
final class ExampleRuleTest extends RuleTestCase
{
    public function testViolations(): void
    {
        $this->analyse(
            [__DIR__ . '/Fixture/InvalidTwigComponent.php'],
            [
                [
                    'Expected error message.',
                    10, // Line number
                    'Expected tip.',
                ],
            ]
        );

        $this->analyse(
            [__DIR__ . '/Fixture/InvalidLiveComponent.php'],
            [
                [
                    'Expected error message.',
                    10,
                    'Expected tip.',
                ],
            ]
        );
    }

    public function testNoViolations(): void
    {
        $this->analyse([__DIR__ . '/Fixture/NotAComponent.php'], []);
        $this->analyse([__DIR__ . '/Fixture/ValidTwigComponent.php'], []);
        $this->analyse([__DIR__ . '/Fixture/ValidLiveComponent.php'], []);
    }

    public static function getAdditionalConfigFiles(): array
    {
        return [__DIR__ . '/config/configured_rule.neon'];
    }

    protected function getRule(): Rule
    {
        return self::getContainer()->getByType(ExampleRule::class);
    }
}
```

#### Configuration (`config/configured_rule.neon`)

```yaml
includes:
    - ../../../../test-extension.neon

rules:
    - Kocal\PHPStanSymfonyUX\Rules\TwigComponent\ExampleRule
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kocal/phpstan-symfony-ux](https://github.com/Kocal/phpstan-symfony-ux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
