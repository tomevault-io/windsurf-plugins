---
trigger: always_on
description: This guide explains how AI agents can effectively use the Upload Post node in n8n workflows to automate social media content publishing across multiple platforms.
---

# Upload Post Node - Agent Integration Guide

This guide explains how AI agents can effectively use the Upload Post node in n8n workflows to automate social media content publishing across multiple platforms.

## Overview

The Upload Post node enables automated publishing to:
- **Instagram** (Photos, Videos, Stories, Reels)
- **Facebook** (Photos, Videos, Text, Pages)
- **LinkedIn** (Photos, Videos, Text, Pages)
- **TikTok** (Photos, Videos)
- **X (Twitter)** (Photos, Videos, Text, Polls)
- **YouTube** (Videos)
- **Pinterest** (Photos, Videos, Boards)
- **Threads** (Photos, Videos, Text)
- **Reddit** (Text posts)

## Agent Integration Capabilities

### Direct Workflow Invocation
Agents can trigger Upload Post workflows by:
- Calling n8n webhooks with content data
- Using the n8n API to execute workflows
- Integrating with n8n's agent framework

### Content Generation & Publishing Flow
```
Agent → Content Generation → Upload Post Node → Multiple Platforms
```

### Supported Agent Operations

#### Content Upload Operations
- **Upload Photos**: Single or multiple images with platform-specific customization
- **Upload Videos**: Video content with thumbnails and metadata
- **Upload Text**: Text posts, including polls on X/Twitter

#### Management Operations
- **Get Upload Status**: Check async upload completion
- **Get Upload History**: Retrieve past uploads
- **List Scheduled Posts**: View future scheduled content
- **Cancel/Edit Scheduled Posts**: Modify scheduled content

#### User Management
- **Create/List/Delete Users**: Manage Upload Post profiles
- **Generate/Validate JWT**: Platform integration tokens

## Agent Usage Examples

### Basic Content Publishing

```json
{
  "operation": "uploadPhotos",
  "user": "my_profile",
  "platform": ["instagram", "facebook"],
  "title": "Beautiful sunset view",
  "description": "Captured this amazing sunset during my evening walk",
  "photos": "{{ $binary.data }}"
}
```

### Video Upload with Platform-Specific Settings

```json
{
  "operation": "uploadVideo",
  "user": "creator_profile",
  "platform": ["youtube", "tiktok", "instagram"],
  "title": "Tutorial: Getting Started with AI",
  "description": "Learn the basics of artificial intelligence",
  "video": "{{ $binary.video }}",
  "youtubeTitle": "AI Tutorial for Beginners",
  "youtubeDescription": "Complete guide to AI fundamentals",
  "youtubeTags": "AI, tutorial, beginners, machine learning",
  "tiktokTitle": "AI Basics in 5 Minutes! 🤖",
  "instagramMediaType": "REELS"
}
```

### Scheduled Content Publishing

```json
{
  "operation": "uploadText",
  "user": "business_profile",
  "platform": ["linkedin", "facebook"],
  "title": "Exciting company news coming soon!",
  "scheduledDate": "2024-12-25T10:00:00Z",
  "facebookTitle": "🚨 Big Announcement 🚨",
  "linkedinTitle": "Important Update from Our Team"
}
```

### Poll Creation on X/Twitter

```json
{
  "operation": "uploadText",
  "user": "social_profile",
  "platform": ["x"],
  "title": "What's your favorite programming language?",
  "xPollOptions": "JavaScript, Python, TypeScript, Go",
  "xPollDuration": 1440
}
```

## Agent Best Practices

### Content Preparation
1. **Optimize Media**: Ensure images/videos meet platform requirements
2. **Platform-Specific Customization**: Use title/description overrides for each platform
3. **Hashtag Strategy**: Include relevant hashtags in platform-specific titles

### Error Handling
```json
{
  "operation": "getStatus",
  "requestId": "{{ $node.uploadPost.output.request_id }}"
}
```

### Workflow Design
1. **Async Processing**: Use `uploadAsync: true` for large uploads
2. **Status Polling**: Implement polling for async operations
3. **Fallback Strategies**: Handle platform-specific failures gracefully

### Rate Limiting & Scheduling
- Schedule content during optimal posting times
- Space out multi-platform publishing to avoid rate limits
- Use the scheduling feature for precise timing

## Platform-Specific Agent Considerations

### Instagram
- **Media Types**: IMAGE (feed), STORIES, REELS
- **Trial Reels**: Test content with non-followers first before sharing with followers
  - `CUSTOM`: Regular Reel (default)
  - `TRIAL_REELS_SHARE_TO_FOLLOWERS_IF_LIKED`: Auto-share if performs well
  - `TRIAL_REELS_DONT_SHARE_TO_FOLLOWERS`: Manual decision later
- **Video Options**: Share to feed, collaborators, cover URL
- **Best Practice**: Use high-quality, vertical videos for Reels

### TikTok
- **Photo Options**: Auto-add music, disable comments
- **Video Options**: Privacy levels, duet/stitch controls
- **AI Content**: Declare AI-generated content when applicable

### YouTube
- **Metadata**: Tags, categories, privacy settings
- **Compliance**: Made for kids, synthetic media declarations
- **Geo-Restrictions**: Country-specific availability

### X (Twitter)
- **Rich Features**: Polls, quote tweets, threads
- **Validation**: Poll options (2-4, max 25 chars each)
- **Timing**: Optimal engagement windows

### LinkedIn
- **Professional Focus**: Use appropriate tone and formatting
- **Page Selection**: Target specific organization pages
- **Visibility**: Public, connections, or logged-in users

## Agent Workflow Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Upload-Post/n8n-nodes-upload-post](https://github.com/Upload-Post/n8n-nodes-upload-post) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
