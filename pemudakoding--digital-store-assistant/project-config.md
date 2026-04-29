---
trigger: always_on
description: This document contains domain-specific knowledge for developing WhatsApp bots for digital store management, covering business logic, technical patterns, and domain expertise.
---

# .cursor/rules/02-whatsapp-bot-domain-knowledge.mdc

## WhatsApp Bot Domain Knowledge

This document contains domain-specific knowledge for developing WhatsApp bots for digital store management, covering business logic, technical patterns, and domain expertise.

## WhatsApp Bot Architecture Patterns

### Message Flow Understanding
```
Incoming Message → Message Handler → Command Parser → Context Creation → Command Execution → Response Queue → WhatsApp API
```

### Context Object Structure
The context object is the core data structure passed to all commands:
```javascript
const context = {
    // Core WhatsApp data
    msg,                    // Original Baileys message object
    from,                   // Chat/Group ID
    sender,                 // User ID who sent message
    pushname,               // User's display name
    body,                   // Message text content
    
    // Group metadata (if applicable)
    isGroup,                // Boolean: is this a group chat
    groupMetadata,          // Group info (name, participants, etc.)
    groupMembers,           // Array of group members
    isGroupAdmin,           // Boolean: is sender group admin
    isBotGroupAdmin,        // Boolean: is bot group admin
    
    // User permissions
    isOwner,                // Boolean: is sender bot owner
    
    // Message analysis
    isQuotedMsg,            // Boolean: is replying to a message
    quotedMsg,              // Quoted message object
    
    // Services and managers
    messageService,         // For sending messages
    groupService,           // For group operations
    listManager,            // Product management
    testiManager,           // Testimonial management
    afkManager,             // AFK status management
    sewaManager,            // Subscription management
    // ... other managers
    
    // Utility functions
    reply: async (text) => {} // Quick reply function
};
```

### Command Categories & Responsibilities

#### **Admin Commands** (`src/commands/admin/`)
- **Purpose**: Group moderation and management
- **Access**: Group admins only
- **Common patterns**: Member management, message moderation, group settings
- **Examples**: `kick`, `promote`, `demote`, `hidetag`, `antilink`

#### **Owner Commands** (`src/commands/owner/`)
- **Purpose**: Bot administration and system management
- **Access**: Bot owner only
- **Common patterns**: System monitoring, data management, bot configuration
- **Examples**: `botstat`, `resetqueue`, `broadcast`, `addproduk`

#### **Store Commands** (`src/commands/store/`)
- **Purpose**: Digital store operations
- **Access**: All users (with business logic restrictions)
- **Common patterns**: Product browsing, payment info, testimonials
- **Examples**: `list`, `produk`, `payment`, `testi`

#### **General Commands** (`src/commands/general/`)
- **Purpose**: Universal utility functions
- **Access**: All users
- **Common patterns**: Information retrieval, utilities, help
- **Examples**: `help`, `ping`, `sticker`, `afk`

## Digital Store Business Logic

### Product Management Workflow
```
Add Product → Validate Data → Store in Database → Update Catalog → Notify Admins
```

### Order Processing States
1. **Inquiry** - Customer asks about product
2. **Quotation** - Price and details provided
3. **Processing** - Order being prepared (`set_proses.json`)
4. **Completion** - Order delivered (`set_done.json`)
5. **Testimonial** - Customer feedback collection

### Payment Flow Understanding
```
Customer Inquiry → Product Selection → Payment Info → Payment Proof → Order Processing → Delivery → Completion
```

## WhatsApp API Specific Knowledge

### Message Types & Handling
```javascript
// Text messages
msg.message?.conversation

// Media messages with captions
msg.message?.imageMessage?.caption
msg.message?.videoMessage?.caption

// Extended text (links, mentions)
msg.message?.extendedTextMessage?.text

// Button responses
msg.message?.buttonsResponseMessage?.selectedButtonId

// List responses
msg.message?.listResponseMessage?.singleSelectReply?.selectedRowId
```

### Group Operations Best Practices
```javascript
// Always check bot admin status before group operations
const isBotAdmin = await groupService.isBotGroupAdmin(groupId);
if (!isBotAdmin) {
    return messageService.reply('Bot harus menjadi admin untuk operasi ini');
}

// Validate target user permissions
const isTargetAdmin = await groupService.isGroupAdmin(groupId, targetUserId);
if (isTargetAdmin && action === 'kick') {
    return messageService.reply('Tidak bisa kick admin grup');
}
```

### Media Handling Patterns
```javascript
// Image processing for products
if (quotedMsg?.imageMessage) {
    const imageUrl = await mediaService.uploadImage(quotedMsg);
    product.image_url = imageUrl;
    product.isImage = true;
}

// Sticker creation
const stickerBuffer = await mediaService.createSticker(mediaBuffer);
await messageService.sendSticker(from, stickerBuffer);
```

## Customer Service Automation Patterns

### AFK System Domain Logic
- **Scope**: Group-specific (user can be AFK in one group but not another)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pemudakoding) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
