---
name: postbase-manage-posts
description: Check, edit, reschedule, retry or cancel posts in Postbase through its REST API. Use when asked whether a post went out, to see what's scheduled, to move or change a scheduled post, or to resend a post that failed.
---

# Manage Postbase posts

- Base URL: `https://www.postbase.so/api/v1`
- Auth: `Authorization: Bearer $POSTBASE_API_KEY`. Never print the key.

## See what's queued

`GET /posts?status=scheduled` (or `draft`, `published`, `failed`) lists posts
with their text, time and channels. Match the user's description ("the
LinkedIn post on Friday") to a post id. Ask if more than one fits.

## Did it go out?

`GET /posts/{id}` returns each channel's outcome:

- `status`: `draft`, `scheduled`, `publishing`, `published` or `failed`
- `url`: the live post, once published. Share it with the user.
- `error`: why a channel failed. `retry_at`: when Postbase will retry it on its
  own (null once it has stopped).
- `warning`: it published, with something worth mentioning.
- `metrics`: latest likes, views and so on, where the network reports them.
- `summary`: counts of published, failed and pending channels.

## Edit or reschedule

`PATCH /posts/{id}` with only the fields to change (same fields as creating):

```bash
curl -X PATCH https://www.postbase.so/api/v1/posts/<id> \
  -H "Authorization: Bearer $POSTBASE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "scheduled_at": "2026-10-02T09:00:00+01:00" }'
```

- Reschedule: send `scheduled_at` (ISO 8601 with an offset) or
  `schedule_in_minutes`.
- Back to a draft: `"scheduled_at": null`. Schedule a draft: send a time.
- New text: `body`, `thread` or `channel_bodies`. Change channels: `channel_ids`.
- A post that has started publishing returns `409`: it can't be changed, so
  offer to create a new one instead.

## Retry a failure

`POST /posts/{id}/retry` resends right away to the failed channels only;
channels that published are untouched, so nothing goes out twice. Read the
`error` first: if a channel's connection expired or was revoked, the user has to
reconnect it in Postbase (Channels) before a retry can work. `409` means there
was nothing to retry.

## Cancel

`POST /posts/{id}/cancel` stops a scheduled post and turns it back into a draft
(nothing is deleted). Channels already mid-send finish. The API can't delete
posts or published content.

Docs: https://docs.postbase.so/api/posts
