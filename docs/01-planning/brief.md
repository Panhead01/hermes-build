# Brief: Hermes build

Status: DRAFT for Boss to edit. Updated 6 Oct 2026 from his "Personal agent architecture, version 2" document (3 Oct 2026, claude.ai). Anything not from that document is marked as Alice's proposal.

## Purpose
A private, self-hosted personal agent that captures what Boss dictates on his phone, keeps work and personal apart, and writes to Obsidian (thinking) and Notion (actions). It runs on the home workstation; the phone is only the interface. Alice (Claude Code) is the development collaborator and works through this repo.

## Requirements (from the architecture document)
- **Capture:** voice or text from a Samsung Android phone through a messaging chat.
- **Always-on feel:** acts only when Boss is active; the phone never hosts the agent.
- **Separation:** work and personal are separate agents in separate VMs, with separate vaults and credentials; they cannot reach each other.
- **Models:** small local models for routine jobs (transcription, tagging, summaries); a closed model as orchestrator for judgment and strict structured writes.
- **Memory:** Obsidian is the memory layer, shared with Claude Code. Hindsight is a candidate for the agent's own long-term memory (task T009).
- **Access:** reachable from Boss's devices without exposing anything to the public internet.

## Version 1 scope (Alice's proposal, for Boss to change)
Personal agent only, in one isolated guest on the workstation:
1. Dictate or type a note to a chat; it is transcribed (if voice), tagged and saved to the personal Obsidian vault.
2. A daily brief to the phone from the personal Gmail and Google Calendar, plus the work calendar feed as titles and times only.
3. Hindsight tried as memory, with a test that something told in one session is recalled in another.
4. Runs on the workstation's GTX 1080 with a small local model where it can; the closed model only for judgment.

**Out of version 1:** the work agent, any Notion writes, any work email, any remote access beyond the home network, finance.

## Done criteria for version 1 (draft)
- [ ] A dictated note appears correctly in the personal vault within a minute or two, and is recalled later by Hermes.
- [ ] The daily brief arrives on the phone at a set time, with the right events and no descriptions, passcodes or personal details from work entries.
- [ ] Hindsight recall test passes across two separate sessions.
- [ ] Measured: speed and video memory use of the local model on the GTX 1080 (scorecard T010).
- [ ] Nothing the agent can reach lets it merge to `main`, change Proxmox, or read finance or work-confidential data.
- [ ] Everything is documented here, with undo steps.

## Constraints
- Home network only. No Proxmox changes by agents without Boss's explicit go-ahead for each change.
- Credentials are entered by Boss at hidden prompts; agents never see them.
- Finance data stays out. Work confidential data stays out; ideas and methods may move freely, company data may not.
- Work email stays with Boss's company-approved tool (Copilot), not this agent.

## Still to decide (see tasks)
- Messaging service (T003). The architecture document names Telegram; Boss said it is undecided.
- VM or container for the trial guest, and sizing (T005).
- What, if anything, the closed orchestrator may see from the work side (open decision in the document; version 1 sidesteps it).
- Write permissions: confirm-before-write for anything that sends or changes an external system (Alice's recommendation).
- Failure mode when the workstation is off: queue messages or wait.

## Phases (document, with this repo's trial tasks)
The document proposes: boot the board (done), prototype the capture flow, build on the workstation, add the work agent, add the rest. This repo does the prototype on the workstation itself, on the GTX 1080, as tasks T005 to T010, so the Fedora PC prototype step is replaced by the trial guest.

## Background
Earlier research on Hermes Agent (Nous Research, MIT licence, v0.21.5 on 24 Sep 2026) is in section 8b of `~/alice/truenas-full-build-map-2026-10-04.md` on Boss's desktop (large codebase and installer; secrets kept in `~/.hermes/.env`); that is from Alice's notes and is re-checked in T007. An earlier Hermes on a VPS was closed on 13 Sep 2026 and is unrelated.
