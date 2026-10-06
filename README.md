# hermes-build

Planning and build workspace for **Hermes**, the personal agent on Boss's home workstation.

- **Status:** scaffold only. Goal, host, model and messaging service are still undecided (see `tasks/active/`).
- **How we work:** Alice and Hermes work on `work/<topic>` branches and open pull requests; only Boss (`gitea-su`) merges to `main`. Rules for the agents are in `CLAUDE.md`.
- **Private by default:** nothing here is published outside the home network. Secrets never go in the repo (see `.gitignore`).

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

