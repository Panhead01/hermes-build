---
id: T009
title: Trial Hindsight as Hermes's memory
status: backlog
branch: work/hindsight
priority: medium
created: 2026-10-06
due:
---

## Goal
Find out whether Hindsight gives Hermes useful long-term memory, self-hosted.

## Acceptance
- [ ] Code read for telemetry and outbound calls before running (page says nothing for self-hosted)
- [ ] Pinned version run in Docker with PostgreSQL and pgvector, in the trial guest only
- [ ] Pointed at the local model, with no cloud component
- [ ] Connected to Hermes through its MCP server, or a note on why that is not possible (the page does not list Hermes)
- [ ] Test: facts told in one session are recalled correctly in another
- [ ] No finance or other sensitive data stored
