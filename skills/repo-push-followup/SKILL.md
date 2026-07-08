---
name: repo-push-followup
description: Use when substantial code changes have been completed in a git/GitHub-backed repository. Before ending the turn, proactively ask the user whether they want the validated changes committed, pushed, or synced to target branches. Keep the prompt concise and do not push without explicit approval.
---

# Repo Push Follow-Up

## Trigger

Use this after meaningful repository work is complete, especially when:
- multiple files changed
- a feature, bugfix, migration, UI refresh, or release artifact was produced
- tests/builds have been run or deployment steps were prepared
- the repository is clearly backed by git or GitHub

## Behavior

Before the final handoff, ask one concise question about repo sync, such as whether the user wants the changes:
- committed locally
- pushed to the current branch
- synced from one branch to another
- packaged first and pushed later

## Guardrails

- Ask after validation, not before the work is done.
- Keep the prompt short if the user prefers low-text interactions.
- Do not commit or push without explicit user approval.
- If branch rules or CI constraints exist, mention them briefly only when relevant.
