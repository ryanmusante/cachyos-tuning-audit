# ry-install Deep-Research Prompt

Version-pinned deep-research prompt for auditing `ry-install.fish` against current
upstream sources. Each release of the prompt tracks one exact script version; the
current file targets **v7.70.1**.

- **File:** `ry-install-research-prompt-v7_70_1.md`
- **Prompt version:** v7.70.1 (matches script version under audit)
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
║ KERNEL_PARAMS       ║ 12    ║
║ ENV_VARS            ║ 10    ║
║ SYSCTL              ║ 11    ║
║ PKGS_ADD            ║ 17    ║
║ PKGS_DEL            ║ 9     ║
║ MASK                ║ 10    ║
║ EXPECTED_SERVICES   ║ 5     ║
║ managed files       ║ 17    ║  (14 system + 3 user)
║ MangoHud directives ║ 15    ║
║ MKINITCPIO_HOOKS    ║ 11    ║
║ LOGIND              ║ 8     ║
```

## Structure

The prompt is organized as:

- **Mission / Rules / Output** — review mandate, the flag-don't-fix discipline,
  required report shape.
- **VERIFY block** — post-reboot `fish` assertions; hard `--verify` asserts vs
  warn-level checks are enumerated.
- **Security delta** — ordered list of where the profile reduces hardening vs
  CachyOS defaults.
- **Investigation §1–§12** — ordered by installer phase: platform baseline, packages,
  kernel cmdline, bootloader/initramfs, GPU/Vulkan, CPU power, memory/storage,
  network/latency, systemd units, security (cross-cutting), known issues/DKMS,
  MangoHud/Bluetooth/hygiene.
- **Scope & non-goals** — out-of-scope surfaces and the reinstatement rule for
  deliberately removed values.

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

Two config choices are explicitly protected from being flagged as regressions
unless current upstream directly contradicts the rationale:

- **IOMMU** — `amd_iommu=off` is itself removed; current is `iommu=pt`. Treat changes
  under the reinstatement rule; record as a restored mitigation.
- **Governor/EPP** — `performance`→`powersave` + `performance`→`balance_performance`
  was deliberate (EPP-honoring under active mode). Do not flag `powersave` as a
  regression without proving the `performance` governor would override the EPP hint.

Do not recommend reinstating deliberately removed values (amdgpu.ppfeaturemask,
PROTON_FSR4_RDNA3_UPGRADE, `--country` flag, TTM/GTT cap, RADV drirc, MangoHud
gpu_power/cpu_temp/toggle_hud) unless upstream contradicts the removal rationale —
and then flag, don't FIX.

## Non-goals

Recommendations only — no modified script emitted. Out of scope: dotfiles, shells,
editors, secrets, backups, multi-user, non-CachyOS, laptops, UKI. Per-game Proton
tuning is secondary to system-wide config.

## Usage

Provide this prompt together with the matching `ry-install.fish` v7.70.1 to a
research-capable model with live source access. When the script version changes, fork
the prompt to a new version file and update the counts, the inline v-deltas, and the
VERIFY/security-delta sections to match.

## See also

`CHANGELOG.md` — prompt revision history.
