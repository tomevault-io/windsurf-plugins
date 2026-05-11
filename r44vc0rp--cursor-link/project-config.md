---
trigger: always_on
description: URL: /docs/plugins/device-authorization
---

# plugins: Device Authorization
URL: /docs/plugins/device-authorization
Source: https://raw.githubusercontent.com/better-auth/better-auth/refs/heads/main/docs/content/docs/plugins/device-authorization.mdx

OAuth 2.0 Device Authorization Grant for limited-input devices
        
***

title: Device Authorization
description: OAuth 2.0 Device Authorization Grant for limited-input devices
---------------------------------------------------------------------------

`RFC 8628` `CLI` `Smart TV` `IoT`

The Device Authorization plugin implements the OAuth 2.0 Device Authorization Grant ([RFC 8628](https://datatracker.ietf.org/doc/html/rfc8628)), enabling authentication for devices with limited input capabilities such as smart TVs, CLI applications, IoT devices, and gaming consoles.

## Try It Out

You can test the device authorization flow right now using the Better Auth CLI:

```bash
npx @better-auth/cli login
```

This will demonstrate the complete device authorization flow by:

1. Requesting a device code from the Better Auth demo server
2. Displaying a user code for you to enter
3. Opening your browser to the verification page
4. Polling for authorization completion

<Callout type="info">
  The CLI login command is a demo feature that connects to the Better Auth demo server to showcase the device authorization flow in action.
</Callout>

## Installation

<Steps>
  <Step>
    ### Add the plugin to your auth config

    Add the device authorization plugin to your server configuration.

    ```ts title="auth.ts"
    import { betterAuth } from "better-auth";
    import { deviceAuthorization } from "better-auth/plugins"; // [!code highlight]

    export const auth = betterAuth({
      // ... other config
      plugins: [ // [!code highlight]
        deviceAuthorization({ // [!code highlight]
          // Optional configuration
          expiresIn: "30m", // Device code expiration time // [!code highlight]
          interval: "5s",    // Minimum polling interval // [!code highlight]
        }), // [!code highlight]
      ], // [!code highlight]
    });
    ```
  </Step>

  <Step>
    ### Migrate the database

    Run the migration or generate the schema to add the necessary tables to the database.

    <Tabs items={["migrate", "generate"]}>
      <Tab value="migrate">
        ```bash
        npx @better-auth/cli migrate
        ```
      </Tab>

      <Tab value="generate">
        ```bash
        npx @better-auth/cli generate
        ```
      </Tab>
    </Tabs>

    See the [Schema](#schema) section to add the fields manually.
  </Step>

  <Step>
    ### Add the client plugin

    Add the device authorization plugin to your client.

    ```ts title="auth-client.ts"
    import { createAuthClient } from "better-auth/client";
    import { deviceAuthorizationClient } from "better-auth/client/plugins"; // [!code highlight]

    export const authClient = createAuthClient({
      plugins: [ // [!code highlight]
        deviceAuthorizationClient(), // [!code highlight]
      ], // [!code highlight]
    });
    ```
  </Step>
</Steps>

## How It Works

The device flow follows these steps:

1. **Device requests codes**: The device requests a device code and user code from the authorization server
2. **User authorizes**: The user visits a verification URL and enters the user code
3. **Device polls for token**: The device polls the server until the user completes authorization
4. **Access granted**: Once authorized, the device receives an access token

## Basic Usage

### Requesting Device Authorization

To initiate device authorization, call `device.code` with the client ID:

<APIMethod path="/device/code" method="POST">
  ```ts
  type deviceCode = {
      /**
       * The OAuth client identifier
       */
      client_id: string;
      /**
       * Space-separated list of requested scopes (optional)
       */
      scope?: string;
  }
  ```
</APIMethod>

Example usage:

```ts
const { data } = await authClient.device.code({
  client_id: "your-client-id",
  scope: "openid profile email",
});

if (data) {
  console.log(`Please visit: ${data.verification_uri}`);
  console.log(`And enter code: ${data.user_code}`);
}
```

### Polling for Token

After displaying the user code, poll for the access token:

<APIMethod path="/device/token" method="POST">
  ```ts
  type deviceToken = {
      /**
       * Must be "urn:ietf:params:oauth:grant-type:device_code"
       */
      grant_type: string;
      /**
       * The device code from the initial request
       */
      device_code: string;
      /**
       * The OAuth client identifier
       */
      client_id: string;
  }
  ```
</APIMethod>

Example polling implementation:

```ts
let pollingInterval = 5; // Start with 5 seconds
const pollForToken = async () => {
  const { data, error } = await authClient.device.token({
    grant_type: "urn:ietf:params:oauth:grant-type:device_code",
    device_code,
    client_id: yourClientId,
    fetchOptions: {
      headers: {
        "user-agent": `My CLI`,
      },
    },
  });

  if (data?.access_token) {
    console.log("Authorization successful!");
  } else if (error) {
    switch (error.error) {
      case "authorization_pending":
        // Continue polling
        break;
      case "slow_down":
        pollingInterval += 5;
        break;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [R44VC0RP/cursor.link](https://github.com/R44VC0RP/cursor.link) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
