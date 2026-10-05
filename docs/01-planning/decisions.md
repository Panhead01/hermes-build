# Decisions log (newest first)

## 2026-10-06: Scaffold the repo from a lean version of the workspace spec
- **Context:** Boss asked to start the project and checked the structure.
- **Decision:** use README, CLAUDE.md, docs, tasks, materials, workflows and CHANGELOG; leave out the NAS dataset hierarchy, archive and share layers until needed.
- **Reason:** the spec is a generic template; this is one project and the agents do the upkeep.

## 2026-10-06: Agents work through the Gitea connector as `alice`
- **Context:** Alice needs to read and write this repo without SSH or Proxmox access.
- **Decision:** official Gitea MCP v1.8.0 with 22 allowlisted tools (no deletes, no repo creation); token read from a file at runtime. The server refuses pushes to `main` and merges for `alice` (tested).
- **Reason:** two layers of protection; Boss approves every merge.

## 2026-10-05: Gitea 1.27.3 in container 211
- **Context:** needed a self-hosted git server with a connector, no GitHub.
- **Decision:** Gitea (not Forgejo), version 1.27.3 rather than 28.0.0 (no patch releases yet).
- **Reason:** official MCP connector with read-only flag and tool filtering; Forgejo's connectors are community ones with no filtering. Moving between the two later is hard, so this is treated as final.