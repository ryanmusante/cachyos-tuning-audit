# ry-install Deep-Research Prompt

Version-pinned deep-research prompt for auditing `ry-install.fish` against current
upstream sources. Each release of the prompt tracks one exact script version; the
current file targets **v7.75.1**.

- **File:** `ry-install-research-prompt-v7_75_1.md`
- **Prompt version:** v7.75.1 (matches script version under audit)
- **Source of truth:** script > README > CHANGELOG

## Purpose

Drives an item-by-item, evidence-backed tuning review of a CachyOS gaming/compute
profile on a single, fixed hardware target. The prompt instructs the reviewer to
evaluate every config decision against current upstream documentation, flag anything
superseded or harmful, quantify safety deltas, and respect deliberate
performance-over-safety trade-offs without auto-reverting them.

The deliverable is a **recommendations report**, not a modified script.

## Target hardware

Beelink GTR9 Pro — Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) ·
Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤96 GB as
VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 ·
140 W TDP · CachyOS · systemd-boot.

## Profile counts

From `_ir_validate_counts`:

```
║ GLOBAL              ║ COUNT ║
║─────────────────────║───────║
║ KERNEL_PARAMS       ║ 16    ║
║ ENV_VARS            ║ 11    ║
║ SYSCTL_VALUES       ║ 9     ║
║ PKGS_ADD            ║ 17    ║
║ PKGS_DEL            ║ 9     ║
║ MASK                ║ 10    ║
║ EXPECTED_SERVICES   ║ 5     ║
║ managed files       ║ 18    ║  (15 system + 3 user)
║ MangoHud directives ║ 18    ║
║ MKINITCPIO_HOOKS    ║ 11    ║
║ LOGIND_IGNORE_KEYS  ║ 8     ║
║ NTSYNC_MODLOAD_CONFS║ 3     ║
```

Hard floors: **KERNEL_MIN 6.18** (preflight hard-fail) · CPU gate `Ryzen AI Max` ·
soft Mesa < 26.0 warn.

## What changed since v7.70.1

The prompt's leading "What changed since v7.70.1" section enumerates the deltas the
reviewer must re-evaluate with fresh evidence. Headline items:

- **KERNEL_MIN 6.18 is now a HARD preflight floor** (was "no hard floor").
- **GPU_DPM_LEVEL flipped `high` → `auto`** — the GPU clock-floor is no longer pinned;
  the §6 premise is inverted.
- **ntsync is now MANAGED** (3 modules-load.d autoload confs + a
  builtin|loaded|loaded_nodev|missing state machine) — previously "observed, not
  deployed."
- **RY_REMOTE_PLAY_PORTS gate added** (default `false`) — opt-in Sunshine/Moonlight +
  Steam Remote Play inbound ports.
- **PROTON_FSR4_RDNA3_UPGRADE=1 re-added** — previously a deliberately-removed value.
- **4 kernel params added:** `btusb.enable_autosuspend=n`, `processor.max_cstate=1`,
  `fsck.mode=force`, `fsck.repair=yes` (KERNEL_PARAMS 12 → 16).
- **2 sysctl keys dropped** as vendor duplicates: `vm.page-cluster=0`,
  `vm.vfs_cache_pressure=50` (SYSCTL_VALUES 11 → 9).
- **MangoHud restored 3 directives** (`gpu_power`, `text_outline`,
  `toggle_hud=Shift_R+F12`); 15 → 18.
- **New modprobe drop-in** `60-ry-mt7925e.conf` (`options mt7925e disable_aspm=1`).
- **New advisory verify sub** `_vss_known_benign` (5 INFO conditions, never fails).

## Structure

The prompt is organized as:

- **What changed since v7.70.1** — the re-audit focus list (re-validate these; do not
  carry forward prior conclusions on them).
- **Mission / Rules / Output** — review mandate, the flag-don't-fix discipline,
  required report shape.
- **VERIFY block** — post-reboot `fish` assertions; hard `--verify` asserts vs
  warn-level checks are enumerated.
- **Security delta** — ordered list of where the profile reduces hardening vs
  CachyOS defaults (now includes the remote-play opt-in).
- **Investigation §1–§12** — ordered by installer phase: platform baseline, packages,
  kernel cmdline, bootloader/initramfs, GPU/Vulkan, CPU power, memory/storage,
  network/latency, systemd units, security (cross-cutting), known issues/DKMS,
  MangoHud/Bluetooth/hygiene.
- **Scope & non-goals** — out-of-scope surfaces and the (updated) reinstatement rule.
- **Deep-pass appendix (§A–§F)** — artifact-level layer beneath the value-level §1–§12:
  the install-phase model (§A), the exact rendered file bodies to validate rule-by-rule
  (§B — cmdline/sdboot, loader.conf, the full nftables ruleset, the 3 udev rules,
  resolved/NM/regdom/BlueZ/cpupower/MangoHud), the complete `--verify` subsystem the
  user VERIFY block omits (§C — `_vss_*`/`_vre_*`/`_vrs_*`, incl. `_vre_zram`,
  `_vre_fstab`, `_vrs_nm_perms`), the fstab normalization logic (§D), preflight gate
  ordering + exit codes (§E), and 10 new actionable deltas (§F).
- **Deepest-pass appendix (§G–§L)** — robustness & correctness audit surface (does the
  installer run *safely*, independent of what it configures): atomic-write guarantees
  (§G — tee+`$pipestatus`, post-write symlink-swap probe, `mv -T`, byte-re-read+`.ry.bak`
  restore), instance-lock & PID-recycle TOCTOU (§H — atomic mkdir, `/proc` starttime vs
  pidfile mtime with CONFIG_HZ recovery, re-read-before-rm, fail-closed), privilege
  handling (§I — `sudo -n`, tri-state rc 0/1/2 drift-vs-lapse, `_run` timeouts),
  boot-wipe gate & boot-critical rollback (§J — `$BOOT`-unresolved refusal, taint gate,
  mkinitcpio snapshot/revert), signal & exit teardown (§K — 128+N codes, cleanup
  ordering, tmpdir-glob completeness), pacman transaction safety (§L). Closes with a
  required **ROBUSTNESS verdict** (PASS/GAP/UNCERTAIN per §G–§L); any GAP in
  atomic-write/lock/boot-wipe is release-blocking and outranks every tuning finding.

## Output contract

A reviewer answering this prompt must return:

1. **Findings matrix** — box-drawn Unicode, code-fenced, grouped by section:
   ITEM · CURRENT · CALL (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK ·
   EVIDENCE (URL + version/date/commit).
2. **Before→after** for each TUNE/FIX — exact current string, exact replacement,
   in-script global.
3. **VERIFY block** — the post-reboot command set.
4. **Security delta vs CachyOS defaults** — ordered.
5. **Verdict** — one per section (OPTIMAL/TUNE/FIX) plus overall
   (PASS/PASS-WITH-FIXES/FAIL).
6. **Methodology** — source list with access dates/versions; conflicts flagged;
   unknowns marked UNCERTAIN.

## Review rules (summary)

- Hardware-anchored to gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified /
  dual 10 GbE.
- Deliberate trade-offs: **flag + quantify, do not auto-FIX.** FIX is reserved for
  incorrect/superseded/deprecated/harmful values.
- Rate IMPACT × RISK. Default KEEP when impact is marginal and risk non-trivial.
- Never invent params/flags/keys/options/URLs — cite a source or mark UNCERTAIN.
- Flag every source conflict and state which is trusted.
- Exact versions (kernel / Mesa / linux-firmware / pkg) and exact before→after,
  mapped to the in-script global.

## Special-case rules

Config choices explicitly protected from being flagged as regressions unless current
upstream directly contradicts the rationale:

- **IOMMU** — `amd_iommu=off` is itself removed; current is `iommu=pt`. Treat changes
  under the reinstatement rule; record as a restored mitigation.
- **Governor/EPP** — `performance`→`powersave` + `performance`→`balance_performance`
  was deliberate (EPP-honoring under active mode). Do not flag `powersave` as a
  regression without proving the `performance` governor would override the EPP hint.
- **GPU_DPM_LEVEL (NEW)** — `high`→`auto` was deliberate (stop pinning SCLK and
  stealing Zen 5 boost). Do not flag `auto` as a GPU-perf regression without proving
  `high` materially improves frametime/1%-lows without costing CPU boost on the shared
  140 W package.

Re-added in this revision and therefore no longer "deliberately removed" — evaluate as
live config (KEEP if upstream-valid, FIX-to-remove if inert/harmful):
PROTON_FSR4_RDNA3_UPGRADE, MangoHud gpu_power/text_outline/toggle_hud.

Still deliberately removed (reinstatement rule applies): amdgpu.ppfeaturemask,
`--country` flag, TTM/GTT cap, RADV drirc, MangoHud cpu_temp/fps_metrics,
vm.page-cluster/vm.vfs_cache_pressure (vendor-provided).

## Non-goals

Recommendations only — no modified script emitted. Out of scope: dotfiles, shells,
editors, secrets, backups, multi-user, non-CachyOS, laptops, UKI. Per-game Proton
tuning is secondary to system-wide config.

## Usage

Provide this prompt together with the matching `ry-install.fish` v7.75.1 to a
research-capable model with live source access. When the script version changes, fork
the prompt to a new version file and update the counts, the inline v-deltas, and the
VERIFY/security-delta sections to match.

## See also

`CHANGELOG.md` — prompt revision history.
