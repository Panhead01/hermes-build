# Closing the Hermes-on-GTX-1080 trial (8 October 2026)

Decision (Boss, 8 October 2026): the chapter on trying to run Hermes on the GTX 1080 is closed. The project is not closed: all files, scripts, notes and this git repository are kept, and the next step is to research the right hardware and setup for running Hermes locally on the workstation. Nothing has been deleted or shut down by this decision.

## What the trial proved

- A GPU passthrough guest (VM 101 `Mimir`, Ubuntu 26.04, driver 580.178.04) runs Ollama on CUDA: qwen3:8b at about 38.5 tokens/s generation and about 440 tokens/s prompt reading with the NVIDIA profile-file workaround (without it, about 33.6 tok/s). See `engine-results.md`.
- The fix is stable: a one-hour soak (517 generations) and a 20-minute soak with the fan service, answers identical before and after, no NVIDIA errors.
- Narrow Proxmox access works: user `Alice@pve` with a privilege-separated token (no start/stop, no start-on-boot), create/clone/snapshot/ISO upload within the `agents` pool.
- Ollama runs as a service (localhost only, prompt cache capped), a fan-curve and monitoring service works headless, and Hermes and Hermes Desktop were researched in depth (nothing installed).

## Limitations and possible fixes

| # | Limitation | Evidence | Possible fix |
|---|---|---|---|
| 1 | Hermes needs at least 64,000 tokens of context and rejects smaller windows at startup | Hermes quickstart and Ollama's context-length page (read directly) | A card with more VRAM, or a model/cache combination that fits (see 2) |
| 2 | qwen3:8b at 64K does not fit in 8 GB: KV cache 9.66 GB at f16, 5.13 GB at q8_0, 2.72 GB at q4_0, on top of about 5 GB of weights (36 layers, 8 KV heads, head dim 128, from the Hugging Face config) | Calculated from the published config; 5.6 GB at 4096 measured | 4B-class model with q4_0 cache, partial CPU offload (slow), MoE model with expert offload to system RAM, or a 16 to 24 GB card |
| 3 | Small models tool-call unreliably: Hermes's own Ollama guide lists only gemma4:31b (about 20 GB) as reliable for tools; 3B to 9B models are listed as unable or unreliable | Hermes `local-ollama-setup` page | A larger model on a larger card, or local for easy turns and a capped API key for hard ones |
| 4 | Hermes's fixed prompt is large (about 19K tokens in one report): a cold first turn is about 43 s at 440 tok/s | One third-party measurement; arithmetic from our speed | Trim toolsets, keep the model loaded (KEEP_ALIVE), rely on prefix caching, faster prompt processing on a bigger card |
| 5 | Known Hermes/Ollama bugs: Ollama silently runs at 4096 tokens unless set server-wide (issue 43900, open); reasoning models can return empty content (issue 46131, open, workaround `reasoning_effort: none`) | GitHub issues, state read via API | Set `OLLAMA_CONTEXT_LENGTH` on the server; use a non-thinking model or the workaround |
| 6 | Flash attention is off on Mimir and KV cache quantisation is untested on Pascal | Observed in the service log | Test `OLLAMA_FLASH_ATTENTION=1` and a q8_0/q4_0 cache (reversible; not done) |
| 7 | The VM is small: 2 generic vCPUs (no AVX2) | Observed | Use CPU type host and more vCPUs (the host has 24 threads) |
| 8 | CUDA holds consumer cards at P2 (memory 4513 instead of 5005 MHz); the fix relies on an NVIDIA application-profile file that is not documented for compute jobs | Measured on/off/on; NVIDIA collaborator's GitHub comment | Keep the file; check again after driver changes; a newer card family may behave differently |
| 9 | Pascal is at the end of its life: driver branch 580 is the last that supports it (supported until mid-2028 per NVIDIA); Ollama needs 570 or newer for CUDA on it | NVIDIA developer blog and supported-chips list (read directly) | Plan the next card |
| 10 | Heat: the blower card (Gigabyte GV-N1080TTOC-8GD) reached 90 C under sustained load; 82 C with the fan service | Measured | Clean and repaste (Boss, weekend); case airflow; a lower power limit if needed (parked) |
| 11 | Hermes safety: its approval gate is documented as a heuristic, not a security boundary; the project changes every few days (updates track `main` only); reported CVEs in older versions; install is a piped vendor script; Desktop has no Linux binary and its remote mode uses plain HTTP with basic auth | SECURITY.md, install.sh and docs read directly; CVE list reported by trackers | VM and firewall containment, `approvals.mode: manual`, `auth.adopt_external_logins: false`, pin a reviewed commit, SSH tunnel or TLS proxy for Desktop |
| 12 | The narrow Proxmox token cannot set the BIOS type, name or start-at-boot (one shared right) or start/stop machines | Proxmox source and tests | Always clone from a template; revisit if it keeps recurring |
| 13 | Unexplained: plain-CUDA baseline read 35.7 tok/s early on and 33.6 in controlled runs (persistence mode ruled out) | Measured | Low priority; recheck after hardware changes |
| 14 | Hindsight needs an LLM (it would compete with the main model for VRAM) plus about 2.5 to 3 GB RAM; recall adds up to 4K tokens per turn | Hindsight docs | Run it on a bigger setup, or use a separate small model and CPU embeddings |

## What is kept and what is still running (no change made)

- Repo `projects/hermes-build` on Gitea (docs, results, this note); research notes and scripts (`gpu-sequence.py`, `gpu-queue.py`, `gpu-fan-curve.py`, `bench.py`, `integrity.py`) live on Mimir and in Boss's session notes; they are not yet in the repo (held until Boss decides).
- VM 101 `Mimir` (stopped or running at Boss's choice): Ollama service, fan service, persistence mode, the profile file, model qwen3:8b, run logs under `~/gpu-seq/`. Start-at-boot is off.
- Proxmox: user `Alice@pve`, token `agent`, roles `AgentLook` and `AgentBuild`, pool `agents`, PCI mapping `gtx1080`. Token secret is in Boss's desktop file only.
- Prepared but not installed: nothing pending.

## Next

Research the recommended hardware (GPU, RAM, storage, case and power fit) and the whole setup for running Hermes locally on the workstation, then bring Boss a recommendation. Hermes itself was not installed in this trial.
