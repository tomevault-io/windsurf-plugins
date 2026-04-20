---
trigger: always_on
description: You're an expert in writing TypeScript with Node.js. Generate **high-quality XMTP Agents** that adhere to the following best practices:
---

# Writing XMTP Agents

You're an expert in writing TypeScript with Node.js. Generate **high-quality XMTP Agents** that adhere to the following best practices:

## Guidelines

1.  Use modern TypeScript patterns and ESM modules. All examples should be structured as ES modules with `import` statements rather than CommonJS `require()`.

2.  Use the @xmtp/node-sdk and default to the latest version.

3.  Only import from @xmtp/node-sdk for XMTP functionality. Do not import from any other XMTP-related packages or URLs. Specifically:

    - Never use the deprecated @xmtp/xmtp-js library, which has been completely replaced by @xmtp/node-sdk
    - Always import directly from @xmtp/node-sdk as shown below:

    ```typescript
    // CORRECT:
    import { Client, type Conversation, type XmtpEnv } from "@xmtp/node-sdk";

    // INCORRECT - DEPRECATED:
    import { Client } from "@xmtp/xmtp-js";
    import { XmtpClient } from "some-other-package";
    ```

4.  Follow the consistent pattern for initializing XMTP clients:

    ```typescript
    const signer = createSigner(WALLET_KEY);
    const encryptionKey = getEncryptionKeyFromHex(ENCRYPTION_KEY);
    const client = await Client.create(signer, {
      dbEncryptionKey: encryptionKey,
      env: XMTP_ENV as XmtpEnv,
    });
    ```

5.  Use proper environment variable validation at the start of each application. Check for required environment variables and show descriptive errors if missing.

6.  Never use the concept of "topic" when working with XMTP. The current SDK doesn't use topics for message organization - work directly with conversations, groups, and DMs instead.

7.  Handle both Group and DM conversations properly. The `Group` and `Dm` classes extend the `Conversation` class and provide specific functionality:

    ```typescript
    if (conversation instanceof Group) {
      // Group-specific functionality like group.name or group.addMembers
    } else if (conversation instanceof Dm) {
      // DM-specific functionality like conversation.peerInboxId
    }
    ```

8.  Always sync conversations before streaming messages:

    ```typescript
    await client.conversations.sync();
    const stream = client.conversations.streamAllMessages();
    ```

9.  Filter out messages from the agent itself to prevent endless loops:

    ```typescript
    if (message?.senderInboxId.toLowerCase() === client.inboxId.toLowerCase()) {
      continue;
    }
    ```

10. Consistent error handling pattern with try/catch blocks and specific error messages.

11. Use the helper functions from the shared helpers directory for common operations:

    - `createSigner` - Creates a signer from a private key
    - `getEncryptionKeyFromHex` - Converts a hex string to an encryption key

12. Always import helpers from the `@helpers` path, not from a relative path:

    ```typescript
    // CORRECT:
    import { createSigner, getEncryptionKeyFromHex } from "@helpers/client";

    // INCORRECT:
    import { createSigner, getEncryptionKeyFromHex } from "./helpers";
    ```

13. When creating a group, use the correct options interface:

    ```typescript
    // CORRECT:
    const group = await client.conversations.newGroup([inboxId], {
      groupName: "My Group Name",
      groupDescription: "My group description",
      groupImageUrlSquare: "https://example.com/image.jpg",
    });

    // INCORRECT:
    const group = await client.conversations.newGroup([inboxId], {
      metadata: {
        name: "My Group Name",
      },
    });
    ```

14. When checking message content types, use string literals instead of importing a non-existent ContentTypeId enum:

    ```typescript
    // CORRECT:
    if (message?.contentType?.typeId !== "text") {
      continue;
    }

    // INCORRECT:
    import { ContentTypeId } from "@xmtp/node-sdk";
    if (message?.contentType?.typeId !== ContentTypeId.Text) {
      continue;
    }
    ```

15. Get information about a message sender by using the conversation's members method, not by trying to call a non-existent sender() method on the message:

    ```typescript
    // CORRECT: Use inboxStateFromInboxIds to get the address from the inboxId
    const inboxState = await client.preferences.inboxStateFromInboxIds([
      message.senderInboxId,
    ]);
    // assuming there is 1 only one identifier
    const addressFromInboxId = inboxState[0].identifiers[0].identifier;

    // INCORRECT:
    const sender = await message.sender();
    const senderAddress = sender.accountIdentifiers[0].identifier;
    ```

16. Remember that ALL conversation types (including DMs) have a members() method. Don't check for "members" in an object:

    ```typescript
    // CORRECT:
    const members = await conversation.members();

    // INCORRECT:
    if (conversation && "members" in conversation) {
      const members = await conversation.members();
    }
    ```

17. Always use toLowerCase() when comparing inboxIds or addresses:

    ```typescript
    // CORRECT:
    if (message.senderInboxId.toLowerCase() === client.inboxId.toLowerCase()) {
      continue;
    }

    // INCORRECT:
    if (message.senderInboxId === client.inboxId) {
      continue;
    }
    ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buildwithkon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
