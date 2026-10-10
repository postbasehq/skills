---
name: postbase-media
description: Attach images and videos to Postbase social posts, from the user's Postbase media library or from public links. Use when a post needs a picture, screenshot, carousel or video, or a YouTube upload.
---

# Images and video in Postbase posts

- Base URL: `https://www.postbase.so/api/v1`
- Auth: `Authorization: Bearer $POSTBASE_API_KEY`. Never print the key.

## Files the user already uploaded

`GET /media?type=image|video&search=<part of a name>` lists the newest 50 files
in the media library: `id`, `name`, `type`, `size_bytes`, `url`. Pass the ids as
`media_ids` on `POST /posts` or `PATCH /posts/{id}`, in the order they should
appear.

## Files on the web

Pass public `https://` links as `media_urls` when creating or editing a post.
Postbase downloads each one into the user's media library and attaches it after
any `media_ids`. Or add one first with:

```bash
curl -X POST https://www.postbase.so/api/v1/media \
  -H "Authorization: Bearer $POSTBASE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "url": "https://example.com/launch.png", "name": "Launch banner" }'
```

- JPG, PNG, WebP, GIF, MP4, MOV or WebM, up to 200 MB a file, up to 10 links a
  request, 60 an hour per workspace. Files count toward the plan's storage.
- The link must be publicly reachable over https (no login, no localhost or
  private addresses). A local file on this machine can't be sent this way: ask
  the user to upload it in Postbase (Media), which takes up to 1 GB, then use
  its id.

## What each network takes

- X and Mastodon: up to 4 images, or 1 video on its own.
- Bluesky: up to 4 images (videos aren't posted).
- LinkedIn: up to 20 images (videos aren't posted), and it needs some text.
- YouTube: exactly one video, plus `youtube.made_for_kids` (ask the user) and a
  title (`youtube.title`, or the post's first line).
- TikTok: drafts only through the API; the user schedules it in Postbase.

A post with media a network can't take is refused with the reason (`400`).

Docs: https://docs.postbase.so/api/media
