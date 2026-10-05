---
id: T002
title: Choose where Hermes runs and which model
status: backlog
branch: work/host-and-model
priority: medium
created: 2026-10-06
due:
---

## Goal
Pick a new VM or container on Proxmox (CPU, RAM, disk) and a model: local CPU-only or hosted.

## Acceptance
- [ ] Host decision and sizing recorded in `decisions.md`
- [ ] Model decision recorded, with the reason
- [ ] Boss runs any Proxmox steps himself, or approves them explicitly

## Notes
The workstation has 24 threads and 64 GB RAM (about 62 GB visible), no GPU. Large local models are impractical without one (INFERRED).
