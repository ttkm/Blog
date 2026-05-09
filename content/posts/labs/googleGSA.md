---
title: "Repurposing a Google Search Appliance Lab Log"
date: 2026-05-07T16:10:31-05:00
draft: false
toc: true
images:
tags:
  - homelab
  - dell
  - linux
  - ubuntu
  - raid
  - networking
  - tailscale
  - wake-on-lan
---

> **Hardware:** I bought a Dell PowerEdge R720xd (Google Search Appliance T4/T5 variant) - 96 GB DDR3 ECC RAM, PERC H710P Mini RAID controller, BCM5720 quad-NIC. This is a raw log of every bug I hit and every step that actually worked while repurposing it from scratch. I wanted to mess around with it and learn from it, so expect beginner mistakes.....
>
> **Want steps only?** See the [quick repurposing guide](/posts/other/google-gsa-repurpose-guide/) (checklist + precautions, no war story).

---

# What Is a Google Search Appliance?

The Google Search Appliance was an enterprise on-premises search product a physical Dell PowerEdge server sold to businesses with Google's branding and a custom OEM BIOS locked on top. Hardware-wise it's a standard 12th-gen Dell (R720 or R730 class depending on the generation), but the firmware is not stock - Google modified it to lock BIOS access, disable standard reset paths, and run their own Lifecycle Controller splash.

When Google discontinued the GSA around 2019, units started flooding eBay. The catch: they look like normal servers but behave differently until you replace the firmware.

---

## Phase 1 - First Boot and Display

### The server has no HDMI

The R720xd only has a VGA port. To connect it to a modern monitor you need a VGA 🠞 HDMI active converter (not a passive cable). A passive cable won't work, it briefly showed the POST screen then the monitor lost signal entirely.

The system was booting fine, but after POST the video mode changed and the passive adapter lost sync. It wasn't a boot failure just a display path failure.

Use an active powered VGA 🠞 HDMI converter, or borrow a native VGA monitor once to do initial setup don't make the same mistake as I did.

### First boot observations

On first power-on with one PSU connected:

- Front health LED: slow amber blink
- Drive bay LEDs: all green
- Fans: slow 🠞 loud during POST 🠞 never settled

The amber blink was just a PSU redundancy warning, perfectly normal with only one PSU plugged in. Not a hardware fault, be sure to have both PSU connected.

### Fans stuck at 100%

The fans never settled down after POST. This turned out to be the iDRAC/BMC not completing initialization, which puts the system into a **thermal failsafe mode** that locks fans at maximum. This is not a BIOS issue or a hardware failure, it's the management controller not coming up cleanly, which on a GSA is caused by the Google firmware. Although of course saying this now may seem basic knowledge but at the moment it took me a while to figure out since this is my first time dealing with a physical server.

---

## Phase 2 - Trying to Access iDRAC

### Direct Ethernet PC <> iDRAC did not work

First attempt: plug iDRAC port directly into PC, open browser. Result: `Media State: Media disconnected`.

The problem wasn't IP configuration, it was a physical layer issue. The iDRAC port had no link LEDs at all because the iDRAC subsystem wasn't initialized yet (again, Google firmware issue). Even after establishing a physical link, a direct PC-to-server cable has no DHCP server, so iDRAC gets no IP unless you set a static one.

**What works:**
- Connect iDRAC to a **local router** (no internet needed, just a LAN router providing DHCP)
- Wait 2–3 minutes for iDRAC to initialize and get an IP
- Check the router's DHCP client list
- Open `https://<idrac-ip>` - default credentials: `root` / `calvin`

However on this GSA the iDRAC port had a **"license required"** sticker on the rear. Without an Enterprise iDRAC license the dedicated port may be limited or disabled. This was secondary to the firmware issue.

---

## Phase 3 - The Google BIOS Lock

### F2, F10, F11 - all blocked

On first successful boot with a working display:

```text
Memory initialization warning detected
DDR3 training failure / write leveling DIMM A5/A4
MEMBIST memory test failure DIMM A4
Type password and... press CTRL+ENTER to disable password security
```

Pressing `CTRL+ENTER` bypassed the password prompt and let the system continue POST. This was the first real entry point.

After that: `Strike F1 to continue, F2 to run System Setup`

The memory errors (DIMM A4/A5) were real, those slots had failed or empty DIMMs. They didn't stop the system from booting but needed to be handled.

### Jumper resets did not clear the password

The R720xd has two relevant jumpers on the motherboard:

| Jumper | Purpose |
|--------|---------|
| `PWRD_EN` | Password enable/disable |
| `NVRAM_CLR` | Clear NVRAM / stored config |

Moving `NVRAM_CLR` to the clear position produced this message on next boot:

```text
Applying configuration changes...
System reboot required for configuration changes to take effect.
Please power off the system and remove NVRAM_CLR jumper.
Press F1 to continue, F2 for Setup, F11 for Boot Menu
Initializing firmware interfaces...
Lifecycle Controller: Collecting System Inventory...
```

The NVRAM reset was detected and applied. But the setup password prompt still appeared afterward. The Google firmware stores the password outside normal NVRAM standard Dell jumper resets don't reach it.

> Jumpers are not useless here. The NVRAM reset does clear some state and is part of the process. But alone, it doesn't remove the Google password.

### BIOS recovery trigger attempts

Tried `CTRL+ESC` + power button to trigger Dell's pre-BIOS recovery mode (which reads a `BIOS_IMG.rcv` file from USB before normal firmware loads). No response.. The Google firmware has this disabled or blocks it.

### The actual unlock path (from Reddit r/servers)

Found a thread from a user who had already solved this exact problem on a GSA T4 (also R720). The method that works:

1. Enter the RAID config utility (`CTRL+R` at boot) - this is accessible even with the BIOS locked
2. Wipe the existing RAID config completely
3. Build a new array (or leave it empty)
4. Boot from a USB (the system will fall through to USB if no bootable disk exists)
5. Install Linux (Ubuntu)
6. From Linux, flash the **Dell standard BIOS** over the Google OEM BIOS
7. After flash, the password is gone - system behaves as a normal Dell R720xd

You don't need to crack or bypass the password. You just replace the firmware that enforces it.

---

## Phase 4 - Booting Ubuntu

### Getting into USB boot

With `CTRL+ENTER` bypassing the password prompt at POST, the system could reach `F11` (Boot Manager). With a bootable Ubuntu USB inserted, it showed up and booted successfully.

Used: Ubuntu 24.04 live USB made with Rufus (MBR, FAT32 - important for BIOS boot mode).

### What was on the original drives

Looking at the Ubuntu installer's storage layout screen showed:

```text
crypto_LUKS  partition 1  existing, not mounted
crypto_LUKS  partition 2  existing, not mounted
crypto_LUKS  partition 3  existing, not mounted
swap         partition     unused
ESP          partition     existing, unused
```

Multiple `crypto_LUKS` encrypted partitions across several disks. No readable root filesystem, no mountable OS partition. The previous Google appliance OS was **LUKS-encrypted** and inaccessible without the passphrase which I do not have.

**This is not "OS encryption" in the way Linux users set it up.** It's remnants of a deployed appliance image that was encrypted at the storage level. Without the key it's unrecoverable, and there's no reason to try in my opinion, don't get me wrong I was very curious and wished to see the contents inside even if it was boring files. But realistically it's just a deployed enterprise appliance, not personal data.

---

## Phase 5 - Flashing the Dell BIOS

### Why this works

The Dell official BIOS updater validates the system model before flashing. If you give it the wrong file it refuses to run. If you give it the right file it stages the update and rewrites the firmware region including the part that stored Google's OEM password. After reboot: standard Dell BIOS, no password, iDRAC initializes normally.

### What file to use

Server is a Dell PowerEdge R720xd (confirmed: DDR3 RAM, PERC H710P, 12th gen). The correct file:

```text
BIOS_DYH8T_LN_2.7.0.BIN
```

- `DYH8T` - Dell package identifier for R720/R720xd
- `LN` - Linux executable format (not the Windows `.EXE`)
- `2.7.0` - version

**Do not use an R730 or R710 BIOS. If you have the same GSA as me** The flash tool will catch mismatches in most cases but it's not worth risking a brick.

### Steps from Ubuntu live session

```bash
# USB was the same one used to boot Ubuntu, with the .BIN added to the root
ls /media/ubuntu/

# Navigate to USB
cd /media/ubuntu/<USB_NAME>

# Make executable
chmod +x BIOS_DYH8T_LN_2.7.0.BIN

# Flash
sudo ./BIOS_DYH8T_LN_2.7.0.BIN
```

The tool ran, verified the system, staged the update, and prompted for reboot. During reboot the BIOS was written. After reboot:

- F2 entered without any password prompt
- iDRAC initialized - port LEDs lit up
- **Fans dropped to normal speed** (this was the confirmation everything worked)

---

## Phase 6 - RAID Setup and OS Installation

### Clearing the old RAID config

Before installing Ubuntu properly, the existing RAID configuration was wiped via `CTRL+R` at boot (PERC configuration utility):

- Deleted VD0 (RAID 10, 3.7 TB)
- Deleted VD1 (RAID 0, 92 GB)
- Cleared all foreign configurations and metadata

Then rebuilt clean:

| VD | Type | Disks | Size | Purpose |
|----|------|-------|------|---------|
| VD0 | RAID 10 | 8x HDD | ~3.7 TB | Storage |
| VD1 | RAID 0 | 1x SSD | 92 GB | OS |

> **Important:** RAID controller metadata persists even after deleting virtual disks. If you layer changes without a full reset first, you end up in inconsistent states where the controller initializes slowly or erratically. Always do a full clear.

### Installing Ubuntu

Booted the Ubuntu installer, targeted VD1 (92 GB RAID 0) for the OS, used default LVM layout. Install completed fine.

### PXE boot loop after install

After removing the USB and rebooting: PXE boot loop. The system ignored the hard drive entirely and fell through to network boot, which failed.

Cause: the BIOS was trying to boot the wrong virtual disk. Dell PERC systems in BIOS mode only boot one virtual disk - and it was pointing at VD0 (the 3.7 TB data disk) instead of VD1 where Ubuntu was installed.

The PERC H710P Mini's virtual disk management menu does not have a "Make Bootable" option. Boot selection is done at the BIOS level:

```text
F2 🠞 System BIOS 🠞 Boot Settings 🠞 Hard Disk Drive Sequence
```

Set VD1 (92 GB RAID 0) as first in the sequence.

Even after that: still booting into PXE. GRUB had been installed to the filesystem but not correctly to the MBR of the RAID0 disk.

**Fix - boot live USB and reinstall GRUB to the correct device:**

```bash
lsblk
# confirm RAID0 OS disk = /dev/sdb

sudo mount /dev/ubuntu-vg/ubuntu-lv /mnt
sudo mount /dev/sdb2 /mnt/boot
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt

grub-install --recheck --force /dev/sdb
update-grub
```

After this: Ubuntu booted correctly.

---

## Phase 7 - Networking Broken on First Boot

### What the server looked like after Ubuntu booted

```text
eno1  UP    xxx.xxx.x.xxx  (active)
eno2  DOWN  NO-CARRIER
eno3  DOWN  NO-CARRIER
eno4  DOWN  NO-CARRIER
zt... UP    (ZeroTier interface - was already installed)
```

IP existed but DNS was broken. Some pings worked, others didn't. NetworkManager commands had no effect on eno1.

### Root cause: dual network stack conflict

Ubuntu server installed with both `NetworkManager` and `systemd-networkd` active simultaneously. `netplan` was configured to use `systemd-networkd` as the backend, so `systemd-networkd` was actually controlling the interfaces. `NetworkManager` was running but had no authority.

This produced:
- `eno1` marked as "unmanaged" by NetworkManager
- DHCP inconsistencies
- DNS failures
- Partial connectivity

**Fix:**

```bash
sudo systemctl restart NetworkManager
```

This was enough to get NetworkManager to take control properly. IPv4 came up stable, DNS resolved, LAN connectivity stable.

The real proper fix is to pick one and commit, either disable `systemd-networkd` and use NetworkManager, or disable NetworkManager and configure `netplan` cleanly for `systemd-networkd`. Running both creates unpredictable states.

---

## Phase 8 - SSH and Remote Access

### SSH was never actually the problem

OpenSSH was installed and running on port 22 from the first boot. UFW was inactive (no rules blocking anything). The server was accessible locally via:

```bash
ssh user@xxx.xxx.x.xxx
```

The hostname on the server was set to `google` (leftover from the appliance image's hostname configuration).

### ZeroTier was already installed

The ZeroTier interface (`zt...`) showed up in `ip a` with a stable peer connection (PLANET + LEAF nodes visible). This was already configured on the network before the OS install ZeroTier persists across reinstalls if the identity files survive.

ZeroTier works for SSH. It does **not** work for Wake-on-LAN because WoL requires Layer 2 broadcast packets, and ZeroTier is a Layer 3 VPN broadcasts don't cross it.

### Tailscale setup and ACL bug

Installed Tailscale for more reliable remote access:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Server got a `100.x.x.x` Tailscale IP. But SSH over Tailscale failed with:

```
tailnet policy does not permit you to SSH to this node
```

**Root cause:** The Tailscale ACL policy used `check` instead of `accept`. In Tailscale's ACL syntax, `check` means "test/evaluate this rule" it does not actually allow traffic. The policy was syntactically valid but didn't permit anything.

**Fix:** Edit the ACL in the Tailscale admin console:

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": ["<server-tag-or-ip>:22"]
    }
  ]
}
```

After saving with `accept` rules: SSH over Tailscale worked immediately.

---

## Phase 9 - Wake-on-LAN

### Local WoL

After confirming the NIC LEDs stayed lit after shutdown (indicating standby power is active), WoL was tested locally:

```powershell
# Windows PowerShell
$mac = "xx-xx-xx-xx-xx-xx"
$broadcast = [System.Net.IPAddress]::Broadcast
$packet = [byte[]]@(0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF)
$macBytes = $mac -split '-' | ForEach-Object { [Convert]::ToByte($_, 16) }
for ($i = 0; $i -lt 16; $i++) { $packet += $macBytes }
$udp = New-Object System.Net.Sockets.UdpClient
$udp.Connect("xxx.xxx.x.xxx", 9)
$udp.Send($packet, $packet.Length)
$udp.Close()
```

Worked. Server powered on.

### WSL WoL failed

Trying to send the magic packet from WSL (Windows Subsystem for Linux) did not work. WSL cannot reliably send Layer 2 broadcast packets since its virtual network stack intercepts them before they reach the physical NIC. This is a known WSL limitation with WoL.

**Fix:** Run WoL from native Windows PowerShell, not from WSL.

### BIOS setting needed

In BIOS (`F2 🠞 System BIOS 🠞 Integrated Devices 🠞 NIC`):

- Set NIC to `Enabled (OS)` mode
- Enable `Preboot Wake-on-LAN`

WoL depends more on the NIC having standby power than on what BIOS labels say if the NIC LEDs are on after shutdown, standby power is present and WoL will work.

### Remote WoL via Raspberry Pi

WoL magic packets must originate from **inside the LAN** - they're Layer 2 broadcasts and don't route over the internet. The solution for remote wake: a Raspberry Pi that stays always-on inside the home network, accessible via Tailscale or ZeroTier.

Architecture:
```text
Remote device 🠞 SSH (Tailscale) 🠞 Raspberry Pi 🠞 WoL broadcast 🠞 Server wakes up
```

Script on the Pi (`wake.sh`):

```bash
#!/bin/bash

MAC="xx:xx:xx:xx:xx:xx"
BCAST="xxx.xxx.x.xxx"

echo "Sending WoL..."
wakeonlan -i "$BCAST" "$MAC"
echo "Done"
```

```bash
sudo apt install wakeonlan -y
chmod +x wake.sh
```

From anywhere in the world:
```bash
ssh pi@100.x.x.x   # Tailscale IP of the Pi
./wake.sh
```

---

## Final System State

| Component | Status |
|-----------|--------|
| BIOS | Standard Dell v2.7.0 - Google firmware gone |
| iDRAC | Initializing normally, fans under control |
| RAM | 86–96 GB DDR3 ECC active (DIMM A4/A5 slots avoided) |
| Storage | RAID 10 (3.7 TB data) + RAID 0 (92 GB OS) - both Optimal |
| OS | Ubuntu, stable, temps 40–52°C |
| Network | eno1 up, 192.168.1.47, NetworkManager controlling interfaces |
| SSH | Working locally + via Tailscale (100.x.x.x) |
| Firewall | UFW active, SSH allowed |
| WoL | Working locally via PowerShell, remote via Pi relay |
| Fans | Normal speed - iDRAC managing thermal properly |

---

## What Actually Caused Each Problem

| Problem | Real cause |
|---------|-----------|
| Fans at 100% | iDRAC not initializing - Google firmware |
| No iDRAC link | Same - BMC not coming up under Google firmware |
| BIOS password bypass failed | Google firmware stores password outside standard NVRAM |
| "Encrypted drives" | LUKS on top of PERC RAID volumes - not recoverable without key |
| PXE boot loop | Wrong RAID virtual disk set as boot target + GRUB not written to MBR correctly |
| RAID instability | Metadata stacking from changes without full controller reset |
| Network broken after install | NetworkManager + systemd-networkd both active, both fighting |
| Tailscale SSH denied | ACL used `check` instead of `accept` |
| WoL failed from WSL | WSL cannot send Layer 2 broadcasts |

---

## Summary

- The GSA is just a Dell R720xd with Google OEM firmware. Once the BIOS is replaced it's a normal server.
- Standard Dell jumper resets (`PWRD_EN`, `NVRAM_CLR`) do partially work - they clear NVRAM state - but don't remove the Google password on their own. The password lives in firmware, not NVRAM.
- The actual unlock path requires booting external media (which is possible via `CTRL+ENTER` 🠞 `F11`), installing Linux, and flashing the standard Dell BIOS from there.
- RAID controller metadata persists through virtual disk deletions. Always do a full controller reset before rebuilding from scratch.
- On Dell PERC systems, boot drive selection is at the BIOS level (`Boot Settings 🠞 Hard Disk Drive Sequence`), not inside the PERC utility.
- Don't run NetworkManager and systemd-networkd at the same time. Pick one.
- Tailscale ACL `check` ≠ `accept`. Easy mistake to make, confusing error message.
- WoL from WSL doesn't work. Use PowerShell or a LAN relay.

