# Changelog

All notable changes to the ry-install deep-research prompt.

Format follows the kernel.org convention: newest first, grouped by change class.
The prompt version tracks the `ry-install.fish` version it audits.

## [v7.75.1] - 2026-06-27

Target script: `ry-install.fish` v7.75.1. Forked from the v7.70.1 prompt; counts,
inline v-deltas, VERIFY block, and security-delta ordering re-pinned to the current
script. A leading "What changed since v7.70.1" section enumerates every delta the
reviewer must re-evaluate.

### Added (deepest pass — robustness & correctness audit §G–§L)

- **§G atomic-write guarantees** (`_awf_*`) — tee-to-tmp with `$pipestatus` two-stage
  check, post-write symlink-swap TOCTOU probe, `mv -T` same-FS atomic rename, post-write
  byte re-read with `.ry.bak` restore on mismatch; flags the `/boot` vfat rename-atomicity
  question and generator-determinism requirement for post-write-verified files.
- **§H instance-lock & PID-recycle TOCTOU** (`_acquire_lock*`, `_lock_pid_started_after`)
  — atomic `mkdir` primitive, pidfile via `mktemp`+`mv -Tf`, recycle detection via
  `/proc/PID/stat` field-22 starttime vs pidfile mtime+2s with USER_HZ recovery from
  `/proc/config.gz`, re-read-before-rm guard, symlink refusal, fail-closed on every
  parse failure; flags the `comm`-embeds-`) ` parsing trap as a correctness spot-check.
- **§I privilege handling** (`_as`, `_run`, `_is_symlink`, `_installed_bytes`) —
  `sudo -n` everywhere with credential-lapse re-check before each critical write, the
  tri-state rc 0/1/2 (drift-vs-sudo-lapse) invariant every caller must branch on, and
  `_run` timeout non-fatality for pacman/mkinitcpio.
- **§J boot-wipe gate & boot-critical rollback** (`_irb_taint_gate`,
  `_install_rebuild_boot`, `_ip_snapshot_mkinitcpio`/`_mkinitcpio_revert`) — the
  `$BOOT`-unresolved refusal that blocks `sdboot-manage REMOVE_EXISTING` against an
  unverified target, the taint gate that skips `mkinitcpio -P` on prior boot-critical
  failure, `EXIT_BOOT_CRIT` terminality, and the snapshot(`/run`)→revert(`/etc` same-FS)
  rollback; flags the cmdline(Phase 3)-vs-initramfs(Phase 5) ordering window to trace.
- **§K signal & exit teardown** (`_cleanup`/`_teardown`/`_do_cleanup`) — INT/TERM/HUP/
  QUIT/ABRT with 128+N codes, idempotent `_CLEANUP_DONE`, SIGPIPE→JSONL-only, the
  children→revert→tmpfiles→fs-sweep→lock-release→globals ordering, and the
  `_RY_TMPDIR_GLOBS` exact-match-the-created-set requirement (leak vs cross-instance
  deletion).
- **§L pacman transaction safety** (`_ip_pacman_invoke`) — full `-Syu`/`-Syyu` only
  (no partials), retry-then-fatal, db.lck pre-check with no self-removal, PKGS_DEL
  removal not inducing a partial-upgrade.
- **ROBUSTNESS verdict** — new required verdict block (PASS/GAP/UNCERTAIN per §G–§L)
  separate from the tuning verdicts; any GAP in §G/§H/§J is release-blocking and
  outranks all config findings. Notes that flag-don't-FIX does NOT apply here —
  safety invariants get FIX, there is no deliberate-trade-off defense for a partial-write
  window or fail-open lock.
- All §G–§L mechanism names, exit codes, and helper signatures fact-checked present in
  `ry-install.fish` v7.75.1 (32/32).

### Added (deep pass — artifact-level appendix §A–§F)

- **§A install-phase model** — the 6 `_RY_PHASE_NAMES` phases, `_RY_BOOT_CRITICAL_DSTS`
  (4), `_RY_BACKUP_TARGETS` (2), and the 18-tag `_RY_POST_HOOKS` reload map, so a
  recommendation valid in isolation is checked against phase ordering.
- **§B exact rendered file bodies** — the literal generator output for `/etc/kernel/cmdline`
  + `sdboot-manage.conf`, `loader.conf`, the full `/etc/nftables.conf` ruleset
  (rule-by-rule: 8-type ICMPv6 set, 4-type IPv4 ICMP set, `flush ruleset` blast radius,
  rule ordering), the 3 udev rules (add|change vs add-only asymmetry, `card[0-9]` match),
  resolved/NM/dispatcher/regdom/BlueZ/cpupower drop-ins (basename-override caution), and
  the 18-directive MangoHud.conf. Reviewer validates content, not paraphrase.
- **§C full `--verify` subsystem** — the `_vss_*` (11), `_vre_*` (7), `_vrs_*` (5)
  families the user-facing VERIFY block omits; flags missing `_vre_zram`, `_vre_fstab`,
  and `_vrs_nm_perms` (0600 root:root) checks to add.
- **§D fstab normalization** — documents that `_install_fstab_opts` strips conflicting
  `relatime/atime/strictatime`, rewrites an existing `commit=N`, and preserves
  already-correct lines (idempotency + atomic-write + boot-critical concerns).
- **§E preflight gate ordering** — the `_ir_*` sequence mapped to distinct exit codes
  (PREFLIGHT 3, GEN_NOFN 11, GEN_NOUUID 12, GEN_SYSCTL 13), the 3 scoped skip-overrides,
  and the `PACTREE_TIMEOUT_S`/`BOOT_SPACE_*`/`ROOT_AVAIL_*` thresholds.
- **§F 10 new actionable deltas** — incl. dual bootloader-management surface
  (cmdline + sdboot LINUX_OPTIONS double-source), `cachyos-iw-set-regdomain` external
  dependency (highest version-fragility), `cpupower-service.conf` path verification,
  `99-cachyos-*` basename-override semantics, `flush ruleset` blast radius, and the
  fallback-entry/timeout-0 recovery posture.
- All §A–§F quoted strings fact-checked present in `ry-install.fish` v7.75.1 (21/21).

### Added (first pass)

- **What changed since v7.70.1** section — 10-item re-audit focus list flagging
  NEW/INVERTED surfaces so prior conclusions are not carried forward on them.
- §3: validation items for the four new kernel params — `processor.max_cstate=1`
  (v7.74.0), `btusb.enable_autosuspend=n` (v7.73.6), `fsck.mode=force` +
  `fsck.repair=yes` (v7.75.0).
- §5: ntsync investigation rewritten — ntsync is now MANAGED (3 modules-load.d
  autoload confs, builtin|loaded|loaded_nodev|missing state machine, v7.71.0+). The
  prior "largest unmanaged gaming surface" framing is retired.
- §5: PROTON_FSR4_RDNA3_UPGRADE=1 re-evaluation (re-added v7.71.0) — now live config,
  KEEP-or-FIX-to-remove rather than a removed value.
- §8/§11: `60-ry-mt7925e.conf` (`disable_aspm=1`, v7.72.0) — symptomatic MT7925
  coredump/BT-reconnect/assoc-fail mitigation; cross-checked against
  `pcie_aspm.policy=performance` and `btusb.enable_autosuspend=n` for redundancy.
- §10: RY_REMOTE_PLAY_PORTS gate (default false, v7.71.0) — exact Sunshine/Moonlight +
  Steam Remote Play TCP/UDP port set added for validation; inserted at position 4 of
  the security delta.
- §12: `_vss_known_benign` advisory verify sub (v7.74.2) — 5 INFO conditions
  (ModemManager D-Bus noise, ACP70 no-machine-driver, boltd NHI-unknown,
  no battery/backlight, USB-mic curve) added for benign-vs-swallowed-regression review.

### Changed

- Counts re-pinned: KERNEL_PARAMS 12 → 16, ENV_VARS 10 → 11, SYSCTL 11 → 9,
  managed files 17 (14+3) → 18 (15+3), MangoHud directives 15 → 18, plus
  NTSYNC_MODLOAD_CONFS 3 added to the manifest.
- §1: KERNEL_MIN 6.18 is now documented as a HARD preflight floor (`_ir_validate_kernel_floor`,
  v7.71.0) rather than "no hard floor"; reviewer must re-confirm 6.18 is the true
  r8169 / RTL8127-suspend-hang landing version.
- §6: GPU clock-floor premise INVERTED — `GPU_DPM_LEVEL=auto` (v7.71.0) replaces the
  prior pinned `high`. The drift-on-≠high assert is removed; VERIFY now expects `auto`.
  New special-case rule protects `auto` from being flagged as a GPU regression.
- §7: `vm.page-cluster=0` and `vm.vfs_cache_pressure=50` documented as DROPPED
  (v7.73.0, vendor duplicates); reviewer must confirm the vendor file sets identical
  values, else flag a silent behavior change.
- §12: MangoHud directives `gpu_power`, `text_outline`, `toggle_hud=Shift_R+F12`
  documented as RESTORED (v7.71.3); the prompt's prior TUNE-to-restore note is
  resolved. `cpu_temp` / `fps_metrics` remain absent.
- §2: noted removal of the advisory x86-64-v4 repo-tier probe (v7.74.0) — script no
  longer assesses repo tier.
- §4: HOOKS review now cross-checks the `fsck` hook against the new
  `fsck.mode=force` cmdline for boot-prompt/hang risk.
- VERIFY block: added asserts/observations for `processor.max_cstate`, `fsck.*`,
  `/dev/ntsync` + autoload conf, the mt7925e drop-in, and `GPU_DPM=auto`; udev
  scheduler node moved to `99-ry-perf.rules`.
- Scope/non-goals: reinstatement rule updated — PROTON_FSR4_RDNA3_UPGRADE and the
  restored MangoHud directives moved out of the "deliberately removed" set;
  vm.page-cluster / vm.vfs_cache_pressure moved in (now vendor-provided).

### Carried unchanged (vs v7.70.1)

- Governor/EPP special case retained: `powersave` + `balance_performance` is the
  EPP-honoring config under `amd_pstate=active` and must not be flagged as a regression
  without proving the `performance` governor would override the EPP hint. (v7.75.1
  reasserts both via cpupower + udev; values unchanged.)
- Security-delta head retained: UMIP off (`clearcpuid=514`) first, then
  `split_lock_detect=off`, plaintext DNS, then (NEW) remote-play opt-in, firewall
  default-deny (net positive), IOMMU restored (closed reduction).
- IOMMU special case retained: `amd_iommu=off` removed; current `iommu=pt`.
- Naming FIX (Low/Low) retained: human-facing "GTR9 Pro" vs internal `gtr_pro`;
  cosmetic, do not change if it breaks function-name refs or log fields.

## [v7.70.1] - 2026-06-26

Target script: `ry-install.fish` v7.70.1.

### Changed

- Editorial pass on the prompt: removed conversational filler, normalized headers and
  list phrasing to professional register. No technical content changed — all 11 counts,
  every config value-string, the VERIFY block, the security-delta ordering, all
  `§1`–`§12` items, every source list, and both r8169 commit hashes preserved.
- Pinned all version-floor language and inline deltas to v7.70.1; source-of-truth order
  set to script > README > CHANGELOG.

### Profile facts carried (vs v7.70.0)

- VERIFY block hardened to assert `scaling_governor=powersave` and
  `EPP=balance_performance` as the EPP-honoring config under `amd_pstate=active`.
- §6 marked the governor/EPP pairing as a protected special case.
- Security-delta ordering retained: UMIP off first, then `split_lock_detect=off`,
  plaintext DNS, firewall tightening, IOMMU restored.

## [v7.70.0] - prior

Target script: `ry-install.fish` v7.70.0. Baseline for the v7.70.1 deltas.

### Added

- Bluetooth `main.conf`: `FastConnectable=true`, `AutoEnable=true`,
  `ReconnectAttempts=3`.
- `rtkit` and `modemmanager` mask added.

### Changed

- Soft Mesa floor raised 25.3 → 26.0 (preflight warn).
- Bluetooth `ReconnectAttempts` lowered 7 → 3.

### Removed

- Second regdomain file `conf.d/wireless-regdom`; single `/etc/iw-regdomain`
  (`COUNTRY=US`) authoritative.
- Bluetooth explicit `ReconnectIntervals` list (1,2,4,8,16,32,64) → BlueZ default
  backoff.
- MangoHud `gpu_power`, `cpu_temp`, `text_outline`, `toggle_hud`, `fps_metrics`
  (15 directives remained). NOTE: `gpu_power`/`text_outline`/`toggle_hud` were
  subsequently restored in v7.71.3 — see the v7.75.1 entry.
- RADV drirc (`95-ry-radv-apu.conf`) — gfx1151 reports `uma:1` natively.
- TTM/GTT module params — kernel auto-sizes GTT.
- Boot-time / THP / KSM checks from VERIFY.

### Security

- IOMMU: `amd_iommu=off` removed; current `iommu=pt` (AMD-Vi on).
- Wi-Fi backend reverted to `wpa_supplicant` (from iwd) as an MT7925-stability move.

---

### Notes

- This changelog documents revisions to the **prompt**, not to `ry-install.fish`.
  The script maintains its own CHANGELOG, which is the authoritative source for profile
  changes; entries above summarize only the deltas the prompt explicitly references.
- When the script version advances, fork the prompt to a new version file and add a
  corresponding entry here. Historical entries are never renumbered.
