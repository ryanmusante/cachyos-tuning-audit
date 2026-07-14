# CachyOS Tuning Profile — Beelink GTR9 Pro

[![profile](https://img.shields.io/badge/profile-7.105.9-1793d1?style=flat-square)](CHANGELOG.md)
[![platform](https://img.shields.io/badge/platform-CachyOS-1793d1?style=flat-square)](#requirements)
[![silicon](https://img.shields.io/badge/silicon-gfx1151%20%2F%20Strix%20Halo-1793d1?style=flat-square)](#hardware-target)
[![kernel](https://img.shields.io/badge/kernel-6.18.4%20advisory-1793d1?style=flat-square)](#requirements)

> A gaming- and LLM-oriented CachyOS desktop tuning profile for the Beelink GTR9 Pro
> (Ryzen AI Max+ 395 "Strix Halo"). It manages 17 system and user configuration files
> spanning the kernel command line, package set, systemd services, network stack,
> GPU/CPU power, memory, storage, and an on-screen HUD.

Corresponds to `ry-install.fish` **7.105.9**. The reference document is formatted as a
deep-research brief: actionable items only, ordered by research priority.

## Hardware target

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) ·
Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤ 96 GB as
VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 ·
140 W TDP (README BIOS ceiling 85 W) · CachyOS · systemd-boot.

## Requirements

```
║ REQUIREMENT ║ MINIMUM                            ║
║─────────────║────────────────────────────────────║
║ Platform    ║ CachyOS · systemd-boot · ext4 root ║
║ Kernel      ║ 6.18.4 ADVISORY floor (not         ║
║             ║ enforced) — RTL8127 r8169 +        ║
║             ║ suspend-hang regression baseline;  ║
║             ║ gfx1151 hang fix is firmware       ║
║             ║ (linux-firmware MES 0x86), not     ║
║             ║ kernel                             ║
║ CPU         ║ matches `Ryzen AI Max` (sole skip: ║
║             ║ RY_INSTALL_SKIP_HARDWARE_CHECK=1)  ║
║ Mesa        ║ ≥ 26.0 (soft warning below)        ║
```

## What it configures

- **Kernel command line (17 params)** — CPU/power: `amd_pstate=active`,
  `processor.max_cstate=1`, `split_lock_detect=off`, `clearcpuid=umip`, `tsc=reliable`;
  I/O + link latency: `pcie_aspm=off` (global ASPM disable — replaces
  `pcie_aspm.policy=performance` and the former per-module MT7925 option),
  `nvme_core.default_ps_max_latency_us=0`, `usbcore.autosuspend=-1`,
  `btusb.enable_autosuspend=n`; platform: `amd_iommu=off`, `ipv6.disable=1`,
  `nowatchdog`, `zswap.enabled=0`, `8250.nr_uarts=0`, `quiet`; filesystem:
  `fsck.mode=force`, `fsck.repair=yes`.
- **Packages** — 18 added (incl. `pacman-contrib`), 9 removed, 12 masked units
  (incl. `avahi-daemon` service + socket); RADV Vulkan stack (`vulkan-radeon` +
  `lib32-vulkan-radeon`).
- **Modules** — `/etc/modprobe.d/60-ry-modules.conf` now carries the `amdxdna`
  blacklist only (the XDNA NPU probes `-EINVAL` under `amd_iommu=off`; opt-in via
  `BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`, validator-enforced — that path
  renders a comment-only file, accepted by the format validator).
- **Gaming environment (12 vars)** — RADV ICD pin, Wayland Proton, FSR4 upgrade on
  RDNA 3.5, local + 16G shader caches, **`VKD3D_CONFIG=descriptor_heap`**, silenced
  DXVK/VKD3D/WINE logging, `MANGOHUD=1`.
- **Network** — IPv4-only nftables ruleset (default-deny inbound, established/related
  allowed, inbound ICMP echo accepted, remote-play ports gated), `nft -c` pre-validated
  before every deploy/reload; systemd-resolved with mDNS/LLMNR/DoT off; NetworkManager
  on wpa_supplicant with Wi-Fi powersave off; regulatory domain US.
- **GPU / CPU power** — amd_pstate EPP `balance_performance` (enum-gated) under the
  `powersave` governor (`dynamic_epp` disabled); GPU DPM level `auto`; udev rules
  pinning EPP and GPU state.
- **Memory / storage (10 sysctls)** — BBR + `fq`; `vm.max_map_count=2147483642`,
  `vm.swappiness=150`, `vm.compaction_proactiveness=0`,
  **`vm.watermark_boost_factor=0`**; ext4 mounted `noatime,lazytime,commit=10`;
  NVMe scheduler `none`; zswap off.
- **HUD** — readout-only MangoHud config (19 active directives; `cpu_temp` shipped
  commented — re-enabling re-trips MangoHud #1794), toggled with `Shift_R+F12`.
- **Safety rails** — all 4 boot-critical files get `.ry.bak` + post-write
  verify/restore; long package/boot operations wall-clock-capped at a 7200 s floor;
  NTP remediation unconditional with a chronyd/ntpd/openntpd conflict guard.

## At a glance

```
║ AREA                ║ COUNT ║
║─────────────────────║───────║
║ kernel cmdline      ║ 17    ║
║ packages added      ║ 18    ║
║ packages removed    ║ 9     ║
║ masked units        ║ 12    ║
║ sysctl values       ║ 10    ║
║ environment vars    ║ 12    ║
║ MangoHud directives ║ 19    ║  (active; +1 commented # cpu_temp)
║ managed files       ║ 17    ║  (15 system + 2 user)
║ backup targets      ║ 4     ║  (all boot-critical files)
║ preflight tripwires ║ 21    ║  (hard-asserted array counts)
```

## Notable design choices

Deliberate trade-offs the profile makes for latency and throughput on this fixed
hardware:

- **`pcie_aspm=off`** — the kernel ASPM driver is disabled outright (MT7925
  coredump/BT-reconnect/assoc mitigation + NVMe latency). Supersedes both
  `pcie_aspm.policy=performance` and the per-module `mt7925e disable_aspm=1` option.
- **`amd_iommu=off`** — AMD-Vi disabled for the unified-memory pool; ROCm on gfx1151
  unaffected. Named cost: the XDNA NPU is blacklisted. NPU/VFIO/SR-IOV users opt back
  in with `BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`.
- **IPv6 disabled + IPv4-only firewall** — `ipv6.disable=1` with an IPv4-only nftables
  ruleset that accepts inbound ping; avahi masked (unit + socket) closes the second
  mDNS responder. Dual-stack users remove the token and restore IPv6 rules.
- **`powersave` governor + EPP `balance_performance`** — the EPP-honoring
  maximum-performance configuration under `amd_pstate=active`.
- **`split_lock_detect=off` + `clearcpuid=umip`** — latency-oriented CPU settings.
- **Kernel floor advisory-only** — the hard-fail validator was removed; 6.18.4 is a
  regression baseline documented in a script comment, not a gate.
- **HUD dual-published** — the readout-only MangoHud config also ships standalone as
  `mangohud-gtr9-pro` (v1.17.0 cross-audited: 19/19 active-directive parity in set and
  order, comment-only byte delta; the installer's embedded generator is the source of
  truth per the repo's own changelog).

## Contents

- `cachyos-tuning-audit.md` — deep-research brief: research priority queue, every
  managed value, the exact rendered configuration bodies, the verify surface, and the
  robustness invariants.
- `CHANGELOG.md` — version history.

## Usage

The profile is applied by `ry-install.fish` on a CachyOS installation matching the
hardware target above. Consult the reference for the exact managed values and the
post-install `--verify` checks. Upgrading from ≤ 7.98.x: remove the two superseded
modprobe drop-ins once (`sudo rm /etc/modprobe.d/60-ry-mt7925e.conf
/etc/modprobe.d/60-ry-blacklist-amdxdna.conf`) — as of 7.105.9 no in-script or README
guard detects the leftovers (open finding, reference §6).
