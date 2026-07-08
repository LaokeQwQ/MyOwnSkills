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
| `github-readme-writer` | Write, rewrite, and audit GitHub project READMEs from repository evidence. |
| `repo-push-followup` | Prompt for commit/push/sync follow-up after substantial validated repository work. |
| `darwin-skill` | Evaluate and iteratively improve skills with rubric-based scoring, validation gates, and result cards. |

## Layout

```text
skills/
  github-readme-writer/
  repo-push-followup/
  darwin-skill/
```

Each skill keeps its own `SKILL.md` and any references, scripts, assets, or templates it needs.

## Notes

- `github-readme-writer` was also published as a standalone repository at `LaokeQwQ/README.skills`.
- `darwin-skill` is a locally modified 2.0 version and includes its own README, references, scripts, templates, and assets.
- This repository intentionally includes only selected personal or modified skills, not the full local installed skill set.

## License

Apache-2.0. See [LICENSE](LICENSE).
