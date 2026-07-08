# Playbook

## High-value patch targets

- Route input parsing: reject invalid UUIDs early.
- Share deletion: enforce owner check before delete.
- EventSource/preview auth: allow GET query token fallback where header cannot be set.
- Direct-link handler: resolve by storage key metadata; block banned resources.
- Admin delete: delete storage objects and reclaim quota.

## Moderation/appeal data design

- `files.moderation_status`: `normal|banned`
- `files.moderation_reason`, `moderated_by`, `moderated_at`
- `file_appeals`: `pending|approved|deleted|rejected`
- Add unique partial index to prevent duplicate pending appeals per file.

## API additions pattern

- User:
  - `POST /api/files/:id/appeal`
- Admin:
  - `PUT /api/admin/files/:id/ban`
  - `PUT /api/admin/files/:id/unban`
  - `GET /api/admin/appeals`
  - `PUT /api/admin/appeals/:id/review`

## Audit log events

- `file_banned`
- `file_unbanned`
- `file_appeal_submitted`
- `file_appeal_approved`
- `file_appeal_deleted`

## Release process summary

1. Backup DB and current binary/static files.
2. Run tests/build.
3. Stop service.
4. Replace artifacts.
5. Run migration.
6. Start service.
7. Run smoke script and manual acceptance checklist.

## Rollback strategy summary

- Stop service immediately on critical regressions.
- Restore previous binary/static assets.
- Restore DB from backup only if compatibility requires it.
- Start old version and rerun smoke tests.

## DogeCloud troubleshooting notes

- `video/info.json` is the primary metadata source for list rendering. Persist `thumbnail` and `thumbnail_small` so the UI can show real video covers before playback.
- DogeCloud transcode callback fields must be parsed exactly as documented: `msg`, `vid`, `vcode`, `callbackString`. Success responses must contain the literal string `DogeCloud Callback Success`.
- `video/streams.json` playback URLs are typically under `data.streams[].url`, not only top-level `play_url`.
- If `userId` is missing from callback/info payloads, recover it from the playback URL query string (`userId`, `userid`, or `uid`) and store it for JS player initialization.
- For China-mainland-first delivery, remove Google Fonts and similar third-party webfont/CDN dependencies from the frontend shell. Prefer bundled assets or system font stacks.
- If DogeCloud player delivery becomes a latency issue, vendor the official loader into `web/public/vendor/` and load it from the site's own static origin.
- Do not set `crossorigin` on `https://player.dogecloud.com/js/loader`. The official SDK expects a plain script tag and may appear "loaded but unusable" if the app treats a modal mount race as an injection failure.
- If DogePlayer reports that the SDK is not injected while the network request succeeds, verify the player container exists before constructing `new DogePlayer(...)`. Ant Design `Modal` content can mount after the async loader resolves.
- DogeCloud-backed COS buckets may reject `path-style` addressing with `PathStyleDomainForbidden`. For `*.myqcloud.com` and `*.cos.*` endpoints, use virtual-hosted-style S3 requests.

## SSE and reverse-proxy notes

- Browser console noise from extensions (`start_content.js`, `index.global.js`, stylesheet timeout warnings) is often unrelated. Focus on first-party requests such as `/api/events`.
- `net::ERR_HTTP2_PROTOCOL_ERROR 200` on `/api/events` usually means the SSE stream is being cut by an HTTP/2 proxy chain. Send an initial SSE frame, periodic comment heartbeats, and `Cache-Control: no-cache, no-transform` from the app.
- On the frontend, close stale `EventSource` instances before reconnecting and clear previous reconnect timers to avoid duplicate connections after proxy resets.

## Static delivery and 403 notes

- If `/api/*` works but `/` returns `403`, inspect the reverse-proxy error log for `Permission denied` on `web/dist/index.html` before changing app routes.
- When serving the frontend from `/data/ideasaver/web/dist`, make sure every directory in the real path chain is executable by the web server process and static files are world/group readable.
- For mixed deployments, keep systemd aligned with the actual production layout: `WorkingDirectory=/data/ideasaver` and `EnvironmentFile=/data/ideasaver/.env`.

## File listing and audit-log notes

- If a file upload reports success but the file disappears after refresh, inspect the DB write/read path before blaming OSS. `CompleteUpload` may have succeeded while later list queries fail.
- PostgreSQL `NULL` values must not be scanned directly into Go `string` fields for file records. Use `sql.NullString`/`sql.NullTime` when reading nullable columns such as `storage_key`, `mime_type`, `public_url`, `thumbnail_key`, `moderation_reason`, and moderation timestamps.
- A broken file-list query can look like "upload succeeded but file does not exist", because the frontend may swallow list errors and show an empty directory.
- Direct-link resolution by `storage_key` can also fail with the same nullable-column scan bug, causing false `404 文件不存在` even though the object exists in storage.
- PostgreSQL `INET` columns do not accept empty strings. When audit log IPs are unavailable, insert `NULL`, not `""`; otherwise audit writes fail silently if the error is ignored.
- After fixing audit log insertion bugs, only new operations will appear in history. Missing historical rows are usually not recoverable unless they were captured elsewhere.
- If audit log IPs are expected to be present, inject request metadata at the HTTP middleware layer and let repository writes fall back to `ClientIP` / `User-Agent` from context. That keeps service-layer logging calls simple and avoids widespread handler plumbing.
- Admin log search usually needs more than an action selector. Add backend filters for `action`, `user`, `keyword`, `start_at`, and `end_at`, then keep the SQL parameterized with generated placeholders instead of string interpolation.
- Nullable audit/file fields are a common source of "UI shows nothing" bugs. When list pages mysteriously go blank after otherwise successful writes, inspect DB scans before blaming storage or frontend state.

## Thumbnail and file-manager notes

- DogeCloud image thumbnail rules such as `/thumb` are relative to the OSS object URL, not the app's business direct link. Do not teach users to append `/thumb` to their copied `/s/...` links.
- For app-side file grids, proxy thumbnail generation through the backend or fetch it with authenticated requests, rather than exposing raw OSS-style thumbnail URLs as user-facing links.
- If the app uses JWT-in-header auth, avoid introducing new `<img src=\"...token=...\">` surfaces for thumbnails. Prefer lazy-loading blobs with authenticated `fetch`/Axios requests and convert them to object URLs in the browser.
- Keep rename semantics at the metadata layer only. The storage object key should stay random and immutable, while the user-visible `name` can change freely.
- Consolidating file actions into shared menu builders reduces UI drift between table view, grid view, and right-click menus. Keep copy/cut/delete/rename/share/link actions defined once and reuse them everywhere.
- Ant Design tables that look fine on desktop can silently break mobile shells. Combine `min-width: 0` on layout containers, breakpoint-aware column hiding, and `scroll.x` on tables to keep admin pages, trash pages, and share pages inside the content frame.
- If a file table's mobile name column appears blank, inspect the flex container inside the cell. The usual fix is a `min-width: 0` wrapper around the icon/name stack plus moving secondary metadata into the name cell on small screens.

## Admin capability notes

- Global admin file/video pages should bypass owner-only checks for preview, direct-link copy, download, playback info, enable/disable, and delete actions, while still respecting deleted/banned state where appropriate.
- When adding admin bypasses, also record audit events that include both the acting admin and the original owner in `details`, so review pages preserve accountability.
- Admin file/video review pages become hard to use quickly without owner labels and fuzzy search. Prefer joining `users.username` into admin list queries and support keyword search over file name, extension-in-name, video title, `vcode`, `vid`, and username.

## DogeCloud sharing notes

- For user-facing video sharing, prefer DogeCloud player-page URLs over raw file-playback URLs. A stable share format is `https://player.dogecloud.com/web/player.html?vcode=...&userId=...`.
- Use `inFrame=true` only when generating iframe embed code; normal share links should stay as plain player-page URLs.
- Keep autoplay off by default. If the UI exposes an autoplay toggle, only append `autoPlay=true` when enabled; omitting the parameter is cleaner for the default state.
- If the deployment enables anti-hotlink protection, do not surface direct-access player URLs in the product UI. Generate iframe embed code instead, and let operators copy HTML/Markdown-safe snippets with configurable width, height, and autoplay.
- DogeCloud `video/info` responses can be inconsistent across fields. When persisting playback counts, extract them defensively from candidates such as `play_count`, `playCount`, `view_count`, `viewCount`, or `pv`.
