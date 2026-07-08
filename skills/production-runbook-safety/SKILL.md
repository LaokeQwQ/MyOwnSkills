---
name: production-runbook-safety
description: Use when writing, reviewing, or executing production server runbooks, deployment/migration steps, backups/restores, Docker/Compose operations, database migrations, kernel updates, or commands that may affect live services. Enforces preflight checks for paths, disk space, images, env vars, dependencies, backups, rollback, and validation before suggesting or running commands.
---

# Production Runbook Safety

Use this skill before giving production commands or editing an operational runbook. Treat every server as production until proven otherwise.

## Non-Negotiable Workflow

1. Identify the blast radius before commands:
   - OS/distribution, shell, package manager, service manager.
   - Relevant containers/services, restart policies, ports, networks, volumes.
   - Whether a database is single-app or shared by other services.
   - Which commands mutate state, restart services, pull images, delete files, or consume large disk.

2. Convert assumptions into checks:
   - Paths: use `test -e`, `find`, or `ls` before putting paths in `tar`, `rsync`, `rm`, `mv`, or config edits.
   - Disk: use `df -h`, `du -sh`, and estimate backup size before creating archives/dumps.
   - Variables: verify required vars are non-empty and point to existing dirs/files before use.
   - Docker image tags: derive from existing containers with `docker inspect`; do not hard-code a tag if an equivalent local image exists.
   - Compose flavor: verify `docker-compose` vs `docker compose`.
   - Network: inspect Docker networks and do not assume `bridge`, `172.17.0.1`, or `host.docker.internal`.

3. Design backups before changes:
   - Create backups in a named directory and record the path.
   - Exclude irrelevant bulk directories (`node_modules`, caches, logs) unless explicitly needed.
   - Use an include list and skip missing paths intentionally; missing optional paths must not fail the whole backup.
   - Remove partial failed artifacts before retrying.
   - Validate backups with list/restore checks, not just file existence.
   - Generate checksums after validation.
   - Recommend off-host copy for production backups.
   - Distinguish online safety backups from final zero-write migration backups.

4. For database migrations:
   - Stop application writers, not the source database, before final logical dumps.
   - Before stopping a database, prove whether it is shared by other services.
   - If stopping a shared PostgreSQL instance, migrate roles and all required business databases, not only one app database.
   - Preserve owner and ACL for full-instance migrations. Do not use `--no-owner` / `--no-acl` unless intentionally doing a single-app ownership remap.
   - Use matching major-version client tools where possible.
   - Validate with `pg_restore --list` and, for real cutovers, restore tests or app smoke checks.

5. For Docker/Compose operations:
   - Never use broad commands like `docker stop $(docker ps -q)` in production runbooks.
   - If container env or compose config changes, recreate the container; `docker start` alone will not reload environment variables.
   - Use `--pull=never` for validation commands when avoiding network pulls and disk surprises.
   - Stop automation/update containers during maintenance windows if they may restart workloads unexpectedly.

6. State stop conditions:
   - Missing required path.
   - Low disk or unknown backup size.
   - Empty critical variable such as `BACKUP`.
   - Unknown database dependency graph.
   - Command tries to pull/build unexpectedly.
   - Backup validation fails.
   - A proposed single-app migration is actually touching a shared service.

## Command Patterns

Validate a backup variable:

```bash
echo "BACKUP=$BACKUP"
test -n "$BACKUP" && test -d "$BACKUP" && echo "BACKUP OK" || {
  echo "BACKUP is empty or missing"; exit 1;
}
```

Build a tar from existing paths only:

```bash
CONFIG_LIST="$BACKUP/config-paths.txt"
sudo awk '
  { if (system("[ -e \"" $0 "\" ]") == 0) print $0; else print "missing, skipped: " $0 > "/dev/stderr" }
' "$CONFIG_LIST" | sudo tar czf "$BACKUP/config-files.tgz" \
  --exclude='*/node_modules/*' \
  --exclude='*/.cache/*' \
  --exclude='*/logs/*' \
  --files-from=-
```

Use the existing PostgreSQL image for validation:

```bash
PG_IMAGE=$(docker inspect -f '{{.Config.Image}}' POSTGRES_CONTAINER)
docker run --rm --pull=never -v "$BACKUP:/backup:ro" "$PG_IMAGE" \
  pg_restore --list /backup/db.dump >/dev/null
```

## Runbook Review Checklist

Before handing a runbook to the user, verify it answers:

- Which exact steps are already completed and should not be rerun?
- Which services are stopped, and why?
- Which services must not be stopped?
- What proves backups are valid?
- What proves the new service is reachable from the real container network?
- What changes require container recreation instead of restart?
- What is the rollback path if the cutover fails?
- Which commands are safe to run online, and which belong only inside the maintenance window?

If a mistake is found during execution, stop the sequence, explain the immediate safe recovery, update the runbook, and only then continue.
