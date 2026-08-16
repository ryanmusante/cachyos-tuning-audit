# cachyos-tuning-audit

Deep-research audit companion for a CachyOS desktop tuning profile on the Beelink GTR9 Pro
(Ryzen AI Max+ 395 "Strix Halo", gfx1151).

This repository holds the **brief, not the installer**. `cachyos-tuning-audit.md` is pinned
to [`ry-install.fish`](#pin) **7.163.0** — edition **r2** — and is ordered by implementation
safety, observation-only actions first, protected decisions last. r2 re-verified the same
archive (hash and all seventeen rendered bodies byte-identical to r1), expanded §3 to twenty
candidates with an effort-ordered shortlist, and cut prose again.

**§3 is a standing hunt for gaming and performance headroom.** For four rebases the queue's
answer to "what should change" was "nothing, the artifact already closed it" — a correct
audit result and a useless research result. §3 is the correction: **twenty** gated candidates
ranked by expected frametime effect, each with a mechanism, a tier, a source and the evidence
that would settle it, plus §3e's shortlist of what to do first by effort against return.

**Two FIX-shaped items are open**, after four rebases with none: **T2-6**
(`autoconnect-retries-default=0` is deployed but unasserted) and **T3-5** (`amd_iommu=on` is
an inert no-op the host proves the kernel rejects).

**No performance value has changed since 7.130.0 — thirty-three releases.** All four
performance scalars are byte-identical.

## Files

| File | Purpose |
|---|---|
| [`cachyos-tuning-audit.md`](cachyos-tuning-audit.md) | the brief |
| `README.md` | this file |
| [`CHANGELOG.md`](CHANGELOG.md) | per-pin history, newest first |

Released as a single `zip -0 -X` archive, top-level directory
`cachyos-tuning-audit-v7_163_0_r2`, documents mode `0644`.

## Pin

Every value in the brief was re-derived from the 7.163.0 script itself: array contents by
live `fish` evaluation, configuration bodies by executing the generator functions. All
seventeen bodies were re-rendered and diffed against the previous edition's fences; two came
back different.

| Artifact | SHA256 | Size |
|---|---|---|
| `ry-install-main.zip` | `c890a83b` | 86,813 B |
| `ry-install.fish` | `1f2a1bae` | 4,919 L / 293,423 B |
| `README.md` | `688d795f` | 304 L / 19,103 B |
| `CHANGELOG.md` | `2ac8288a` | 143 L / 5,440 B |
| `LICENSE` | `2e1e7c8a` | 21 L / 1,069 B |

The copy audited is the GitHub `main` zip (git-archive form, topdir `ry-install-main`, no
mode bits). **No release zip was supplied this rebase, so no release anchors are recorded** —
do not reconstruct them, and re-apply `0755` to the script when repacking. Disambiguate by
zip, README or CHANGELOG hash — never by `--version`, and never by script hash alone:
7.162.0, 7.162.1 and 7.162.2 shipped inside one week, two of them differing by two bytes of
script.

The upstream set was re-checked on 2026-08-16: mainline 7.2.0-rc7, linux-cachyos 7.1.8-1,
proton-cachyos 11.0-20260703, MangoHud #1794 and systemd #33579 both still open. Mesa's
`VERSION` returned HTTP 504 twice and is carried forward at 26.3.0-devel rather than
re-derived.

## Hardware target

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S
(40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified · dual M.2 NVMe · dual 10 GbE
(RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP with an **85 W BIOS ceiling** ·
CachyOS · systemd-boot.

As deployed, **both 10 GbE links are down and `wlan0` carries the default route** (T0-8),
and the 85 W ceiling frames every performance candidate in §3. A recommendation that assumes
either away is describing the hardware rather than the machine.

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
hardware precondition.

## What the profile configures

| Area | Detail |
|---|---|
| Kernel command line | 15 params. CPU and power: `amd_pstate=active`, `processor.max_cstate=1`, `split_lock_detect=off`. Link latency: `pcie_aspm.policy=performance`, `nvme_core.default_ps_max_latency_us=0`, `usbcore.autosuspend=-1`, `btusb.enable_autosuspend=n`. Wireless: `mt7925e.disable_aspm=1`. Platform: `amd_iommu=on` (inert — T3-5), `iommu=pt`, `ipv6.disable=1`, `zswap.enabled=0`, `quiet`. Filesystem: `fsck.mode=auto`, `fsck.repair=yes` |
| CPU and GPU power | governor `performance`, EPP `performance` via udev, GPU DPM `high`, driver asserted as `amd-pstate-epp`. `power-profiles-daemon` and `ananicy-cpp` masked so no second authority competes |
| Packages | 16 added (incl. `cachyos-gaming-meta`), 9 removed, 11 units masked, RADV Vulkan via chwd. A package in both add and remove sets, or a unit in both mask and enable sets, is refused at preflight |
| Memory and network | 9 sysctl keys at priority 95, loading after CachyOS's vendor `70-cachyos-settings.conf`; BBR + `fq`. Two of the four ArchWiki zram keys are absent — candidate G-3 |
| DNS | **Nothing.** No upstream, no DoT, no DNSSEC, no NetworkManager `[global-dns]`; per-link DHCP DNS from the router, which forwards to AdGuard over DoT |
| Firewall | nftables default-deny-inbound with inbound IPv4 ping and the ICMPv6 base accept; `ufw.service` masked behind a gate that confirms a live default-deny ruleset first |
| Boot | systemd-boot with `timeout 0`, mkinitcpio `MODULES=(amdgpu)` for early KMS, zstd `-3` |
| Session | **10** environment variables (Proton, DXVK/VKD3D, MangoHud, PowerDevil, `FSR4_WATERMARK`, and `GSK_RENDERER=ngl` new at 7.163.0) and an 18-directive readout-only MangoHud HUD |

## How the brief is organized

| § | Section | Read it for |
|---|---|---|
| 0 | Provenance | archive hashes and the upstream set — verify before trusting anything |
| 1 | Delta vs 7.162.2 | what moved, and the roster of retired questions |
| 2 | Action queue | the eight T0 results, then tiers T1 through T5 |
| 3 | **Gaming and performance** | the open candidate surface, its gates and the measurement method |
| 4 | Settled | closed by upstream evidence, each claim dated; do not re-research |
| 5 | Corrections | what this edition withdraws or reframes |
| 6 | Security posture | ordered exposure deltas, quantify only |
| 7 | Verify block | post-reboot commands, grouped, all self-resolving |
| 8 | Reference data | counts, scalars, the 11 perf sites, all 17 generated bodies, byte anchors, gates |
| 9 | Verify-surface ownership | which sub asserts which value, and what nothing asserts |
| 10 | Reproduction method | the harness and every trap in it |
| 11 | Scope and output contract | rules and required deliverable shape |

## The safety tiers

| Tier | Blast radius | State at 7.163.0 |
|---|---|---|
| T0 | none — observation only | all eight returned; results stand |
| T1 | user scope, no root, no reboot | no open items; the MangoHud companion repo is still divergent |
| T2 | managed config value; self-heals on the next deploy | **T2-6 FIX candidate** — a managed value with no assertion; nftables order KEEP; `energy_uj` DECLINED |
| T3 | kernel command line; reboot required | **T3-5 reopened as a FIX candidate**; `max_cstate` KEEP by decision; ASPM pair confirmed load-bearing |
| T4 | boot chain, firewall handoff, detector severity | fallback-entry window, ufw gate and the DRIFT design remain open |
| T5 | closed — do not recommend changing | DNS posture, DPM `high`, no version gates, `BLACKLIST_AMDXDNA=false` |

## What changed in this edition

- Re-pinned 7.162.2 to **7.163.0**. `ENV_VARS` 9 → 10 (`GSK_RENDERER=ngl`), a new `[main]`
  section in the NetworkManager drop-in (`autoconnect-retries-default=0`), and Σ across the
  seventeen generated bodies 5,338 → 5,393 B. Everything else held, including the harness
  cut at L4790 — its first unmoved rebase in six.
- **Added §3**, the gaming and performance candidate surface: **20** levers with gates, from
  the 85 W ceiling and the GTT/VRAM budget down to `RADV_PERFTEST=nggc`, a frame cap under
  the power wall, per-app drirc and per-game gamescope — plus a measurement protocol, an
  effort-ordered shortlist, and a protected list recording why sched_ext and `profile_peak`
  are not candidates.
- **Added T2-6.** The new NetworkManager value is hardcoded in the generator and `_vss_nm`
  does not assert it — the first recorded instance of a managed value with no verifier.
  `GSK_RENDERER` needed no verifier edit at all, because the env verifiers iterate the array.
- **Reopened T3-5 as a FIX candidate.** The host logs `AMD-Vi: Unknown option - 'on'`;
  `iommu=pt` is the load-bearing half and `amd_iommu=on` does nothing. Removal cascade costed
  at `KERNEL_PARAMS` 15 → 14 and Σ −26 B.
- **Recorded a class of blind spot:** the verify surface tests token *presence*, never
  *efficacy*, so a token the kernel rejects passes every check.
- **The `--verify` OK count is 268** (189 static + 79 runtime), from the clean post-deploy
  host run of 2026-08-16 — exactly +2 over 7.162.2 for the one new environment variable.
- **The 7.162.0 standing check passed its first real exercise:** a count moved and
  `_ir_validate_counts` moved with it.
- **Trimmed throughout.** Closed items collapse into a single roster instead of repeating as
  tier entries, and **all line numbers are gone** — they moved on five of the last six
  rebases and were never load-bearing. Locate a symbol. Across r1 and r2 the non-§3 material
  is down roughly a fifth from the 7.162.2 edition while §3 grew to ~26 KB.

## Reproducing the numbers

§10 records the full harness and its traps. In short: cut the script just before the
`# ── MAIN: ARGPARSE` banner (**L4790** at 7.163.0 — always locate it, never hardcode),
delete the line-3 source guard, shadow `exit`, and `source` the result as a non-root user
with a writable `$HOME` and `/tmp` — **with stderr visible**, or `_err_loud` refusals vanish.

```fish
sed -n '1,4789p' ry-install.fish | sed '3d' > harness.fish
```

Array counts must come from live `fish` evaluation, never text parsing; generated bytes must
be measured as written files, never through `string collect`; and the function census must
come from `^function ` at column 0 in source, because the fallen-through top level erases its
own signal handlers before any post-source probe. WARN/INFO text is QUIET-suppressed in the
harness — assert warn branches by rc and JSONL. Determinism this edition: 3/3 renders, sorted
per-file manifest sha `d9b5b3f3e4bea768`, Σ 5,393 B across all seventeen generators.
