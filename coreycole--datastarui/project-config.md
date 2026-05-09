---
trigger: always_on
description: Generates Go code from templ files.
---

# Injection attacks

templ is designed to prevent user-provided data from being used to inject vulnerabilities.

`<script>` and `<style>` tags could allow user data to inject vulnerabilities, so variables are not permitted in these sections.

```html
templ Example() {
  <script>
    function showAlert() {
      alert("hello");
    }
  </script>
  <style type="text/css">
    /* Only CSS is allowed */
  </style>
}
```

`onClick` attributes, and other `on*` attributes are used to execute JavaScript. To prevent user data from being unescaped, `on*` attributes accept a `templ.ComponentScript`.

```html
script onClickHandler(msg string) {
  alert(msg);
}

templ Example(msg string) {
  <div onClick={ onClickHandler(msg) }>
    { "will be HTML encoded using templ.Escape" }
  </div>
}
```

Style attributes cannot be expressions, only constants, to avoid escaping vulnerabilities. templ style templates (`css className()`) should be used instead.

```html
templ Example() {
  <div style={ "will throw an error" }></div>
}
```

Class names are sanitized by default. A failed class name is replaced by `--templ-css-class-safe-name`. The sanitization can be bypassed using the `templ.SafeClass` function, but the result is still subject to escaping.

```html
templ Example() {
  <div class={ "unsafe</style&gt;-will-sanitized", templ.SafeClass("&sanitization bypassed") }></div>
}
```

Rendered output:

```html
<div class="--templ-css-class-safe-name &amp;sanitization bypassed"></div>
```

```html
templ Example() {
  <div>Node text is not modified at all.</div>
  <div>{ "will be escaped using templ.EscapeString" }</div>
}
```

`href` attributes must be a `templ.SafeURL` and are sanitized to remove JavaScript URLs unless bypassed.

```html
templ Example() {
  <a href="http://constants.example.com/are/not/sanitized">Text</a>
  <a href={ templ.URL("will be sanitized by templ.URL to remove potential attacks") }</a>
  <a href={ templ.SafeURL("will not be sanitized by templ.URL") }</a>
}
```

Within css blocks, property names, and constant CSS property values are not sanitized or escaped.

```css
css className() {
	background-color: #ffffff;
}
```

CSS property values based on expressions are passed through `templ.SanitizeCSS` to replace potentially unsafe values with placeholders.

```css
css className() {
	color: { red };
}
```
# Content security policy

## Nonces

In templ [script templates](mdc:syntax-and-usage/script-templates#script-templates) are rendered as inline `<script>` tags.

Strict Content Security Policies (CSP) can prevent these inline scripts from executing.

By setting a nonce attribute on the `<script>` tag, and setting the same nonce in the CSP header, the browser will allow the script to execute.

:::info
It's your responsibility to generate a secure nonce. Nonces should be generated using a cryptographically secure random number generator.

See https://content-security-policy.com/nonce/ for more information.
:::

## Setting a nonce

The `templ.WithNonce` function can be used to set a nonce for templ to use when rendering scripts.

It returns an updated `context.Context` with the nonce set.

In this example, the `alert` function is rendered as a script element by templ.

```templ title="templates.templ"
package main

import "context"
import "os"

script onLoad() {
    alert("Hello, world!")
}

templ template() {
    @onLoad()
}
```

```go title="main.go"
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func withNonce(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		nonce := securelyGenerateRandomString()
		w.Header().Add("Content-Security-Policy", fmt.Sprintf("script-src 'nonce-%s'", nonce))
		// Use the context to pass the nonce to the handler.
		ctx := templ.WithNonce(r.Context(), nonce)
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}

func main() {
	mux := http.NewServeMux()

	// Handle template.
	mux.HandleFunc("/", templ.Handler(template()))

	// Apply middleware.
	withNonceMux := withNonce(mux)

	// Start the server.
	fmt.Println("listening on :8080")
	if err := http.ListenAndServe(":8080", withNonceMux); err != nil {
		log.Printf("error listening: %v", err)
	}
}
```

```html title="Output"
<script nonce="randomly generated nonce">
  function __templ_onLoad_5a85() {
    alert("Hello, world!")
  }
</script>
<script nonce="randomly generated nonce">
  __templ_onLoad_5a85()
</script>
```
# Code signing

Binaries are created by the GitHub Actions workflow at https://github.com/a-h/templ/blob/main/.github/workflows/release.yml

Binaries are signed by cosign. The public key is stored in the repository at https://github.com/a-h/templ/blob/main/cosign.pub

Instructions for key verification at https://docs.sigstore.dev/verifying/verify/
---
sidebar_position: 1
---

# Introduction

## templ - build HTML with Go

Create components that render fragments of HTML and compose them to create screens, pages, documents, or apps.

* Server-side rendering: Deploy as a serverless function, Docker container, or standard Go program.
* Static rendering: Create static HTML files to deploy however you choose.
* Compiled code: Components are compiled into performant Go code.
* Use Go: Call any Go code, and use standard `if`, `switch`, and `for` statements.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoreyCole/datastarui](https://github.com/CoreyCole/datastarui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
