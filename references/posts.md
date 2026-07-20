# Posts

Create a post by sending a `providers` map. Each key is a **bare provider name** and each value is one channel's entry.

## Request shape (CreatePostRequest)

```jsonc
{
  "providers": {
    "twitter": {                              // BARE name. NOT "twitter:@handle".
      "channelName": "@yourbrand",            // the real connected handle (from GET /connections)
      "providerId":  "tw_9f3a1c20e5",         // connection.providerId (NOT uniqueId)
      "content": { "text": "We shipped v2.",
                   "media": [{ "url": "https://cdn.postnext.io/assets/hero.png", "type": "IMAGE" }],
                   "hashtags": ["launch"] },
      "firstComment": "..."                   // instagram + linkedin ONLY; ignored elsewhere
    }
  },
  "title": "Launch",                          // optional, internal label
  "tags": ["launch"]                          // optional
}
```

`providerId` and `channelName` come from `GET /api/connections` (see `channels.md`). Resolve them by matching `provider` + `channelName`. A wrong/missing `providerId` makes the publish worker reject the row silently.

## Three ways to send it

| Goal | Endpoint |
|------|----------|
| Save a draft | `POST /api/posts` |
| Publish immediately | `POST /api/posts/publish` |
| Schedule | `POST /api/posts/schedule` with `"scheduledAt": "<future ISO8601>"` |

Existing post by id: `POST /api/posts/{postId}/publish`, `POST /api/posts/{postId}/schedule`, `POST /api/posts/{postId}/cancel-schedule` (back to draft), `DELETE /api/posts/{groupId}` (removes from PostNext; does not delete an already-published post from the platform).

`userId` and `teamId` are derived from the key - never send them.

Note: fetching a deleted or non-existent post (`GET /api/posts/{id}`) returns HTTP 500 with `message: "Post group not found"`, not 404 (see `errors.md`).

## Single post vs thread

`content` is either an object (single post) or an array (thread).

**Single** (`LegacyPostContent`):
```jsonc
{ "text": "...", "media": [{ "url": "<asset.url>", "type": "IMAGE|VIDEO|GIF" }],
  "hashtags": ["a"], "instagramPostType": "feed|post|story|reel" }
```

**Thread** (array, `twitter` and `threads` ONLY):
```jsonc
[ { "order": 1, "text": "1/", "media": [] },
  { "order": 2, "text": "2/" } ]
```
`order` must be exactly 1..N with no gaps. Sending an array to any other provider silently keeps only element `[0]`.

Attach media by URL (`content.media[].url` or the legacy `content.mediaUrls[]`). `assetIds` is never dereferenced - a post with only `assetIds` has no media.

## Per-provider rules enforced at create time

| Provider | Rules |
|----------|-------|
| instagram | text or media required; caption <= 2200; <= 30 hashtags; `firstComment` <= 2200; `instagramPostType` in feed/post/story/reel |
| bluesky | text or image; <= 300 graphemes; <= 4 images; **no video** |
| linkedin | **no video** (422 UNSUPPORTED_MEDIA_FOR_PLATFORM); IMAGE + GIF ok; `firstComment` <= 1250 |
| twitter | text required; no char limit enforced at create; polls 2-4 options |
| threads, tiktok | no create-time content validator |
| youtube | publish path via this route is UNVERIFIED - test before relying on it |

## Scheduling

`scheduledAt` must be a future ISO8601 datetime (past or now is a 400). `timezone` only affects recurrence and display; it never shifts the fire time.

**Idempotency:** raw `POST /api/posts/{postId}/schedule` will create a duplicate job if called twice, causing a double-publish. Before re-scheduling, check the post is not already scheduled to that time (the `postnext` helper guards a +/- 60s window and refuses past-due times).

## Raw curl

```bash
# publish now
curl -sS -X POST https://api-app.postnext.io/api/posts/publish \
  -H "x-api-key: $POSTNEXT_API_KEY" -H 'Content-Type: application/json' \
  -d '{"providers":{"twitter":{"channelName":"@brand","providerId":"tw_123",
       "content":{"text":"hi","media":[{"url":"https://cdn/x.png","type":"IMAGE"}]}}}}'

# list posts (optionally by status)
curl -sS "https://api-app.postnext.io/api/posts?status=SCHEDULED" -H "x-api-key: $POSTNEXT_API_KEY"

# per-channel publish outcomes
curl -sS https://api-app.postnext.io/api/posts/results -H "x-api-key: $POSTNEXT_API_KEY"
```
