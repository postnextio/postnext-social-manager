# Analytics

All analytics endpoints return `{success, data}`. Several require a paid tier (noted). Metrics exist only for **PUBLISHED** posts - per-post calls error on drafts or scheduled posts.

## Freshness

Stored metrics are periodically snapshotted. To pull fresh numbers on demand:

`POST /api/v1/analytics/sync` (paid) -> `{synced, summary:{twitterSnapped, instagramSnapped, tiktokSnapped, linkedinSnapped}}`. Rate-limited and bounded by per-platform caps. Call it before reading if you need current data.

## Endpoints

| Endpoint | Purpose | Params |
|----------|---------|--------|
| `GET /api/v1/analytics` | last-7-day counts of posts/ideas/assets | - |
| `GET /api/v1/analytics/overview` | totals (impressions, reach, likes, comments, engagementRate) + vsPrevious | `channel`, `period` |
| `GET /api/v1/analytics/posts` | per-post rows | `channel`, `period`, `sort=engagement|impressions|likes|recent` |
| `GET /api/v1/analytics/daily` | per-day activity counts | `days` (1-60) |
| `GET /api/v1/analytics/audience-growth` | follower time series | `channel`, `period` |
| `GET /api/v1/analytics/engagement-over-time` (paid) | engagement rate per day | `channel`, `period` |
| `GET /api/v1/analytics/best-time-to-post` (paid) | ranked posting slots from your own performance | `platform` (required), `timezone`, `topN` |
| `GET /api/v1/analytics/post/{postId}` | latest metrics, one row per provider | - |
| `GET /api/v1/analytics/post/{postId}/daily` | daily snapshots + day-over-day deltas | - |

`period` is one of `7d`, `30d`, `90d`, `all` (default `30d`). `channel` filters to one provider/channel id; omit or `all` for no filter.

## Capability flag

`overview`, `posts`, and `engagement-over-time` may include a `capability` field: `ok`, `insights_unavailable`, `api_tier_limited`, `no_data_yet`, or `unsupported` (it can also be `null`/absent, which means normal). If it is present and not `ok`, tell the user why the numbers are thin (for example a platform whose API tier does not expose insights) rather than presenting zeros as real.

`engagementRate` is a fraction (multiply by 100 for a percentage).

## Examples

```bash
BASE=https://api-app.postnext.io
curl -sS "$BASE/api/v1/analytics/overview?period=30d" -H "x-api-key: $POSTNEXT_API_KEY"
curl -sS "$BASE/api/v1/analytics/best-time-to-post?platform=instagram&timezone=Europe/Bucharest&topN=5" -H "x-api-key: $POSTNEXT_API_KEY"
curl -sS -X POST "$BASE/api/v1/analytics/sync" -H "x-api-key: $POSTNEXT_API_KEY"
```
