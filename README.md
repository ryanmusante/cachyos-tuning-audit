# CachyOS Tuning Profile — Beelink GTR9 Pro

[![profile](https://img.shields.io/badge/profile-7.135.1-1793d1?style=flat-square)](CHANGELOG.md)
[![platform](https://img.shields.io/badge/platform-CachyOS-1793d1?style=flat-square)](#requirements)
[![silicon](https://img.shields.io/badge/silicon-gfx1151%20%2F%20Strix%20Halo-1793d1?style=flat-square)](#hardware-target)
[![gates](https://img.shields.io/badge/version%20gates-none-1793d1?style=flat-square)](#requirements)

> A gaming- and LLM-oriented CachyOS desktop tuning profile for the Beelink GTR9 Pro
> (Ryzen AI Max+ 395 "Strix Halo"). It manages 17 system and user configuration files
> spanning the kernel command line, package set, systemd services, network stack,
> GPU/CPU power, memory, storage, and an on-screen HUD.

Corresponds to `ry-install.fish` **7.135.1**. The reference document is formatted as a
deep-research brief: actionable items only, ordered by research priority.

Every value in this README and in the reference was re-derived from the 7.135.1 script
itself — array contents read from a live evaluation, configuration bodies captured from
the actual generator functions. Nothing was carried forward from the 7.130.0 edition.

**No tuning value changed between 7.130.0 and 7.135.1.** All 21 count tripwires, all 13
profile scalars and all 17 generated file bodies are byte-identical across that window,
confirmed by diffing the previous edition's rendered bodies against fresh generator
output. What changed is structural: two long-standing detection gaps closed, a fourth
preflight validator added, and a silent data-loss defect found and fixed.

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
║ CPU         ║ matches `Ryzen AI Max` (sole skip: ║
║             ║ RY_INSTALL_SKIP_HARDWARE_CHECK=1)  ║
║ Kernel      ║ no floor — neither enforced nor    ║
║             ║ advisory                           ║
║ Mesa        ║ no floor — MESA_MIN removed        ║
```

As of 7.135.1 the profile still carries **no version gates of any kind**. The hard kernel
validator was dropped earlier in the 7.10x series; the advisory kernel comment and the
`MESA_MIN` soft warning were both removed by 7.122.0 and have not returned. The
CPU-model match is the only remaining hardware precondition. Version-sensitivity is
therefore a research question, not a runtime guarantee — see the reference's priority
queue.

## What it configures

- **Kernel command line (15 params)** — CPU/power: `amd_pstate=active`,
  `processor.max_cstate=1`, `split_lock_detect=off`, `clearcpuid=umip`; I/O + link
  latency: `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`,
  `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`; wireless:
  `mt7925e.disable_aspm=1`; platform: `amd_iommu=off`, `ipv6.disable=1`,
  `zswap.enabled=0`, `quiet`; filesystem: `fsck.mode=force`, `fsck.repair=yes`.
- **Packages** — 16 added (incl. `pacman-contrib`), 9 removed, 11 masked units
  (incl. `avahi-daemon` service + socket and `ufw.service`); RADV Vulkan stack
  (`vulkan-radeon` + `lib32-vulkan-radeon`). A package in both the add and remove sets,
  or a unit in both the mask and enable sets, is now refused at preflight.
- **Modules** — `/etc/modprobe.d/60-ry-modules.conf` carries the `amdxdna` blacklist
  only (the XDNA NPU probes `-ENODEV (-19)` under `amd_iommu=off`; opt-in via
  `BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`, validator-enforced — that path
  renders a comment-only file, accepted by the format validator). The MT7925 ASPM
  mitigation rides the kernel command line, not a module drop-in. **Superseded
  `60-ry-*` drop-ins are now detected**: `--verify` warns, `--check` records them.
- **Gaming environment (10 vars)** — Wayland Proton, `PROTON_FSR4_UPGRADE=1`, local +
  16G shader caches, silenced DXVK/VKD3D/WINE logging, `MANGOHUD=1`, and
  `POWERDEVIL_NO_DDCUTIL=1` (disables PowerDevil's DDC/CI external-monitor brightness
  path). `VKD3D_CONFIG=descriptor_heap` is absent — vkd3d-proton's own default governs.
  No ICD is pinned: `AMD_VULKAN_ICD` was removed, so RADV selection rests on the
  installed ICD set alone.
- **Network** — IPv4-only nftables ruleset (invalid dropped first, then
  established/related accepted, then loopback; default-deny inbound, inbound ICMP echo
  accepted, remote-play ports gated), `nft -c` pre-validated before every deploy/reload;
  systemd-resolved with mDNS/LLMNR off, `DNSOverTLS=no` and `DNSSEC=no`, upstreams
  pinned to the AdGuard ad-block tier (94.140.14.14 / 94.140.15.15) and repeated in the
  NetworkManager drop-in under `[global-dns-domain-*]`, which is the line that actually
  beats DHCP-supplied per-link DNS; NetworkManager on wpa_supplicant with Wi-Fi
  powersave off plus a dispatcher logging drop-in; regulatory domain US.
- **GPU / CPU power (maximum-performance posture)** — amd_pstate EPP `performance`
  (enum-gated) under the `performance` governor (`dynamic_epp` disabled); GPU DPM
  level `high` (clocks pinned, gating stays active); udev rules pinning EPP and GPU
  state. The profile now documents that the `performance` governor pins the EPP hint to
  its maximum, so `EPP_PREFERENCE` restates rather than adds — verifying that claim
  against upstream is the reference's #2 question.
- **Memory / storage (11 sysctls)** — BBR + `fq`; `vm.max_map_count=2147483642`,
  `vm.swappiness=150`, `vm.compaction_proactiveness=0`, `vm.watermark_boost_factor=0`,
  and `kernel.nmi_watchdog=0` (which backs the long-standing runtime assert);
  ext4 mounted `noatime,lazytime,commit=10`; NVMe scheduler `none`; zswap off.
- **HUD** — readout-only MangoHud config (19 active directives; `cpu_temp` shipped
  commented — re-enabling re-trips MangoHud #1794, still open upstream), toggled with
  `Shift_R+F12`.
- **Safety rails** — all 4 boot-critical files get `.ry.bak` + post-write
  verify/restore; the other 13 managed files get a one-time `<path>.ry.orig` preserve on
  first adoption; long package/boot operations wall-clock-capped at a 7200 s floor;
  NTP remediation unconditional with a chronyd/ntpd/openntpd conflict guard.

## At a glance

```
║ AREA                ║ COUNT ║
║─────────────────────║───────║
║ kernel cmdline      ║ 15    ║  (unchanged vs 7.130.0)
║ packages added      ║ 16    ║
║ packages removed    ║ 9     ║
║ masked units        ║ 11    ║
║ sysctl values       ║ 11    ║  (unchanged vs 7.130.0)
║ environment vars    ║ 10    ║  (unchanged vs 7.130.0)
║ MangoHud directives ║ 19    ║  (active; +1 commented # cpu_temp)
║ managed files       ║ 17    ║  (15 system + 2 user; 5,093 B generated)
║ backup targets      ║ 4     ║  (boot-critical; .ry.bak + verify/restore)
║ first-adoption      ║ 13    ║  (.ry.orig, one-time — live since 7.135.1)
║ preflight tripwires ║ 21    ║  (hard-asserted array counts)
║ preflight validators║ 4     ║  (counts, keys, sets, post-hooks)
║ script functions    ║ 292   ║  (+5 vs 7.130.0)
║ verify functions    ║ 60    ║  (12 orchestrators + 48 subs)
```

## Notable design choices

Deliberate trade-offs the profile makes for latency and throughput on this fixed
hardware:

- **`pcie_aspm.policy=performance` + `mt7925e.disable_aspm=1`** — ASPM is addressed at
  two layers: the kernel ASPM driver stays active and is held at the performance policy
  (blunt `pcie_aspm=off` from the 7.10x series remains reverted), and the MT7925 gets
  its device-level ASPM disabled by module parameter on the command line. Whether the
  pair is redundant or complementary on this board's actual link set is the reference's
  #3 question, and the two available descriptions of what the policy token *does* now
  disagree — the reference flags that conflict rather than picking a side.
- **`amd_iommu=off`** — AMD-Vi disabled for the unified-memory pool; ROCm on gfx1151
  unaffected. Named cost: the XDNA NPU is blacklisted. NPU/VFIO/SR-IOV users opt back
  in with `BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`.
- **IPv6 disabled + IPv4-only firewall** — `ipv6.disable=1` with an IPv4-only nftables
  ruleset that accepts inbound ping; avahi masked (unit + socket) closes the second
  mDNS responder. Dual-stack users remove the token and restore IPv6 rules.
- **`ufw.service` masked, not removed** — the profile ships its own nftables ruleset and
  neutralizes ufw by mask rather than by uninstalling it, leaving the package in place.
- **Removals reconcile in three tiers** — a value inside a generated file disappears on
  the next deploy, because the generators rewrite each file wholesale. A masked unit
  dropped from the mask list, and a superseded `60-ry-*` drop-in, both persist — but
  both are now *detected*: `--verify` reports them and `--check` records them, without
  counting either as drift, since a re-run cannot clear them. A package dropped from the
  add list stays installed and is detected by nothing, deliberately: tracking it would
  need a persisted manifest, which would add its own drift surface.
- **Plaintext DNS with filtering upstream** — `DNSOverTLS=no` and `DNSSEC=no` both
  deliberate. Filtering is identical either way, and strict DoT fails closed; the
  posture prioritizes uninterrupted resolution for every device on the LAN. The router
  runs the same AdGuard upstreams in plaintext, so both layers agree.
- **`performance` governor + EPP `performance` + DPM `high`** — the maximum-performance
  stack under `amd_pstate=active`, in place since 7.128.0/7.129.0. Peak draw is
  unchanged because firmware caps the package at 85 W; idle draw rises because clocks no
  longer scale down. **That is a stated expectation, not a measurement** — quantifying
  the idle floor is the reference's #1 question and has been open for five releases.
- **`split_lock_detect=off` + `clearcpuid=umip`** — latency-oriented CPU settings.
- **No version floors** — neither kernel nor Mesa is gated, enforced, or advised. The
  profile assumes a current CachyOS rolling installation.
- **HUD dual-published** — the readout-only MangoHud config also ships standalone as
  `mangohud-gtr9-pro` (v1.17.0 cross-audited: 19/19 active-directive parity in set and
  order, comment-only byte delta; the installer's embedded generator is the source of
  truth per the repo's own changelog). The generator body is byte-identical across
  7.106 → 7.135.1.

## Contents

- `cachyos-tuning-audit.md` — deep-research brief: baseline delta, research priority
  queue, every managed value, the exact rendered configuration bodies, the verify
  surface, and the robustness invariants.
- `CHANGELOG.md` — version history.

## Usage

The profile is applied by `ry-install.fish` on a CachyOS installation matching the
hardware target above. Consult the reference for the exact managed values and the
post-install `--verify` checks.

Upgrading from ≤ 7.98.x: the two superseded modprobe drop-ins
(`60-ry-mt7925e.conf`, `60-ry-blacklist-amdxdna.conf`) are no longer a silent leftover —
`--verify` warns about any unmanaged `/etc/modprobe.d/60-ry-*.conf` and `--check`
records it. Confirm ownership with `pacman -Qo` and remove by hand; neither mode
removes files for you, and neither counts the leftover as drift.

Deployed between 7.109.0 and 7.135.0: the one-time `<path>.ry.orig` preserve did not
run in that window, so pre-existing content at the 13 non-boot managed paths was
overwritten with no backup and no log record. The behavior is fixed in 7.135.1, but
nothing recovers what that window replaced — see the reference's #15 question and its
backup-surface audit commands.
