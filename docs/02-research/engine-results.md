# Engine results: qwen3:8b on the GTX 1080 (trial guest 212)

Measured 7 October 2026 on trial guest `agent-trial` (VM 212). This is the engine-only step from `trial-plan.md`: driver, Ollama, one model, speed and fit.

## Setup (OBSERVED)

| Item | Value |
|---|---|
| Guest | Debian 13.7 (trixie), kernel 6.12.111, 15 GB RAM, 64 GB disk |
| GPU | GeForce GTX 1080 (Pascal, compute 6.1), 8 GiB, passed through |
| NVIDIA driver | 550.163.01 (Debian's own package; the newest Debian offers) |
| Ollama | v0.40.0, tarball unpacked in the user's home folder (no root, started by hand, not a service); archive sha256 c94aa415...d5ad matched the published digest |
| Model | `qwen3:8b`, 5.6 GB, id 500a1f067a9f |
| Context | 4096 (Ollama default at the time of the run) |
| Thinking | off (`think: false`) |

## Finding: Ollama did not use CUDA (OBSERVED)

Ollama's log at start-up:

> NVIDIA driver too old ... compute=6.1 driver=550 required_driver="570 or newer"

It fell back to the Vulkan backend on the same card. `ollama ps` showed `100% GPU`, and `nvidia-smi` showed 5332 MiB of 8192 MiB in use. Ollama's own GPU documentation (docs/gpu.mdx, read directly) says cards with compute capability 5.0 to 6.2 need driver 570 or newer.

## Speed (OBSERVED, six timed runs after one discarded warm-up)

Each run asked for about 200 words, capped at 300 tokens, temperature 0.2, through the local API.

| Run | Tokens generated | Generation (tok/s) | Prompt reading (tok/s) |
|---|---|---|---|
| 1 | 256 | 39.45 | 158.2 |
| 2 | 260 | 39.62 | 168.3 |
| 3 | 263 | 39.83 | 157.7 |
| 4 | 233 | 39.58 | 164.7 |
| 5 | 221 | 39.80 | 162.6 |
| 6 | 288 | 39.34 | 158.1 |

- Generation: mean about 39.6 tok/s, range 39.3 to 39.8.
- Prompt reading: mean about 162 tok/s.
- An earlier single run (with `--verbose`) gave 38.2 tok/s generation, 59 tok/s prompt reading, consistent with the above (that run included model load).
- GPU memory 5332 MiB used, 71 C and 1746 MHz right after the runs.

## Limits of these numbers

- Six runs, one model, one prompt style, short answers, 4096 context. Longer contexts, other prompts, and tool-calling were not measured.
- Backend was Vulkan, not CUDA. A CUDA run needs driver 570 or newer.
- Not compared against CUDA on the same card: no one has shown CUDA is faster for generation here. Third-party figures for a GTX 1080 Ti suggest Vulkan generation is similar and prompt reading is slower under Vulkan, but those used different models and builds and are not comparable (agent-reported, not verified).

## Driver options (checked 7 October 2026)

- NVIDIA's developer blog (read directly): branch 580 is the last driver branch that supports GPUs before compute capability 7.5, as a long-term branch supported until mid-2028. NVIDIA's 590 supported-chips list names the GTX 1080 under "legacy GPUs that are no longer supported".
- Debian 13 offers only 550.163.01 in stable (confirmed on the guest). Debian's backports and testing were reported at the same version (not verified).
- The 580.178.04 `.run` installer exists on NVIDIA's server (397,214,534 bytes, 13 July 2026).
- Ubuntu's archive lists `nvidia-driver-580` for both 24.04 (noble) and 26.04 (resolute); the 26.04.1 live server image is on releases.ubuntu.com. The exact 580 version in the 26.04 archive was not checked.

## Second run: CUDA on Ubuntu 26.04, driver 580 (7 October 2026, later the same day)

The Debian trial guest was destroyed by Boss afterwards. The same test ran on a new guest, VM 101 `Mimir`, built through the narrow API token from Ubuntu's official 26.04 cloud image (checksum and signature verified).

| Item | Value |
|---|---|
| Guest | Ubuntu 26.04.1 LTS, kernel 7.0.0-38, 4 GB RAM, 2 cores, UEFI (OVMF) firmware, q35 |
| GPU | same GTX 1080 via the `gtx1080` PCI mapping |
| NVIDIA driver | 580.178.04 (`nvidia-driver-580-server` from Ubuntu's own archive, DKMS modules) |
| Ollama | same v0.40.0 archive, same sha256, same model (`qwen3:8b`, id 500a1f067a9f), same script, same prompts |
| Backend chosen | CUDA (CUDA 12 runner; the CUDA 13 files skip this card as unsupported) |

| Run | Tokens generated | Generation (tok/s) | Prompt reading (tok/s) |
|---|---|---|---|
| 1 | 250 | 35.75 | 425.9 |
| 2 | 249 | 35.85 | 446.8 |
| 3 | 220 | 35.85 | 427.4 |
| 4 | 255 | 35.75 | 449.7 |
| 5 | 231 | 35.77 | 444.8 |
| 6 | 300 | 35.59 | 426.1 |

Mean generation about 35.8 tok/s (range 35.6 to 35.9), mean prompt reading about 437 tok/s. GPU memory 5471 MiB of 8192, 71 C, 100% on GPU.

## Comparison (OBSERVED, six runs each)

| | Vulkan, driver 550, Debian 13 | CUDA, driver 580, Ubuntu 26.04 |
|---|---|---|
| Generation | about 39.6 tok/s | about 35.8 tok/s (about 10% slower) |
| Prompt reading | about 162 tok/s | about 437 tok/s (about 2.7 times faster) |
| GPU memory | 5332 MiB | 5471 MiB |

Caveat: more than the backend changed between the two runs (operating system, kernel, driver, firmware type), so the difference cannot be put down to CUDA versus Vulkan alone. Both used a 4096-token context and short answers. Longer contexts, tool-calling and other models were not measured.

## Notes from building the second guest

- The first boot with the card attached hung on the default BIOS firmware (silent console, one vCPU spinning, no network). Switching to OVMF with an EFI disk fixed it (a single observation).
- The API token cannot set the BIOS type, name or start-at-boot (one shared Proxmox right, VM.Config.Options), so the BIOS change was made by Boss in the web page.

## Why CUDA is slower at generating: the P2 memory clock (7 October 2026)

Control experiment on the same Ubuntu 26.04 / driver 580 machine, same model and script, forcing the Vulkan backend (`OLLAMA_LLM_LIBRARY=vulkan`, `CUDA_VISIBLE_DEVICES=-1`):

| Run (six each) | Generation tok/s | Prompt reading tok/s | Performance state | Memory clock |
|---|---|---|---|---|
| CUDA, default | 35.5 to 35.9 (mean about 35.7), repeated twice | about 437 | P2 | 4513 MHz |
| Vulkan | 40.7 to 41.0 (mean about 40.9) | about 162 | P0 | 5005 MHz |

The card's maximum memory clock is 5005 MHz. CUDA on GeForce cards is held in P2, which lowers the memory clock about 10%, and generation is memory-bandwidth bound. 4513/5005 = 0.90; the observed speed ratio was 0.87, so the clock explains most but not all of the gap.

Other observations (all OBSERVED on Mimir): GPU 99% utilised and at the 180 W power limit (range 90 to 216 W); PCIe gen 3 x16; persistence mode was off (fixed, see below); vCPU is the generic default type without AVX2 (unchanged); no NVIDIA error (Xid) messages. `nvidia-smi --lock-memory-clocks` is refused on this card (probed once, no change made).

## Fix under test: the CudaNoStablePerfLimit application profile

Source (read directly): an NVIDIA collaborator's comment dated 18 December 2025 on https://github.com/NVIDIA/open-gpu-kernel-modules/issues/333 gives a profile named `CudaNoStablePerfLimit` with setting `0x166c5e` set to 0, said to work on all supported drivers including pre-Turing cards. NVIDIA's README (580.126.09) says application profiles are only supported by the GLX implementation and matches `procname` exactly against the executable name; it also documents the `__GL_APPLICATION_PROFILE_LOG=1` debug switch.

File created on Mimir: `/etc/nvidia/nvidia-application-profiles-rc` (did not exist before), rule `procname` = `llama-server` (the name of the process that holds the GPU; Ollama launches it as a child, binary `.../lib/ollama/llama-server`). Settings are the flat form `["0x166c5e", 0]` as in the NVIDIA comment.

| Condition | Performance state | Memory clock | Generation tok/s (six runs) | Prompt tok/s |
|---|---|---|---|---|
| Profile ON | P0 | 5005 MHz | 38.1 to 38.6 (mean about 38.4) | about 437 |
| Profile OFF (control, file moved away, Ollama restarted) | P2 | 4513 MHz | 32.8 to 33.5 | about 417 |
| Profile restored | (file back in place) | | | |

Result: the profile moves the compute process to P0 at the full 5005 MHz, and removing it returns P2 (CONFIRMED by switching it on, off and on). The debug log printed nothing, so the file parsing was not directly visible; the change in behaviour is the evidence. Speed: about 38.4 tok/s with the profile against about 35.7 earlier without it (roughly +7%); still about 2.5 tok/s under Vulkan's 40.9.

Caveat on the speed numbers: the card warms during a test (about 65 to 78 C over six runs) and speed drifts down slightly during each set; the control set ran on a warm card and read lower than the earlier cool baseline (about 33 against 35.7). Speeds in the table are therefore only roughly comparable between rows; the clock readings are the solid evidence. A fair A/B needs a cooldown between sets.

Safety: no Xid errors in the log (0) after the P0 runs. The card's GDDR5X has no error correction, so output was not yet checked against a stock-clock baseline and no long soak has been run (see next steps).

## Housekeeping done on Mimir

- NVIDIA persistence mode enabled (systemd drop-in `/etc/systemd/system/nvidia-persistenced.service.d/persistence-on.conf`; Ubuntu starts the daemon with `--no-persistence-mode`).
- Ollama is still started by hand (not a service); it must be restarted after a reboot.

## Research reports behind this (summary, sources checked where marked)

- Published llama.cpp tables (CONFIRMED by reading the discussion text): GTX 1080 Ti about 62.5 tok/s CUDA and 67.8 Vulkan (Llama 2 7B Q4_0), GTX 1070 Ti 37.8 CUDA and 42.9 Vulkan. No plain GTX 1080 row exists; interpolation (estimate) puts it at roughly 34 to 42 tok/s for qwen3 8B, so our results are in the normal range. Flash attention changes generation by about 1% on Pascal.
- Other routes to P0 and their risks (agent report; pstated source and the library files on Mimir checked): the `nvidia-pstate` tool can force a state through an unsupported NVIDIA call and is untested on GeForce; `nvidia-settings` with Coolbits and a memory offset needs an X server (X and NVIDIA's Xorg driver are already installed on Mimir); NVML cannot lock memory clocks on Pascal. Memory overclocking can lower speed before showing errors and can corrupt output silently.

## Integrity check at the full memory clock (7 October 2026)

12 prompts, fixed seed, temperature 0, about 180 tokens each, run with the profile file ON (P0, 5005 MHz) and again with it OFF (P2, 4513 MHz): **12 of 12 outputs identical**; no NVIDIA Xid errors. (Short outputs; this is a sanity check, not a proof against rare memory errors.)

## Cool-start run, six mirrored blocks (script `gpu-sequence.py` on Mimir, started 7 Oct 2026 21:40 AEDT)

Method: each block starts only after the idle card is at 42 C or less and four readings 10 minutes apart (30 minutes) are within 1 C; blocks run in mirrored order (A B C C B A) so heat and time drift cancel; each block is one discarded warm-up plus six timed runs (same prompts as before, 300-token cap); Ollama is restarted per block; the profile file is moved aside for the OFF blocks and always put back. Safety stops (never triggered): GPU above 92 C, any new Xid error, nvidia-smi not answering.

| Block | Condition | Start temp | End temp | Memory clock (end) | Generation tok/s | Prompt tok/s |
|---|---|---|---|---|---|---|
| 1 | CUDA, profile ON | 37 C | 68 C | 5005 MHz, P0 | 38.48 | 440 |
| 2 | CUDA, profile OFF | 37 C | 71 C | 4513 MHz, P2 | 33.65 | 440 |
| 3 | Vulkan | 36 C | 67 C | 5005 MHz, P0 | 40.91 | 161 |
| 4 | Vulkan | 36 C | 67 C | 5005 MHz, P0 | 40.90 | 162 |
| 5 | CUDA, profile OFF | 36 C | 71 C | 4513 MHz, P2 | 33.61 | 435 |
| 6 | CUDA, profile ON | 36 C | 67 C | 5005 MHz, P0 | 38.52 | 443 |

Repeat blocks agree within 0.1 tok/s, so the numbers are reproducible on a cool start. Means: CUDA ON 38.50, CUDA OFF 33.63, Vulkan 40.91 tok/s generation. The profile is worth about +14.5% over CUDA without it (38.50 against 33.63); Vulkan is still about 6% faster than CUDA with the profile at generation, and about 2.7 times slower at reading prompts (161 against about 440 tok/s). Mean clocks while busy: SM about 1767 MHz (ON), 1840 MHz (OFF), 1776 MHz (Vulkan); power about 172 to 181 W (limit 180 W). Card temperatures 45 to 71 C during the six-run blocks.

### Unexplained discrepancy

The earlier CUDA-default baselines on this machine (two sets of six, before persistence mode was turned on) read about 35.7 tok/s; the cool-start OFF blocks read about 33.6. Clocks were similar (P2, 4513 MHz, SM about 1822 MHz then against 1836 to 1844 MHz now), the profile-ON result did not change (about 38.4 then, 38.5 now), and the repeat OFF blocks agree with each other. Between the two, persistence mode was enabled and Ollama was restarted many times; no cause is established (GUESS: persistence mode or a state left over from the first boot). Treat 33.6 as the baseline of the controlled run and 35.7 as unexplained; the +14.5% figure uses the controlled one, the earlier +7% used the other.

## Soak: stopped early by an out-of-memory kill (NOT completed)

The one-hour soak (profile ON, continuous generation) ran about 7 minutes and about 45 generations at a steady 37.7 tok/s, then the script stopped itself on an Ollama HTTP 500. Cause (OBSERVED in the guest's kernel log and Ollama's log): the Linux out-of-memory killer killed the `llama-server` process. The guest has 3.8 GB of RAM and no swap; llama-server held 2.85 GB resident, and Ollama's prompt cache had grown to 69 cached prompts (2.56 GB, configured limit 8192 MiB, larger than the machine's RAM). It was not a GPU fault: 0 Xid errors, temperature never above the 92 C limit.

While it ran, the card settled at 86 to 88 C (mean 86 C over the soak, range 70 to 88 C) with the SM clock down to about 1642 MHz, power about 168 W: over the card's reported 83 C target, under its 96 C slowdown and 99 C shutdown points. So sustained load is hot even though short tests are fine. Long-run stability at the full memory clock is therefore NOT yet proven.

## Soak rerun with 12 GB RAM: completed (7 October 2026, 18:45 to 19:47 UTC)

Mimir's memory was raised from 4 GB to 12 GB (set through the API token; Boss did the Stop and Start), then the soak-only run repeated: a cool start (35 C idle, four readings within 1 C), then one hour of continuous generation with the profile file ON, with the 12 fixed-seed prompts run before and after.

| Measure | Result |
|---|---|
| Generations | 517 over 60 minutes |
| Generation speed | first 10 runs 38.07 tok/s, last 10 runs 37.74 tok/s (about 1% slower after an hour; steady at 37.6 to 37.8 for most of it) |
| Answer integrity | **12 of 12 identical** before and after (fixed seed, temperature 0) |
| NVIDIA Xid errors | 0 |
| Out-of-memory kills | 0 |
| Guest memory | rose about 350 MB a minute early on (Ollama's prompt cache), reached about 9.6 GB of 11.9 GB and levelled off; no OOM |
| Peak temperature | 90 C (mean about 86 to 88 C); SM clock settled about 1630 MHz; power about 160 to 175 W; memory clock stayed 5005 MHz, P0 |

Conclusion (CONFIRMED for this one-hour run): CUDA with the CudaNoStablePerfLimit profile held 5005 MHz memory for an hour with no errors and no corrupted output on the 12-prompt check. The 4 GB machine's earlier kill was a RAM shortage, not a GPU fault. Peak 90 C is within 2 C of the script's 92 C stop and 6 C under the card's 96 C slowdown point, so cooling deserves attention.

## Cooling findings (7 October 2026)

- The card is a Gigabyte GV-N1080TTOC-8GD (Turbo OC, blower cooler, 2 slots, one 8-pin; Guru3D launch article confirms the blower and an 8+2 power-phase design). The board is probably not the NVIDIA reference layout (reported, not confirmed on a page), so drop-in aftermarket coolers may not fit; availability is poor (Gelid Icy Vision discontinued; Arctic Accelero and Raijintek Morpheus availability unverified).
- A logger on Mimir showed the card's own fan curve already at 87% when the card sits at 88 C, so a steeper curve can only add a small amount; cleaning dust and repasting (planned by Boss for the weekend) is likelier to help.
- Fan control works headless in the guest through NVML as root (CONFIRMED by test: set 35% took effect and was restored to automatic; range 24 to 100%). A fan-curve and monitoring service is prepared (`gpu-fan-curve.py`): automatic below 55 C, then 50% at 55 C, 65% at 65 C, 80% at 72 C, 100% at 78 C, with hysteresis; hands control back to automatic on any error or stop; logs CSV. Simulation passed.
- Power limit range on this card: 90 W minimum, 180 W default, 216 W maximum. Lowering it is parked: Boss will only try it if temperatures cause problems in normal use.

## Unattended test queue (script `gpu-queue.py` on Mimir, 7 to 8 October 2026)

Same safety stops as before (abort above 92 C, on any new Xid error, or if nvidia-smi stops answering) and the same cool-start rule for T2 and T3. Nothing aborted; Xid errors 0, out-of-memory kills 0. Final state restored: settings file in place, persistence mode on, fan service running, Ollama stopped.

**T1, fan service under load: PASS.** Installed as systemd service `gpu-fan` (`/opt/gpu-fan/gpu-fan-curve.py`, log `/var/log/gpu-fan/gpu-fan.csv`). In a 130 second load (19 generations) it took manual control past 56 C, reached 100% fan, peak 82 C, and handed control back to automatic about 120 seconds after the load ended.

**T2, persistence mode OFF baseline (CUDA, profile OFF, fan service stopped, cool start 37 C):** 33.64 tok/s generation, 439.7 tok/s prompt, P2 and 4513 MHz. Persistence ON gave 33.63 in the earlier cool-start run. **Persistence mode is not the cause of the earlier 35.7 baseline.** That discrepancy is still unexplained.

**T3, 20-minute soak with the fan service (profile ON, cool start):** 175 generations, first 10 at 38.27 tok/s, last 10 at 37.89, 12 of 12 fixed-seed answers identical before and after.

| | One-hour soak, stock fan | 20-minute soak, fan service |
|---|---|---|
| Peak temperature | 90 C | 82 C |
| Mean temperature | about 86 C | 80 C |
| SM clock (mean) | about 1640 MHz | 1671 MHz |
| Generation, last 10 | 37.74 tok/s | 37.89 tok/s |
| Answers identical | yes (12 of 12) | yes (12 of 12) |

The fan curve lowers peak temperature by 8 C (to 82 C, just under the card's 83 C target) and gains about 0.4% in speed; it is worth keeping on for heat, not for speed. Caveat: the second soak was 20 minutes against 60, though the first soak had already reached 87 C within about 3 minutes.

## Next steps (each reversible, Boss approves)

1. Boss cleans (and possibly repastes) the card at the weekend; then a short cool-start A/B (profile ON, fan service ON) to see the temperature change.
2. Rerun CUDA and Vulkan on different driver versions only if the result still disappoints.
3. Make Ollama a service so the setup survives a reboot (also consider limiting its prompt cache).
4. Turn Mimir into a cloud-init template (card removed, start-at-boot off) once the engine settles.
5. Unexplained: the earlier 35.7 tok/s plain-CUDA baseline against 33.6 now (persistence mode ruled out).
