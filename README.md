# cachyos-tuning-audit

Deep-research audit companion for a CachyOS desktop tuning profile on the Beelink GTR9 Pro
(Ryzen AI Max+ 395 "Strix Halo", gfx1151).

This repository holds the **brief, not the installer**. `cachyos-tuning-audit.md` is pinned
to [`ry-install.fish`](#pin) **7.141.0 r2** and is ordered by implementation safety —
observation-only actions first, protected decisions last.

**No tuning value has changed since 7.130.0 — eleven releases.** All 21 count tripwires,
all four performance scalars and 16 of 17 generated bodies are byte-identical. The single
delta in the window is a redundant `-T0` dropped from the initramfs compression options,
moving the 17-file total from 5,093 B to **5,089 B**.

## Files

| File | Purpose |
|---|---|
| [`cachyos-tuning-audit.md`](cachyos-tuning-audit.md) | the brief |
| `README.md` | this file |
| [`CHANGELOG.md`](CHANGELOG.md) | per-pin history, newest first |

Released as a single `zip -0 -X` archive, top-level directory
`cachyos-tuning-audit-v7_141_0`, documents mode `0644`.

## Pin

Every value in the brief was re-derived from the 7.141.0 script itself: array contents by
live `fish` evaluation, configuration bodies by executing the generator functions. Upstream
claims were re-checked against primary sources on 2026-07-27 and each carries a
verification date.

| Artifact | SHA256 | Size |
|---|---|---|
| `ry-install-v7_141_0.zip` | `76c47d95` | 327,084 B |
| `ry-install.fish` | `f8210c9e` | 4,990 L / 295,103 B |
| `README.md` | `ab63febc` | 291 L / 21,911 B |
| `CHANGELOG.md` | `79661db3` | 183 L / 8,313 B |
| `LICENSE` | `2e1e7c8a` | 21 L / 1,069 B |

7.141.0 shipped twice with an **identical script hash**. Disambiguate by zip, README or
CHANGELOG hash — never by `--version`, and never by script hash alone.

## Hardware target

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S
(40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤ 96 GB as VRAM) · dual M.2
NVMe · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP with an 85 W BIOS
ceiling · CachyOS · systemd-boot.

## Requirements

| Requirement | Minimum |
|---|---|
| Platform | CachyOS (Arch-based), systemd-boot with BLS entries |
| CPU | matches `Ryzen AI Max` — sole bypass `RY_INSTALL_SKIP_HARDWARE_CHECK=1` |
| Kernel | no floor, neither enforced nor advisory |
| Mesa | no floor, `MESA_MIN` removed |
| systemd | 250 or newer |
| fish | 3.6 or newer |

The profile carries **no version gates of any kind**. The CPU-model match is the only
hardware precondition; version sensitivity is a research question, not a runtime guarantee.
The root filesystem type is **unconfirmed** — the fstab rewrite path is ext4-only, which
does not establish the root. That is brief item T0-3.

## What the profile configures

| Area | Detail |
|---|---|
| Kernel command line | 15 params. CPU and power: `amd_pstate=active`, `processor.max_cstate=1`, `split_lock_detect=off`, `clearcpuid=umip`. Link latency: `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`, `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`. Wireless: `mt7925e.disable_aspm=1`. Platform: `amd_iommu=off`, `ipv6.disable=1`, `zswap.enabled=0`, `quiet`. Filesystem: `fsck.mode=force`, `fsck.repair=yes` |
| CPU and GPU power | governor `performance`, EPP `performance` via udev, GPU DPM `high`, scaling driver asserted as `amd-pstate-epp`. `power-profiles-daemon` and `ananicy-cpp` masked so no second authority competes |
| Packages | 16 added, 9 removed (`-Rns`, rdep-aware), 11 units masked, RADV Vulkan stack via chwd. A package in both add and remove sets, or a unit in both mask and enable sets, is refused at preflight |
| Memory and network | 11 sysctl keys at priority 95, loading after CachyOS's vendor `70-cachyos-settings.conf`; BBR + `fq`; `netdev_budget` 600/5000 for dual 10 GbE |
| DNS | AdGuard ad-block tier, plaintext by explicit decision, with the NetworkManager `[global-dns-domain-*]` mechanism that beats per-link DHCP DNS |
| Firewall | IPv4-only nftables default-deny-inbound; `ufw.service` masked rather than removed, behind a gate that confirms a live default-deny ruleset first |
| Boot | systemd-boot with `timeout 0`, mkinitcpio `MODULES=(amdgpu)` for early KMS, zstd `-1` |
| Session | 10 environment variables (Proton, DXVK/VKD3D, MangoHud, PowerDevil) and a 19-directive MangoHud HUD |

## How the brief is organised

| § | Section | Read it for |
|---|---|---|
| 0 | Provenance | archive hashes and the upstream version set — verify before trusting anything |
| 1 | Delta vs 7.139.0 | what moved, what was removed, what must not be re-derived from it |
| 2 | Action queue | **the operative section** — tiers T0 through T5 |
| 3 | Settled | closed by upstream evidence, each claim dated; do not re-research |
| 4 | Corrections | what this edition withdraws or reframes |
| 5 | Security posture | ordered exposure deltas, quantify only |
| 6 | Verify block | post-reboot commands, grouped by tier |
| 7 | Reference data | counts, scalars, the 13 perf sites, all 17 generated bodies, byte anchors, gates |
| 8 | Verify-surface ownership | which sub asserts which value, by line number |
| 9 | Reproduction method | the harness and every trap in it |
| 10 | Scope and output contract | rules and required deliverable shape |

## The safety tiers

| Tier | Blast radius | Examples |
|---|---|---|
| T0 | none — observation only | turbostat idle floor, lspci ASPM, root FS type, softnet_stat pressure |
| T1 | user scope, no root, no reboot | HUD sensor selection, Proton environment variables |
| T2 | managed config value; self-heals on the next deploy | sysctl, nftables rule order, `energy_uj` permissions |
| T3 | kernel command line; reboot required | `max_cstate`, `clearcpuid`, `fsck.mode`, the ASPM pair |
| T4 | boot chain, firewall handoff, detector severity | fallback entry, ufw gate, sdboot drop-ins, DRIFT design |
| T5 | closed — do not recommend changing | plaintext DNS, DPM `high`, no version gates |

Five of the seven T0 items gate a decision in a lower tier, which is why they run first.
**T0-1, the idle-floor measurement, is the highest-value action in the brief and has never
been run** — eleven releases now.

## What changed in this edition

- Re-pinned 7.139.0 r3 to **7.141.0 r2**. Every count, scalar, byte anchor and line number
  re-derived live rather than shifted.
- **All 17 generated bodies are now embedded byte-exact**, up from two. This closes the
  defect class where a fence captured from a toggled variant silently disagrees with the
  size table beside it.
- Three upstream corrections, each withdrawing or reframing a prior finding. The
  `netdev_budget` 600/4000 pairing is Red Hat's guidance and not ESnet's;
  `PROTON_FSR4_UPGRADE=1` is current rather than near-obsolete; the CachyOS `dynamic_epp`
  backport question is closed rather than unchecked.
- New T0 items for softnet_stat pressure and `dynamic_epp` state. New T1 items moving
  `PROTON_ENABLE_WAYLAND` per-game and pairing the RDNA3 FSR4 workaround.
- Every entry in the settled table now carries a verification date.

## Reproducing the numbers

The brief's §9 records the full harness and its traps. In short: cut the script just before
the `# ── MAIN: ARGPARSE` banner (**L4861** at 7.141.0 — always locate it, never hardcode),
delete the line-3 source guard, shadow `exit`, and `source` the result as a non-root user
with a writable `$HOME` and `/tmp`.

```fish
sed -n '1,4860p' ry-install.fish | sed '3d' > harness.fish
```

Array counts must come from live `fish` evaluation, never text parsing. Generated bytes
must be measured as written files, never through `string collect`. The function census must
come from `^function ` at column 0 in source, because the fallen-through top level erases
its own signal handlers before any post-source probe.
