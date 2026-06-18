---
trigger: always_on
description: Idiomatic patterns and first-party components for Symfony 7.4+ projects - routing, Doctrine, security, forms, testing, and all core tasks
---


# Symfony 7.4+ Skill

Helps with any task in a Symfony 7.4+ project: idiomatic patterns, first-party
components, and configuration. All examples pin to the 7.4 documentation.

## Tooling — ddev when present

Before running any shell command:

- If the project has `.ddev/config.yaml`, prefix with `ddev`. Examples:
  `ddev php bin/console make:migration`, `ddev composer require lock`,
  `ddev php bin/console doctrine:migrations:migrate`.
- Otherwise use the host environment directly:
  `php bin/console …`, `composer …`. The Symfony CLI (`symfony console …`)
  is fine if installed.

This applies to every command in the references below — substitute the
prefix as needed.

## Version check — gate version-specific features

Some patterns in this skill require a minimum Symfony version. Detect the
installed version once per session before using them:

```bash
composer show symfony/framework-bundle --no-ansi | grep versions
# fallback: grep '"symfony/framework-bundle"' composer.json
```

| Installed version | What applies |
|-------------------|--------------|
| >= 8.1 | Everything, including [whats-new-8.1.md](references/whats-new-8.1.md) |
| 7.4 – 8.0 | Core references + [whats-new-7.4.md](references/whats-new-7.4.md); **no** 8.1 features |
| < 7.4 | Core references only; skip both what's-new pages |

Never suggest a feature from a what's-new page without confirming the
version meets its gate. When a newer feature would be the ideal fit but
the project's version is too old, use the classic pattern and mention the
upgrade option.

## 1. Working in a Symfony project

Standard layout:

```
.
├── bin/console               # CLI entrypoint
├── config/
│   ├── bundles.php
│   ├── packages/             # bundle configuration (one file per bundle)
│   ├── routes/               # route imports
│   ├── routes.yaml
│   └── services.yaml
├── migrations/               # DoctrineMigrationsBundle
├── public/                   # web root, contains index.php
├── src/                      # PSR-4 namespace App\
│   ├── Controller/
│   ├── Entity/
│   ├── Repository/
│   ├── Form/
│   ├── Security/
│   ├── EventListener/
│   ├── Command/
│   └── ...
├── templates/                # Twig
├── tests/                    # PSR-4 namespace App\Tests\
├── translations/
├── var/                      # cache, log (git-ignored)
├── vendor/                   # composer (git-ignored)
├── .env, .env.local          # env vars
├── composer.json
├── symfony.lock              # Flex lock
```

Conventions in `config/services.yaml`:

- `autowire: true`, `autoconfigure: true`, `public: false`
- Classes under `App\` are auto-registered as services
- Add new services by just creating the class — no XML/YAML wiring needed
  unless tags or arguments need overriding

For details, see [`references/project-layout.md`](references/project-layout.md).

## 2. Common tasks

| Task | Reference | Docs |
|------|-----------|------|
| Use a Symfony 7.4 feature (requires >= 7.4) | [whats-new-7.4.md](references/whats-new-7.4.md) | https://symfony.com/blog/symfony-7-4-curated-new-features |
| Use a Symfony 8.1 feature (requires >= 8.1) | [whats-new-8.1.md](references/whats-new-8.1.md) | https://symfony.com/blog/symfony-8-1-curated-new-features |
| Add a route / controller | [controllers-and-routing.md](references/controllers-and-routing.md) | https://symfony.com/doc/7.4/controller.html |
| Add an entity / repository / migration | [doctrine.md](references/doctrine.md) | https://symfony.com/doc/7.4/doctrine.html |
| Add a console command | [console-commands.md](references/console-commands.md) | https://symfony.com/doc/7.4/console.html |
| Debug routes / services / config / DB | [debugging.md](references/debugging.md) | — |
| React to an event | [events.md](references/events.md) | https://symfony.com/doc/7.4/event_dispatcher.html |
| Secure an endpoint / firewall / voter | [security.md](references/security.md) | https://symfony.com/doc/7.4/security.html |
| Write a unit / functional / browser test | [testing.md](references/testing.md) | https://symfony.com/doc/7.4/testing.html |
| Configure the framework (`framework:` tree) | [framework-config.md](references/framework-config.md) | https://symfony.com/doc/current/reference/configuration/framework.html |
| Validate input | [validator.md](references/validator.md) | https://symfony.com/doc/7.4/validation.html |
| Send email | [mailer.md](references/mailer.md) | https://symfony.com/doc/7.4/mailer.html |
| Send notification (chat/SMS/push) | [notifier.md](references/notifier.md) | https://symfony.com/doc/7.4/notifier.html |
| Run background / queued work | [messenger.md](references/messenger.md) | https://symfony.com/doc/7.4/messenger.html |
| Schedule recurring work | [scheduler.md](references/scheduler.md) | https://symfony.com/doc/7.4/scheduler.html |
| Make HTTP calls outbound | [http-client.md](references/http-client.md) | https://symfony.com/doc/7.4/http_client.html |
| Cache expensive work | [cache.md](references/cache.md) | https://symfony.com/doc/7.4/cache.html |
| Acquire a lock | [lock.md](references/lock.md) | https://symfony.com/doc/7.4/lock.html |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recranet/symfony-skills](https://github.com/recranet/symfony-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
