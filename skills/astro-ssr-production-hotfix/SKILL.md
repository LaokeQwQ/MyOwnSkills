---
name: astro-ssr-production-hotfix
description: Safely patch, validate, package, and release a production Astro SSR application running in Docker/Compose. Use when fixing a live Astro/Node SSR site, creating a hotfix release, rebuilding a container from an existing release directory, preserving rollback, or pushing changes through dev/canary/master-style branch promotion.
---

# Astro SSR Production Hotfix

Use this workflow for live Astro SSR applications where a small UI/API/config bug must be fixed without losing rollback capability.

## Workflow

1. Establish the live state before editing:
   - Identify current release path, compose files, env file, container name, published port, healthcheck, and rollback artifact.
   - Confirm which files are ignored and must not be committed: `.env`, local logs, `dist`, `node_modules`, private runbooks, uploaded packages.
   - Record the exact user-visible bug and the page/route where it occurs.

2. Patch the source of truth:
   - Edit the tracked source file, not only built output.
   - For an urgent server-side patch, mirror the same change into the current release directory and then rebuild/recreate the container.
   - Keep a timestamped backup of the changed production file before using `sed`/`perl` on the server.

3. Validate locally:
   - Run `npm run build` in the Astro app.
   - Search for leaked secrets before committing:
     `git grep -n "SECRET_VALUE_OR_PATTERN" -- . ':!*.example'`.
   - Verify version strings and release notes if the hotfix changes a release version.

4. Promote through branches:
   - Commit only intended tracked files to `dev`.
   - Push `dev`, open PR `dev -> canary/cannary`, wait for CI, merge.
   - Open PR `canary/cannary -> master`, wait for CI, merge.
   - Do not jump directly from `dev` to `master` when branch policy forbids it.

5. Tag and package from production branch:
   - Tag `origin/master`, not the local working tree.
   - Create deployment packages with `git archive origin/master`.
   - Generate SHA256 sidecar files and upload both artifacts to the release.

6. Production hotfix deploy:
   - Rebuild with the same compose project and env file.
   - Recreate only the target service.
   - Wait for health to become `healthy`.
   - Check local loopback, public route, API JSON route, and logs.

## Server Patch Pattern

Use a script instead of pasting `set -e` commands directly into an interactive SSH shell:

```bash
cat >/root/hotfix.sh <<'SH'
#!/usr/bin/env bash
set -Eeuo pipefail

APP=/opt/app
RELEASE="$(readlink -f "$APP/current")"
ENV_FILE="$APP/shared/.env"
BK="$APP/backups/hotfix-$(date +%F-%H%M%S)"
mkdir -p "$BK"

FILE="$RELEASE/path/to/source"
cp -a "$FILE" "$BK/source.before"
sed -i 's/old/new/' "$FILE"

cd "$RELEASE"
ln -sfn "$ENV_FILE" .env
docker-compose -p project --env-file .env -f docker-compose.prod.yml build service
docker-compose -p project --env-file .env -f docker-compose.prod.yml up -d --no-build --force-recreate service
docker inspect -f '{{.Name}} {{.State.Status}} {{if .State.Health}}{{.State.Health.Status}}{{end}}' container
SH
bash /root/hotfix.sh || echo "script failed; SSH session kept alive"
```

## Stop Conditions

- New container is not healthy.
- Public route or local route returns non-200 for core pages.
- Logs contain repeated errors, redirect loops, missing env, or SSR exceptions.
- The patch needs a secret or real origin IP committed to Git.
