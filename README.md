# CachyOS Tuning Profile — Beelink GTR9 Pro

A gaming- and LLM-oriented CachyOS desktop tuning profile for the Beelink GTR9 Pro
(Ryzen AI Max+ 395 "Strix Halo"). It manages 17 system and user configuration files
spanning the kernel command line, package set, systemd services, network stack, CPU/GPU
power, memory, storage, and an on-screen HUD.

This repository holds the **audit companion**, not the installer.
`cachyos-tuning-audit.md` is a deep-research brief pinned to `ry-install.fish` **7.139.0
r3** and **ordered by implementation safety** — observation-only actions first, protected
decisions last.

Every value in the brief was re-derived from the 7.139.0 script itself: array contents by
live evaluation, configuration bodies by executing the generator functions. Nothing was
carried forward from the 7.135.1 edition.

## Headline

**No tuning value has changed since 7.130.0 — nine releases.** All 21 count tripwires and
all 17 generated bodies are byte-identical at a 5,093 B total, confirmed by diffing the
previous edition's rendered bodies against fresh generator output. Everything that moved
in this window is verification surface, logging, documentation, or feature removal.

## Hardware target

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S
(40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤ 96 GB as VRAM) · dual M.2
NVMe · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP (BIOS ceiling 85 W) ·
CachyOS · systemd-boot.

## Requirements

```
║ REQUIREMENT ║ MINIMUM                                             ║
║─────────────║─────────────────────────────────────────────────────║
║ Platform    ║ CachyOS · systemd-boot · ext4 root                  ║
║ CPU         ║ matches `Ryzen AI Max`                              ║
║             ║ (sole skip: RY_INSTALL_SKIP_HARDWARE_CHECK=1)       ║
║ Kernel      ║ no floor — neither enforced nor advisory            ║
║ Mesa        ║ no floor — MESA_MIN removed                         ║
║ fish        ║ 3.6+                                                ║
```

The profile carries **no version gates of any kind**. The CPU-model match is the only
hardware precondition. Version sensitivity is a research question, not a runtime guarantee.

## What it configures

- **Kernel command line (15 params)** — CPU/power: `amd_pstate=active`,
  `processor.max_cstate=1`, `split_lock_detect=off`, `clearcpuid=umip`; I/O and link
  latency: `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`,
  `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`; wireless:
  `mt7925e.disable_aspm=1`; platform: `amd_iommu=off`, `ipv6.disable=1`, `zswap.enabled=0`,
  `quiet`; filesystem: `fsck.mode=force`, `fsck.repair=yes`.
- **CPU/GPU power** — governor `performance`, EPP `performance` via udev, GPU DPM `high`,
  scaling driver asserted as `amd-pstate-epp`. `power-profiles-daemon` and `ananicy-cpp`
  are masked so no second authority competes for the governor.
- **Packages** — 16 added, 9 removed (`-Rns`, rdep-aware), 11 units masked, RADV Vulkan
  stack via chwd. A package in both the add and remove sets, or a unit in both the mask and
  enable sets, is refused at preflight.
- **Memory and network** — 11 sysctl keys at priority 95, loading after CachyOS's vendor
  `70-cachyos-settings.conf`; BBR + fq; `netdev_budget` 600/5000 for dual 10 GbE.
- **DNS** — AdGuard ad-block tier, plaintext by explicit decision, with the NetworkManager
  `[global-dns-domain-*]` mechanism that beats per-link DHCP DNS.
- **Firewall** — IPv4-only nftables default-deny-inbound; `ufw.service` masked rather than
  removed, behind a gate that confirms a live default-deny ruleset first.
- **Boot** — systemd-boot with `timeout 0`, mkinitcpio `MODULES=(amdgpu)` for early KMS,
  zstd `-1 -T0`.
- **Session** — 10 environment variables (Proton, DXVK/VKD3D, MangoHud, PowerDevil) and a
  19-directive MangoHud HUD.

## How the brief is organised

```
║ §  ║ SECTION                    ║ READ IT FOR                              ║
║────║────────────────────────────║──────────────────────────────────────────║
║ 0  ║ Provenance                 ║ archive hashes — verify before trusting  ║
║ 1  ║ Delta vs 7.135.1           ║ what moved, what was removed, what this  ║
║    ║                            ║ rebase corrects in the previous edition  ║
║ 2  ║ Action queue               ║ THE OPERATIVE SECTION — tiers T0…T5      ║
║ 3  ║ Settled                    ║ closed by upstream evidence; do not      ║
║    ║                            ║ re-research                              ║
║ 4  ║ Security posture           ║ ordered exposure deltas, quantify only   ║
║ 5  ║ Verify block               ║ post-reboot commands, grouped by tier    ║
║ 6  ║ Reference data             ║ counts, scalars, byte anchors, gates     ║
║ 7  ║ Verify-surface ownership   ║ which sub asserts which value            ║
║ 8  ║ Reproduction method        ║ the harness and every trap in it         ║
║ 9  ║ Scope and output contract  ║ rules and required deliverable shape     ║
```

## The safety tiers

```
║ TIER ║ BLAST RADIUS                          ║ EXAMPLES                    ║
║──────║───────────────────────────────────────║─────────────────────────────║
║ T0   ║ none — observation only               ║ turbostat idle floor,       ║
║      ║                                       ║ lspci ASPM, root FS type    ║
║ T1   ║ user scope, no root, no reboot        ║ MangoHud sensor selection   ║
║ T2   ║ managed config value; self-heals on   ║ sysctl, env, nftables order ║
║      ║ the next deploy                       ║                             ║
║ T3   ║ kernel command line; reboot required  ║ max_cstate, clearcpuid,     ║
║      ║                                       ║ fsck.mode, the ASPM pair    ║
║ T4   ║ boot chain, firewall handoff,         ║ fallback entry, ufw gate,   ║
║      ║ detector severity                     ║ orphan DRIFT design         ║
║ T5   ║ closed — do not recommend changing    ║ plaintext DNS, DPM=high,    ║
║      ║                                       ║ no version gates            ║
```

Four of the six T0 items gate a decision in a lower tier, which is why they run first.
**T0-1, the idle-floor measurement, is the highest-value action in the brief and has never
been run.**

## Files

```
║ FILE                     ║ PURPOSE                                        ║
║──────────────────────────║────────────────────────────────────────────────║
║ cachyos-tuning-audit.md  ║ the brief                                      ║
║ README.md                ║ this file                                      ║
║ CHANGELOG.md             ║ per-pin history, newest first                  ║
```

Released as a single `zip -0 -X` archive, top-level directory
`cachyos-tuning-audit-v7_139_0`, documents mode 0644.
