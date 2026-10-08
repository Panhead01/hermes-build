# Alternatives to a consumer GPU for the Hermes workstation (8 October 2026)

Addendum to `workstation-hermes-recommendation.md`. Two research agents; key claims checked against sources where marked CONFIRMED. Nothing bought or installed. Australian prices are thin: most retailers were out of stock or not reachable, and tracker pages date from June to August 2026.

## Short answer

Nothing found beats a used RTX 3090 on speed per dollar. The realistic alternatives that fit the workstation are the **Radeon AI PRO R9700 32 GB** (fits the case without modification, cheapest new 32 GB card, real benchmarks exist, but a documented passthrough-reset risk family) and the **RTX PRO 4500 / 4000 Blackwell** (good software, much dearer, reset behaviour unknown). Outside the workstation, a 128 GB **Strix Halo** box suits mixture-of-experts models only; a **DGX Spark / GX10** or a **Mac Studio** is the fast-prefill route at two to three times the cost of a 3090.

## Cards that fit this chassis (SilverStone RM41-H08: 278 mm with the 5.25" cage, 426 mm without; two weak 80 mm fans; Corsair RM1000x 2021 with no 12VHPWR cable)

| Card | Memory / bandwidth | Power | Size and cooler | AUD found | Measured speed | Verdict |
|---|---|---|---|---|---|---|
| Radeon AI PRO R9700 | 32 GB GDDR6 / about 640 GB/s | 300 W, one 12V-2x6 (adapter needed) | 267 x 111 x 40 mm, 2-slot blower (CONFIRMED on Puget's page) | about 2,300 to 2,500 (out of stock at Newegg AU) | community posts: Qwen3.5/3.6 35B-A3B Q4 prefill about 2,400 to 3,300 tok/s, decode about 150; 27B Q4 prefill about 800 to 990, decode 28 to 32 (45 with multi-token prediction) | best value per GB; fits without removing the cage; RDNA4 reset risk (inferred for this card) |
| RTX PRO 4500 Blackwell | 32 GB GDDR7 / 896 GB/s, 200 W | | dual-slot; length unknown | about 4,765 ex-GST, out of stock | none found | strong on paper; costly; reset behaviour unknown |
| RTX PRO 4000 Blackwell | 24 GB / 672 GB/s, 140 W | | single-slot | Melbourne reseller lists it, price not seen | none found | best airflow and power fit; 24 GB is tight |
| Arc Pro B70 | 32 GB / 608 GB/s, 230 W, one 8-pin | | 267 mm reference; partner cards 271 to 289 mm | no AU listing found (list US 949) | one blog: very slow prompt reading at 150K context | needs software tuning |
| Used RTX A5000 / A6000 | 24 / 48 GB / 768 GB/s | one 8-pin | dual-slot blower, 267 mm (memory) | one A5000 listing at about 2,930; A6000 about 6,822 | none | poor value |

**Avoid:** Tesla P40/P100, V100, T4 (Pascal and Volta are on the last driver branch, 580; CUDA 13 cannot target Pascal); MI50/MI60 (unclear ROCm support, reset problems, passive); A100, A40, L40S, A10, L4 (passive, need ducted airflow and EPS cables; no AU prices found); Radeon Pro W7800/W7900 (280 mm, over the limit unless the cage is removed); modified RTX 4090 48 GB (65 dB peak noise in one test, no warranty, import duty, scam risk); RTX PRO 6000 (host-hang reset bug, far over budget); passive Arc Pro B60.

## Passthrough risk (the main issue)

- Blackwell RTX 5090 and RTX PRO 6000: function-level reset fails after guest shutdown and the host hangs; whether the PRO 4000/4500/5000 are affected is unknown.
- AMD RDNA4 (9070 XT, 9060 XT): no function-level reset; users rely on hookscripts and avoid in-guest shutdown. No R9700 report found; same silicon, so assume the same.
- Ampere and Ada workstation cards and the RTX 3090 and 4090: no reset reports found (absence of evidence). A separate report has a 5070 Ti hanging a host on a different 7.0 kernel build; this host runs 7.0.14-20-pve.
- Whatever is bought: at least 5 to 10 VM start, stop and reboot cycles, check `dmesg` for reset timeouts, and keep a return window.

## Boxes instead of a card

| Option | Price | Speed | Fit and notes |
|---|---|---|---|
| Strix Halo (Ryzen AI Max+ 395, 128 GB, 256 GB/s) | no AUD found; US 2,459 to 3,650, conflicting and rising | Qwen3.6-35B-A3B Q8: about 53 tok/s decode at empty context; a daily-driver post (CONFIRMED on Level1Techs) shows about 25 tok/s at 153K live context under 100 W; dense 27B only about 7 to 8 tok/s | MoE models only; Vulkan more stable than ROCm at long context |
| DGX Spark / ASUS GX10 / MSI EdgeXpert (GB10, 128 GB) | AU January 2026: 6,249 to 7,999 (a blog, not retailer pages); probably higher now | other models: gpt-oss-120b about 60 tok/s; Qwen3-30B-A3B 77 to 85; prefill about 1,800 to 3,600; Qwen3.6 not measured | best software (CUDA); Hermes has a Spark playbook |
| Mac Studio M5 Max / Ultra | M5 Max from 4,299 (128 GB price not found); Ultra from 9,499 | no Qwen3.6 numbers; vendor figure 1,810 prefill / 112 decode on Qwen3.5-35B-A3B | strongest for big models; macOS, outside Proxmox, dearest |

## Using what is already there

- **GTX 1080 with expert offload:** a second-hand report of a r/LocalLLaMA post (13 May 2026, read via an aggregator) says about 24 tok/s on Qwen3.6-35B-A3B Q4_K_M with 128K context on a GTX 1080 8 GB, `--n-cpu-moe 30`, and non-standard cache types (turbo4/turbo3, apparently from a llama.cpp fork), on an older i7-6700 with 32 GB. Prompt-processing speed was not stated and is the likely weak point (CPU-held weights are copied to the GPU for each large batch, limited by PCIe and the 1080's compute). Free to try and unmeasured on this machine.
- **CPU only on the 285K:** no published figures; the research agent's guess is 15 to 25 tok/s decode and slow prompt reading (Arrow Lake lacks AVX-512 and AMX; unverified).
- **Not worth it:** the desktop's RX 9070 over llama.cpp RPC (one test: decode halved), cloud GPU rental (breaks the privacy rule except for throwaway benchmarking).

## Ranking for this workstation

1. Free test first: 35B-A3B with expert offload on the existing 1080; measure prefill at 4K and 19K prompts.
2. Used RTX 3090 (about A$1,500 to 1,900, estimate): best speed per dollar; check the 278 mm limit and heat; safest passthrough record.
3. Radeon AI PRO R9700 (about A$2,300 to 2,500): fits as is, 32 GB, warranty; accept the reset risk and the 12V-2x6 adapter, and test.
4. RTX PRO 4000 or 4500 Blackwell: if the reset behaviour proves benign and the price is acceptable.
5. Strix Halo only for a separate low-power MoE-only box; Spark or Mac only if fast prefill on large models is worth two to three times the cost.

## Before ordering

Measure card length and height in the case; confirm the RM1000x version and the cable or adapter for 12V-2x6 or EPS leads; confirm Above 4G decoding and Resizable BAR in the BIOS; get live Australian stock and prices in writing (Techom, PB Tech AU, Scorptec, Centrecom, PC Case Gear, Computer Orbit, eBay AU); run the real workload (35B-A3B and 27B, 64K cache, 4K and 19K prompts) with Vulkan and ROCm or CUDA builds on any card that can be trialled.

## Could not verify

Current AU prices for most items; any R9700 or RTX PRO passthrough report; measured Qwen3.6 speeds on the Spark and Mac; CPU-only numbers for the 285K; the Strix Halo, Spark and Mac AUD prices; NVIDIA's own current Pascal driver page.
