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

## Decision pending

Boss will turn Mimir into a cloud-init template (card removed, start-at-boot off). Nothing beyond the engine is built yet.
