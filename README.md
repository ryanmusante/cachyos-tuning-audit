# cachyos-tuning-audit

Deep-research audit companion for a CachyOS desktop tuning profile on the Beelink GTR9 Pro
(Ryzen AI Max+ 395 "Strix Halo", gfx1151).

This repository holds the **brief, not the installer**. `cachyos-tuning-audit.md` is pinned
to [`ry-install.fish`](#pin) **7.158.0** and is ordered by implementation safety —
observation-only actions first, protected decisions last.

**All eight T0 observation gates have returned.** For twenty-five releases the brief carried
eight unrun measurements gating eleven lower-tier items. Every one is now answered, and the
action queue collapsed accordingly.

**No performance value has changed since 7.130.0 — twenty-eight releases.** All four
performance scalars are byte-identical.

**Two count-oracle values did move**, the first oracle movement since 7.140.0, and **one
generated body changed content at a byte delta of zero.** No count check, no byte anchor and
no Σ total can see that change — only a byte-exact diff of the embedded fence against fresh
generator output catches it, which is why all seventeen bodies are embedded and re-rendered
on every rebase.

## Files

| File | Purpose |
|---|---|
| [`cachyos-tuning-audit.md`](cachyos-tuning-audit.md) | the brief |
| `README.md` | this file |
| [`CHANGELOG.md`](CHANGELOG.md) | per-pin history, newest first |

Released as a single `zip -0 -X` archive, top-level directory
`cachyos-tuning-audit-v7_158_0`, documents mode `0644`.

## Pin

Every value in the brief was re-derived from the 7.158.0 script itself: array contents by
live `fish` evaluation, configuration bodies by executing the generator functions. The
upstream version set and both tracked issue states were re-checked on 2026-08-07 and none
had moved since the previous edition; every claim in the settled table carries the date it
was last verified.

| Artifact | SHA256 | Size |
|---|---|---|
| `ry-install-v7_158_0.zip` | `cc97b5a5` | 317,575 B |
| `ry-install.fish` | `13a467b8` | 4,915 L / 292,715 B |
| `README.md` | `061e3b79` | 307 L / 18,711 B |
| `CHANGELOG.md` | `91634f1b` | 131 L / 4,392 B |
| `LICENSE` | `2e1e7c8a` | 21 L / 1,069 B |

Disambiguate by zip, README or CHANGELOG hash — never by `--version`, and never by script
hash alone. A single script hash routinely covers several shipped artifacts: 7.141.0 shipped
twice with one, 7.140.0 nine times with two, and 7.151.0 through 7.153.0 are
version-string-only edits of one another.

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
| Kernel command line | 15 params. CPU and power: `amd_pstate=active`, `processor.max_cstate=1`, `split_lock_detect=off`, `clearcpuid=umip`. Link latency: `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`, `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`. Wireless: `mt7925e.disable_aspm=1`. Platform: `amd_iommu=off`, `ipv6.disable=1`, `zswap.enabled=0`, `quiet`. Filesystem: `fsck.mode=auto`, `fsck.repair=yes` |
| CPU and GPU power | governor `performance`, EPP `performance` via udev, GPU DPM `high`, scaling driver asserted as `amd-pstate-epp`. `power-profiles-daemon` and `ananicy-cpp` masked so no second authority competes |
| Packages | 16 added, 9 removed (`-Rns`, rdep-aware), 11 units masked, RADV Vulkan stack via chwd. A package in both add and remove sets, or a unit in both mask and enable sets, is refused at preflight |
| Memory and network | 9 sysctl keys at priority 95, loading after CachyOS's vendor `70-cachyos-settings.conf`; BBR + `fq`. Both `netdev_budget` keys were removed at 7.157.0 after the softnet squeezed counter measured zero |
| DNS | **Nothing.** Since 7.147.0–7.148.0 the host pins no upstream, no DoT and no DNSSEC, and NetworkManager carries no `[global-dns]` block; it takes per-link DHCP DNS from the router, which forwards to AdGuard over DoT |
| Firewall | IPv4-only nftables default-deny-inbound; `ufw.service` masked rather than removed, behind a gate that confirms a live default-deny ruleset first |
| Boot | systemd-boot with `timeout 0`, mkinitcpio `MODULES=(amdgpu)` for early KMS, zstd `-3` |
| Session | 9 environment variables (Proton, DXVK/VKD3D, MangoHud, PowerDevil, `FSR4_WATERMARK`) and a 19-directive MangoHud HUD |

## How the brief is organised

| § | Section | Read it for |
|---|---|---|
| 0 | Provenance | archive hashes and the upstream version set — verify before trusting anything |
| 1 | Delta vs 7.155.0 | what moved, what was removed, what must not be re-derived from it |
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

| Tier | Blast radius | State at 7.158.0 |
|---|---|---|
| T0 | none — observation only | **all eight returned**; results recorded, no action remains |
| T1 | user scope, no root, no reboot | one open item: comment the MangoHud CPU keys |
| T2 | managed config value; self-heals on the next deploy | nftables order KEEP; `energy_uj` drop-in DECLINED |
| T3 | kernel command line; reboot required | `fsck.mode` shipped; `max_cstate` and `clearcpuid` KEEP by decision; ASPM pair confirmed |
| T4 | boot chain, firewall handoff, detector severity | compression shipped; fallback entry, ufw gate, DRIFT design and two LOW/INFO items open |
| T5 | closed — do not recommend changing | the three-layer DNS posture, DPM `high`, no version gates |

## What changed in this edition

- Re-pinned 7.155.0 to **7.158.0**. Every count, scalar, byte anchor and line number
  re-derived live rather than shifted.
- **All eight T0 gates returned.** Idle floor 21.33 / 21.69 W package and 3.93 / 4.20 W core;
  every PCIe link reads `ASPM Disabled`; root is `ext4`; `k10temp` exposes only `Tctl`
  (`temp1_input`) and `energy_uj` is mode 400; softnet squeezed is 0 on all 32 CPUs;
  `dynamic_epp` is disabled; the backup inventory is exactly as designed; `/etc/resolv.conf`
  is in foreign mode.
- **Two items shipped at 7.158.0.** `fsck.mode=force` → `auto`, because the root is ext4 and
  `force` was running a full check on every boot — the previous edition's "largely inert on a
  Btrfs root" reading was wrong. And `COMPRESSION_OPTIONS=(-1)` → `(-3)`.
- **`mkinitcpio.conf` changed with a byte delta of zero.** First instance on record of a body
  edit invisible to counts, byte anchors and the Σ total simultaneously.
- **T2-1 closed on measurement and both `netdev_budget` keys were removed at 7.157.0.** The
  tuning was inert. Reinforcing it: both 10 GbE links are down and `wlan0` is the default
  route, so the premise behind the tuning did not describe the deployment.
- **T1-2 closed by the artifact** — `PROTON_ENABLE_WAYLAND=1` was removed after 7.155.0,
  taking `ENV_VARS` 10 → 9. That is the second time in three rebases that the artifact retired
  the brief's own open finding.
- **T3-1 and T3-2 retired to KEEP by maintainer decision.** `processor.max_cstate=1` was
  measured and kept; `clearcpuid=umip` is kept despite the taint and the missing
  documentation. Record the trades, do not re-litigate them.
- **T2-3 declined on security grounds, not scope.** `energy_uj` is mode 400 and relaxing RAPL
  permissions re-opens the PLATYPUS side channel.
- **The 278 expected `--verify` OK count is now stale and marked UNKNOWN.** Three assertion
  counts fell without a single verify-function edit, because the verifiers iterate the profile
  arrays rather than carrying literal lists.
- **Line anchors did not move**, the first rebase where that has happened. The standing rule
  is unchanged: always locate a symbol, never hardcode its line.
- New §7f tier-3 row: an environment variable dropped from `ENV_VARS` self-heals in the file
  but persists in a live `systemd --user` session until logout. Nothing detects that, and it
  is not drift.

## Reproducing the numbers

The brief's §9 records the full harness and its traps. In short: cut the script just before
the `# ── MAIN: ARGPARSE` banner (**L4786** at 7.158.0 — always locate it, never hardcode;
this is the first rebase where it did not move), delete the line-3 source guard, shadow
`exit`, and `source` the result as a non-root user with a writable `$HOME` and `/tmp`.

```fish
sed -n '1,4785p' ry-install.fish | sed '3d' > harness.fish
```

Array counts must come from live `fish` evaluation, never text parsing; generated bytes must
be measured as written files, never through `string collect`; and the function census must
come from `^function ` at column 0 in source, because the fallen-through top level erases its
own signal handlers before any post-source probe. Determinism this edition: 3/3 renders,
sorted-manifest sha `6bf7f8ea53c36c40`, Σ 4,858 B across all seventeen generators.
