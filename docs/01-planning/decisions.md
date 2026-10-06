# Decisions log (newest first)

## 2026-10-07: Revised Workstation Agent Map adopted as the target design; kick off with the engine
- **Context:** Boss reviewed the revised map from another chat and asked for it to be recorded and the build started.
- **Decision:** the revised map is the destination for tools and the Proxmox tool (brief, section "Target design"), with Alice's two notes. The first build step stays small: a trial guest with the GTX 1080 and one local model, measured; no gatekeeper, Hermes or Hindsight yet.
- **Reason:** the engine is the biggest unknown (GPU passthrough, Pascal support, real speed) and is fully reversible. Boss runs the Proxmox steps; Alice writes them and does everything inside the guest.

## 2026-10-06: Hermes is the always-on layer; approval gates; tools and runners
- **Context:** the unattended-Claude question (subscription limits and terms) made an always-on Claude bot uncertain, and Boss put the phone link on hold.
- **Decision:** the always-on assistant is Hermes with a local model, so nothing unattended touches the Claude subscription. Claude is used interactively by Boss, or by Hermes only through a capped API key. Approval gates always apply. Deterministic tools and allowlisted runner jobs are preferred to model improvisation.
- **Reason:** no usage-limit or terms risk, low running cost, and a bounded blast radius if the small model errs. Matrix with Synapse and the phone link are tabled for later.

## 2026-10-06: Memory-harness testing ground; Claude orchestrates a local model; Notion and Obsidian set aside
- **Context:** Boss redirected the build after reviewing his Claude artifacts on agent design.
- **Decision:** Hindsight is the first memory harness; Claude Code is orchestrator and judge; a small local model on the GTX 1080 does routine work; Notion and Obsidian are set aside for Hermes. Version 1 compares harnesses (none, plain files, Hindsight, Hermes's own) on one fixed set of tasks (T011).
- **Reason:** the goal is to learn which memory approach earns its place before spending on hardware. Claude Code, not claude.ai chat, because it can reach home-network services without exposing them.

## 2026-10-06: Trial first, on the workstation's GTX 1080, before spending on AI hardware
- **Context:** Boss wants to see whether Hermes with a memory layer (Hindsight) is worth investing in.
- **Decision:** run a trial in an isolated guest using the GTX 1080 (8 GB), a small local model, then Hindsight; judge it with a scorecard (tasks T005 to T010).
- **Reason:** cheap and reversible; produces real numbers to guide any hardware purchase. Hindsight is only a candidate: its page does not list Hermes, and telemetry and version are unconfirmed.

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