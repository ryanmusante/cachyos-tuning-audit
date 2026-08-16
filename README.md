# cachyos-tuning-audit

Deep-research audit companion for a CachyOS desktop tuning profile on the Beelink GTR9 Pro
(Ryzen AI Max+ 395 "Strix Halo", gfx1151).

This repository holds the **brief, not the installer**. `cachyos-tuning-audit.md` is pinned
to [`ry-install.fish`](#pin) **7.162.2** and is ordered by implementation safety —
observation-only actions first, protected decisions last.

**The action queue has no open change item.** Between 7.159.0 and 7.162.0 the installer
itself closed four of the brief's tracked items — the MangoHud CPU keys, `clearcpuid`, the
`--check` drop-in hoist and the ICMPv6 accept. What remains open is observational: the
fallback-entry window, the ufw handoff gate, the detector-severity design question, and one
advisory read.

**No performance value has changed since 7.130.0 — thirty-two releases.** All four
performance scalars are byte-identical.

**One count-oracle value moved twice and round-tripped** (`KERNEL_PARAMS` 15 → 14 → 15), and
the tripwire that guards it shipped unsynced at 7.162.0, refusing every run until 7.162.1 —
the reason this brief now certifies validator runs unshadowed, with stderr visible.

## Files

| File | Purpose |
|---|---|
| [`cachyos-tuning-audit.md`](cachyos-tuning-audit.md) | the brief |
| `README.md` | this file |
| [`CHANGELOG.md`](CHANGELOG.md) | per-pin history, newest first |

Released as a single `zip -0 -X` archive, top-level directory
`cachyos-tuning-audit-v7_162_2`, documents mode `0644`.

## Pin

Every value in the brief was re-derived from the 7.162.2 script itself: array contents by
live `fish` evaluation, configuration bodies by executing the generator functions. The
upstream version set was re-checked on 2026-08-15 — mainline moved to 7.2.0-rc7 and
linux-cachyos to 7.1.8-1, neither of which changes a settled verdict; every claim in the
settled table carries the date it was last verified.

| Artifact | SHA256 | Size |
|---|---|---|
| `ry-install-v7_162_2.zip` | `d81475e6` | 320,167 B |
| `ry-install.fish` | `9ff2fae0` | 4,919 L / 293,359 B |
| `README.md` | `55695efa` | 305 L / 19,186 B |
| `CHANGELOG.md` | `f9ea3017` | 156 L / 5,865 B |
| `LICENSE` | `2e1e7c8a` | 21 L / 1,069 B |

The copy audited for this edition is the GitHub `main` zip (`153d75b3`, 86,954 B, topdir
`ry-install-main` — git-archive form, no mode bits); its four members are byte-identical to
the release members above, so the brief holds for both. Disambiguate by zip, README or
CHANGELOG hash — never by `--version`, and never by script hash alone: 7.162.0, 7.162.1 and
7.162.2 shipped inside one week, two of them differing by two bytes of script; 7.141.0
shipped twice with one script hash and 7.140.0 nine times with two.

## Hardware target

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S
(40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤ 96 GB as VRAM) · dual M.2
NVMe · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP with an 85 W BIOS
ceiling · CachyOS · systemd-boot.

As deployed, **both 10 GbE links are down and `wlan0` carries the default route** (brief item
T0-8). A networking recommendation that assumes the wired links are in use is describing the
hardware rather than the machine.

## Requirements

| Requirement | Minimum |
|---|---|
| Platform | CachyOS (Arch-based), systemd-boot with BLS entries |
| CPU | matches `Ryzen AI Max` — sole bypass `RY_INSTALL_SKIP_HARDWARE_CHECK=1` |
| Kernel | no floor, neither enforced nor advisory |
| Mesa | no floor, `MESA_MIN` removed |
| systemd | 250 or newer |
| fish | 3.6 or newer |
| Root filesystem | `ext4`, confirmed by T0-3 |

The profile carries **no version gates of any kind**. The CPU-model match is the only
hardware precondition; version sensitivity is a research question, not a runtime guarantee.

## What the profile configures

| Area | Detail |
|---|---|
| Kernel command line | 15 params. CPU and power: `amd_pstate=active`, `processor.max_cstate=1`, `split_lock_detect=off`. Link latency: `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`, `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`. Wireless: `mt7925e.disable_aspm=1`. Platform: `amd_iommu=on`, `iommu=pt`, `ipv6.disable=1`, `zswap.enabled=0`, `quiet`. Filesystem: `fsck.mode=auto`, `fsck.repair=yes` |
| CPU and GPU power | governor `performance`, EPP `performance` via udev, GPU DPM `high`, scaling driver asserted as `amd-pstate-epp`. `power-profiles-daemon` and `ananicy-cpp` masked so no second authority competes |
| Packages | 16 added, 9 removed (`-Rns`, rdep-aware), 11 units masked, RADV Vulkan stack via chwd. A package in both add and remove sets, or a unit in both mask and enable sets, is refused at preflight |
| Memory and network | 9 sysctl keys at priority 95, loading after CachyOS's vendor `70-cachyos-settings.conf`; BBR + `fq`. Both `netdev_budget` keys were removed at 7.157.0 after the softnet squeezed counter measured zero |
| DNS | **Nothing.** Since 7.147.0–7.148.0 the host pins no upstream, no DoT and no DNSSEC, and NetworkManager carries no `[global-dns]` block; it takes per-link DHCP DNS from the router, which forwards to AdGuard over DoT |
| Firewall | nftables default-deny-inbound with inbound IPv4 ping and the ICMPv6 base accept (live on the fallback entry); `ufw.service` masked rather than removed, behind a gate that confirms a live default-deny ruleset first |
| Boot | systemd-boot with `timeout 0`, mkinitcpio `MODULES=(amdgpu)` for early KMS, zstd `-3` |
| Session | 9 environment variables (Proton, DXVK/VKD3D, MangoHud, PowerDevil, `FSR4_WATERMARK`) and an 18-directive MangoHud HUD — the CPU keys ship commented |

## How the brief is organized

| § | Section | Read it for |
|---|---|---|
| 0 | Provenance | archive hashes and the upstream version set — verify before trusting anything |
| 1 | Delta vs 7.158.0 | what moved, what was removed, what must not be re-derived from it |
| 2 | Action queue | **the operative section** — the eight T0 results, then tiers T1 through T5 |
| 3 | Settled | closed by upstream evidence, each claim dated; do not re-research |
| 4 | Corrections | what this edition withdraws or reframes |
| 5 | Security posture | ordered exposure deltas, quantify only |
| 6 | Verify block | post-reboot commands, grouped by tier, all self-resolving |
| 7 | Reference data | counts, scalars, the 11 perf sites, all 17 generated bodies, byte anchors, gates |
| 8 | Verify-surface ownership | which sub asserts which value, by line number |
| 9 | Reproduction method | the harness and every trap in it |
| 10 | Scope and output contract | rules and required deliverable shape |

## The safety tiers

| Tier | Blast radius | State at 7.162.2 |
|---|---|---|
| T0 | none — observation only | all eight returned at the 7.158.0 edition; results stand |
| T1 | user scope, no root, no reboot | **no open items** — the CPU keys shipped commented (`<= 7.160.0`) |
| T2 | managed config value; self-heals on the next deploy | nftables order KEEP; `energy_uj` drop-in DECLINED |
| T3 | kernel command line; reboot required | `clearcpuid` removed at 7.160.0; `amd_iommu=on iommu=pt` shipped at 7.162.0 (new note T3-5); `max_cstate` KEEP by decision; ASPM pair confirmed |
| T4 | boot chain, firewall handoff, detector severity | ICMPv6 shipped and the `--check` hoist landed; fallback entry, ufw gate and the DRIFT design remain open; T4-8 INFO |
| T5 | closed — do not recommend changing | the three-layer DNS posture, DPM `high`, no version gates, `BLACKLIST_AMDXDNA=false` |

## What changed in this edition

- Re-pinned 7.158.0 to **7.162.2** (4,919 lines, +644 B; anchors moved +2 to +4). Every
  count, scalar, byte anchor and line number re-derived live rather than shifted.
- **Four tracked items closed by the artifact:** T1-1 (the MangoHud CPU keys shipped
  commented, `<= 7.160.0`), T3-2 (`clearcpuid` removed at 7.160.0, boot untainted), T4-6
  (the `--check` drop-in sweep hoisted before the sudo bail) and T4-7 (the ICMPv6 base
  accept, 7.159.0).
- **`KERNEL_PARAMS` moved 15 → 14 → 15** and five generated bodies changed; the 17-file
  total went 4,858 → 5,338 B. New tracked note **T3-5**: `amd_iommu=on` is not a value the
  AMD IOMMU parser accepts — `iommu=pt` is the load-bearing half; the cost is one boot
  notice.
- **7.162.0 shipped with the count tripwire unsynced** and refused every run at rc 3 until
  7.162.1. Standing check added: run `_ir_validate_counts` unshadowed, stderr visible, in
  every certification.
- **The `--verify` OK count is re-baselined at 266** (188 static + 78 runtime), from the
  clean post-deploy host run of 2026-08-14.
- **The security list is rewritten.** The "AMD-Vi fully disabled" exposure is gone; the
  residual is the passthrough identity-domain trade, and the fallback entry is now the
  *more*-isolated DMA posture. UMIP restored is recorded as a posture win.
- **T1-1's sensor framing is superseded:** `cpu_custom_temp_sensor` is inert on this APU —
  MangoHud reads `apu_cpu_temp` from `gpu_metrics` before any hwmon lookup — and `cpu_power`
  comes from the APU metric, not RAPL.
- **The MES firmware read returned:** the unit reports MES 0x91 / KIQ 0x75, past the 0x86
  gfx1151 hang fix. One advisory read remains open (the installed proton-cachyos build).
- Determinism manifest method replaced: sorted per-file `sha256sum` manifest, hashed once
  (`8c6623d5db5f8b23`); prior editions' whole-directory shas are method-incompatible.

## Reproducing the numbers

The brief's §9 records the full harness and its traps. In short: cut the script just before
the `# ── MAIN: ARGPARSE` banner (**L4790** at 7.162.2 — moved again; always locate it,
never hardcode), delete the line-3 source guard, shadow `exit`, and `source` the result as a
non-root user with a writable `$HOME` and `/tmp` — **with stderr visible**, or `_err_loud`
refusals vanish.

```fish
sed -n '1,4789p' ry-install.fish | sed '3d' > harness.fish
```

Array counts must come from live `fish` evaluation, never text parsing; generated bytes must
be measured as written files, never through `string collect`; and the function census must
come from `^function ` at column 0 in source, because the fallen-through top level erases its
own signal handlers before any post-source probe. WARN/INFO text is QUIET-suppressed in the
harness — assert warn branches by rc and JSONL. Determinism this edition: 3/3 renders,
sorted per-file manifest sha `8c6623d5db5f8b23`, Σ 5,338 B across all seventeen generators.
