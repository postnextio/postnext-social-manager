---
name: postnext-social-manager
description: Manage social media through PostNext - schedule, publish, and analyze posts across Twitter/X, Instagram, LinkedIn, Threads, YouTube, TikTok, and Bluesky via the PostNext public API. Use when the user wants to draft, schedule, or publish social posts, upload media for posts, check what is queued, or read social analytics (engagement, best time to post, audience growth) and has a PostNext account and API key.
---

# PostNext Social Manager

Drive a PostNext account programmatically: connect channels, upload media, compose and schedule or publish posts across platforms, and read analytics. Wraps the PostNext public REST API with an API key.

## Setup

1. The user creates an API key at https://postnext.io/account/api-keys (shown only once).
2. Export it: `export POSTNEXT_API_KEY=<key>`
3. Base URL is `https://api-app.postnext.io`. Auth header on every request: `x-api-key: $POSTNEXT_API_KEY`.

Two ways to use this skill:
- **Helper (recommended where a shell is available):** the bundled `postnext` script (needs `curl` + `jq`). It handles the fiddly, error-prone parts for you. Run `./postnext help`.
- **Raw curl (works anywhere, including no-filesystem sandboxes):** every operation has a copy-paste curl recipe in `references/`.

## The core loop

1. **Connect** (one-time, done by the user in the PostNext web app - there is no API to connect a channel). Confirm what is connected: `postnext channels`.
2. **Upload media** if the post has an image or video: `postnext upload ./clip.mp4` prints an asset URL.
3. **Compose + publish now** or **schedule**:
   - `postnext post --provider twitter --text "..." --media ./a.png`
   - `postnext schedule --provider instagram --text "..." --media ./a.jpg --at 2026-07-25T14:00:00Z`
4. **Measure**: `postnext analytics overview`, `postnext analytics best-time twitter`, `postnext results`.

## Four traps this skill exists to prevent

The PostNext API has four behaviors that silently produce broken posts. The `postnext` helper handles all four; if you use raw curl, you must handle them yourself (see `references/posts.md` and `references/media.md`).

1. **Key posts by the bare provider name** (`twitter`), never `twitter:@handle`. A colon key is rejected at publish. Select the account with the entry's `providerId` + `channelName`.
2. **Attach media by its URL**, never by `assetId`. A post that carries only `assetIds` publishes with no media and no error. Put the uploaded `asset.url` into `content.media`.
3. **`providerId` is the connection's `providerId`** (the platform-native account id from `GET /api/connections`), not `uniqueId`. A wrong or missing `providerId` fails at the publish worker silently.
4. **Only twitter and threads thread.** An array of posts sent to any other provider is silently reduced to the first item.

## When to read which reference

| Task | Read |
|------|------|
| Build a post, threads, per-provider rules, char limits | `references/posts.md` |
| Upload an image or video, size routing, the 3-step flow | `references/media.md` |
| List or check connected channels, resolve `providerId` | `references/channels.md` |
| Read analytics, best time to post, sync | `references/analytics.md` |
| Response envelope shapes, error codes, rate limits | `references/errors.md` |

## Known limitations

- No channel connect over the API (browser-only in PostNext).
- No team switching: the key operates on its account's default team.
- One entry per provider per request (no multi-account-per-provider in a single post).
- Analytics and per-post metrics require PUBLISHED posts.
- Not supported by this skill: WordPress/blog posts, brand-profile edits, team management (use the PostNext MCP or web app).
