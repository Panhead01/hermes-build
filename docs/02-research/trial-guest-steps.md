# Trial guest: steps (draft, updated 7 Oct 2026)

Goal: a new guest on the workstation with the GTX 1080 passed through, running one small model, measured. Boss runs every Proxmox step; Alice works inside the guest once it exists. Anything marked UNSURE is to be checked against the current Proxmox documentation before running.

## Stage 1: read-only look (DONE 7 Oct 2026)
Run by Boss in the Proxmox shell (node `pve`). Results, all OBSERVED on screen:
- **GPU:** NVIDIA GP104 GeForce GTX 1080 `[10de:1b80]` at `02:00.0`, its audio function `[10de:10f0]` at `02:00.1`. Also seen: an Intel device at `80:14.5` and the management controller's ASPEED graphics at `86:00.0` (not relevant).
- **Host driver now:** `nouveau` on the GPU, `snd_hda_intel` on the audio function. The host is using the card until a VM claims it.
- **IOMMU:** on. The log shows the VT-d tables loaded, interrupt remapping enabled, and devices assigned to IOMMU groups.
- **IOMMU groups:** `02:00.0` and `02:00.1` are alone in their group (nothing else in it). The SATA controller `80:17.0` (the one the TrueNAS VM uses) is alone in its own group, at an unrelated address.
- **Boot options:** `initrd=\EFI\proxmox\7.0.14-20-pve\initrd.img-7.0.14-20-pve root=ZFS=rpool/ROOT/pve-1 boot=zfs`. Plain ZFS with systemd-boot, no passthrough options. If a boot option is ever needed, it goes in `/etc/kernel/cmdline`, not GRUB.

## Stage 2: decision (7 Oct 2026)
GPU in a clean group, IOMMU on, NAS controller unaffected: go on to stage 3. Host driver changes (stage 4) are tried only if the first VM start fails.

## Stage 3: the guest (Boss, Proxmox web page)
1. **Install image.** Node `pve`, storage `local`, ISO Images, Download from URL: `https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.7.0-amd64-netinst.iso`, checksum algorithm SHA-256, value `a7ef94ac2fb9a7fec454552abd629b7cc9d5155c886165a45649f5ce6167e355` (read from Debian's SHA256SUMS on 7 Oct 2026).
2. **Create VM.** General: ID `212`, name `agent-trial`. OS: that ISO, type Linux. System: machine `q35`, BIOS OVMF (UEFI), Add EFI Disk, Pre-Enroll keys unticked (so Secure Boot does not block the NVIDIA driver), SCSI controller VirtIO SCSI single, Qemu Agent ticked. Disks: 60 GiB on `local-zfs`, Discard ticked. CPU: 1 socket, 8 cores, type `host`. Memory: 16384 MiB, Ballooning unticked (passthrough needs it off). Network: bridge `vmbr0`, VirtIO, firewall off for now. Do not start after creation.
3. **Add the GPU.** VM 212, Hardware, Add, PCI Device, Raw Device `0000:02:00`, All Functions and PCI-Express ticked, Primary GPU unticked.
4. **Install Debian** from the VM Console: no desktop, SSH server ticked, hostname `agent-trial`, a normal user (no root login), passwords typed only by Boss.
5. **Alice's access.** A key pair for this guest only was generated on the desktop (`~/.config/alice-ssh/id_ed25519_agenttrial`, no passphrase); Boss pastes the public half into the user's `authorized_keys`. Router DHCP reservation for the guest, then Alice connects.
6. **Snapshot** the guest before any driver install.

**Risk, stated plainly:** the host is using the GPU now. The first start with the card attached could rarely hang the host's console or the host itself, which would restart every other guest including the NAS. Do the first start at a quiet moment. If the host stops responding, restart it from the management controller's web page.

## Stage 4: host changes (only if the first start fails; Boss decides each one)
Likely needs: keep the host off the card (blacklist `nouveau`, bind the GPU to `vfio-pci`) and rebuild the boot image, which needs a planned host reboot. UNSURE whether Proxmox's own start-up binding is enough without that; try the plain start first. Exact edits to be written only if needed, each with an undo.

## Stage 5: inside the guest (Alice, over SSH)
- NVIDIA driver (a version that still supports Pascal; UNSURE which; check first), then `nvidia-smi`.
- Ollama, pinned; one model such as `qwen3:8b` at 4-bit.
- Measure: tokens per second, GPU memory used, and a set of sample jobs. Results go in `docs/02-research/engine-results.md`.

## Undo
Delete the guest and its PCI mapping; revert any host change from stage 4. The TrueNAS VM and everything else is untouched at every stage.
