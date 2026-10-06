# Trial plan: Hermes on the workstation GPU, then Hindsight memory

Written 6 Oct 2026. Purpose: find out cheaply whether Hermes with a memory layer earns its place before Boss spends on better AI hardware.

## Facts (with source)
- **Workstation GPU:** NVIDIA GeForce GTX 1080, 8 GB, Pascal (per Boss, 6 Oct 2026; not verified from the machine).
- **Desktop GPU:** AMD RX 9070, 16 GB (OBSERVED 6 Oct 2026). ROCm 7.2 officially supports it (from an earlier research pass, 25 Sep 2026; not re-verified).
- **Workstation:** Core Ultra 9 285K (24 threads), about 62 GB RAM visible, Proxmox 9.2.21, no other GPU known.
- **Hindsight** (github.com/vectorize-io/hindsight, read 6 Oct 2026): agent memory system, MIT licence, runs from Docker (`ghcr.io/vectorize-io/hindsight`), pip (`hindsight-api`) or Helm; stores data in PostgreSQL with pgvector; needs an LLM (25+ providers, or local via Ollama, LM Studio, llama.cpp); has a built-in MCP server; page lists Claude Code, Cursor, LangGraph and others, but does **not** mention Hermes Agent; telemetry for self-hosted not stated; version not stated.
- **Hermes Agent** (Nous Research, MIT, v0.21.5 on 24 Sep 2026, per earlier research, not re-verified): large codebase, installer pulls in Python, keeps secrets in `~/.hermes/.env`.

## Unknowns to settle first
1. Is the GTX 1080 actually usable in a guest (IOMMU group, passthrough, reset behaviour)? INFERRED possible, untested.
2. Does current software still support Pascal (CUDA and driver versions; newer CUDA drops Pascal)? UNSURE; check before installing.
3. VM or container for the GPU? A VM with GPU passthrough keeps the Proxmox host clean (no NVIDIA driver on the host) but gives the card to that VM exclusively. A container needs the driver on the host, which is a host change. Boss decides; Boss runs any Proxmox steps.
4. BIOS: Internal Graphics is disabled (needed for the BMC console); confirm how the host treats the 1080 as primary.

## Order of work
T005 container or VM, T006 GPU access, T007 Hermes install, T008 local model, T009 Hindsight, T010 scorecard.

## Rules for the trial
- Home network only. No messaging service connected yet. No finance data. No credentials seen by agents (Boss types keys at hidden prompts).
- Pin versions. Read the code of third-party tools before running them. Record every step in the task file.
- Stop and report if two attempts at the same step fail.
