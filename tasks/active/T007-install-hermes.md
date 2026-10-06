---
id: T007
title: Install Hermes Agent in the trial guest
status: backlog
branch: work/install-hermes
priority: high
created: 2026-10-06
due:
---

## Goal
Hermes Agent installed and running in the trial guest, with nothing sensitive in it.

## Acceptance
- [ ] Current official install steps fetched and recorded (version, date); the old note of v0.21.5 on 24 Sep 2026 is re-checked
- [ ] Installer and dependencies read or scanned before running
- [ ] Installed with a pinned version; secrets file location noted (`~/.hermes/.env`) and permissions 600
- [ ] Boss types any API key at a hidden prompt; agents never see it
- [ ] Runs in manual mode first
