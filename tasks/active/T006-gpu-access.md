---
id: T006
title: Give the guest the GTX 1080
status: backlog
branch: work/gpu-access
priority: high
created: 2026-10-06
due:
---

## Goal
The trial guest can run models on the GTX 1080.

## Acceptance
- [ ] IOMMU enabled and the card's IOMMU group checked (read-only look first)
- [ ] Card (and its audio function) passed to the guest, with a recorded way to undo
- [ ] Guest shows the card and a driver version that supports Pascal
- [ ] A simple GPU test passes (for example `nvidia-smi` and a short model run)

## Notes
Check current NVIDIA driver and CUDA support for Pascal before choosing versions. BIOS Internal Graphics is currently disabled.
