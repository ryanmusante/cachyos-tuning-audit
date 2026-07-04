# CachyOS Tuning Profile — Beelink GTR9 Pro

A gaming- and LLM-oriented CachyOS desktop tuning profile for the Beelink GTR9 Pro
(Ryzen AI Max+ 395 "Strix Halo"), applied by `ry-install.fish`. It manages 17 system
and user configuration files spanning the kernel command line, package set, systemd
services, network stack, GPU/CPU power, memory, storage, and an on-screen HUD.

Corresponds to `ry-install.fish` **7.88.3**.

## Hardware target

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) ·
Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤96 GB as
VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 ·
140 W TDP · CachyOS · systemd-boot.

## Requirements

- **Kernel ≥ 6.19** (hard floor) — anchored to gfx1151 "MES-0x86" amdgpu stability plus
  the RTL8127 suspend/shutdown-hang fix and r8169 support, all of which land at ≥ 6.19.
- **CPU** matching `Ryzen AI Max` (install gate).
- **Mesa ≥ 26.0** (soft warning below).

## What it configures

- **Kernel command line (17 params)** — CPU/power: `amd_pstate=active`,
  `processor.max_cstate=1`, `split_lock_detect=off`, `clearcpuid=514`, `tsc=reliable`;
  I/O latency: `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`,
  `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`; platform: `amd_iommu=off`,
  `ipv6.disable=1`, `nowatchdog`, `zswap.enabled=0`, `8250.nr_uarts=0`, `quiet`;
  filesystem: `fsck.mode=force`, `fsck.repair=yes`.
- **Packages** — 17 added, 9 removed, 10 masked units; RADV Vulkan stack
  (vulkan-radeon + lib32-vulkan-radeon).
- **Network** — IPv4-only nftables ruleset (default-deny inbound, established/related
  allowed, inbound ICMP echo accepted, remote-play ports gated); systemd-resolved with
  mDNS/LLMNR/DoT off; NetworkManager on wpa_supplicant; regulatory domain US.
- **GPU / CPU power** — amd_pstate EPP `balance_performance` under the `powersave`
  governor (`dynamic_epp` disabled); GPU DPM level `auto`; udev rules pinning EPP and
  GPU state.
- **Memory / storage** — BBR congestion control + `fq` qdisc; tuned sysctls
  (`vm.max_map_count=2147483642`, `vm.swappiness=150`, `vm.compaction_proactiveness=0`);
  ext4 mounted `noatime,lazytime,commit=10`; zswap off.
- **HUD** — a readout-only MangoHud config (19 directives: fps/frametime, GPU/CPU
  clocks, temps, and power, VRAM/RAM), toggled with `Shift_R+F12`, enabled via
  `MANGOHUD=1`.

## At a glance

```
║ AREA                ║ COUNT ║
║─────────────────────║───────║
║ kernel cmdline      ║ 17    ║
║ packages added      ║ 17    ║
║ packages removed    ║ 9     ║
║ masked units        ║ 10    ║
║ sysctl values       ║ 9     ║
║ environment vars    ║ 11    ║
║ MangoHud directives ║ 19    ║
║ managed files       ║ 17    ║  (15 system + 2 user)
```

## Notable design choices

Deliberate trade-offs the profile makes for latency and throughput on this fixed
hardware:

- **`amd_iommu=off`** — AMD-Vi disabled for the unified-memory pool; does not affect
  ROCm on gfx1151. VFIO/SR-IOV users opt back in with `amd_iommu=on iommu=pt`.
- **IPv6 disabled + IPv4-only firewall** — `ipv6.disable=1` with an IPv4-only nftables
  ruleset that accepts inbound ping. Dual-stack users remove the token and restore IPv6
  rules.
- **`powersave` governor + EPP `balance_performance`** — the EPP-honoring
  maximum-performance configuration under `amd_pstate=active`.
- **`split_lock_detect=off` + `clearcpuid=514`** — latency-oriented CPU settings.

## Contents

- `cachyos-tuning-audit.md` — full parameter-by-parameter tuning reference: every
  managed value, the exact rendered configuration bodies, and the `ry-install --verify`
  surface.
- `CHANGELOG.md` — version history.

## Usage

The profile is applied by `ry-install.fish` on a CachyOS installation matching the
hardware target above. Consult the tuning reference for the exact managed values and the
post-install `--verify` checks.
