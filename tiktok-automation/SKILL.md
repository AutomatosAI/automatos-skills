---
name: tiktok-automation
description: Upload videos and photos to TikTok with two-phase publish workflow
version: "1.0.0"
tags: [tiktok, video, social-media, content, short-form]
category: social-media
tools:
  - name: composio_execute
    description: Execute Composio actions
  - name: scratchpad_read
    description: Read data from previous recipe steps
  - name: scratchpad_write
    description: Export data for downstream recipe steps
---

# TikTok Automation

## Prerequisites

- Active Composio connection: `TIKTOK` app linked to user's TikTok account
- Content Creator or Business account required for API access
- Videos must meet TikTok specs (MP4, max 287.6 MB for direct upload, max 4GB for chunked)

## Core Workflows

### 1. Upload and Publish a Video (Two-Phase)

**Phase 1: Initialize Upload**

```
composio_execute action="TIKTOK_INIT_VIDEO_UPLOAD"
  params:
    title: "Quick automation demo"
    description: "How to automate your workflow #automation #tech"
    privacy_level: "PUBLIC_TO_EVERYONE"
    disable_comment: false
    disable_duet: false
    disable_stitch: false
    video_size: 15728640
    chunk_size: 10485760
    total_chunk_count: 2
```

Response returns `upload_url` and `publish_id`. Upload the binary to `upload_url` via HTTP PUT.

**Phase 2: Check Publish Status**

```
composio_execute action="TIKTOK_PUBLISH_STATUS_CHECK"
  params:
    publish_id: "<publish_id from phase 1>"
```

Poll this until status is `PUBLISH_COMPLETE` or `FAILED`.

### 2. Upload a Photo Post

```
composio_execute action="TIKTOK_INIT_PHOTO_UPLOAD"
  params:
    title: "Behind the scenes"
    description: "Building in public #startup"
    privacy_level: "PUBLIC_TO_EVERYONE"
    photo_count: 3
```

Response returns upload URLs for each photo. Upload images to each URL.

### 3. Status Polling Pattern

```
1. composio_execute action="TIKTOK_INIT_VIDEO_UPLOAD" ...
2. Upload binary to returned upload_url (external HTTP step)
3. Wait 10-30 seconds for processing
4. composio_execute action="TIKTOK_PUBLISH_STATUS_CHECK" publish_id=<id>
5. If status == "PROCESSING_UPLOAD", wait and retry step 4
6. If status == "PUBLISH_COMPLETE", done
7. If status == "FAILED", check error field
```

## Key Parameters

| Parameter | Format | Notes |
|-----------|--------|-------|
| `title` | String | Display title for the post |
| `description` | String | Caption with hashtags |
| `privacy_level` | Enum | `PUBLIC_TO_EVERYONE`, `MUTUAL_FOLLOW_FRIENDS`, `SELF_ONLY` |
| `video_size` | Integer (bytes) | Required for chunked upload |
| `chunk_size` | Integer (bytes) | Max 10MB per chunk |
| `total_chunk_count` | Integer | Ceil(video_size / chunk_size) |
| `publish_id` | String | Returned from init, used for status check |

## Pitfalls

- **Two-phase upload is mandatory:** `TIKTOK_INIT_VIDEO_UPLOAD` only creates the upload session. You must upload the actual binary to the returned URL separately, then poll for completion.
- **Processing time:** Videos take 10 seconds to several minutes to process depending on length and resolution. Always poll `TIKTOK_PUBLISH_STATUS_CHECK`.
- **File size limits:** Direct upload max 287.6 MB. Chunked upload max 4 GB. Each chunk max 10 MB.
- **Privacy defaults:** If `privacy_level` is omitted, TikTok may default to private. Always set explicitly.
- **Hashtags:** Include in `description` field, not a separate parameter. Use `#tag` format inline.
- **Rate limits:** TikTok enforces daily upload limits per account. Exceeding them returns opaque errors.

## Quick Reference

| Task | Action |
|------|--------|
| Start video upload | `TIKTOK_INIT_VIDEO_UPLOAD` |
| Start photo upload | `TIKTOK_INIT_PHOTO_UPLOAD` |
| Check publish status | `TIKTOK_PUBLISH_STATUS_CHECK` |

## What NOT to Do

- Do NOT skip the binary upload step — `INIT_VIDEO_UPLOAD` does not upload the video itself
- Do NOT assume the video is live after init — always poll `PUBLISH_STATUS_CHECK`
- Do NOT upload files exceeding size limits — the upload will silently fail
- Do NOT omit `privacy_level` — defaults are unpredictable
- Do NOT set `chunk_size` above 10 MB — the upload will be rejected
