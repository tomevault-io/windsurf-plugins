---
trigger: always_on
description: This is the primary class to interact with SecretAgent. The following is a simple example:
---

# Agent

This is the primary class to interact with SecretAgent. The following is a simple example:

```js
const agent = require('secret-agent');

(async () => {
  await agent.goto('https://www.google.com');
  // other actions...
  await agent.close();
})();
```

An Agent instance can be thought of as a single user-browsing session. A default instance is automatically initialized and available as the default export of `secret-agent`. Each additional instance you create has the following attributes:

#### Replayable

An instance has a [replayable](/docs/advanced/session-replay)&nbsp;[Session](/docs/advanced/session) that will record all commands, dom changes, interaction and page events.

#### Lightweight

Instances are very lightweight, sharing a pool of browsers underneath. To manage concurrent scrapes in a single script, you can create one Agent for each scrape, or manage load and concurrency with a [Handler](/docs/basic-interfaces/handler).

#### Single Active Tab

Agent instances can have multiple [Tabs](/docs/basic-interfaces/tab), but only a single tab can be focused at a time. Clicks and other user interaction will go to the active tab (interacting with multiple tabs at once by a single user is easily detectable).

#### Sandboxed

Each Agent instance creates a private environment with its own cache, cookies, session data and [BrowserEmulator](/docs/plugins/browser-emulators). No data is shared between instances -- each operates within an airtight sandbox to ensure no identities leak across requests.

## Default Instance {#default}

A default instance is automatically initialized and available as the default export of `secret-agent`.

The default instance can receive configuration via command line arguments. Any args starting with `--input.*` will be processed. The resulting json object is available as [`agent.input`](#input)

```js
// script.js
const agent = require('secret-agent');

console.log(agent.input); // { secret: "true", agent: "true" }
```

```bash
$ node script.js --input.secret=true --input.agent=true
```

## Constructor

### new Agent*(options)* {#constructor}

Creates a new sandboxed browser instance with [unique user session and fingerprints](/docs/overview/basic-concepts). Or pass in an existing UserProfile to reconstruct a previously used user session.

You can optionally await an instance (or constructor) to cause the connection to the underlying SecretAgent to be initialized. If you don't await, the connection will be established on the first call.

Note: If you provide a `name` that has already been used to name another instance then a counter will be appended to your string to ensure its uniqueness. However, it's only unique within a single NodeJs process (i.e., rerunning your script will reset the counter).

```js
const { Agent } = require('secret-agent');

(async () => {
  // connection established here
  const agent = await new Agent({
    userAgent: '~ mac 13.1 & chrome > 14'
  });
})();
```

#### **Arguments**:

- options `object` Accepts any of the following:
  - connectionToCore `options | ConnectionToCore`. An object containing `IConnectionToCoreOptions` used to connect, or an already created `ConnectionToCore` instance. Defaults to automatically booting up and connecting to a local `Core`.
  - name `string`. This is used to generate a unique sessionName.
  - userAgent `strong`. This sets your browser's user agent string. Prefixing this string with a tilde (~) allows for dynamic options.
  - browserEmulatorId `string` defaults to `default-browser-emulator`. Chooses the BrowserEmulator plugin which emulates the properties that help SecretAgent look like a normal browser.
  - humanEmulatorId `string` defaults to `default-human-emulator`. Chooses the HumanEmulator plugin which drives the mouse/keyboard movements.
  - geolocation `IGeolocation`. Overrides the geolocation of the user. Will automatically grant permissions to all origins for geolocation.
    - latitude `number`. Latitude between -90 and 90.
    - longitude `number`. Longitude between -180 and 180.
    - accuracy `number`. Non-negative accuracy value. Defaults to random number 40-50.
  - timezoneId `string`. Overrides the host timezone. A list of valid ids are available at [unicode.org](https://unicode-org.github.io/cldr-staging/charts/37/supplemental/zone_tzid.html)
  - locale `string`. Overrides the host languages settings (eg, en-US). Locale will affect navigator.language value, Accept-Language request header value as well as number and date formatting rules.
  - viewport `IViewport`. Sets the emulated screen size, window position in the screen, inner/outer width and height. If not provided, the most popular resolution is used from [statcounter.com](https://gs.statcounter.com/screen-resolution-stats/desktop/united-states-of-america).
    - width `number`. The page width in pixels (minimum 0, maximum 10000000).
    - height `number`. The page height in pixels (minimum 0, maximum 10000000).
    - deviceScaleFactor `number` defaults to 1. Specify device scale factor (can be thought of as dpr).
    - screenWidth? `number`. The optional screen width in pixels (minimum 0, maximum 10000000).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ulixee/secret-agent](https://github.com/ulixee/secret-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
