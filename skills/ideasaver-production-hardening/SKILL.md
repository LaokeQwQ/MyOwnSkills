---
name: ideasaver-production-hardening
description: Use when hardening IdeaSaver-like file platforms for production readiness, especially for moderation/appeal workflows, direct-link access control, security bugfixes, release checklists, rollback planning, and smoke-test design.
---

# IdeaSaver Production Hardening

## Workflow

1. Baseline validation
- Run backend and frontend builds first.
- Capture concrete failures before patching.

2. Security and reliability fixes
- Validate auth behavior for browser limitations (for example EventSource/image preview token paths).
- Validate ownership checks on destructive operations.
- Validate UUID/input parsing and fail fast on invalid IDs.
- Validate direct-link behavior for deleted/banned resources.

3. Moderation and appeal implementation
- Add file moderation status to schema and models.
- Add appeal tickets with pending-state uniqueness.
- Implement user appeal submit and admin review endpoints.
- Enforce banned resource blocking in direct link + preview.
- Persist audit log entries for moderation actions and review outcomes.

4. Production release assets
- Create release checklist, rollback plan, and smoke scripts.
- Add CI checks for `go test ./...`, `go build ./...`, and frontend build.
- Remove packaging noise (`web/node_modules`, `web/dist`) before source archive delivery.

## Validation Standard

- Backend: `go test ./...` and `go build ./...` must pass.
- Frontend: `npm run build` must pass before cleanup.
- API behavior: moderation and appeal flow must be manually smoke tested.

## References

- Read [references/playbook.md](references/playbook.md) for concrete API/routes, query patterns, deployment steps, and regression checklist.