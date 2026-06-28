# Changelog

All notable changes to the ry-install deep-research prompt.

Format follows the kernel.org convention: newest first, grouped by change class.
The prompt version tracks the `ry-install.fish` version it audits.

## [v7.77.1] - 2026-06-28

Target script: `ry-install.fish` v7.77.1. Forked from the v7.75.1 prompt; counts,
inline v-deltas, VERIFY block, security-delta ordering, and the verify-subsystem map
re-pinned to the current script. A leading "What changed since v7.75.1" section
enumerates every delta the reviewer must re-evaluate. This revision also adds a
deeper-pass gaming-first investigation layer (§13) and value-anchors several previously
generic items.

### Added (deeper pass — §13 candidate enhancements, gaming-first)

- **§13 Candidate enhancements** — a new investigation section auditing knobs the
  profile does NOT set, each as an ADD-default / ADD-opt-in / KEEP-omitted decision with
  IMPACT × RISK. Subsections:
  - §13a kernel cmdline: `mitigations=off` (Zen 5 cost vs the already-accepted UMIP-off
    threat model), `amdgpu.ppfeaturemask` (undervolt/OC via CoreCtrl/LACT),
    `preempt=full` (frame-pacing, gated on the CachyOS default PREEMPT_DYNAMIC mode),
    `nvme_core.io_timeout`/`pcie_port_pm`.
  - §13b RADV/Mesa: `RADV_PERFTEST` (gpl/sam), `RADV_DEBUG`, present-mode/`vblank_mode`,
    `mesa_glthread` — each as default-vs-win.
  - §13c DXVK/VKD3D-Proton: GPL + compiler-threads (with the explicit "async→GPL,
    do-not-recommend-old-async" note), upscaler envs, `VKD3D_CONFIG` dxr.
  - §13d firmware/platform: Resizable BAR / Smart Access Memory verification (proposed
    advisory INFO for ReBAR-off), gaming-vs-compute GTT verdict.
  - §13e scheduler/memory: `read_ahead_kb`/`nr_requests` (un-deployed), `vm.max_map_count`
    exact-requirement confirm, `isolcpus`/`nohz_full`/`rcu_nocbs` KEEP-omitted with
    rationale.
- Bias documented as KEEP-omitted unless the gaming win is concrete and low-risk.
- All §13 "absent" claims fact-checked against `ry-install.fish` v7.77.1 (probe
  confirmed `mitigations`/`RADV_PERFTEST`/`DXVK_ASYNC`/`gamemode`/`rebar`/`read_ahead_kb`
  all absent from source).

### Added (value-anchored sharpening of existing items)

- §4: zstd surfaced as the literal `COMPRESSION_OPTIONS=(-1 -T0)` (negative level =
  fastest/lowest-ratio, not default-3) — ESP-budget vs boot-time trade with the
  `BOOT_SPACE_*` gates, replacing the generic "confirm zstd level."
- §6: EPP item now demands a quantified `balance_performance` → `performance` gaming
  opt-in (udev EPP ATTR only; governor stays powersave).
- §7: `nr_requests`/`read_ahead_kb` documented as NOT deployed (only `scheduler=none`
  is) with an ADD-or-KEEP decision, replacing "evaluate nr_requests and read_ahead_kb."
- §3: preempt item tied to the actual `_vrk_cmdline` dmesg `Dynamic Preempt:` detection.
- §2: added the GameMode-integration gap (no explicit `gamemode` pkg, no `gamemode.ini`,
  no `gamemoderun` env; static profile may already cover GameMode's governor switch).
- §5: ntsync 6.14-mainline vs 6.18-floor consistency confirm.

### Added (re-architected verify subsystem — §C rewrite)

- §C rewritten to the real v7.77.1 architecture: 12 orchestrators across six sub-families
  — `_vss_*` (static on-disk), `_kb_*` (5 known-benign INFO), `_vrk_*` (runtime-kernel:
  cmdline/gpu/cpu/module/clocksource), `_vrkm_*` (module-state: amdgpu/**iommu**/blacklist),
  `_vrsv_*` (runtime-services), `_vre_*` (runtime-env), `_vrs_*` (runtime-session). The
  prior `_vss(11)/_vre(7)/_vrs(5)`-only map is retired.
- `_vrk_cmdline` documented as a generic loop asserting EVERY `KERNEL_PARAMS` token +
  `rw` (so `amd_iommu=off` is auto-asserted).
- `_vrkm_iommu`, `_vrk_clocksource` (TSC-demotion correlation), and `_vrkm_blacklist`
  added to the verify surface.

### Changed (IOMMU inversion — the headline delta)

- IOMMU premise INVERTED a second time: `iommu=pt` → `amd_iommu=off` (v7.77.0).
  §3 re-audits AMD-Vi-off on merits (incl. a ROCm/SVM-breakage escalation gate),
  §10 + security-delta reorder it to the #2 OPEN reduction, the IOMMU special case flips
  (do not re-add `iommu=pt` as a default), and the VERIFY block + §C swap the
  `iommu=pt` assert for `amd_iommu=off`/0-groups via `_vrkm_iommu`.
- Counts re-pinned to all 19 hard-asserted globals; KERNEL_PARAMS stays 16 (IOMMU swap),
  NTSYNC_MODLOAD_CONFS removed from the manifest, MangoHud 17 active (+1 commented).
- §5: ntsync framing changed from MANAGED (3 autoload confs) to assert-only;
  `PROTON_NO_NTSYNC=1` opt-out documented; the "largest unmanaged surface" framing
  retired.
- §12 / §B9: MangoHud `cpu_temp` documented as commented out (re-enable via
  `cpu_custom_temp_sensor=k10temp`).
- §1 / §11: linux-firmware preflight advisory added (`20251125*` hard-warn,
  `< 20260110` soft-warn).
- §9 / §A: RTC write-back (`_ry_rtc_writeback`) added to the Services phase.
- VERIFY block: asserts for `amd_iommu=off`/0-groups, `pacman -Q linux-firmware`,
  `_vrk_cmdline` generic param loop; GPU DPM assert stays `auto`.
- Security-delta head: UMIP off first, then `amd_iommu=off` (INVERTED, now open), then
  `split_lock_detect=off`, plaintext DNS, remote-play opt-in, firewall default-deny.
- Scope/non-goals: reinstatement rule cross-linked `amdgpu.ppfeaturemask` to §13a;
  ntsync autoload moved into the deliberately-removed set.

### Carried unchanged (vs v7.75.1)

- Governor/EPP special case retained (`powersave` + `balance_performance` is the
  EPP-honoring config under `amd_pstate=active`).
- GPU_DPM_LEVEL special case retained (`auto`, un-pinned SCLK).
- nftables ruleset, udev rules, remote-play ports, fstab normalization (§D), preflight
  gates (§E), and the entire §G–§L robustness layer verified byte/behavior-identical;
  all robustness machinery (atomic-write, lock, boot-wipe, signal teardown, pacman) and
  the ROBUSTNESS verdict carried forward, re-scoped to §1–§13.
- Naming FIX (Low/Low) retained: human-facing "GTR9 Pro" vs internal `gtr_pro`.

## [v7.75.1] - 2026-06-27

Target script: `ry-install.fish` v7.75.1. Forked from the v7.70.1 prompt; counts,
inline v-deltas, VERIFY block, and security-delta ordering re-pinned to the current
script. A leading "What changed since v7.70.1" section enumerates every delta the
reviewer must re-evaluate.

### Added (deepest pass — robustness & correctness audit §G–§L)

- §G atomic-write guarantees, §H instance-lock & PID-recycle TOCTOU, §I privilege
  handling, §J boot-wipe gate & boot-critical rollback, §K signal & exit teardown,
  §L pacman transaction safety, plus the required ROBUSTNESS verdict block. All §G–§L
  mechanism names, exit codes, and helper signatures fact-checked present (32/32).

### Added (deep pass — artifact-level appendix §A–§F)

- §A install-phase model, §B exact rendered file bodies, §C full `--verify` subsystem,
  §D fstab normalization, §E preflight gate ordering, §F 10 actionable deltas. All §A–§F
  quoted strings fact-checked present (21/21).

### Added (first pass)

- "What changed since v7.70.1" section; §3 four new kernel params; §5 ntsync-as-MANAGED
  + PROTON_FSR4_RDNA3_UPGRADE re-eval; §8/§11 mt7925e drop-in; §10 RY_REMOTE_PLAY_PORTS;
  §12 `_vss_known_benign` advisory sub.

### Changed

- Counts re-pinned: KERNEL_PARAMS 12 → 16, ENV_VARS 10 → 11, SYSCTL 11 → 9, managed
  files 17 → 18, MangoHud 15 → 18, NTSYNC_MODLOAD_CONFS 3 added; KERNEL_MIN 6.18 hard
  floor; GPU clock-floor `high` → `auto`; MangoHud directives restored;
  vm.page-cluster/vm.vfs_cache_pressure dropped as vendor duplicates.

## [v7.70.1] - 2026-06-26

Target script: `ry-install.fish` v7.70.1.

### Changed

- Editorial pass: removed conversational filler, normalized headers and list phrasing to
  professional register. No technical content changed. Pinned version-floor language and
  inline deltas to v7.70.1; source-of-truth order script > README > CHANGELOG.

### Profile facts carried (vs v7.70.0)

- VERIFY hardened to assert `scaling_governor=powersave` and `EPP=balance_performance`;
  §6 governor/EPP special case; security-delta ordering retained.

## [v7.70.0] - prior

Target script: `ry-install.fish` v7.70.0. Baseline for the v7.70.1 deltas.

### Added

- Bluetooth `main.conf` (FastConnectable/AutoEnable/ReconnectAttempts); `rtkit` +
  `modemmanager` mask.

### Changed

- Soft Mesa floor 25.3 → 26.0; Bluetooth ReconnectAttempts 7 → 3.

### Removed

- Second regdomain file; Bluetooth explicit ReconnectIntervals; MangoHud
  gpu_power/cpu_temp/text_outline/toggle_hud/fps_metrics (some later restored); RADV
  drirc; TTM/GTT module params; boot-time/THP/KSM checks from VERIFY.

### Security

- IOMMU: `amd_iommu=off` removed; `iommu=pt` (AMD-Vi on). NOTE: subsequently
  re-inverted to `amd_iommu=off` in v7.77.0 — see the v7.77.1 entry. Wi-Fi backend
  reverted to `wpa_supplicant`.

---

### Notes

- This changelog documents revisions to the **prompt**, not to `ry-install.fish`.
  The script maintains its own CHANGELOG, which is the authoritative source for profile
  changes; entries above summarize only the deltas the prompt explicitly references.
- When the script version advances, fork the prompt to a new version file and add a
  corresponding entry here. Historical entries are never renumbered.
