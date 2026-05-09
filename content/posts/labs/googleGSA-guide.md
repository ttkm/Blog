---
title: "Google Search Appliance - Repurposing (quick guide)"
date: 2026-05-07T17:21:45-05:00
draft: false
toc: true
slug: "google-gsa-repurpose-guide"
images:
tags:
  - homelab
  - dell
  - linux
  - ubuntu
  - raid
  - networking
---

Short checklist for turning a **Google Search Appliance** back into a normal Dell server.  
Hardware here is **PowerEdge R720xd-class (GSA T4/T5)**. Other GSA generations may be **R730 or different**, verify your chassis before you download BIOS or assume jumper layout.

For the full debug trail (display, iDRAC, every wrong turn), see the [long lab log](/posts/other/googlegsa/).

---

## Before you start

- Expect Google OEM BIOS: locked setup, odd Lifecycle Controller behavior, iDRAC may not init fans at 100% is common until firmware is normal Dell.
- Original appliance disks are often LUKS-encrypted. Treat as wipe and reinstall. Do not expect to recover the old OS without keys.
- Match BIOS to exact model (e.g. R720xd vs R730). Wrong package = brick risk. Use Dell's Linux `.BIN` self-extractor for your model, not another server's.
- VGA only on many units, use active VGA to HDMI if you need a modern monitor.

---

## High-level path (what actually works)

1. POST: if you see Ctrl+Enter to bypass a password gate, use it so boot can continue.
2. Enter PERC config with Ctrl+R (works even when BIOS setup is locked).
3. Clear all foreign configs and virtual disks, full clean slate on the RAID controller before you rebuild layouts (`metadata stacking` bites if you only 'delete VD' and layer changes).
4. Boot USB installer, often F11 Boot Manager and once POST you can proceed to use a BIOS/MBR-friendly Ubuntu USB if the machine is legacy BIOS.
5. From Linux live session, run Dell's Linux BIOS updater for **your** model. Reboot and confirm F2 works without password, iDRAC link, and if fans normalize if fans were having issues.
6. In PERC, create the RAID layout you want. Install OS on the boot VD.
7. In BIOS: Boot Settings > Hard Disk Drive Sequence, put the VD that holds the OS first (PERC often has no separate 'make bootable' toggle).
8. If the installer boots but PXE loops after reboot the boot order is wrong or GRUB not on the correct disk MBR. Fix `grub-install` to the whole block device for the OS VD (see lab log for chroot example).
9. Networking: avoid NetworkManager + systemd-networkd both 'owning' interfaces. Pick one stack and align `netplan` (I made that mistake...).
10. Tailscale: ACL rules need `accept`, not `check` if you plan to use Tailscale, or SSH looks 'denied by policy'.
11. Wake-on-LAN: needs LAN broadcase. If you plan using WSL, It usually cannot send magic packets to the wire, ZeroTier/Tailscale won't carry WoL. Use native OS, LAN, or a small on-LAN relay (e.g. Pi is what I currently use).

---

## iDRAC tips

- Dedicated iDRAC port may need router DHCP (not a direct PC cable with no DHCP). Wait a few minutes after link for init.
- Enterprise license sticker sometimes matters for dedicated port behavior, plan on lifecycle-handset or shared NIC paths if stuck.

---

## Aftercare

- Rename hostname if installer inherited **`google`** (cosmetic).
- Document which VD is boot, which NIC is management vs data, and MAC*used for WoL.

---

## Related

- [Repurposing a Google Search Appliance (full lab log)](/posts/other/googlegsa/) - narrative, commands, and failure modes.
