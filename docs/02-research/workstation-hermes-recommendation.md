# Recommended setup for running Hermes locally on the workstation (8 October 2026)

Three research agents, with the key claims checked against primary sources where marked CONFIRMED. Nothing was bought, installed or changed. Prices are AUD from retailer pages read on 8 October 2026 by the research agent unless stated; used prices are estimates.

## Short answer

- **Recommended:** a 24 GB GPU (used RTX 3090 or 4090, headless) with the existing 128 GB of system RAM, serving **Qwen3.6-35B-A3B** (or Qwen3.6-27B) through llama.cpp with `--jinja` at 64K to 128K context and an 8-bit cache. Safest fit if buying new: Intel Arc Pro B70 32 GB (A$2,199), at about half the speed and with rougher software.
- **Avoid for Proxmox passthrough:** Blackwell cards (RTX 5090, RTX PRO 6000 class): a function-level-reset failure that can hang the host after VM shutdown is reported by several sources; the 5090 is also about A$9,600 or more. 16 GB cards cannot hold the target models.
- **A free test first:** Qwen3.6-35B-A3B has a tiny cache (20 KiB per token, 1.3 GB at 64K, CONFIRMED from the model config) because only 10 of its 40 layers use full attention. With all experts held in system RAM it may run on the existing GTX 1080. The research agent's guess is 20 to 30 tok/s; this has not been measured on this hardware.

## Why 8 GB was not enough (CONFIRMED)

Hermes requires at least 64,000 tokens of context. qwen3:8b needs 144 KiB of cache per token (36 layers, 8 KV heads, head dim 128): 9.66 GB at f16 for 64K. The Qwen3.6 models are different: 35B-A3B needs 20 KiB per token (10 full-attention layers, 2 KV heads, head dim 256) and 27B needs 64 KiB per token (16 full-attention layers, 4 KV heads, head dim 256).

## Requirement tiers

| Tier | GPU | System RAM | Model and context | Expected generation | Basis |
|---|---|---|---|---|---|
| Minimum, no purchase | existing GTX 1080 8 GB | 128 GB | Qwen3.6-35B-A3B Q4, all experts in RAM (`--cpu-moe`), 64K, 8-bit cache | 20 to 30 tok/s (GUESS) | cache maths CONFIRMED; speed unmeasured here |
| Better minimum | 12 GB | 64 to 128 GB | same, about 24 expert layers on CPU | about 40 tok/s, prompt about 900 tok/s | reported on DDR4 rigs |
| **Recommended** | **24 GB (3090/4090)** | **128 GB** | **35B-A3B IQ4/Q4 at 64K to 128K, 0 to 12 expert layers on CPU; or 27B Q4_K_M with 8-bit cache** | **35B-A3B: 85 to 140 tok/s (CONFIRMED on a 3090); 27B: 37 to 50** | benchmark read directly: 3090 CUDA 139.6 gen / 3,360 prompt tok/s all on GPU, 89 gen / 1,153 prompt with 10 layers offloaded |
| Ideal | 32 GB (5090 or B70) | 128 GB | 27B at Q6/Q8 with 128K, or 35B-A3B Q6 resident | about 79 tok/s (estimate) | weak sources |

Memory totals (weights + cache + about 1.5 GB buffers, calculated): 27B Q4_K_M about 21 to 23 GB at 64K with 8-bit cache; 35B-A3B Q4 about 24 to 25 GB; the 3090 benchmark ran 35B-A3B IQ4_NL_XL fully on the card at 89.6K context. Hermes's own managed runtime places overflow in system RAM expert-weights-first and never offers builds below 4-bit (CONFIRMED on its local-models page).

Tool calling: Hermes's Ollama guide names only gemma4:31b as reliable (CONFIRMED); Qwen3.6-27B is the community favourite (anecdotal). No published tool-calling benchmark (BFCL) exists for any candidate, so a reliability test is part of the plan. Qwen3.6 thinks by default; Hermes issue 46131 (empty content from reasoning models) is open, so disable thinking (`enable_thinking=false`) or use the workaround. llama.cpp needs `--jinja` and an explicit `-c` of at least 64000 (`-np 1`).

## GPU shortlist (fit against the case, board and power supply still to be measured)

| Rank | Card | Price (AUD) | Why | Concerns |
|---|---|---|---|---|
| 1 | Used RTX 3090 24 GB | about 1,500 to 1,900 (estimate, UNVERIFIED) | best speed per dollar, mature CUDA, no passthrough reset reports found | about 300 to 340 mm long on most models; the RM41-H08 allows 278 mm with its 5.25" cage fitted and 426 mm without; 350 W heat in a 4U case with 80 mm fans; cap its power |
| 2 | Intel Arc Pro B70 32 GB | 2,199 new | 267 mm, blower, one 8-pin, 230 W, warranty, 32 GB | about half the speed; software needs SYCL builds and patches; no passthrough reports found |
| 3 | Arc Pro B60 24 GB | 1,099 (blower) | cheapest 24 GB new | lower bandwidth; same software caveats |
| 4 | RTX PRO 4000 Blackwell 24 GB | about 4,999 | single slot, 241 mm, 140 W | Blackwell passthrough risk; 672 GB/s |
| 5 | RTX 4090 24 GB | about 3,100 new (estimate) | fastest 24 GB | 450 W, cable, size |
| Avoid | RTX 5090 | about 9,600 to 9,800 | | Blackwell reset bug; 600 W cable; cage out |
| Avoid | RX 7900 XTX, A4000/4000 Ada | | | passthrough reports are poor |

CONFIRMED by direct read: RTX 5090 price at PC Case Gear is about A$9,600 to 9,700 (older A$5,400 to 6,200 figures are stale). Not independently confirmed by me: the other prices, the board slot layout, and the case's card-length limit (taken from retailer copies of SilverStone's spec; one older source says 200 mm).

## Fit checks to complete before ordering

- Board (ASUS Pro WS W880-ACE SE): two PCIe 5.0 x16 slots from the CPU (x16 or x8/x8) and one PCIe 4.0 x16 slot wired x4 from the chipset, per ASUS's spec page as read by the agent. Slot spacing, M.2 positions and BIOS x8/x8 support: NOT VERIFIED.
- Case (SilverStone RM41-H08): measure the space from the 5.25" cage to the slot, and the internal height with the lid on (maximum card height NOT FOUND).
- Power supply: the 2021 Corsair RM1000x has six 6+2-pin PCIe connectors and no 12VHPWR cable in the box; the newer ATX 3.1 version has a native 12V-2x6 socket. Check which one is installed.
- Cooler: Noctua NH-D12L is about 145 mm tall (recollection); the case limit is 148 mm (130 mm with the card retainer).
- Host: Proxmox VE 9.2.21 on kernel 7.0.14-20-pve (read from the API). One report has a different build, 7.0.6-2-pve, hanging the host on a 5070 Ti VM start; test any new card with at least 10 start/stop cycles and check `dmesg` for reset timeouts.
- Memory: the host has 125 GiB; 52 GiB is in use now, so plan the inference VM at about 32 to 40 GB unless other guests are trimmed.
- Pascal note: driver branch 580 is the last that supports the GTX 1080; a new card on a newer driver means the 1080 cannot share that driver.

## Target architecture (CONFIRMED mechanisms; the layout is a design)

- **Guests on one bridge, isolated by the Proxmox firewall (default drop in and out, explicit IP rules, no DNS from the agent):** `inf` (GPU passthrough, model server only, no internet), `agent` (Hermes, no internet; reaches only inf, mem and Gitea), `mem` (Hindsight in local_external mode with a small CPU model), optional `fetch` (the only guest with internet; search tools; no agent, no system access), optional `broker` (holds a capped Anthropic key; exposes one text-only "ask Claude" tool through an allowlist proxy), optional `gatekeeper` (read-only Proxmox token, fixed read endpoints).
- **Front end:** Hermes Desktop on Fedora connects with its SSH connection type (it opens the tunnel and starts the dashboard; CONFIRMED on Hermes's multi-connection page). Do not use the plain-HTTP remote gateway mode over the LAN. Build Desktop from source; run it with `HERMES_HOME` pointing at an empty directory and `auth.adopt_external_logins: false`, never the Local connection (the Fedora desktop holds the Claude Code login).
- **Hermes settings:** `approvals.mode: manual` (it only prompts on dangerous-looking commands; its own SECURITY.md calls the gate a heuristic, so the VM and firewall are the boundary), `approvals.deny` globs, unattended modes deny, `security.allow_lazy_installs: false`, `updates.check: false`, web/browser/messaging toolsets off, no gateway platforms, MCP servers with `tools.include` allowlists and `sampling.enabled: false` (Gitea read-only; Proxmox read-only via the gatekeeper only). Install from a reviewed pinned commit (mirror the repo into Gitea); never run `hermes update` (it tracks main only).
- **Claude:** use a capped API key only (Anthropic console spend limit and a separate workspace limit, both CONFIRMED; reaching the limit returns an error and stops spend). Hermes's `fallback_providers` would put Claude in the agent's seat on 429/5xx/auth errors, which conflicts with the rule that no model has both system access and internet, so use the text-only broker or no Claude at all. Subscription logins stay out of every Hermes machine.
- **Hindsight:** `local_external` on its own guest, with a small CPU model for extraction (retain runs every turn by default); about 4 GB RAM; embeddings on CPU; choose the embedding model before storing anything.
- **Backups:** vzdump of agent and mem to the TrueNAS storage, plus a nightly dump inside each; model files excluded; TrueNAS and the backup share a host, so the offsite copy is what makes it a real backup.

## Staged plan with success criteria

0. Free test on the existing card: Qwen3.6-35B-A3B with all experts in RAM and Hermes pointed at it with non-sensitive prompts; success = starts at 64K, tool calls parse, fixed-prompt time and tok/s recorded.
1. Guests and firewall without the GPU; success = from the agent, the internet, DNS, Proxmox port 8006 and the router all fail; inf, mem and Gitea succeed.
2. GPU passthrough and llama.cpp with `--jinja -c 64000+`; sweep the expert-offload layers; success = acceptable speed at 64K with the real prompt, and 10 clean VM start/stop cycles.
3. Tool-call reliability: about 30 scripted calls across three simple tools, with thinking on and off.
4. Install Hermes at a pinned reviewed commit.
5. Approvals test: dangerous command prompts, ordinary runs, denied globs refuse, unanswered prompt denies after 300 s.
6. Hindsight retain and recall.
7. Gitea read-only MCP (a write attempt must fail at the server).
8. Desktop over SSH from Fedora.
9. Optional broker and fetch VM.
10. Backup and restore drill.

## Decisions needed from Boss

1. Whether to run the free test on the GTX 1080 first (it decides whether a card is needed at all, and which).
2. Card: used 3090 (cheapest, fit and heat to be measured) against Arc Pro B70 (safer fit, slower, rougher software).
3. Claude: text-only broker, or none.
4. How much of Hermes's dependency tree you will accept without reading.

## Could not verify

- Any measurement on the Core Ultra 9 285K with DDR5-6000; any tool-calling benchmark for these models with Hermes; prefix-cache behaviour with Qwen's hybrid attention in llama.cpp (could decide how long a warm turn takes).
- Used prices in Australia (eBay, Facebook Marketplace, Scorptec, Umart, Mwave, Centrecom were not reachable); Arc passthrough reports; the board's slot spacing and the case's card height.
- Official details of `hermes serve`, the semantics of several config keys, and whether two reported Desktop bugs were fixed.
