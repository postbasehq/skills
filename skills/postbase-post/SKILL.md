---
name: postbase-post
description: Draft or schedule social media posts and threads to X, LinkedIn, Bluesky, Mastodon and YouTube through the Postbase REST API. Use when asked to post, schedule, announce or cross-post something to social media with Postbase.
---

# Post to social media with Postbase

Postbase (postbase.so) schedules posts to connected social accounts ("channels")
and publishes them through each network's official API. Everything you create
appears on the user's Postbase calendar.

## Setup

- Base URL: `https://www.postbase.so/api/v1`
- Auth: `Authorization: Bearer $POSTBASE_API_KEY` (a `pb_live_…` key from
  Postbase → Developers). Never print the key or write it into files.
- Check the key: `GET /me` returns the workspace name and channel count.

If the user's tool supports MCP, the hosted server at `https://mcp.postbase.so/mcp`
(sign in with Postbase, no key) offers the same actions as tools.

## Workflow

1. `GET /channels` to find channel ids. Match what the user said ("LinkedIn",
   "my X account") to `platform` and `handle`. Ask if it's ambiguous.
2. Write the post. Respect each network's limit per post: X 280 (each link
   counts 23), Bluesky 300, Mastodon 500, LinkedIn 3,000, YouTube 5,000
   (description).
3. `POST /posts` with an `Idempotency-Key` header (generate a UUID per post).
   If the request fails or times out, retry with the **same** key: it can't
   post twice.
4. Tell the user what was scheduled and when, in their timezone.
5. Later, `GET /posts/{id}` shows each channel's status, live `url` or `error`.

Prefer scheduling (or a draft) over posting "now", so the user can review it on
their calendar. Confirm the text and time with the user before scheduling
unless they've told you to go ahead.

## Create a post

```bash
curl -X POST https://www.postbase.so/api/v1/posts \
  -H "Authorization: Bearer $POSTBASE_API_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{
    "body": "The long LinkedIn version of the announcement…",
    "channel_ids": ["<linkedin-id>", "<x-id>"],
    "channel_bodies": { "<x-id>": "Short X version" },
    "scheduled_at": "2026-10-01T09:00:00+01:00"
  }'
```

Fields:

- `body` (one post) or `thread` (array of posts, published as a reply chain on
  X, Bluesky and Mastodon; on LinkedIn the extra parts become the first comment).
- `channel_ids`: from `GET /channels`.
- `channel_bodies`: `{ channel_id: text }` to give some channels their own text
  (a single post each). Use it instead of separate requests per network.
- `scheduled_at`: ISO 8601 **with** `Z` or an offset. A time without one is
  refused. Or `schedule_in_minutes` for relative times ("in 2 hours" → 120).
  Omit both to save a draft.
- `media_ids` (from `GET /media`) and/or `media_urls` (public https links,
  downloaded into the user's media library). See the postbase-media skill.
- `youtube`: `{ "title", "privacy": "public|unlisted|private", "made_for_kids" }`.
  YouTube needs a video and `made_for_kids` (ask the user, never guess).

Response `201` with `{ "post": { "id", "status", "scheduled_at", ... } }`, or
`200` with `"idempotent_replay": true` when a key was reused. A `warning` (for
example about X charging for links) should be passed on to the user.

## Rules that will reject a post

- TikTok can only be saved as a **draft** through the API: the creator must pick
  who sees it in the Postbase composer. Tell the user to schedule it there.
- Times in the past, text over a network's limit, or media a network can't take
  return `400` with the reason. Fix and retry (new key if the content changed).
- No active plan: `402`. Rate limits: `429` with `Retry-After` (120 requests a
  minute, 60 posts an hour per workspace).
- Errors look like `{ "error": "<readable message>", "code": "<code>" }`. Show
  the user the message.

Docs: https://docs.postbase.so/api/posts
