---
id: T005
title: Create the trial VM or container for Hermes
status: backlog
branch: work/trial-vm
priority: high
created: 2026-10-06
due:
---

## Goal
A new, isolated guest on the workstation for the Hermes trial.

## Acceptance
- [ ] Boss decides VM (GPU passthrough, cleaner host) or container (needs host driver); recorded in `decisions.md`
- [ ] Sizing agreed (suggest 8 cores, 16 GB RAM, 60 GB disk for a VM; Boss to confirm)
- [ ] Guest created by Boss in Proxmox; Alice connects with a dedicated key; backup job entry added
- [ ] Home network only, no port forwards

## Notes
Alice does not touch Proxmox without Boss's explicit go-ahead for each change.
