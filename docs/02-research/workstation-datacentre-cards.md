# Data-centre cards and boxes for the workstation (9 October 2026)

Companion to `workstation-hermes-recommendation.md` and `workstation-hermes-alternatives.md` (pull requests #12 and #13). Marks: CONFIRMED = primary source read; agent-reported = a research sub-agent said it, not independently checked. Prices are undated snippets unless stated.

## Fit limits of the workstation

ASUS Pro WS W880-ACE SE, SilverStone RM41-H08 4U case with two weak 80 mm fans, Corsair RM1000x (2021). Card length limit 278 mm with the 5.25-inch cage, 426 mm without (retailer copies). Slot spacing for two cards not verified.

## Findings

| Option | Verdict | Why |
|---|---|---|
| Used RTX 3090 24 GB | Best value | about A$1,500 to 2,300 on eBay AU and Gumtree (agent-reported, undated); Boss found one near A$5,000, source not recorded. Large cooler, check clearance and 12V connectors. |
| Radeon AI PRO R9700 32 GB | Best non-consumer fit | 267 mm blower, 300 W, one 12V-2x6 plug (CONFIRMED, Puget); about A$2,300 to 2,500, thin stock; RDNA4 passthrough reset risk (agent-reported). |
| Blackwell cards (RTX 5090 and similar) | Avoid for Proxmox passthrough | reset bug reported; RTX 5090 about A$9,600 (CONFIRMED). |
| Tesla P40, P100, V100, T4 | Avoid | end-of-life drivers; passive cooling needs ducting; EPS power cables. |
| Passive data-centre cards in general | Avoid in this case | two weak 80 mm fans cannot push air through a passive heatsink. |
| Radeon Pro W7800, W7900 | Avoid | 280 mm, over the 278 mm limit with the cage. |
| Modified 4090 48 GB | Avoid | unsupported modification, warranty and driver risk. |
| Strix Halo 128 GB box | Niche | good for mixture-of-experts models only. |
| DGX Spark and OEM GB10 boxes | Possible | A$6,249 to 7,999 in January (agent-reported); probably more now. Separate box, not a card. |
| Mac Studio M5 | Possible | separate box, different software stack. |

## Two cards

Board slots allow it (two PCIe 5.0 x16 from the CPU, one x4). Case clearance, the 1,000 W supply and heat are the limits.

## Not checked

Current prices, stock, slot spacing, and whether the passthrough reset reports still apply to current drivers.
