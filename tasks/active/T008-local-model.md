---
id: T008
title: Serve a small local model on the GPU
status: backlog
branch: work/local-model
priority: medium
created: 2026-10-06
due:
---

## Goal
A 7 to 8 billion parameter model running on the GTX 1080, used by Hermes.

## Acceptance
- [ ] Model server (for example Ollama) installed and pinned; installed from a packaged or vendor source, flagged to Boss first
- [ ] Two or three candidate models tried at 4-bit; tokens per second and video memory use recorded
- [ ] Hermes pointed at the local model and a test task completed

## Notes
8 GB fits roughly 7 to 8 billion parameters at 4-bit (INFERRED). Compare with the desktop RX 9070 (16 GB) only if the 1080 is too limited.
