---
name: cdn-origin-host-override
description: Configure server-side services to bypass CDN/ESA/DogeCloud/Cloudflare protection while keeping application URLs, logs, Host headers, and public links on the canonical domain. Use when JSON APIs, assets, avatars, or SSR fetches hit redirect loops, WAF challenges, CDN anti-bot pages, or need origin-only access from Docker/Compose.
---

# CDN Origin Host Override

Use this skill when a backend service must fetch a public site through its origin without exposing origin IPs in app URLs or public repositories.

## Preferred Pattern

Keep public URLs and app config domain-based:

```env
PUBLIC_URL=https://service.example.com
API_URL=http://service.example.com:ORIGIN_PORT
API_HOST=service.example.com
```

Then override DNS inside the container:

```yaml
services:
  app:
    extra_hosts:
      - "host.docker.internal:host-gateway"
      - "service.example.com:ORIGIN_IP"
```

This keeps:

- User-facing links on `https://service.example.com`.
- Request Host header on `service.example.com`.
- Server-side route to `ORIGIN_IP:ORIGIN_PORT`.
- Origin IP out of public `.env.example` and tracked docs when needed.

## Implementation Checklist

1. Split public and fetch URLs:
   - Public links use `PUBLIC_URL`.
   - SSR/API fetches use `API_URL`.
   - If fetching an IP directly, set an explicit `Host` header.
   - Prefer domain + `extra_hosts` over IP URLs for maintainability.

2. Use a private compose override:
   - Commit only `docker-compose.origin-hosts.example.yml`.
   - Store the real override on the server, for example `/opt/app/shared/docker-compose.origin-hosts.yml`.
   - Start with both files:
     `docker-compose -f docker-compose.prod.yml -f /opt/app/shared/docker-compose.origin-hosts.yml up -d`.

3. Validate from inside the container:
   - `docker exec app sh -lc "grep service.example.com /etc/hosts"`.
   - `curl -I http://127.0.0.1:APP_PORT/path`.
   - Check the specific JSON route and at least one proxied asset.

4. Handle assets:
   - Rewrite origin asset paths through an app-owned proxy route when public CDN asset URLs redirect-loop.
   - Cache proxied assets with conservative `Cache-Control` and preserve content type.

5. Prepare CDN refresh URLs:
   - Include full URLs for HTML routes, API routes, static assets, favicon/logo, and representative proxied assets.
   - Keep one URL per line for CDN tooling.

## Red Flags

- `ERR_TOO_MANY_REDIRECTS` on emoji/avatar/upload assets.
- JSON endpoints returning HTML challenge pages.
- Public API works in browser but SSR fetch fails.
- `.env` or docs contain raw origin IPs where a private compose override should be used.
