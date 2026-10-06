# CLAUDE.md: rules for Claude (Alice) in this repo

Purpose: plan and build Hermes, the personal agent on Boss's workstation. Boss owns every decision; agents propose.

## Rules
- Never commit to `main`. Work on a `work/<topic>` branch and open a pull request. Only Boss (`gitea-su`) merges.
- Small commits, format `type(scope): summary`. Types: `research`, `draft`, `plan`, `process`, `config`.
- Never put secrets in the repo: passwords, tokens, API keys, `.env` files, config exports, logs, private hostnames beyond what is already in the docs.
- Keep `materials/raw/` add-only; log every addition in `materials/MANIFEST.md`.
- Update the task file, `docs/01-planning/decisions.md` and `CHANGELOG.md` as part of each unit of work.
- Mark every claim OBSERVED or INFERRED and give confidence (CONFIRMED, LIKELY, GUESS). Say "unknown" instead of guessing.
- Nothing leaves the home network. Do not publish, send or share anything from this repo; hand it to Boss for review.
- Finance material never goes in this repo.

## Session start
Read this file and `tasks/active/`, then summarise status and the next step.

## Where things are
Gitea: http://192.168.0.70:3000 (org `projects`). Workstation notes: `~/Documents/Workstation/` on Boss's desktop. Handoff: `~/Downloads/workstation-handoff.md`.