---
name: remote-deploy-defaults
description: Prefer remote/server-side deployment execution by default. Use when the user says "在服务器上部署", "自部署", "self-host", "self-hosted", "deploy to a server", or similar without explicitly asking for local deployment. Interpret the target environment as a remote server and keep all non-essential deployment steps, commands, and operational actions on the remote host rather than the local machine.
---

# Remote Deploy Defaults

## Overview

Interpret server deployment and self-hosting requests as remote-first unless the user explicitly asks for local deployment. Minimize local execution and keep the working plan anchored to the target server.

## Default Assumption

- Treat the deployment target as a remote machine, VPS, dedicated server, container host, or similar environment.
- Present commands as server-side commands by default, typically to be run after `ssh` into the host or through the host's remote shell.
- Use the local machine only for steps that are genuinely necessary, such as editing repository files, opening an SSH session, transferring artifacts, or preparing assets that cannot reasonably be built on the server.

## Execution Rules

- Prefer remote variants when a task can be done either locally or on the server.
- Do not assume the user's current terminal is the deployment environment unless they explicitly say the deployment is local.
- If a step must run locally, say that clearly and explain why it is necessary.
- When mixing local and remote steps, label each command block as `Local` or `Server` to avoid ambiguity.
- Keep secrets, service configuration, process management, and verification steps centered on the server environment.

## Clarify Only When It Changes the Plan

- Do not stop just to ask whether deployment is local or remote if the request already says server deployment or self-hosting.
- Ask a follow-up only when the remote target materially changes the commands, such as Linux distribution, container vs bare metal, or whether SSH access already exists.
- If the user later says `本地部署` or otherwise requests local execution, stop using this default and switch to a local-first plan.

## Output Style

- Write deployment plans in a remote-first order: connect, prepare server, configure runtime, deploy, start, verify.
- Make the execution context obvious whenever the plan includes both local and remote actions.
- Avoid presenting localhost-only assumptions, desktop tooling, or local package managers as the primary path unless the user explicitly wants them.
