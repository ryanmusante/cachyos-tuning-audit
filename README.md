# CachyOS Tuning Profile — Beelink GTR9 Pro

[![profile](https://img.shields.io/badge/profile-7.99.1-1793d1?style=flat-square)](CHANGELOG.md)
[![platform](https://img.shields.io/badge/platform-CachyOS-1793d1?style=flat-square)](#requirements)
[![silicon](https://img.shields.io/badge/silicon-gfx1151%20%2F%20Strix%20Halo-1793d1?style=flat-square)](#hardware-target)
[![kernel](https://img.shields.io/badge/kernel-%E2%89%A5%206.19-1793d1?style=flat-square)](#requirements)

> A gaming- and LLM-oriented CachyOS desktop tuning profile for the Beelink GTR9 Pro
> (Ryzen AI Max+ 395 "Strix Halo"). It manages 17 system and user configuration files
> spanning the kernel command line, package set, systemd services, network stack,
> GPU/CPU power, memory, storage, and an on-screen HUD.

Corresponds to `ry-install.fish` **7.99.1**.

## Hardware target

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) ·
Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤ 96 GB as
VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 ·
140 W TDP · CachyOS · systemd-boot.

## Requirements

| Requirement | Minimum |
|---|---|
| Platform | CachyOS · systemd-boot · ext4 root |
| Kernel | ≥ 6.19 — gfx1151 MES-0x86 amdgpu; **unconditional** (no override; the RTL8127 r8169 + suspend-hang fixes land below the floor) |
| CPU | matches `Ryzen AI Max` (sole skip: `RY_INSTALL_SKIP_HARDWARE_CHECK=1`) |
| Mesa | ≥ 26.0 (soft warning below; `vercmp` output validated) |

## What it configures

- **Kernel command line (17 params)** — CPU/power: `amd_pstate=active`,
  `processor.max_cstate=1`, `split_lock_detect=off`, `clearcpuid=umip` (renamed from
  `514`; version-stable string form), `tsc=reliable`; I/O latency:
  `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`,
  `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`; platform: `amd_iommu=off`,
  `ipv6.disable=1`, `nowatchdog`, `zswap.enabled=0`, `8250.nr_uarts=0`, `quiet`;
  filesystem: `fsck.mode=force`, `fsck.repair=yes`.
- **Packages** — **19** added (now incl. `pacman-contrib` + `archlinux-contrib`),
  9 removed, **12** masked units (now incl. `avahi-daemon` service + socket); RADV
  Vulkan stack (`vulkan-radeon` + `lib32-vulkan-radeon`).
- **Modules** — single merged `/etc/modprobe.d/60-ry-modules.conf`: MT7925 PCIe ASPM
  off + `amdxdna` blacklisted by default (the XDNA NPU probes `-EINVAL` under
  `amd_iommu=off`; opt-in via `BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`,
  validator-enforced).
- **Network** — IPv4-only nftables ruleset (default-deny inbound, established/related
  allowed, inbound ICMP echo accepted, remote-play ports gated), now `nft -c`
  pre-validated before every deploy/reload; systemd-resolved with mDNS/LLMNR/DoT off;
  NetworkManager on wpa_supplicant; regulatory domain US.
- **GPU / CPU power** — amd_pstate EPP `balance_performance` (hoisted, enum-gated)
  under the `powersave` governor (`dynamic_epp` disabled); GPU DPM level `auto`; udev
  rules pinning EPP and GPU state (GPU matcher fixed to `ENV{DEVTYPE}` — the prior rule
  never applied).
- **Memory / storage** — BBR congestion control + `fq` qdisc; tuned sysctls
  (`vm.max_map_count=2147483642`, `vm.swappiness=150`, `vm.compaction_proactiveness=0`);
  ext4 mounted `noatime,lazytime,commit=10`; zswap off.
- **HUD** — readout-only MangoHud config (19 active directives; `cpu_temp` shipped
  commented with an explanatory note), toggled with `Shift_R+F12`, enabled via
  `MANGOHUD=1`.
- **Safety rails** — all 4 boot-critical files get `.ry.bak` + post-write
  verify/restore; long package/boot operations are wall-clock-capped at a 7200 s floor
  instead of exempt; NTP remediation is unconditional with a chronyd/ntpd conflict
  guard.

## At a glance

```
║ AREA                ║ COUNT ║
║─────────────────────║───────║
║ kernel cmdline      ║ 17    ║
║ packages added      ║ 19    ║
║ packages removed    ║ 9     ║
║ masked units        ║ 12    ║
║ sysctl values       ║ 9     ║
║ environment vars    ║ 11    ║
║ MangoHud directives ║ 19    ║  (active; +1 commented # cpu_temp)
║ managed files       ║ 17    ║  (15 system + 2 user)
║ backup targets      ║ 4     ║  (all boot-critical files)
║ preflight tripwires ║ 21    ║  (hard-asserted array counts)
```

## Notable design choices

Deliberate trade-offs the profile makes for latency and throughput on this fixed
hardware:

- **`amd_iommu=off`** — AMD-Vi disabled for the unified-memory pool; does not affect
  ROCm on gfx1151. Named cost: the XDNA NPU is blacklisted (`amdxdna` cannot probe
  without the IOMMU). VFIO/SR-IOV/NPU users opt back in with `BLACKLIST_AMDXDNA=false`
  + `amd_iommu=on iommu=pt`.
- **IPv6 disabled + IPv4-only firewall** — `ipv6.disable=1` with an IPv4-only nftables
  ruleset that accepts inbound ping; avahi masked (unit + socket) closes the second
  mDNS responder. Dual-stack users remove the token and restore IPv6 rules.
- **`powersave` governor + EPP `balance_performance`** — the EPP-honoring
  maximum-performance configuration under `amd_pstate=active`.
- **`split_lock_detect=off` + `clearcpuid=umip`** — latency-oriented CPU settings.

## Contents

- `cachyos-tuning-audit.md` — full parameter-by-parameter reference: every managed
  value, the exact rendered configuration bodies, and the post-install `--verify`
  checks.
- `CHANGELOG.md` — version history.

## Usage

The profile is applied by `ry-install.fish` on a CachyOS installation matching the
hardware target above. Consult the reference for the exact managed values and the
post-install `--verify` checks. Upgrading from ≤ 7.98.x: remove the two superseded
modprobe drop-ins once (`sudo rm /etc/modprobe.d/60-ry-mt7925e.conf
/etc/modprobe.d/60-ry-blacklist-amdxdna.conf`).
