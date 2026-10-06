# Brief: Hermes build

Status: DRAFT for Boss to edit. Redirected 6 Oct 2026 after Boss set aside Notion and Obsidian for Hermes. Sources: his claude.ai documents "Personal agent architecture, version 2" (3 Oct 2026), "Agent Designs" (4 Oct 2026) and "Alice Target Map" (28 Sep 2026). Anything not from those is marked as Alice's proposal.

## Purpose
A private, self-hosted personal agent on the home workstation, and a **testing ground for memory harnesses**. Claude (Claude Code) acts as the orchestrator and judge; a small local model on the workstation does the routine work; Hindsight is the first memory layer under test. Alice (Claude Code) is also the development collaborator and works through this repo.

## Requirements
- **Capture:** voice or text from Boss's Android phone through a messaging chat (service still undecided). The phone is only the interface.
- **Models:** a small local model (7 to 8 billion parameters on the workstation's GTX 1080, 8 GB) for routing, tagging, summaries and drafts; Claude for judgment, planning and checking. Local calls cost time, not tokens, so a local judge loop becomes affordable (Agent Designs).
- **Memory:** Hindsight first (self-hosted, PostgreSQL with pgvector, MCP server). **Set aside for Hermes: Notion and Obsidian.**
- **Separation:** work and personal stay apart; version 1 is personal-only.
- **Access:** home network only; nothing exposed to the public internet.

## Architecture (proposal, from the artifacts)
```
phone chat -> gateway (allowlist) -> router (local model)
                                       |-- local worker: tag, summarise, draft
                                       |-- Claude Code orchestrator: plan, judge, strict writes
memory harness under test (Hindsight, via MCP)  <-> orchestrator and worker
gate outside the agent: manual approval, tests, read-only reviewer, round caps
```
Designs used from Agent Designs: **router** (a small model sorts requests), **orchestrator with workers** (Claude leads, the local model works), **maker and checker** (local maker, Claude or a read-only checker, round cap 2), **memory harness** (borrow the ideas of Hermes and OpenClaw), **graph with checkpoints** for long builds. Skipped: swarms.
Rules that apply to every design: each agent gets its own write surface (branch or worktree); the gate sits outside the agent (Hermes manual approval mode, not its "smart" mode, which was shown injectable); durable checkpoints in files and git; a log of what actually ran that the agent cannot edit; a failure ledger.

## Operating principles (Boss, 6 Oct 2026)
- **Hermes with the local model is the always-on layer.** Scheduled checks, tagging, sorting and watching run there at no per-token cost and with no dependence on Boss's Claude subscription. Nothing unattended uses that subscription.
- **Claude is interactive, or an API-key helper.** Boss uses Claude Code himself when he is present. If Hermes calls Claude, it does so with an API key (pay-per-use, spend capped), never by copying subscription logins.
- **Approval gates regardless.** Any command, write or change outside the guest needs Boss's approval, enforced in code by Hermes's manual approval mode, not by the model's promise. Always-on jobs start read-only or low-risk, and what they ran is logged where the agent cannot edit it.
- **Tools and runners wherever possible.** Prefer fixed, deterministic tools and named runner jobs over the model improvising commands. The model's job is to turn a loose request into a named job from an allowlist; the runner executes the exact command and returns raw output untouched. Background: `~/alice/runner-build-plan-2026-10-04.md` on Boss's desktop (a draft, never built). See task T012.
- **Costs to accept:** the model holds about 5 to 6 GB of the GTX 1080's 8 GB around the clock, so the GPU is not free for other work; and a small model's mistakes happen unattended, so first jobs are ones where a wrong answer is harmless.

## The learning loop (from Alice Target Map)
Every job: brief (goal, what done looks like, known dead ends, when to stop) -> work -> check (code first, then an independent read) -> deliver and react -> record a lesson. Memory sits in the middle, with evidence levels: **hint** (seen once), **pattern** (2 to 3), **rule** (4 or more, or Boss said so), and a fresh observation can overturn a rule. Blocking signals (stall after two attempts, a large surprise, out of scope, safety floor) go to a step-back, not to a third variation.

## Orchestrator choice: Claude Code, not claude.ai chat (Alice's recommendation)
Claude Code runs on the workstation or desktop, can call the local model and Hindsight through MCP, and can be run headless (`claude -p`, subscription, no API key, as Boss accepted earlier). claude.ai chat cannot reach services on the home network without exposing them, which Boss does not want. claude.ai stays the place for design chats and artifacts.

## Memory harness test plan (task T011)
Same fixed set of tasks run against each harness, scored the same way:
1. **Baseline:** no memory.
2. **Plain files:** markdown lessons with evidence counts, as Alice uses today.
3. **Hindsight.**
4. **Hermes's own memory**, if it has one (to be verified).
Measure: recall accuracy across separate sessions, handling of a fact that changes, handling of a conflict between memory and a fresh observation, speed and video memory on the 1080, tokens used by Claude, upkeep effort, and anything that leaks.

## Version 1 scope (Alice's proposal, for Boss to change)
1. Trial guest on the workstation with Hermes, a local model server on the GTX 1080 and Hindsight, all home-network only (tasks T005 to T009).
2. A dictated or typed note goes to the chat, is routed by the local model, and saved to Hindsight (and to a plain markdown file as the baseline for comparison).
3. Claude Code orchestrates a small set of test jobs and judges them; the local model does the volume.
4. The memory harness comparison (T011) and the scorecard (T010).

**Out of version 1:** Notion, Obsidian for Hermes, the work agent, any work email, remote access beyond the home network, finance, the daily briefing (candidate for a later version).

## Done criteria for version 1 (draft)
- [ ] A note told to Hermes in one session is recalled correctly in a separate session, under each harness tested.
- [ ] The local model and Claude split a test set of jobs, with the split and Claude's token use recorded.
- [ ] The harness comparison table is filled in (T011) and Boss has a recommendation.
- [ ] Speed and video memory on the GTX 1080 measured (T010).
- [ ] Nothing the agents can reach lets them merge to `main`, change Proxmox, or read finance or work-confidential data.
- [ ] Everything is documented here with undo steps.

## Constraints
- Home network only. No Proxmox changes by agents without Boss's explicit go-ahead for each change.
- Credentials are entered by Boss at hidden prompts; agents never see them.
- Finance data stays out. Work confidential data stays out; ideas and methods may move freely, company data may not. Work email stays with Boss's company-approved tool.
- Pin versions and read third-party code before running it.

## Still to decide (see tasks)
- Messaging service (T003).
- VM or container, and sizing (T005); GPU access method (T006).
- Whether Hermes is the agent shell at all, or whether Claude Code plus Hindsight is the whole test rig. Hindsight's page does not list Hermes (T009 checks).
- Write permissions: confirm-before-write for anything that sends or changes an external system (Alice's recommendation).
- Failure mode when the workstation is off.

## Background
Earlier research on Hermes Agent (Nous Research, MIT licence, v0.21.5 on 24 Sep 2026) is in section 8b of `~/alice/truenas-full-build-map-2026-10-04.md` on Boss's desktop (large codebase and installer; secrets kept in `~/.hermes/.env`); re-checked in T007. An earlier Hermes on a VPS was closed on 13 Sep 2026 and is unrelated.
