# Media

Upload an image or video, then put the returned **`asset.url`** into a post's `content.media[].url`. The `assetId` is not used by the publish path.

## Which upload path

| Case | Endpoint | Cap | Needs `userId`? |
|------|----------|-----|-----------------|
| image or file <= 50 MB, one of jpeg/png/webp/gif/mp4/webm | 3-step presigned (`create-url` -> PUT -> `complete`) | 50 MB | no (from key) |
| large video (> 50 MB, up to 500 MB) or other type | multipart `POST /api/assets/upload/single` | 500 MB | **yes** = account `uniqueId` |
| a remote image URL | `POST /api/assets/upload/url` | 25 MB | **yes**; **image only** (a video URL is stored as .jpg) |

Get the account `uniqueId` from `GET /api/v1/account` (`.uniqueId`).

## 3-step presigned flow (preferred for images and small video)

```bash
BASE=https://api-app.postnext.io
# 1. request an upload URL (declares type + exact byte size; pre-charges storage quota)
INIT=$(curl -sS -X POST $BASE/api/assets/upload/create-url \
  -H "x-api-key: $POSTNEXT_API_KEY" -H 'Content-Type: application/json' \
  -d '{"filename":"hero.png","contentType":"image/png","sizeBytes":48213}')
UPLOAD_URL=$(echo "$INIT" | jq -r .data.uploadUrl)
UPLOAD_ID=$(echo "$INIT" | jq -r .data.uploadId)

# 2. PUT the bytes. Content-Type MUST equal the declared contentType or S3 rejects the signature.
curl -sS -X PUT "$UPLOAD_URL" -H 'Content-Type: image/png' --data-binary @hero.png

# 3. finalize -> returns the asset; use .data.url in the post
curl -sS -X POST $BASE/api/assets/upload/complete \
  -H "x-api-key: $POSTNEXT_API_KEY" -H 'Content-Type: application/json' \
  -d "{\"uploadId\":\"$UPLOAD_ID\"}" | jq -r .data.url
```

`contentType` allow-list (presigned): `image/jpeg, image/png, image/webp, image/gif, video/mp4, video/webm`. `sizeBytes` must be the true byte size and <= 50 MB (else 413). `complete` verifies the bytes by magic number (415 on mismatch) and reconciles the storage charge to the real size.

## Multipart single (large video)

```bash
UID=$(curl -sS https://api-app.postnext.io/api/v1/account -H "x-api-key: $POSTNEXT_API_KEY" | jq -r .uniqueId)
curl -sS -X POST https://api-app.postnext.io/api/assets/upload/single \
  -H "x-api-key: $POSTNEXT_API_KEY" \
  -F "file=@big.mp4;type=video/mp4" -F "userId=$UID" | jq -r .asset.url
```
Note the different response shape: single/url return a flat `{message, asset}` (no `data` wrapper); presigned `complete` returns `{success, data: asset}`.

## Then attach it

```jsonc
"content": { "text": "...", "media": [{ "url": "<asset.url>", "type": "IMAGE" }] }
```
`type` is `IMAGE`, `VIDEO`, or `GIF`. For a multi-image Instagram carousel, verify whether a top-level `mediaType: "CAROUSEL"` is also required (unverified).

## Manage assets

- `GET /api/assets/mine?page=1&limit=50` -> `{success, data, pagination}` (pagination is top-level).
- `GET /api/assets/{assetId}` -> `{success, data}`.
- `DELETE /api/assets/{assetId}` -> flat `{message, success}`.
