---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`ok_keycloak` (composer: `oliverkroener/ok-keycloak`) signs TYPO3 frontend and backend users in
through Keycloak using the OpenID Connect authorization code flow. It talks to the Keycloak
endpoints over plain HTTP through TYPO3's `RequestFactory` and has **no third-party dependencies**.

**Extension key**: `ok_keycloak`
**Namespace**: `OliverKroener\OkKeycloak\`
**Version**: 5.0.0
**TYPO3**: 13.4, 14.x — v13-only APIs are avoided, v12 compatibility shims are deliberately absent
**PHP**: ^8.2 (requires `sodium`)

## Development context

This package lives at `packages/ok_keycloak/` in the TYPO3 13 monorepo at `/home/oliver/typo3-13`,
registered through the `packages/*` path repository. DDEV runs the stack
(project `typo3-13`, docroot `public`, https://typo3-13.ddev.site).

```bash
ddev composer update oliverkroener/ok-keycloak --no-interaction
ddev exec vendor/bin/typo3 database:updateschema   # after ext_tables.sql changes
ddev exec vendor/bin/typo3 cache:flush             # after config / class / DI changes
ddev exec vendor/bin/typo3 site:sets:list          # confirm the site set is registered
ddev exec vendor/bin/typo3 site:list               # site identifiers, needed for OK_KEYCLOAK_<SITE>_* vars
ddev mysql -e "SELECT * FROM tx_okkeycloak_configuration\G"
```

`cache:flush` is the cheapest real smoke test: it compiles the DI container, so a broken
`Services.yaml`, a bad `#[AsEventListener]` or an unresolvable constructor dependency fails here.

### Quality checks

There is **no test suite**, and the project ships no PHPStan or php-cs-fixer. Install them into a
throwaway directory rather than adding dev dependencies to the project:

```bash
cd /home/oliver/typo3-13
mkdir -p .tools-okkeycloak && cd .tools-okkeycloak
cat > composer.json <<'JSON'
{
    "require": {
        "phpstan/phpstan": "^2.1",
        "friendsofphp/php-cs-fixer": "^3.68",
        "typo3/coding-standards": "^0.8"
    },
    "config": { "allow-plugins": false }
}
JSON
```

PHPStan needs the project autoloader, because the extension is only resolvable through it:

```neon
# .tools-okkeycloak/phpstan.neon
parameters:
    level: 5
    paths:
        - /var/www/html/packages/ok_keycloak/Classes
    bootstrapFiles:
        - /var/www/html/vendor/autoload.php
    scanDirectories:
        - /var/www/html/vendor/typo3
```

```bash
ddev exec "cd /var/www/html/.tools-okkeycloak && composer install --no-interaction"
ddev exec "cd /var/www/html/.tools-okkeycloak && php -d memory_limit=1G vendor/bin/phpstan analyse -c phpstan.neon --no-progress"
# then, and only then, the fixer - point its finder at Classes/ plus the root *.php files
ddev exec "cd /var/www/html/.tools-okkeycloak && php vendor/bin/php-cs-fixer fix --config=.php-cs-fixer.php --dry-run --diff --using-cache=no"
rm -rf /home/oliver/typo3-13/.tools-okkeycloak
```

The php-cs-fixer config is `\TYPO3\CodingStandards\CsFixerConfig::create()` with the finder
pointed at the extension. Both currently pass clean (level 5, 0 of 26 files needing fixes) — keep
them that way.

Other checks:

```bash
ddev exec "cd /var/www/html/packages/ok_keycloak && composer validate --no-check-publish --no-check-lock"
ddev exec vendor/bin/yaml-lint packages/ok_keycloak/Configuration/Services.yaml packages/ok_keycloak/Configuration/Sets/Keycloak/*.yaml
xmllint --noout Resources/Private/Language/*.xlf Configuration/FlexForms/*.xml Resources/Public/Icons/*.svg
make docs        # renders Documentation/ via the TYPO3 render-guides Docker image
```

Label keys drift easily; this catches it:

```bash
comm -23 \
  <(grep -rhoE "locallang_be_module\.xlf:[A-Za-z0-9_.]+" Resources/Private Classes | sed 's/.*xlf://' | sort -u) \
  <(grep -oE 'trans-unit id="[^"]+"' Resources/Private/Language/locallang_be_module.xlf | sed 's/.*id="//;s/"//' | sort -u)
```

### Exercising the login flow without a Keycloak server

Most of the flow can be verified offline. Insert a configuration row whose `server_url` points at
an unreachable host (this also exercises the discovery fallback), then fetch `/typo3/` and read the
rendered authorize URL: the button, the resolved configuration, the decrypted secret, endpoint
derivation, the signed state and the PKCE challenge are all covered by that one request.

The client secret column is encrypted, so seed it by running `EncryptionService::encrypt()` in a
throwaway CLI script — it only needs `$GLOBALS['TYPO3_CONF_VARS']['SYS']['encryptionKey']` from
`config/system/settings.php`, no TYPO3 bootstrap. To confirm the callback would work, re-derive
`hash_hmac('sha256', 'pkce:' . nonce, encryptionKey)` from the state in the URL and check that its
SHA-256 matches the `code_challenge`. **Delete the seeded row afterwards** — an enabled row puts a
dead button on the real backend login screen.

## Architecture

### Configuration resolution — the core idea

`Service/ConfigurationService` resolves **each field separately** and takes the first layer with a
non-empty value, so the client secret can come from the environment while the redirect URI comes
from the database.

| Context | Order |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oliverkroener/ok_keycloak](https://github.com/oliverkroener/ok_keycloak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
