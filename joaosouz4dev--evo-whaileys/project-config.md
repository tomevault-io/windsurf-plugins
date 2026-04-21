---
trigger: always_on
description: Type definitions and interfaces for Evolution API
---


# Evolution API Type Rules

## Namespace Pattern

### WhatsApp Types Namespace
```typescript
/* eslint-disable @typescript-eslint/no-namespace */
import { JsonValue } from '@prisma/client/runtime/library';
import { AuthenticationState, WAConnectionState } from 'whaileys';

export declare namespace wa {
  export type QrCode = {
    count?: number;
    pairingCode?: string;
    base64?: string;
    code?: string;
  };

  export type Instance = {
    id?: string;
    qrcode?: QrCode;
    pairingCode?: string;
    authState?: { state: AuthenticationState; saveCreds: () => void };
    name?: string;
    wuid?: string;
    profileName?: string;
    profilePictureUrl?: string;
    token?: string;
    number?: string;
    integration?: string;
    businessId?: string;
  };

  export type LocalChatwoot = {
    enabled?: boolean;
    accountId?: string;
    token?: string;
    url?: string;
    nameInbox?: string;
    mergeBrazilContacts?: boolean;
    importContacts?: boolean;
    importMessages?: boolean;
    daysLimitImportMessages?: number;
    organization?: string;
    logo?: string;
  };

  export type LocalProxy = {
    enabled?: boolean;
    host?: string;
    port?: string;
    protocol?: string;
    username?: string;
    password?: string;
  };

  export type LocalSettings = {
    rejectCall?: boolean;
    msgCall?: string;
    groupsIgnore?: boolean;
    alwaysOnline?: boolean;
    readMessages?: boolean;
    readStatus?: boolean;
    syncFullHistory?: boolean;
  };

  export type LocalWebHook = {
    enabled?: boolean;
    url?: string;
    events?: string[];
    headers?: JsonValue;
    byEvents?: boolean;
    base64?: boolean;
  };

  export type StatusMessage = 'ERROR' | 'PENDING' | 'SERVER_ACK' | 'DELIVERY_ACK' | 'READ' | 'DELETED' | 'PLAYED';
}
```

## Enum Definitions

### Events Enum
```typescript
export enum Events {
  APPLICATION_STARTUP = 'application.startup',
  INSTANCE_CREATE = 'instance.create',
  INSTANCE_DELETE = 'instance.delete',
  QRCODE_UPDATED = 'qrcode.updated',
  CONNECTION_UPDATE = 'connection.update',
  STATUS_INSTANCE = 'status.instance',
  MESSAGES_SET = 'messages.set',
  MESSAGES_UPSERT = 'messages.upsert',
  MESSAGES_EDITED = 'messages.edited',
  MESSAGES_UPDATE = 'messages.update',
  MESSAGES_DELETE = 'messages.delete',
  SEND_MESSAGE = 'send.message',
  SEND_MESSAGE_UPDATE = 'send.message.update',
  CONTACTS_SET = 'contacts.set',
  CONTACTS_UPSERT = 'contacts.upsert',
  CONTACTS_UPDATE = 'contacts.update',
  PRESENCE_UPDATE = 'presence.update',
  CHATS_SET = 'chats.set',
  CHATS_UPDATE = 'chats.update',
  CHATS_UPSERT = 'chats.upsert',
  CHATS_DELETE = 'chats.delete',
  GROUPS_UPSERT = 'groups.upsert',
  GROUPS_UPDATE = 'groups.update',
  GROUP_PARTICIPANTS_UPDATE = 'group-participants.update',
  CALL = 'call',
  TYPEBOT_START = 'typebot.start',
  TYPEBOT_CHANGE_STATUS = 'typebot.change-status',
  LABELS_EDIT = 'labels.edit',
  LABELS_ASSOCIATION = 'labels.association',
  CREDS_UPDATE = 'creds.update',
  MESSAGING_HISTORY_SET = 'messaging-history.set',
  REMOVE_INSTANCE = 'remove.instance',
  LOGOUT_INSTANCE = 'logout.instance',
}
```

### Integration Types
```typescript
export const Integration = {
  WHATSAPP_BUSINESS: 'WHATSAPP-BUSINESS',
  WHATSAPP_BAILEYS: 'WHATSAPP-BAILEYS',
  EVOLUTION: 'EVOLUTION',
} as const;

export type IntegrationType = typeof Integration[keyof typeof Integration];
```

## Constant Arrays

### Message Type Constants
```typescript
export const TypeMediaMessage = [
  'imageMessage',
  'documentMessage',
  'audioMessage',
  'videoMessage',
  'stickerMessage',
  'ptvMessage', // Evolution API includes this
];

export const MessageSubtype = [
  'ephemeralMessage',
  'documentWithCaptionMessage',
  'viewOnceMessage',
  'viewOnceMessageV2',
];

export type MediaMessageType = typeof TypeMediaMessage[number];
export type MessageSubtypeType = typeof MessageSubtype[number];
```

## Interface Definitions

### Service Interfaces
```typescript
export interface ServiceInterface {
  create(instance: InstanceDto, data: any): Promise<any>;
  find(instance: InstanceDto): Promise<any>;
  update?(instance: InstanceDto, data: any): Promise<any>;
  delete?(instance: InstanceDto): Promise<any>;
}

export interface ChannelServiceInterface extends ServiceInterface {
  sendMessage(data: SendMessageDto): Promise<any>;
  connectToWhatsapp(data?: any): Promise<void>;
  receiveWebhook?(data: any): Promise<void>;
}

export interface ChatbotServiceInterface extends ServiceInterface {
  processMessage(
    instanceName: string,
    remoteJid: string,
    message: any,
    pushName?: string,
  ): Promise<void>;
}
```

## Configuration Types

### Environment Configuration Types
```typescript
export interface DatabaseConfig {
  CONNECTION: {
    URI: string;
    DB_PREFIX_NAME: string;
    CLIENT_NAME?: string;
  };
  ENABLED: boolean;
  SAVE_DATA: {
    INSTANCE: boolean;
    NEW_MESSAGE: boolean;
    MESSAGE_UPDATE: boolean;
    CONTACTS: boolean;
    CHATS: boolean;
  };
}

export interface AuthConfig {
  TYPE: 'apikey' | 'jwt';
  API_KEY: {
    KEY: string;
  };
  JWT?: {
    EXPIRIN_IN: number;
    SECRET: string;
  };
}

export interface CacheConfig {
  REDIS: {
    ENABLED: boolean;
    URI: string;
    PREFIX_KEY: string;
    SAVE_INSTANCES: boolean;
  };
  LOCAL: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joaosouz4dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
