# Trial guest: steps (draft, 7 Oct 2026)

Goal: a new guest on the workstation with the GTX 1080 passed through, running one small model, measured. Boss runs every Proxmox step; Alice works inside the guest once it exists. Anything marked UNSURE is to be checked against the current Proxmox documentation before running.

## Stage 1: read-only look (nothing changes)
Run in the Proxmox web page, node `pve`, Shell. Type the commands rather than pasting (pasting into that window adds stray characters). Send back the output or a screenshot.
1. `lspci -nn | grep -i -E "vga|3d|nvidia"` : finds the GPU and its PCI address (for example `01:00.0`) and the audio function beside it (`01:00.1`).
2. `lspci -nnk | grep -i -A3 nvidia` : which driver, if any, currently has it.
3. `dmesg | grep -i -E "iommu|dmar" | head -20` : whether the IOMMU (VT-d) is on.
4. `cat /proc/cmdline` : the boot options now in use.
5. IOMMU groups: `for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do echo "IOMMU Group ${g##*/}:"; for d in $g/devices/*; do echo -e "\t$(lspci -nns ${d##*/})"; done; done` : the GPU should sit in a group with only its own audio function (and perhaps a PCIe bridge). If it shares a group with other devices, stop and tell Alice.
6. `lspci | grep -i -E "sata|ahci"` : confirms which controller already goes to the TrueNAS VM, so the GPU change cannot touch it.

## Stage 2: decide (Alice reads the output first)
- GPU in a clean group, IOMMU on: go on to stage 3.
- IOMMU off: BIOS VT-d and a kernel option are needed first (UNSURE: check the Proxmox PCI passthrough page for the current option on kernel 7.0).
- GPU not listed: it is not detected; check seating and power before anything else.

## Stage 3: the guest (Boss, Proxmox web page)
- New VM, machine type q35, UEFI (OVMF), Debian 13 server image, 8 cores, 16 GB RAM, 60 GB disk on local-zfs, VirtIO network, start at boot off for now.
- Add the GPU (and its audio function) as a PCI device, "All Functions", PCI-Express ticked; do not make it the primary display.
- SSH key for Alice pasted at install (Alice supplies a key pair for this guest only).
- A snapshot before the first driver install so the guest can be put back.

## Stage 4: host changes (only if stage 1 shows they are needed; Boss decides each one)
Binding the GPU to the passthrough driver at boot (vfio-pci) and keeping the host from using it. Exact file edits to be written once stage 1 output is in; they are host system changes, each with an undo.

## Stage 5: inside the guest (Alice, over SSH)
- NVIDIA driver (a version that still supports Pascal; UNSURE which; check first), then `nvidia-smi`.
- Ollama, pinned; one model such as `qwen3:8b` at 4-bit.
- Measure: tokens per second, GPU memory used, and a set of sample jobs. Results go in `docs/02-research/engine-results.md`.

## Undo
Delete the guest and its PCI mapping; revert any host change from stage 4. The TrueNAS VM and everything else is untouched at every stage.
