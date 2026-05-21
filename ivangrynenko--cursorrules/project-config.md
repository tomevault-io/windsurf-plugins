---
trigger: always_on
description: | Class | Description |
---

# Behat Steps - Claude Memory

## Available steps

### Index of Generic steps

| Class | Description |
| --- | --- |
| [CookieTrait](mdc:#cookietrait) | Verify and inspect browser cookies. |
| [DateTrait](mdc:#datetrait) | Convert relative date expressions into timestamps or formatted dates. |
| [ElementTrait](mdc:#elementtrait) | Interact with HTML elements using CSS selectors and DOM attributes. |
| [FileDownloadTrait](mdc:#filedownloadtrait) | Test file download functionality with content verification. |
| [KeyboardTrait](mdc:#keyboardtrait) | Simulate keyboard interactions in Drupal browser testing. |
| [LinkTrait](mdc:#linktrait) | Verify link elements with attribute and content assertions. |
| [PathTrait](mdc:#pathtrait) | Navigate and verify paths with URL validation. |
| [ResponseTrait](mdc:#responsetrait) | Verify HTTP responses with status code and header checks. |
| [WaitTrait](mdc:#waittrait) | Wait for a period of time or for AJAX to finish. |

### Index of Drupal steps

| Class | Description |
| --- | --- |
| [Drupal\BigPipeTrait](mdc:#drupalbigpipetrait) | Bypass Drupal BigPipe when rendering pages. |
| [Drupal\BlockTrait](mdc:#drupalblocktrait) | Manage Drupal blocks. |
| [Drupal\ContentBlockTrait](mdc:#drupalcontentblocktrait) | Manage Drupal content blocks. |
| [Drupal\ContentTrait](mdc:#drupalcontenttrait) | Manage Drupal content with workflow and moderation support. |
| [Drupal\DraggableviewsTrait](mdc:#drupaldraggableviewstrait) | Order items in the Drupal Draggable Views. |
| [Drupal\EckTrait](mdc:#drupalecktrait) | Manage Drupal ECK entities with custom type and bundle creation. |
| [Drupal\EmailTrait](mdc:#drupalemailtrait) | Test Drupal email functionality with content verification. |
| [Drupal\FieldTrait](mdc:#drupalfieldtrait) | Manipulate Drupal form fields and verify widget functionality. |
| [Drupal\FileTrait](mdc:#drupalfiletrait) | Manage Drupal file entities with upload and storage operations. |
| [Drupal\MediaTrait](mdc:#drupalmediatrait) | Manage Drupal media entities with type-specific field handling. |
| [Drupal\MenuTrait](mdc:#drupalmenutrait) | Manage Drupal menu systems and menu link rendering. |
| [Drupal\MetatagTrait](mdc:#drupalmetatagtrait) | Assert `<meta>` tags in page markup. |
| [Drupal\OverrideTrait](mdc:#drupaloverridetrait) | Override Drupal Extension behaviors. |
| [Drupal\ParagraphsTrait](mdc:#drupalparagraphstrait) | Manage Drupal paragraphs entities with structured field data. |
| [Drupal\SearchApiTrait](mdc:#drupalsearchapitrait) | Assert Drupal Search API with index and query operations. |
| [Drupal\TaxonomyTrait](mdc:#drupaltaxonomytrait) | Manage Drupal taxonomy terms with vocabulary organization. |
| [Drupal\TestmodeTrait](mdc:#drupaltestmodetrait) | Configure Drupal Testmode module for controlled testing scenarios. |
| [Drupal\UserTrait](mdc:#drupalusertrait) | Manage Drupal users with role and permission assignments. |
| [Drupal\WatchdogTrait](mdc:#drupalwatchdogtrait) | Assert Drupal does not trigger PHP errors during scenarios using Watchdog. |


---

## CookieTrait

[Source](mdc:src/CookieTrait.php), [Example](mdc:tests/behat/features/cookie.feature)

>  Verify and inspect browser cookies.
>  - Assert cookie existence and values with exact or partial matching.
>  - Support both WebDriver and BrowserKit drivers for test compatibility.


<details>
  <summary><code>@Then a cookie with the name :name should exist</code></summary>

<br/>
Assert that a cookie exists
<br/><br/>

```gherkin
Then a cookie with the name "session_id" should exist

```

</details>

<details>
  <summary><code>@Then a cookie with the name :name and the value :value should exist</code></summary>

<br/>
Assert that a cookie exists with a specific value
<br/><br/>

```gherkin
Then a cookie with the name "language" and the value "en" should exist

```

</details>

<details>
  <summary><code>@Then a cookie with the name :name and a value containing :partial_value should exist</code></summary>

<br/>
Assert that a cookie exists with a value containing a partial value
<br/><br/>

```gherkin
Then a cookie with the name "preferences" and a value containing "darkmode" should exist

```

</details>

<details>
  <summary><code>@Then a cookie with a name containing :partial_name should exist</code></summary>

<br/>
Assert that a cookie with a partial name exists
<br/><br/>

```gherkin
Then a cookie with a name containing "session" should exist

```

</details>

<details>
  <summary><code>@Then a cookie with a name containing :partial_name and the value :value should exist</code></summary>

<br/>
Assert that a cookie with a partial name and value exists
<br/><br/>

```gherkin
Then a cookie with a name containing "user" and the value "admin" should exist

```

</details>

<details>
  <summary><code>@Then a cookie with a name containing :partial_name and a value containing :partial_value should exist</code></summary>

<br/>
Assert that a cookie with a partial name and partial value exists
<br/><br/>

```gherkin
Then a cookie with a name containing "user" and a value containing "admin" should exist

```

</details>

<details>
  <summary><code>@Then a cookie with the name :name should not exist</code></summary>

<br/>
Assert that a cookie does not exist
<br/><br/>

```gherkin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
