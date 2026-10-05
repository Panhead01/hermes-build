# Brief: Hermes build

Status: DRAFT for Boss to edit. Written 6 Oct 2026.

## Purpose
A personal agent (Hermes) on the home workstation that works alongside Alice, using this Gitea repo as the shared, reviewed place for its work.

## What is already decided (see decisions.md)
- Git server is self-hosted Gitea in container 211; Alice and Hermes have non-admin accounts and cannot push to or merge `main`.

## What is not decided (tasks T001 to T004)
- The goal and what "done" looks like for the first version.
- Where it runs (new VM or container on Proxmox) and its resources.
- Model: local CPU-only model or a hosted one.
- Messaging service (undecided as of 5 Oct 2026).
- How it makes and shows documents and pages locally (an "artifacts" folder and a viewer).

## Constraints
- Home network only unless Boss says otherwise; no Proxmox automation by agents without his explicit go-ahead for that change.
- Credentials are entered by Boss at hidden prompts; agents never see them.
- Finance data stays out.

## Background
Earlier research on Hermes Agent (Nous Research, MIT licence, v0.21.5 on 24 Sep 2026) is in section 8b of `~/alice/truenas-full-build-map-2026-10-04.md` on Boss's desktop. Summary: large codebase and installer, secrets kept in `~/.hermes/.env`, Boss leaned towards it on 4 Oct but the runner and local-model work was shelved the same day. That summary is from Alice's notes, not re-verified.