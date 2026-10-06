---
id: T012
title: Runner with an allowlist of named jobs
status: backlog
branch: work/runner
priority: medium
created: 2026-10-06
due:
---

## Goal
Give Hermes a small set of named, deterministic jobs it can call, so the model never types commands. The runner executes the exact command and returns the raw output untouched.

## Acceptance
- [ ] Start from Boss's earlier draft (`~/alice/runner-build-plan-2026-10-04.md`); review and update it for this setup
- [ ] First jobs are read-only: for example pool and disk health, last backup results, service up checks, and the duplicate-report summary
- [ ] Each job has a name, an exact command, a time limit and an owner; nothing outside the list can run
- [ ] Output is logged to a file the agent cannot edit
- [ ] Anything that changes a system goes through Boss's approval, and Proxmox changes stay with Boss
- [ ] Hermes calls jobs by name; test that it cannot run anything not on the list

## Notes
The model's job is only to pick a named job from the list. Depends on T005 to T007.
