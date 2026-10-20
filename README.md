# MyOwnSkills

Personal and modified agent skills maintained by LaokeQwQ.

This repository is structured as a multi-skill package. Install all skills with:

```bash
npx skills add LaokeQwQ/MyOwnSkills --all
```

List available skills without installing:

```bash
npx skills add LaokeQwQ/MyOwnSkills --list
```

Install one skill:

```bash
npx skills add LaokeQwQ/MyOwnSkills --skill github-readme-writer
```

## Skills

| Skill | Purpose |
| --- | --- |
| `ask-questions-if-underspecified` | Ask the minimum useful clarifying questions before implementation when a request is under-specified. |
| `astro-ssr-production-hotfix` | Safely patch and release production Astro SSR apps running in Docker/Compose. |
| `backend-development` | Backend API, schema, auth, caching, rate limiting, and observability patterns. |
| `cdn-origin-host-override` | Route server-side fetches to an origin while preserving canonical public domains and Host headers. |
| `dogecloud-cdn-api-ops` | Work with DogeCloud CDN domain listing, cache refresh APIs, signing, and operational debugging. |
| `github-readme-writer` | Write, rewrite, and audit GitHub project READMEs from repository evidence. |
| `ideasaver-production-hardening` | Harden IdeaSaver-like file platforms for moderation, appeals, rollback, and smoke testing. |
| `parallel-research-gap-audit` | Run parallel research subagents, cross-check their findings against the current codebase, and consolidate a verified gap report. |
| `production-runbook-safety` | Review production runbooks, backups, migrations, Docker operations, and rollback steps safely. |
| `remote-deploy-defaults` | Treat self-hosting and server deployment requests as remote-first by default. |
| `repo-push-followup` | Prompt for commit/push/sync follow-up after substantial validated repository work. |
| `darwin-skill` | Evaluate and iteratively improve skills with rubric-based scoring, validation gates, and result cards. |
| `ui-layout-responsive-hardening` | Harden dashboards and CRUD/admin UIs against responsive layout regressions. |

## Layout

```text
skills/
  ask-questions-if-underspecified/
  astro-ssr-production-hotfix/
  backend-development/
  cdn-origin-host-override/
  dogecloud-cdn-api-ops/
  github-readme-writer/
  ideasaver-production-hardening/
  parallel-research-gap-audit/
  production-runbook-safety/
  remote-deploy-defaults/
  repo-push-followup/
  darwin-skill/
  ui-layout-responsive-hardening/
```

Each skill keeps its own `SKILL.md` and any references, scripts, assets, or templates it needs.

## Notes

- `github-readme-writer` was also published as a standalone repository at `LaokeQwQ/README.skills`.
- `darwin-skill` is a locally modified 2.0 version and includes its own README, references, scripts, templates, and assets.
- This repository intentionally includes only selected personal or modified skills, not the full local installed skill set.
- Some included skills preserve attribution in their own files when adapted from external sources.

## License

Apache-2.0. See [LICENSE](LICENSE).
