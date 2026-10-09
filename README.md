# hermes-build

> **Read this first.** This repository is a working notebook, not a guide. It records what one person tried on one home server, including the mistakes, dead ends and things that were later undone. Nothing here is advice, a tutorial or a recommended procedure. Settings, commands and numbers were right for that machine on that day and may be wrong, unsafe or out of date for yours. Do not run anything from here on your own systems without understanding it and checking it against the official documentation. The live copy lives on a private server; this is a read-only mirror and pull requests or issues here are not watched.

Planning and build workspace for **Hermes**, the personal agent on Boss's home workstation.

- **Status: CLOSED (9 October 2026).** The Hermes-on-GTX-1080 trial ended on 8 October: Hermes needs a 64K-token context and an 8 GB card cannot hold it (see `docs/02-research/engine-results.md` and the closure note once merged). This repo is kept as a record. Any Hermes work with a bigger card would start a new repo. The GTX 1080 and its VM continue as the separate **1080 AI Playground** project, in its own repo.
- **How we worked:** Alice and Hermes work on `work/<topic>` branches and open pull requests; only Boss (`gitea-su`) merges to `main`. Rules for the agents are in `CLAUDE.md`.
- **Published as a read-only mirror** on GitHub since October 2026 (the home server stays the source of truth). Secrets never go in the repo (see `.gitignore`).

## Layout

| Folder | Purpose |
|---|---|
| `docs/01-planning/` | brief and decisions log |
| `docs/02-research/` | findings and source notes |
| `docs/03-drafts/` | working drafts |
| `tasks/` | one markdown file per task (`active/`, `done/`) |
| `materials/` | inputs and references, listed in `MANIFEST.md` |
| `workflows/` | scripts, prompts, checklists |
| `CHANGELOG.md` | one line per merged unit of work |

The original generic template is `WORKSPACE-SPEC.md` on the NAS (Documents share, Projects/admin). This repo uses a leaner version of it.
