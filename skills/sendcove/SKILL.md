---
name: sendcove
description: Draft, schedule and publish social media posts, upload media, check engagement and best times to post, and answer inbox comments with the Sendcove MCP server (X, Instagram, TikTok, Threads, Bluesky, YouTube, Facebook Pages, LinkedIn). Use when the user asks to write, schedule, post or cross-post social content, attach an image or video to a post, review how posts performed, or read and reply to comments and mentions in Sendcove.
license: MIT
compatibility: Needs the Sendcove MCP server (https://www.sendcove.app/api/mcp) connected with OAuth or a Sendcove API key, on a paid Sendcove plan.
metadata:
  author: Minner Labs LLC
  homepage: https://www.sendcove.app/features/mcp
---

# Using Sendcove

Sendcove is a social media scheduler. Its MCP server gives you tools to manage the user's posts, media, analytics and inbox. If no Sendcove tools are available, tell the user to connect Sendcove (setup guides: https://www.sendcove.app/features/mcp).

## Start with the accounts

1. Call `list_accounts` before creating or changing a post. Pass the returned `id`s as `accountIds`; Sendcove works out the platforms from them.
2. If the user has several accounts and didn't say which to use, ask. Don't post to every account by default.
3. An account with `needsReconnect: true` will fail to publish. Tell the user to reconnect it in Sendcove.

## Creating posts

- `create_post` makes a **draft** unless you set `status`. Drafts are the safe default: create one, show the user what you made, and let them schedule or publish it.
- `content` is the default text for every platform. Use `platformContent` to give a platform its own text, e.g. a 280-character version for `twitter` and a longer caption for `instagram`. Keys: `twitter`, `instagram`, `tiktok`, `threads`, `bluesky`, `facebook`, `linkedin` (strings) and `youtube` (`{ title, description }`).
- Per-platform options (Instagram post type, YouTube video type and visibility, TikTok privacy, Bluesky reply settings, X community, and so on) go in `platformSettings`. Take the allowed fields and values from the tool's input schema and its descriptions instead of guessing. Before setting `platformSettings.tiktok`, call `get_tiktok_creator_info` for the privacy levels and interactions that account allows.
- Character limits are checked when the post is published: X 280, Bluesky 300, Threads 500, Instagram and TikTok 2200, LinkedIn 3000, YouTube title 100 and description 5000. Keep each platform's text within its limit (use `platformContent`) so the post doesn't fail later.
- Instagram and TikTok posts need media; YouTube needs one video.
- Threads: pass follow-up posts as `threadItems` (each `{ content, mediaUrls? }`). They're published as a thread on X, Bluesky and Threads (Pro and Max); other platforms get `content` only. `update_post` with `threadItems: []` stops threading.
- Hashtags: `list_hashtag_sets` returns the user's saved sets (stored without `#`).
- Pass an `idempotencyKey` (any unique string, 8 to 200 characters) to `create_post`, `duplicate_post` and `reply_to_inbox_item`. If a call times out or you're unsure it went through, retry with the **same** key: you get the first result instead of a duplicate post or reply.
- `update_post` changes only the fields you pass, but `platformContent`, `platformSettings` and `threadItems` each replace the whole stored value. Call `get_post` first and send the complete object.

## Media

- Media must be in the user's Sendcove media library before it can go in `mediaUrls`. `list_media` shows what's already there.
- A public https URL: `upload_media_from_url`, then use the returned `url`.
- A local file: call `create_media_upload_url` and run the returned curl command with the real file path (for example `curl -s -X POST '<uploadUrl>' -F 'file=@/path/to/photo.jpg'`). The JSON response has the `url` to use. The upload URL lasts 10 minutes and needs no API key. For files over 4 MB, pass `contentType` and run the two returned commands (PUT to storage, then the finalize POST) in that order. Add `-F convertForInstagram=true` to convert an image to JPEG for Instagram.
- Allowed types: JPEG, PNG, GIF, WebP, MP4, MOV, AVI.

## Scheduling and publishing

Only schedule, publish or reply when the user asked for it in this conversation. Writing a post is not permission to publish it.

- Schedule: `status: "scheduled"` with `scheduledFor` as ISO 8601 with an offset (`2026-11-20T15:00:00Z`). Convert the user's local time and confirm the time zone if it's unclear. Or pass `nextOpenSlot: true` to use the next free posting time the user set for those accounts.
- Publish now: `status: "pending"` on `create_post`, or `publish_post` for an existing post.
- After publishing, call `get_post`: each account has its own `status` (`published`, `failed`, `pending`) and `error`. A failed post usually means one account failed. `publish_post` on that post retries only the failed accounts; it never re-posts where it's already live.
- `delete_post` cannot be undone. Confirm before deleting.

## Drafts-only connections (403)

Scheduling, publishing and replying need a Pro or Max plan **and** a connection that allows publishing. On the Basic plan, or with an API key or connected app set to drafts only, those calls return 403 "This API key or connected app can only create drafts".

When that happens, don't retry and don't work around it. Save the post as a draft and tell the user:

> I saved this as a draft in Sendcove. This connection is drafts only, so I can't schedule or publish it. To let me publish, you need a Pro or Max plan, then reconnect Sendcove here and choose "Allow publishing" (or use an API key created with "Allow publishing" on, under Account > API keys in Sendcove). You can also schedule the draft yourself in Sendcove.

## Analytics and inbox

- `get_engagement`: totals, per-platform breakdown, top posts, and each account's best recent posts against its own median (`timesMedian` shows how much better a post did, not why).
- `get_post_analytics`: one published post's metrics per account; pass `refresh: true` for fresh numbers (cached for an hour). Not every platform reports every metric.
- `get_best_times`: best days and hours to post, from the user's own engagement.
- `list_inbox` / `get_inbox_item`: replies, mentions and comments. `reply_to_inbox_item` posts a **public** reply, so show the user the text and get a yes first. `update_inbox_item` marks items read, unread or archived.
