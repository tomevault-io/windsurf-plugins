---
trigger: always_on
description: **Ethereum Address**: `0x` + 40 hex chars (e.g., `0xfb55CB623f2aB58Da17D8696501054a2ACeD1944`)
---


## Identifiers Reference

### XMTP Identifiers

**Ethereum Address**: `0x` + 40 hex chars (e.g., `0xfb55CB623f2aB58Da17D8696501054a2ACeD1944`)

**Private Key**: `0x` + 64 hex chars (e.g., `0x11567776b95bdbed513330f503741e19877bf7fe73e7957bf6f0ecf3e267fdb8`)

**Encryption Key**: 64 hex chars, no prefix (e.g., `11973168e34839f9d31749ad77204359c5c39c404e1154eacb7f35a867ee47de`)

**Inbox ID**: 64 hex chars, no prefix (e.g., `1180478fde9f6dfd4559c25f99f1a3f1505e1ad36b9c3a4dd3d5afb68c419179`)

**Installation ID**: 64 hex chars, no prefix (e.g., `a83166f3ab057f28d634cc04df5587356063dba11bf7d6bcc08b21a8802f4028`)

### Example User Credentials Set

```json
{
  "accountAddress": "0xfb55CB623f2aB58Da17D8696501054a2ACeD1944",
  "privateKey": "0x11567776b95bdbed513330f503741e19877bf7fe73e7957bf6f0ecf3e267fdb8",
  "encryptionKey": "11973168e34839f9d31749ad77204359c5c39c404e1154eacb7f35a867ee47de",
  "inboxId": "1180478fde9f6dfd4559c25f99f1a3f1505e1ad36b9c3a4dd3d5afb68c419179",
  "installationId": "a83166f3ab057f28d634cc04df5587356063dba11bf7d6bcc08b21a8802f4028"
}
```

## Working with Members

All conversations, both Groups and DMs, have a members() method that returns an array of GroupMember objects:

```typescript
// Get members from any conversation type (DM or Group)
const members = await conversation.members();

// Find a specific member
const member = members.find(
  (member) => member.inboxId.toLowerCase() === targetInboxId.toLowerCase(),
);

// Get member's Ethereum address
if (member) {
  const ethIdentifier = member.accountIdentifiers.find(
    (id) => id.identifierKind === IdentifierKind.Ethereum,
  );

  if (ethIdentifier) {
    const ethereumAddress = ethIdentifier.identifier;
    console.log(`Found Ethereum address: ${ethereumAddress}`);
  }

  // Get installation ID
  if (member.installationIds.length > 0) {
    const installationId = member.installationIds[0];
    console.log(`Found installation ID: ${installationId}`);
  }
}
```

## Working with Conversations

XMTP provides two main conversation types:

### Direct Messages (DMs)

```typescript
// Create a new DM via Inboxid
const dm = await client.conversations.newDm("inboxId123");

// Or create using an Ethereum address
const dmByAddress = await agent.createDmWithIdentifier({
  identifier: "0x7c40611372d354799d138542e77243c284e460b2",
  identifierKind: IdentifierKind.Ethereum,
});

// Send a message
await dm.send("Hello!");

// Access peer's inbox ID
const peerInboxId = dm.peerInboxId;
```

### Groups

```typescript
// Create a new group
const group = await agent.createGroupWithAddresses(addresses, {
  groupName: "My Group",
  groupDescription: "Group description",
});

// Update group metadata
await group.updateName("New Group Name");
await group.updateDescription("Updated description");

// Manage members
await group.addMembers(["newMemberInboxId"]);
await group.removeMembers(["memberToRemoveInboxId"]);

// Manage permissions
await group.addAdmin("memberInboxId");
await group.addSuperAdmin("memberInboxId");
```

Example usage:

```typescript
// Create a group with some initial settings
const group = await agent.createGroupWithAddresses(addresses, {
  groupName: "Project Discussion",
  groupDescription: "A group for our project collaboration",
  groupImageUrlSquare: "https://example.com/image.jpg",
});

// Update group settings later
await group.updateName("Updated Project Name");
await group.updateDescription("Our awesome project discussion");
await group.updateImageUrl("https://example.com/new-image.jpg");
```

Retrieve all messages at once from the local database:

```typescript
// First sync the conversations from the network to update the local db
await client.conversations.sync();

// Then get all messages as an array
const messages = await conversation.messages();
```

## Key Type References

```tsx
// Client Class
declare class Client {
  constructor(client: Client$1, signer: Signer, codecs: ContentCodec[]);
  static create(
    signer: Signer,
    encryptionKey: Uint8Array,
    options?: ClientOptions,
  ): Promise<Client>;
  get inboxId(): string;
  get installationId(): string;
  get conversations(): Conversations;
  get preferences(): Preferences;
}

// Conversations Class
declare class Conversations {
  constructor(client: Client, conversations: Conversations$1);
  getConversationById(id: string): Promise<Dm | Group | undefined>;
  newGroupWithIdentifiers(
    identifiers: Identifier[],
    options?: CreateGroupOptions,
  ): Promise<Group>;
  newGroup(inboxIds: string[], options?: CreateGroupOptions): Promise<Group>;
  newDmWithIdentifier(
    identifier: Identifier,
    options?: CreateDmOptions,
  ): Promise<Dm>;
  newDm(inboxId: string, options?: CreateDmOptions): Promise<Dm>;
  sync(): Promise<void>;
  streamAllMessages(
    callback?: StreamCallback<DecodedMessage>,
  ): Promise<AsyncStream<DecodedMessage<any>>>;
}

// Conversation Base Class
declare class Conversation {
  client: Client;
  constructor(
    client: Client,
    conversation: Conversation$1,
    lastMessage?: Message | null,
  );
  get id(): string;
  send<T>(content: T, options?: SendOptions): Promise<string>;
  messages<T>(options?: PaginationOptions): Promise<Array<DecodedMessage<T>>>;
  members(): Promise<GroupMember[]>;
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xmtplabs/xmtp-agent-examples](https://github.com/xmtplabs/xmtp-agent-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
