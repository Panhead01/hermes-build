---
id: T011
title: Memory harness comparison on a fixed task set
status: backlog
branch: work/memory-harness-test
priority: high
created: 2026-10-06
due:
---

## Goal
Compare memory harnesses fairly: none, plain markdown files with evidence counts, Hindsight, and Hermes's own memory if it has one.

## Acceptance
- [ ] A fixed set of 15 to 20 tasks written: recall across sessions, a fact that changes, a conflict between memory and a new observation, and a few routing and summary jobs
- [ ] A scoring sheet agreed with Boss before any run (what counts as correct)
- [ ] Each harness run on the same tasks; results in a table in `docs/02-research/harness-results.md`
- [ ] Measured per run: correctness, speed, video memory, Claude tokens, setup and upkeep effort, anything that leaked
- [ ] A recommendation to Boss (keep, drop, or combine)

## Notes
Uses the evidence levels hint, pattern and rule from the Alice Target Map. Depends on T005 to T009. The local model runs the volume; Claude judges.
