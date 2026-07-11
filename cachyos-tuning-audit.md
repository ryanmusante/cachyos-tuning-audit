#  cachyos-tuning-audit — ry-install Tuning (CachyOS · Beelink GTR9 Pro)

Target: `ry-install.fish` **v7.99.1** (attached). Source of truth: script > README > CHANGELOG.

**Platform:** Beelink GTR9 Pro · Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤96 GB as VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP · CachyOS · systemd-boot.

**Counts (from `_ir_validate_counts`, all 21 hard-asserted — WAS 19 at v7.91.0; two tripwires ADDED, three values MOVED):** KERNEL_PARAMS **17** (composition changed: `clearcpuid=umip`) · MKINITCPIO_HOOKS 11 · MKINITCPIO_MODULES 1 · LOGIND_IGNORE_KEYS 8 · ENV_VARS 11 · SYSCTL_VALUES 9 · PKGS_ADD **19** (was 17) · PKGS_DEL 9 · MASK **12** (was 10) · EXPECTED_VULKAN_PKGS 2 · EXPECTED_SERVICES 5 · _RY_PKG_MANAGED_SERVICES 1 · _RY_POST_HOOKS 17 · **_RY_ARGPARSE_SPEC 7 (NEW tripwire, v7.94/95)** · _RY_BOOT_CRITICAL_DSTS 4 · _RY_PHASE_NAMES 6 · _RY_BACKUP_TARGETS **4** (was 2 — now derived `= $_RY_BOOT_CRITICAL_DSTS`, v7.96/97) · _RY_TMPDIR_GLOBS 6 · SYSTEM_DESTINATIONS 15 · USER_DESTINATIONS 2 · **MKINITCPIO_COMPRESSION_OPTIONS 2 (NEW tripwire, v7.96/97)**. Managed files = **17** (15 system + 2 user; `_RY_MANAGED_FILE_COUNT=17`) — a runtime drift-guard now recomputes `count(SYSTEM+USER destinations)` at load and refuses (exit 3) on mismatch with the declared constant. The modprobe surface is a **single merged `/etc/modprobe.d/60-ry-modules.conf`** (mt7925e ASPM-off + conditional `blacklist amdxdna`, v7.99.0) — `60-ry-mt7925e.conf` and the interim `60-ry-blacklist-amdxdna.conf` no longer exist as destinations (README ships a one-time `sudo rm` migration note for pre-7.99 installs). MangoHud.conf = **19 active directives + 1 commented** — composition unchanged from v7.91.0 (`# cpu_temp` off, `cpu_power` live, `gpu_temp` before `gpu_core_clock`), but the commented line's TEXT expanded to `# cpu_temp intentionally disabled — enable if you want CPU temperature in the HUD` (prefix-greps still match; byte-exact checks must use the new string, §B9). NTSYNC autoload confs = 0 (assert-only, unchanged).

**Hard floors:** KERNEL_MIN **6.19** — now **UNCONDITIONAL** for deploy and `--check` (both exit 3; `--verify` warns and continues): `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` was **REMOVED (v7.98.x)** and no floor bypass exists. The rationale was **RE-SCOPED (v7.94/95)**: the floor is anchored to **gfx1151 "MES-0x86" amdgpu** stability ONLY — the refusal message now states the RTL8127 r8169 support + suspend/shutdown-hang fix (`ae1737e7339b`) are "already present below this floor" (they are guaranteed-satisfied, no longer floor rationale; every "floor = gfx1151 + RTL8127" statement is retired). · CPU gate `Ryzen AI Max` (override `RY_INSTALL_SKIP_HARDWARE_CHECK=1` — the SOLE remaining skip env; fail-closed on unreadable model). · Soft Mesa < 26.0 warn — the `vercmp` output is now **validated `^-?\d+$` before the compare** (v7.94/95; garbage/empty output logs `MESA_SOFT_FLOOR_SKIP` instead of reaching `test(1)`).

> **MES-0x86 label conflict — STATUS CHANGED (resolve at audit time).** The prior open item stands, with a narrowed source surface: the README's Known-Issues table — which carried the "resolved (MES 0x86; current linux-firmware + shipped mkinitcpio-firmware)" note — was **DROPPED in the v7.98.x docs condensation**. The "MES-0x86" label now survives **ONLY in the script**: the `KERNEL_MIN` comment (line 15, `# gfx1151 MES-0x86 amdgpu floor (RTL8127 + suspend fixes sit below)`) and the two `_ir_validate_kernel_floor` refusal strings. The v7.77.1-era counter-claim (0x83 bad / 0x80 good) remains unreconciled. Per precedence (script > README > CHANGELOG) the in-force label is still "MES 0x86", now uncorroborated by any README prose. Re-verify the shipping gfx1151 GC 11.5.1 MES revision against current upstream (git.kernel.org/kernel-firmware, ROCm #5724, Launchpad #2129150) and state which label matches; flag the disagreement rather than silently carrying either. Still the top factual reconciliation.

## What changed since v7.91.0 (this re-audit's delta — READ FIRST)

The prior revision targeted v7.91.0. The script is now **v7.99.1** (releases 7.92.0 → 7.99.1). Function count **288 → 289**, script **4951 → 4995 lines**, `PROFILE_NAME` **`gtr_pro` → `gtr9_pro`**. Every item below is re-derived directly from `ry-install.fish` v7.99.1 and OVERRIDES the corresponding v7.91.0 conclusion. Ordered by audit impact (tuning values → managed surface → verify coverage → robustness → docs):

1. **`clearcpuid=514` → `clearcpuid=umip` (v7.94/95) — the only KERNEL_PARAMS composition change.** Count stays 17; the token string changed to the name form, which is **version-stable** (the numeric bit index is kernel-layout-fragile; README: "String form is version-stable. Drop the token if no `umip_printk` stutter appears."). Everything downstream updates: B1 cmdline body, `_vsb_cmdline`/`_vrk_cmdline` asserted token, VERIFY command expectation, security-delta wording. The §3 evaluation (UMIP-off benefit vs descriptor-leak/taint cost) is unchanged in substance.
2. **modprobe MERGED + `amdxdna` blacklist NEW (v7.94 → v7.99.0).** `/etc/modprobe.d/60-ry-modules.conf` is now the single managed modprobe destination: `options mt7925e disable_aspm=1` + (when `BLACKLIST_AMDXDNA=true`, the default) `blacklist amdxdna`. Rationale in-file: the **XDNA NPU driver probes `-EINVAL` (ret -22) under `amd_iommu=off`** — blacklisting silences a guaranteed-failing probe. NEW scalar `BLACKLIST_AMDXDNA` (true|false, `_ir_validate_keys`-gated) with a HARD COUPLING: **`BLACKLIST_AMDXDNA=false` while `amd_iommu=off` ∈ KERNEL_PARAMS refuses deploy** ("requires the IOMMU; drop amd_iommu=off; set amd_iommu=on iommu=pt"). The NPU opt-in path is therefore validator-enforced, mirror-image of the nftables↔ipv6 coupling. Audit: confirm `-EINVAL/-22` is the current amdxdna failure mode under AMD-Vi-off, and that blacklist (vs install-time removal or udev unbind) is the right mechanism; evaluate the accepted functional loss (no NPU) as part of the §10 `amd_iommu=off` trade.
3. **PKGS_ADD 17 → 19 (v7.92/93):** `+pacman-contrib` (`pactree`/`paccache` — the script's own removal-safety and cache-trim tooling is now a declared dependency instead of an assumed one) `+archlinux-contrib` (in-script comment: "extra admin scripts, not invoked"). Actionable: `pacman-contrib` is self-evidently justified; **`archlinux-contrib` installs code nothing in the profile invokes** — KEEP-documented or FIX-to-remove, decide on lean-profile grounds. **"Explicit post-`Syu`" is install-REASON marking, not ordering:** after `-Syu`, present PKGS_ADD members are re-marked `pacman -D --asexplicit` so the Phase-4 `-Rns` of PKGS_DEL can never orphan-cascade one that pre-existed as a dependency (`-D` is idempotent; failure warns + logs `PKG_ASEXPLICIT_FAIL` and the orphan exemption is "not guaranteed").
4. **MASK 10 → 12 (v7.96/97):** `+avahi-daemon.service` `+avahi-daemon.socket` — in-script rationale: "second mDNS responder collided with resolved". resolved already ships `MulticastDNS=no`; masking avahi (unit AND socket, so no socket-activation resurrection) closes the other responder. Audit: confirm nothing on this host needs avahi (printer discovery/CUPS, `.local` resolution is already off), and that masking the socket alongside the service is the complete closure.
5. **Kernel-floor bypass REMOVED (v7.98.x).** `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` has 0 occurrences; the floor is deploy/`--check`-unconditional (exit 3), `--verify`-warn. The documented skip-env inventory is now exactly one: `RY_INSTALL_SKIP_HARDWARE_CHECK=1`. Update §E ("the two skip-override env vars" → one) and drop the override from every floor mention.
6. **NTP remediation is now unconditional + NTP-client conflict guard NEW (v7.96/97).** `RY_NO_NTP_REMEDIATION` REMOVED (0 occurrences). An unsynced clock always takes the remediation path, BUT a NEW guard first scans `chronyd.service`/`ntpd.service`: if either is enabled or active, the script **refuses to enable systemd-timesyncd** ("two NTP clients would conflict; repair <unit> manually") and warns instead. RTC write-back (`_ry_rtc_writeback`, `hwclock --systohc --utc`, `RTCInLocalTZ=yes` defer-to-systemd guard) unchanged at both sync-confirmed paths. Audit: the conflict guard resolves the prior "does auto-enable fight an existing NTP client" concern by design — verify the two-unit scan list is sufficient (chrony/ntpd; ntpsec ships `ntpd.service`, openntpd ships `openntpd.service` — is the latter a gap?).
7. **udev GPU rule FIXED AGAIN — `ENV{DEVTYPE}` (v7.94/95).** The v7.79-form rule used bare `DEVTYPE=="drm_minor"`, which is **not a udev match key — the rule NEVER APPLIED**. Now `ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="$GPU_DPM_LEVEL"`. Net effect on ≤7.94 installs was nil ONLY because `GPU_DPM_LEVEL=auto` is also the kernel default — any non-`auto` choice would have been silently inert. This is the second inert-rule fix (EPP rule, v7.85) — B4 rewritten; the "confirm the rule now matches" item re-opens for the GPU rule. The EPP rule's ATTR value is now interpolated from the hoisted `$EPP_PREFERENCE` global (item 12).
8. **Backup surface 2 → 4 (v7.96/97).** `_RY_BACKUP_TARGETS = $_RY_BOOT_CRITICAL_DSTS` (derived, count-asserted 4): loader.conf, `/etc/kernel/cmdline`, sdboot-manage.conf, mkinitcpio.conf ALL get `.ry.bak` + post-write verify/restore. The prior §G risk note ("cmdline is regenerable, only 2 backed up") is RESOLVED-by-change; the remaining §G question narrows to non-boot files (nftables.conf et al.) which stay atomic-write + detect-only — now partially mitigated by item 9.
9. **nftables pre-commit validation NEW (v7.96/97).** The rendered ruleset must pass `nft -c -f <tmpfile>` BEFORE the atomic rename ("refusing deploy (live ruleset and installed file unchanged)", `NFT_PREVALIDATE_FAIL`); `_post_nft` re-validates the installed file before `systemctl restart nftables` (oneshot re-runs `nft -f`; on restart failure the validated ruleset applies at next boot). Closes the deploy-a-syntactically-broken-firewall window the v7.91.0 §G flagged for non-backup files.
10. **Long-op timeout policy INVERTED (v7.92/93).** `pacman`, `mkinitcpio`, `sdboot-manage`, `paccache`, `updatedb`, `pkgfile` are **no longer timeout-EXEMPT**: `_run`'s resolver PATH-resolves the command (basename of `command -s`) and **FLOORS the effective timeout to `_RY_LONGOP_HARD_CAP=7200 s`** — user values below 7200 are raised ("short SIGKILL would bypass rollback"), values above pass through, and `RY_RUN_TIMEOUT=0` still fully disables. Every §I/§K "pacman/mkinitcpio are exempt from the cap" statement is retired. Audit: confirm a worst-case `-Syu` (slow mirror + big initramfs) fits 2 h, and that a 7200 s kill still surfaces as fatal with rollback intact.
11. **Verify surface ADDED (v7.98.x) — four upgrades.** (a) NEW leaf **`_vrkm_blacklist_modprobe`** under `_vrk_module_state`: parses every `blacklist <mod>` entry out of the MANAGED modprobe.d content (generator-sourced, not on-disk), normalizes `-`→`_`, and `lsmod`-checks each — a loaded blacklisted module (amdxdna) is a **FAIL**; `lsmod` absent → warn-skip. This partially restores the module-state live coverage dropped in v7.90.0, on the blacklist side. (b) `_vsb_mkinitcpio` now compares the LIVE `COMPRESSION=` (and `COMPRESSION_OPTIONS` tokens) against `MKINITCPIO_COMPRESSION`/`_OPTIONS` via `_ry_mkinitcpio_array` — which joins **multi-line `KEY=( … )` blocks** and takes the LAST assignment (shell-sourced semantics) with a multiple-assignment warn. (c) `_chk_grep` now **strips inline comments** (awk `sub(/[[:space:]]+#.*$/,"")` + comment-line skip) before the `grep -wF` token match — a commented-out `key=value` can no longer satisfy a presence check; rc 1 distinguishes "no non-comment lines". (d) `_vrk_gpu_state` compares quoted (`test "$level" = "$GPU_DPM_LEVEL"`) — the unquoted form could mis-evaluate on empty sysfs reads. Plus: `_vsp_pacman_conf` gains a **sudo-read fallback** for a 0600-hardened pacman.conf with grep-rc>1 and mid-read sudo-lapse gates (warn-skip, never false-"not set").
12. **Data hoists + enum gates (v7.98.x).** `EPP_PREFERENCE=balance_performance` is a global, enum-gated against NEW `_RY_EPP_LEVELS` (default performance balance_performance balance_power power) in `_ir_validate_keys` — like `GPU_DPM_LEVEL`, the value is interpolated UNQUOTED into a udev ATTR, so the enum bounds injection. `EXPECTED_SCALING_DRIVER=amd-pstate-epp` hoisted (verify-only; non-empty-gated). `CPUPOWER_GOVERNOR` charset aligned to its format validator: `^[a-z][a-z0-9_-]*$` ("the domain `_grep_cpupower_entry` accepts").
13. **Preflight/init hardening (v7.94 → v7.99.1).** `id(1)` is now the FIRST external command and hard-required (UID resolution precedes every privilege decision); `find(1)` hard-required with a `-maxdepth`/`-printf` capability probe; `timeout(1)` probed for `--foreground --kill-after`; **`mv -T` capability live-probed via two mktemp files at init**; `stat(1)` and `date(1)` (`%z` output probed) required — busybox/uutils explicitly rejected on each; systemd ≥ 250 gate. Input hygiene: `KERNEL_PARAMS` tokens charset-gated `^[A-Za-z0-9._,=-]+$` (spliced into shell-sourced boot configs + the cmdline); boot scalars (`MKINITCPIO_COMPRESSION`, `SDBOOT_DEFAULT_ENTRY`, `LOADER_DEFAULT/CONSOLE_MODE/EDITOR`, `CPUPOWER_GOVERNOR`) metachar-gated with a PCRE class using `\x27` for the single quote; `MKINITCPIO_COMPRESSION_OPTIONS` tokens `^-?[A-Za-z0-9]+$`.
14. **CLI/signal/teardown contract (v7.94 → v7.99.1).** Root `--check` is a **silent exit 3**, and the silence now holds through the **pre-argparse window** (v7.99.1; `_RY_ARGV_CHECK_ONLY` hint set before MODE exists; `-V`/glued `-VV` are `--check`-compatible; any other flag/positional restores exit-2 parity). Repeated `--install-file` resolves **last-wins**. The root guard emits **one `@@LEFT@@` line per leftover positional** (display-only sentinel). Unmatched post-hook patterns log `POST_HOOK_NONE` (file deployed, live-apply skipped) vs `POST_HOOK_SKIP_UNCHANGED`. Log rename is `mv -T` with a **`cp -pT` + `rm` recovery** (dir-squat safe; both-fail keeps logging at the old path); a pre-existing symlinked LOG_FILE is removed and re-created 0600; logged `_run` argv redacts `/tmp/ry-*` paths to `[REDACTED]`. `TMPDIR` is **erased at init** (tmp pinned to `/tmp`; children must not honor an inherited TMPDIR). The umask is set as the **`umask` variable directly** (v7.99.1 — the `umask` builtin is autoloaded in fish; a signal mid-autoload leaked "Unknown command" to stderr). Lock: `getconf CLK_TCK` failure falls back to **USER_HZ=100** (correct — `starttime` is USER_HZ, fixed at 100 on Linux, NOT the kernel tick rate; the old `/proc/config.gz` CONFIG_HZ recovery was wrong in principle and is gone) and the mkdir-success flag is set beside the rc capture. Probe helpers dropped builtin→pipe captures (SIGPIPE risk).
15. **Stale-description ledger: 3 FIXED, 1 NEW.** The three v7.91.0-flagged orchestrator `--description` strings are corrected (`_verify_static_system` no longer says "ntsync"; `_verify_runtime_kparams` no longer says "clocksource"; `_verify_runtime_env` no longer says "TCP, ZRAM"). **NEW (FIX-doc, Low/Low): `_verify_runtime_session` `--description` still ends "…, Vulkan packages"** while its body calls only `_vrs_nm_perms` + `_vrs_installed_file_perms` + `_vrs_parent_dirs` (the Vulkan check moved to `_vsp_required` in v7.90.0). Same drift class, one release behind.
16. **`gtr_pro` → `gtr9_pro` (v7.92/93).** The v7.91.0 §12 naming FIX (human-facing "GTR9 Pro" vs internal `gtr_pro`) is RESOLVED — `PROFILE_NAME=gtr9_pro`; drop the finding.
17. **Docs surface (v7.98.x).** README BIOS section condensed to a **flat 85 W ceiling recommendation** (`SPL = fPPT = sPPT = 85 W`, `TjMax 90`; Strix Halo multi-thread gains flatten past ~85 W) with a link-out to `gtr9pro-bios-reference` — this is now a documented HOST-POWER posture the §6/§13d power analysis must acknowledge (an 85 W package budget, not 140 W, is what the governor/EPP/DPM arbitrate if the user follows the README). Known-Issues/Benign-Log tables DROPPED (see the MES note above). Help/README env table trimmed to exactly `RY_RUN_TIMEOUT` / `RY_INSTALL_SKIP_HARDWARE_CHECK` / `NO_COLOR`. Root/TTY contract documented: non-TTY runs require a pre-cached sudo (`sudo -v`), a TTY prompts once; the sudo-lapse mitigation banner names `timestamp_timeout=60`, a `sudo -v` keepalive, or a SCOPED NOPASSWD drop-in (pacman/mkinitcpio/sdboot-manage/systemctl — "avoid ALL").
18. **NEW source conflict (Low/Low): SYSCTL comment vs platform.** The `SYSCTL_VALUES` trailing comment now reads `# netdev=2.5GbE, …` while the platform line and §8 target **dual 10 GbE (RTL8127)**. The values themselves (`netdev_budget=600`, `netdev_budget_usecs=5000`) are unchanged since v7.75.x. Reconcile: either the comment is stale/wrong (RTL8127 is 10 GbE) or the budget values were sized for 2.5 GbE and §8's 10 G-validation question gains urgency. Flag, cite `lspci`/driver evidence, do not silently adopt either.

**Everything else the v7.91.0 body asserted about tuning VALUES is unchanged** — ENV_VARS (11), SYSCTL_VALUES (9), PKGS_DEL (9), the nftables IPv4-only ruleset shape + inbound-ping accept + remote-play port set (TCP 47984/47989/48010/27036/27037, UDP 47998-48010/27031-27036), `amd_iommu=off`, `ipv6.disable=1`, governor `powersave` + EPP `balance_performance` + `dynamic_epp=disabled`, `GPU_DPM_LEVEL=auto`, MangoHud composition (19+1), fstab `noatime,lazytime,commit=10`, regdom US, resolved/NM/logind/bluetooth/cpupower keys, loader/sdboot/mkinitcpio keys — all re-verified byte-for-byte against v7.99.1. NOTE for byte-exact (checksum-class) validation only: **every generator now emits a leading `#` header-comment line** (loader.conf, sdboot-manage.conf, resolved, logind, nm, nm-dispatcher, iw-regdomain, bluetooth, cpupower, sysctl, udev, modprobe, environment.d, MangoHud) and `mkinitcpio.conf` includes explicit `BINARIES=()` + `FILES=()` lines — the KEY=VALUE content is identical; the rendered bodies are not (see §B).

> **Verification note (2026-07-11, second pass).** This revision re-derives every value directly from `ry-install.fish` **v7.99.1** (extracted array-by-array, 289 fns, 4995 lines; parallel `rg` extraction over the data section, generators, validators, verify subsystem, and teardown). A second sweep grounded the identifier surface: **147 backticked script identifiers cited in this document — 136 present in v7.99.1, 11 whitelisted-absent removed surfaces, 0 unexpected; reverse coverage 58/58 verify-family functions accounted for.** Where the script and any prior prompt disagree, the script wins and the disagreement is flagged (headline: the **MES-0x86 label**, now script-only; the **netdev=2.5GbE comment** vs the 10 GbE platform). The v7.91.0→v7.99.1 deltas above are folded throughout §1–§13 and §A–§L. Live-upstream anchors the script did not contradict (ntsync CONFIG_NTSYNC=y, `amd_iommu=off` not breaking ROCm on gfx1151, governor/EPP semantics, §13 candidate calls) are retained pending re-confirmation. Internal-state globals (progress-bar, matrix counters, boot-enum scratch) remain out of scope.

## Lineage (v7.77.1 → v7.91.0 — condensed; still-operative facts only)

Retained so no retired surface is re-audited and no once-inverted value is trusted from stale memory. Each fact below still holds in v7.99.1 unless marked with its supersession:

1. **KERNEL_MIN 6.18 → 6.19** (v7.79) — now unconditional (v7.98.x, delta 5) with the re-scoped gfx1151-only rationale (delta header).
2. **IPv6 disabled system-wide + IPv4-only nftables + inbound-ping accept** (v7.87.x) — `ipv6.disable=1` ∈ KERNEL_PARAMS, hard-coupled in `_ir_validate_keys` (nftables managed ⇒ token required); all ICMPv6/NDP rules gone; `echo-request` accept is a REGRESSION GUARD (`_vss_nft` hard-fails if missing; `_vrsv_nft_assert_ping` warn-live).
3. **`amd_iommu=off`** (v7.77) — AMD-Vi fully disabled; ROCm on gfx1151 unaffected (`IOMMU Support: None`); VFIO/SR-IOV opt-in `amd_iommu=on iommu=pt`. NOW additionally paired with the amdxdna blacklist + `BLACKLIST_AMDXDNA` coupling (delta 2) — the NPU is a named casualty of the trade, with a validator-enforced opt-in.
4. **MangoHud posture** (v7.91): `# cpu_temp` commented (text expanded, delta header), `cpu_power` live, `gpu_temp` before `gpu_core_clock` — unchanged through 7.99.1. MangoHud #1794 (cpu_temp zeroing cpu_power on Zen 5) avoided by design; the #1825/sensor-key question dormant. Live-verification target: `cpu_power` populating from the Zen 5 RAPL/`power1_average` hwmon under Wayland.
5. **Six verify subs REMOVED** (v7.90): `_vre_tcp`, `_vre_zram`, `_vss_ntsync_modules`, `_vrkm_iommu`, `_vrk_clocksource`, `_vrs_vulkan` — all still absent. Directive-level asserts retained (cmdline token loop + `_vsb_cmdline` for `amd_iommu=off`/`tsc=reliable`; `_vss_sysctl`+`_vre_sysctl_runtime` for bbr; `_vsp_required` for the folded Vulkan pkgs). v7.98's `_vrkm_blacklist_modprobe` (delta 11a) partially restores live module-state coverage on the blacklist side. `_vre_ntsync` + the `_ntsync_state` 4-state machine survive.
6. **Removed surfaces — do NOT audit, 0 occurrences each:** baloofilerc/`_post_baloo` (v7.77), the `_kb_*` benign-INFO family + `_vss_known_benign` (v7.77), `_ry_check_umip_disabled` (v7.77), ICMPv6/NDP rules (v7.87), the linux-firmware version advisory + `20251125`/`20260110` strings (v7.81/82), `iommu=pt` (v7.77), `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` (v7.98), `RY_NO_NTP_REMEDIATION` (v7.96/97), `60-ry-mt7925e.conf` as a standalone destination (v7.99.0), `clearcpuid=514` numeric form (v7.94/95).
7. **udev inert-rule history:** the EPP rule never fired until the `SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*"` fix (v7.85; `_post_udev` retriggers cpu beside block so it live-applies); the GPU rule never fired until the `ENV{DEVTYPE}` fix (v7.94/95, delta 7). Treat any "the rule has always applied" assumption as false for both.
8. **Args/root contract:** root guard defers to argparse (v7.89, exit-2 message parity); root `--check` silent exit 3 (v7.85, window-hardened v7.99.1); stdin/pipe/source execution refused (v7.86/87).
9. **`_vmh_*` attribution:** `_vmh_existence_only`/`_vmh_order_checks` are `_ry_validate_mkinitcpio_hooks` leaves (mkinitcpio HOOKS existence/ordering), NOT MangoHud verify. MangoHud static verify = inline `_chk_file` + `_chk_grep fps`; format validation = `_grep_mangohud_entry` via `_rvc_dispatch`.

## Mission

Evaluate every config decision against current upstream sources for this exact silicon. Return a prioritized, evidence-backed tuning report. The profile deliberately trades PCI-passthrough capability, the XDNA NPU, power-saving, IPv6, and host inbound-firewalling-of-ping for performance, latency, and a simpler IPv4-only ruleset; confirm each choice is current and correct, surface anything superseded or harmful, and quantify safety deltas without second-guessing intentional design.

## Rules

1. Item-by-item, hardware-anchored to gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE.
2. Respect deliberate trade-offs: **flag and quantify, do not auto-FIX.** Reserve FIX for incorrect, superseded, deprecated, or harmful values.
3. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is non-trivial.
4. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN.
5. Flag every source conflict and state which is trusted (pre-flagged: MES-0x86 label, script-only; SYSCTL `netdev=2.5GbE` comment vs 10 GbE platform).
6. Give exact versions (kernel / Mesa / linux-firmware / pkg) and exact before→after, mapped to the in-script global.

## Output

- **Findings matrix** (box-drawn Unicode, code fence, grouped by section): ITEM · CURRENT (v7.99.1 value) · CALL (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE (URL + version/date/commit).
- **Candidate-enhancement matrix** (§13, separate): ITEM · PRESENT?(no) · CALL (ADD-default/ADD-opt-in/KEEP-omitted) · IMPACT · RISK · EVIDENCE.
- **Before→after** for each TUNE/FIX/ADD: exact current string, exact replacement, in-script global.
- **VERIFY block** (post-reboot commands, below).
- **Security delta vs CachyOS defaults** (ordered, below).
- **Verdict:** one per section (OPTIMAL/TUNE/FIX) plus overall (PASS/PASS-WITH-FIXES/FAIL).
- **ROBUSTNESS verdict** (§G–§L; separate from tuning verdicts).
- **Methodology:** source list with access dates and versions; conflicts flagged (MES-0x86 first); unknowns marked UNCERTAIN.

### VERIFY block

```fish
cat /proc/cmdline
cat /sys/.../cpu0/cpufreq/scaling_driver                    # amd-pstate-epp (EXPECTED_SCALING_DRIVER)
cat /sys/.../cpu0/cpufreq/scaling_governor                  # powersave (EPP-honoring under pstate=active)
cat /sys/.../cpu0/cpufreq/energy_performance_preference     # balance_performance (EPP_PREFERENCE)
cat /sys/.../amd_pstate/status                              # active
cat /sys/.../amd_pstate/dynamic_epp                         # disabled (else EPP pin -EBUSY; absent pre-6.16)
cat /sys/.../amd_pstate/prefcore                            # enabled
cat /sys/.../cpufreq/boost                                  # 1
cat /sys/.../clocksource0/current_clocksource               # tsc — INFORMATIONAL ONLY (no _vrk_clocksource since v7.90.0)
cat /sys/block/nvme0n1/queue/scheduler                      # [none] (adjust node)
cat /sys/class/drm/card*/device/power_dpm_force_performance_level   # auto (GPU_DPM_LEVEL; udev rule LIVE only since the v7.94/95 ENV{DEVTYPE} fix)
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l   # 0 — INFORMATIONAL ONLY (no _vrkm_iommu since v7.90.0)
cat /proc/cmdline | rg -o 'amd_iommu=\S+'                   # amd_iommu=off (AMD-Vi disabled)
cat /proc/cmdline | rg -o 'ipv6\.disable=\S+'               # ipv6.disable=1 (IPv6 OFF system-wide)
cat /proc/cmdline | rg -o 'clearcpuid=\S+'                  # clearcpuid=umip (RENAMED from 514, v7.94/95)
cat /proc/cmdline | rg -o 'processor.max_cstate=\S+'        # 1 (C-state cap)
cat /proc/cmdline | rg -o 'fsck\S+'                         # fsck.mode=force fsck.repair=yes
ls -l /dev/ntsync                                           # present (assert-only)
lsmod | rg -c '^amdxdna'                                    # 0 (blacklisted via 60-ry-modules.conf; loaded = verify FAIL, _vrkm_blacklist_modprobe)
sudo dmesg | rg -i 'AMD-Vi|DMAR'                            # expect NO "AMD-Vi: Enabled" (amd_iommu=off)
ip -6 addr                                                  # expect no IPv6 addresses (ipv6.disable=1)
cat /etc/modprobe.d/60-ry-modules.conf                      # options mt7925e disable_aspm=1 + blacklist amdxdna (MERGED file, v7.99.0)
ls /etc/modprobe.d/60-ry-mt7925e.conf /etc/modprobe.d/60-ry-blacklist-amdxdna.conf 2>/dev/null   # ENOENT both (pre-7.99 leftovers must be removed once — README migration note)
pacman -Q linux-firmware                                    # currency check only (no version gate in the script)
pacman -Q pacman-contrib archlinux-contrib                  # present (PKGS_ADD 19, v7.92/93)
vulkaninfo | rg -i 'driverName|deviceName'                 # RADV / Radeon 8060S; confirm uma heap
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc vm.max_map_count vm.compaction_proactiveness vm.swappiness
findmnt -no OPTIONS /                                       # noatime,lazytime,commit=10
swapon --show; zramctl                                      # zram active (advisory; not managed, not asserted)
iw reg get | rg -i country                                 # US
cat /etc/iw-regdomain                                       # COUNTRY=US
sudo nft list chain inet filter input                      # policy drop + lo + established/related + IPv4 ICMP incl echo-request (inbound ping ALLOWED); +remote-play ports IFF RY_REMOTE_PLAY_PORTS=true. NO ICMPv6/NDP rules (IPv4-only).
sudo nft -c -f /etc/nftables.conf                          # syntax-valid (the script now refuses to deploy/reload a ruleset failing this, v7.96/97)
stat -c '%a %U:%G' /etc/NetworkManager/system-connections/* # 0600 root:root
systemctl is-enabled bluetooth.service                     # enabled
systemctl is-enabled avahi-daemon.service avahi-daemon.socket   # masked masked (MASK 12, v7.96/97)
printenv MANGOHUD                                          # 1
grep -c '^cpu_temp' ~/.config/MangoHud/MangoHud.conf       # 0 (cpu_temp commented)
grep -c '^cpu_power' ~/.config/MangoHud/MangoHud.conf      # 1 (cpu_power live)
grep -c '^# cpu_temp' ~/.config/MangoHud/MangoHud.conf     # 1 (comment TEXT expanded v7.9x — prefix match still 1; byte-exact checks use the new string, §B9)
```

Hard `--verify` asserts (mismatch → exit 1/3): every `KERNEL_PARAMS` token present in `/proc/cmdline` + `rw` (generic loop in `_vrk_cmdline` — `amd_iommu=off`, `ipv6.disable=1`, `clearcpuid=umip`, `tsc=reliable` all auto-asserted at the cmdline layer); scaling_driver=`$EXPECTED_SCALING_DRIVER` (amd-pstate-epp), scaling_governor=`$CPUPOWER_GOVERNOR` (powersave), EPP=`$EPP_PREFERENCE` (balance_performance), amd_pstate status/prefcore/boost, `dynamic_epp=disabled`; GPU `power_dpm_force_performance_level=$GPU_DPM_LEVEL` (auto; comparison QUOTED, v7.98.x); usbcore.autosuspend=-1, nvme_core.default_ps_max_latency_us=0, zswap.enabled∈{N,0}, nmi_watchdog=0, NVMe sched `[none]`; **managed modprobe blacklist entries NOT loaded (`_vrkm_blacklist_modprobe` — amdxdna loaded ⇒ FAIL; NEW v7.98.x)**; live mkinitcpio `COMPRESSION=`/`COMPRESSION_OPTIONS` match the embedded values (`_vsb_mkinitcpio` via `_ry_mkinitcpio_array`, multi-line-join, last-wins; NEW v7.98.x); regdom (`/etc/iw-regdomain`); nftables IPv4 ping accept present (`_vss_nft` hard static — regression guard) + live warn (`_vrsv_nft_assert_ping`); mt7925e `disable_aspm=1` (now in `60-ry-modules.conf`); NM system-connections 0600 root:root (`_vrs_nm_perms`); PKGS_ADD (19) present + Vulkan pkgs folded into `_vsp_required`. Presence checks are comment-proof: `_chk_grep` strips inline comments and skips comment lines before matching (v7.98.x). **REMOVED asserts (unchanged since v7.90.0 — DO NOT verify): `_vrkm_iommu` (0-iommu-groups), `_vrk_clocksource` (HPET-fail/TSC-demotion), `_vre_zram`, `_vre_tcp`.** No THP, KSM, `ttm.*`, drirc, `iommu=pt`, ICMPv6/NDP, baloo, or `_kb_*` assert exists — do not verify them.

### Security delta (ordered)

1. **UMIP off (`clearcpuid=umip`, renamed from 514)** — descriptor-table base leak, kernel tainted; headline open reduction. The name form is version-stable; the exposure is identical.
2. **AMD-Vi fully disabled (`amd_iommu=off`)** — no DMA isolation/remapping; any DMA-capable device (USB4/Thunderbolt, NVMe, NIC) can in principle DMA over system RAM unmediated. NOW with a named functional casualty: the **XDNA 2 NPU is blacklisted** (amdxdna probes `-EINVAL` without the IOMMU; `BLACKLIST_AMDXDNA=true` default). The validator-enforced opt-back-in (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`) restores both DMA isolation and the NPU together. Quantify the exposure; confirm it stays an accepted trade for a no-passthrough, no-NPU single-user desktop. Open reduction.
3. **IPv6 disabled + inbound IPv4 ping allowed (net wash-to-slight-reduction)** — `ipv6.disable=1` removes the entire IPv6 stack; the IPv4 ruleset accepts inbound `echo-request` (LAN discoverability up slightly). NEW counterweight: **avahi masked (unit+socket, v7.96/97)** removes the second mDNS responder — with resolved's `MulticastDNS=no` this closes multicast discovery entirely, slightly tightening what the ping-accept loosened. Quantify both directions on a trusted dual-10GbE LAN.
4. **split_lock_detect=off** — a misbehaving app can degrade the system.
5. **Plaintext DNS** (`DNSOverTLS=no`, `DNSSEC=allow-downgrade`) reverting the CachyOS DoH default — DNS observable and spoofable on-path.
6. **Optional inbound remote-play ports** (`RY_REMOTE_PLAY_PORTS`, default OFF) — when enabled, opens TCP 47984/47989/48010/27036/27037 + UDP 47998-48010/27031-27036; quantify the opt-in exposure.
7. **Firewall default-deny-inbound ships** (nftables, IPv4-only; lo + established/related + IPv4 diagnostic ICMP incl. inbound ping; all else dropped) — net positive, NOW with a `nft -c` pre-commit gate so a malformed managed ruleset can never replace a working one (v7.96/97).

## Investigation (§1–§12 ordered by installer phase; §13 = candidate enhancements)

### 1. Platform baseline and version floors

Current: **hard kernel floor KERNEL_MIN 6.19, UNCONDITIONAL** (deploy and `--check` exit 3; `--verify` warns and continues; fail-closed on unreadable `uname -r`; **no override — `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` removed v7.98.x**) — rationale re-scoped to **gfx1151 "MES-0x86" amdgpu ONLY** (the refusal message states the RTL8127 r8169 + suspend-hang fixes are "already present below this floor"); CPU gate `Ryzen AI Max` (override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`, fail-closed on unreadable model — the sole skip env); soft Mesa < 26.0 warn (`vercmp`, output validated `^-?\d+$` before compare, garbage → logged skip). No firmware-version advisory of any kind.

- **6.19 floor — RE-VERIFY under the narrowed rationale.** The floor now rests on ONE claim: gfx1151 MES-0x86 amdgpu native support requires ≥6.19. Confirm (a) that claim against current upstream; (b) that the RTL8127 commits (`f24f7b2f3af9` support, `ae1737e7339b` suspend/shutdown-hang) indeed land below 6.19 as the script now asserts (they were previously stated as ≥6.19-guaranteed — the direction of the claim flipped; verify the actual landing kernels); (c) whether the true gfx1151-stability floor sits above 6.19. State per-subsystem floors and whether integer 6.19 is conservative, exact, or insufficient.
- **MES-0x86 label — RESOLVE THE CONFLICT (headline note).** Now script-only (README table dropped v7.98.x). Determine against current upstream which MES revision the shipping gfx1151 GC 11.5.1 blob reports; report (i) current-correct, (ii) carry-over/typo, or (iii) a third revision — with the trusted source named. Do not silently adopt either side.
- **Floor override removal — assess the recoverability trade.** With no bypass, a user on a 6.18 CachyOS snapshot cannot deploy at all (`--verify` still runs). Confirm unconditional-floor is the right posture vs the misuse risk the override carried; note the README documents no workaround (correct).
- Confirm the soft Mesa 26.0 floor matches current RADV guidance; enumerate open gfx1151 RADV issues.
- Confirm gfx1151 reports `uma:1` natively (removed drirc stays redundant).
- **README BIOS posture (NEW context):** the README now prescribes a flat `SPL=fPPT=sPPT=85 W` + `TjMax 90` BIOS ceiling (multi-thread gains flatten past ~85 W). Out of installer scope, but §6/§13d power calls must state which package budget (85 W README-recommended vs 140 W stock) they assume.
- Sources: wiki.cachyos.org, docs.kernel.org gpu/amdgpu, gitlab.freedesktop.org/mesa (gfx1151), git.kernel.org linux-firmware + r8169.

### 2. Packages

PKGS_ADD (**19**, explicit post-`Syu`): nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa, mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, git-delta, lm_sensors, rtkit, realtime-privileges, ddcutil, nftables, **pacman-contrib, archlinux-contrib (NEW v7.92/93)**. PKGS_DEL (9, `-Rns`, rdep-aware): plymouth stack (plymouth, cachyos-plymouth-bootanimation, cachyos-plymouth-theme, breeze-plymouth, plymouth-kcm) + micro + cachyos-micro-settings + cachy-update + kdeconnect. AUR: none. Vulkan (chwd): vulkan-radeon, lib32-vulkan-radeon.

- **pacman-contrib (NEW) — KEEP, self-consistent.** The script itself invokes `pactree` (removal-safety, `PACTREE_TIMEOUT_S=60`) and `paccache` (`-rk2` + `-ruk0`, separate runs); declaring the provider closes a formerly-assumed dependency. Confirm both binaries are owned by pacman-contrib on current Arch/CachyOS.
- **archlinux-contrib (NEW) — DECIDE.** In-script comment: "extra admin scripts, not invoked." Nothing in the profile calls it; on lean-profile grounds this is a FIX-to-remove candidate, on convenience grounds a KEEP-documented. Make the call explicit; do not let it ride as incidental.
- Confirm cachyos-gaming-meta/-applications supply RADV/Proton/gamescope/MangoHud/GameMode; **GameMode omission — KEEP** (governor/EPP/DPM already pinned profile-wide; GameMode's governor switch is redundant). Confirm the meta pulling MangoHud does not conflict with the shipped MangoHud.conf.
- `rtkit` + realtime-privileges: confirm correct for PipeWire thread priority; rtkit-daemon stays socket-activated (not in EXPECTED_SERVICES).
- Confirm `lib32-mesa` still needed alongside `lib32-vulkan-radeon`.
- Confirm PKGS_DEL has no dependency fallout (`-Rns` skips + logs external dependants; `_RY_PKG_REMOVE_SKIPS`).
- Advisory one-liner: state whether znver/x86-64-v4 (AVX-512) repos benefit this build over v3 (script no longer probes repo tier).
- Sources: wiki.cachyos.org, wiki.archlinux.org/Gaming + PipeWire + RealtimeKit, archlinux.org/packages (pacman-contrib, archlinux-contrib).

### 3. Kernel cmdline (17)

`8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0`

NEW/changed since v7.91.0 — audit fresh:

- **`clearcpuid=umip` (RENAMED from `clearcpuid=514`, v7.94/95):** identical semantics (UMIP trapping off; kernel tainted), version-stable string form — the numeric CPUID-bit index is fragile across kernel feature-word relayouts; the name form is looked up symbolically. Confirm `clearcpuid=` accepts the `umip` name on the ≥6.19 floor (name support landed well before; state the exact kernel). The §10 exposure evaluation is unchanged; the README posture ("drop the token if no `umip_printk` stutter appears") stands. There is still no UMIP-specific check function — the token is asserted generically (`_vrk_cmdline` + `_vsb_cmdline`).
- **`amd_iommu=off` (unchanged directive, NEW coupling):** the amdxdna blacklist (§2-delta/§8/§10) is now validator-paired to this token — `BLACKLIST_AMDXDNA=false` refuses to deploy while `amd_iommu=off` is present. ROCm on gfx1151 remains unaffected (`IOMMU Support: None`); the NPU is the named casualty. Re-evaluate as before: latency win (marginal) vs DMA-isolation loss (§10) vs the now-explicit NPU loss; the documented opt-back-in is `BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt` + re-run.
- **`ipv6.disable=1` (unchanged, v7.87.x):** hard-coupled to the managed IPv4-only nftables ruleset (`_ir_validate_keys`). Evaluate as before: LAN impact of no-IPv6-stack, app fallback (Steam/Proton netcode), README dual-stack opt-out.

Carry-forward params (still validating):

- **processor.max_cstate=1:** idle power/thermal cost on the 140 W package (or 85 W per the README BIOS ceiling — state which) vs wake-latency/jitter win; confirm no conflict with amd_pstate=active or boost headroom; is `1` the right cap?
- **btusb.enable_autosuspend=n:** correct fix for MT7925/BT 5.4 reconnect/stutter alongside the mt7925e ASPM option (now in `60-ry-modules.conf`, §8); overlap with `usbcore.autosuspend=-1`?
- **fsck.mode=force + fsck.repair=yes:** every-boot fsck on dual NVMe ext4 — boot-time cost, auto-repair safety, systemd-fsck/mkinitcpio-hook interaction; durability posture vs periodic.
- **amd_pstate=active:** confirm recommended on Zen 5; interaction with powersave governor + EPP (§6) + `dynamic_epp=disabled`.
- **split_lock_detect=off:** perf vs stability; current default; blast radius.
- No `preempt=` pinned: `_vrk_cmdline` INFOs the dmesg `Dynamic Preempt:` model, `_ok` only on `full`. KEEP-omitted (CachyOS desktop kernel boot-defaults full).
- Zero amdgpu/ttm module params: hands-off GPU-param posture (`_vrkm_amdgpu` hex-aware, no-ops without `amdgpu.*`).
- Validate the rest: tsc=reliable, nowatchdog, 8250.nr_uarts=0, usbcore.autosuspend=-1, nvme_core.default_ps_max_latency_us=0, pcie_aspm.policy=performance, zswap.enabled=0.
- **Input hygiene (NEW, v7.96/97):** every KERNEL_PARAMS token is charset-gated `^[A-Za-z0-9._,=-]+$` at preflight (spliced into shell-sourced boot configs). A recommendation introducing a token outside that charset must also change the validator — flag any such collision.
- Sources: docs.kernel.org kernel-parameters + pm/amd-pstate + x86 UMIP + admin-guide + IOMMU/AMD-Vi + networking/ipv6-sysctl, wiki.archlinux.org/AMDGPU + IOMMU + fsck + IPv6, amd.com ROCm.

### 4. Bootloader and initramfs

loader.conf: default @saved, timeout 0, console-mode keep, editor no (body now header-commented, §B2). sdboot-manage: DEFAULT_ENTRY manual, OVERWRITE/REMOVE_EXISTING/REMOVE_OBSOLETE yes, LINUX_FALLBACK_OPTIONS "quiet" (header-commented, §B1). mkinitcpio: MODULES=(amdgpu), HOOKS (11) = base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck, COMPRESSION zstd, COMPRESSION_OPTIONS=(-1 -T0) — body now includes explicit `BINARIES=()` + `FILES=()`. `mkinitcpio.conf` pre-deployed in Phase 2 so `pacman -Syu` triggers exactly one initramfs rebuild (the pre-`Syu` seed is a tagged log line, v7.98.x).

- Verify HOOKS order (systemd hook; microcode/kms/sd-vconsole/block placement); amdgpu early-KMS; `fsck.mode=force` + fsck-hook handshake with no boot prompt/hang.
- **COMPRESSION_OPTIONS=(-1 -T0):** zstd level -1 (fastest/lowest-ratio) with all threads. Quantify boot-decompress delta vs default-3 (likely sub-100 ms on NVMe) and the larger image vs ESP `BOOT_SPACE_*` gates (200/500 MB) with multiple kernels + fallback. TUNE toward default-3 or `-T0`-alone if size threatens the budget for negligible win. NOTE: the tokens are now charset-gated (`^-?[A-Za-z0-9]+$`) and count-asserted (tripwire `MKINITCPIO_COMPRESSION_OPTIONS:2`) — any TUNE must update both.
- **Live verify upgraded (v7.98.x):** `_vsb_mkinitcpio` now compares the LIVE `COMPRESSION=` value and `COMPRESSION_OPTIONS` tokens (via `_ry_mkinitcpio_array` — multi-line `KEY=( )` join, last-assignment-wins with a warn) — a hand-edited or vendor-updated mkinitcpio.conf drifting on compression is now caught, not just hooks/modules. Confirm the last-wins semantics match shell-sourcing.
- `install-file` format-validates before write; loader.conf/cmdline changes regenerate sdboot entries only (no full initramfs rebuild). Confirm the Phase-3 pre-deploy + single-rebuild model holds.
- timeout 0 + DEFAULT_ENTRY manual + REMOVE_EXISTING=yes wipes foreign BLS entries; recovery ergonomics (live-USB → arch-chroot) — confirm intended.
- Confirm sdboot-manage current vs kernel-install/UKI (UKI out of scope).
- Sources: wiki.archlinux.org/Mkinitcpio + systemd-boot, sdboot-manage upstream.

### 5. GPU / Vulkan / gaming

No drirc shipped (gfx1151 uma:1 native). No ttm/modprobe GPU params. ENV_VARS (11, unchanged): AMD_VULKAN_ICD=RADV, DXVK_LOG_LEVEL=none, DXVK_LOG_PATH=none, MANGOHUD=1, MESA_SHADER_CACHE_MAX_SIZE=16G, PROTON_ENABLE_WAYLAND=1, PROTON_FSR4_RDNA3_UPGRADE=1, PROTON_LOCAL_SHADER_CACHE=1, VKD3D_DEBUG=none, VKD3D_SHADER_DEBUG=none, WINEDEBUG=-all.

- **ntsync (assert-only, unchanged):** no autoload conf; `_vre_ntsync` + `_ntsync_state` (builtin|loaded|loaded_nodev|missing) survive; README documents ≥6.14 mainline + `PROTON_NO_NTSYNC=1` per-title opt-out. Confirm (a) ntsync current vs esync/fsync; (b) CachyOS kernel `CONFIG_NTSYNC=y` so `/dev/ntsync` exists without autoload; (c) `loaded_nodev` still a real failure mode; (d) Proton consumes it with frametime benefit on 16C/32T; (e) the opt-out current. The 6.19 floor subsumes ntsync's requirement.
- **PROTON_FSR4_RDNA3_UPGRADE=1:** confirm current Proton/Proton-CachyOS consume it for FSR3.1→FSR4 on RDNA 3.5, the minimum version, and RDNA3.5's `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` companion; no-op/harm on non-FSR titles. Unverified upstream ⇒ FIX-to-remove; verified ⇒ KEEP + cite.
- **RADV unified heap (drirc removed):** confirm uma:1 on current Mesa; regression if not.
- **GTT sizing (ttm removed):** kernel auto-sizes (~62 GiB ceiling); README directs >62 GiB single allocations to the BIOS UMA carveout (≤96 GB), not deprecated `amdgpu.gttsize`; verify via `/sys/module/ttm/parameters/pages_limit`. `amd_iommu=off` does not change the ceiling.
- PROTON_ENABLE_WAYLAND=1 maturity/fallback; AMD_VULKAN_ICD=RADV vs VK_DRIVER_FILES; shader-cache pair sizing; MANGOHUD=1 global overhead vs per-launch, clean with gamescope/GameMode.
- **XDNA NPU note (NEW):** the NPU is blacklisted by default (§8/§10) — no gaming impact (no title uses XDNA), but any future LLM/NPU workload on this host requires the documented opt-in. State it once here; evaluate in §10.
- Sources: docs.mesa3d.org (RADV, APU heap), gitlab.freedesktop.org/mesa + drm/amd, github Proton/Proton-CachyOS (FSR4, ntsync), amd.com ROCm, wiki.cachyos.org.

### 6. CPU performance and power

amd_pstate=active; governor **powersave** (`CPUPOWER_GOVERNOR`, sourced from `/etc/default/cpupower-service.conf` — the generated file's own header now names the consumer: `/usr/lib/systemd/scripts/cpupower`, cpupower.service); EPP **balance_performance** via udev (`ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*"`, ATTR value from the hoisted **`$EPP_PREFERENCE`** global, enum-gated against `_RY_EPP_LEVELS`); **GPU_DPM_LEVEL=auto** (add-only udev rule, matcher FIXED v7.94/95: `ENV{DEVTYPE}=="drm_minor"` — the prior bare-`DEVTYPE` rule NEVER APPLIED); verify expectation `EXPECTED_SCALING_DRIVER=amd-pstate-epp` hoisted. Masked: power-profiles-daemon, ananicy-cpp, modemmanager. `dynamic_epp=disabled` asserted.

- **governor=powersave + EPP=balance_performance (LIVE, special case):** under amd_pstate=active, `powersave` honors EPP (dynamic), `performance` pins max and ignores it — this triple IS the documented EPP-honoring max-perf config on Zen 5; do not flag the governor. The `balance_performance`→`performance` EPP question stays **UNCERTAIN** (no gfx1151/Zen-5 frametime comparison); if evidence appears, the change is the `EPP_PREFERENCE` global only (now enum-gated — `performance` is in `_RY_EPP_LEVELS`, so the change is validator-clean).
- **GPU_DPM_LEVEL=auto:** accepted set `_RY_DPM_LEVELS` (auto low high manual profile_standard profile_min_sclk profile_min_mclk profile_peak perf_determinism), enum-gated. Re-evaluate `auto` vs `high` for frametime/1%-lows on the shared package budget — and note BOTH inertia facts: the rule is add-only (no re-assert after GPU reset) AND it only began applying at all with the v7.94/95 matcher fix, so any pre-fix "high made no difference" observation is void (the rule wasn't firing). State which package budget (85 W README BIOS ceiling vs 140 W stock) the arbitration analysis assumes.
- Confirm the EPP rule live-applies (`_post_udev`: `udevadm verify` when systemd ≥254, then reload + retrigger block AND cpu) and the GPU rule now matches at enumeration on this iGPU.
- prefcore=enabled + boost=1 correct on Strix Halo; `dynamic_epp=disabled` node exists ≥6.16 (6.19 floor guarantees).
- Mask power-profiles-daemon / ananicy-cpp / modemmanager: confirm each remains safe and beneficial on current CachyOS.
- Sources: docs.kernel.org pm/amd-pstate, wiki.archlinux.org/CPU_frequency_scaling + AMDGPU, freedesktop ppd.

### 7. Memory and storage

zswap.enabled=0; NVMe scheduler none (udev, sorts after vendor 60-ioschedulers.rules; NVMe-rule comment now states the divergence: "diverges from CachyOS kyber default"); **SYSCTL_VALUES (9, unchanged):** net.core.default_qdisc=fq, net.core.netdev_budget=600, net.core.netdev_budget_usecs=5000, net.ipv4.tcp_congestion_control=bbr, net.ipv4.tcp_notsent_lowat=16384, net.ipv4.tcp_slow_start_after_idle=0, vm.compaction_proactiveness=0, vm.max_map_count=2147483642, vm.swappiness=150 (priority 95, after vendor 70-cachyos-settings.conf); fstab ext4 noatime,lazytime,commit=10; THP/KSM/systemd-oomd left to CachyOS; vm.page-cluster + vm.vfs_cache_pressure remain dropped (vendor duplicates).

- **NEW source conflict — the SYSCTL trailing comment says `netdev=2.5GbE`** while the platform is dual 10 GbE (RTL8127). Values unchanged; the label is new. Resolve: stale comment (RTL8127 is 10 GbE — cite lspci/driver) OR the 600/5000 budget pair was actually sized for 2.5 GbE, in which case the 10 G netdev_budget validation below is no longer a formality. Flag; do not silently adopt.
- **Vendor-duplicate drop still a no-op?** Confirm CachyOS 70-cachyos-settings.conf sets vm.page-cluster=0 + vm.vfs_cache_pressure=50; list exact vendor values (a differing vendor default makes the drop a silent change).
- zram pair: swappiness>100 accepted; 150 on 128 GB — gratuitous or LLM-reclaim-helpful? zswap=0 vs CachyOS zram (no double compression).
- NVMe none vs mq-deadline/kyber current best practice; `nr_requests`/`read_ahead_kb` not set — propose concrete ATTRs only with game-load/LLM-read evidence, else state defaults optimal. Confirm 99- sorts after 60- (last ATTR wins).
- fstab noatime+lazytime coexistence; commit=10 durability with every-boot forced fsck (§3); fstrim.timer over continuous discard.
- vm.max_map_count (MAX_INT−5, SteamOS value) Proton/anti-cheat sufficiency; compaction_proactiveness=0 for gaming + large unified allocs; oomd stays disabled on 128 GB.
- Sources: docs.kernel.org (block, sysctl/vm), wiki.archlinux.org/Zram + SSD + Ext4, wiki.cachyos.org.

### 8. Network and latency

sysctl net values unchanged (§7). IPv6 disabled system-wide (§3); nftables IPv4-only (§10). NM: wifi.backend=wpa_supplicant, wifi.powersave=2 (off), logging WARN. **Modprobe surface MERGED (v7.99.0): `/etc/modprobe.d/60-ry-modules.conf`** = `options mt7925e disable_aspm=1` + conditional `blacklist amdxdna` (default on). resolved: MulticastDNS=no, LLMNR=no, DNSOverTLS=no, DNSSEC=allow-downgrade (plaintext; diverges from CachyOS DoH default — the drop-in's own header now states the divergence). regdom COUNTRY=US (`/etc/iw-regdomain`; reserved ISO codes rejected). Masked: NetworkManager-wait-online, modemmanager, **avahi-daemon.service + .socket (NEW v7.96/97)**. Enabled: NetworkManager.

- **avahi masked (NEW):** in-script rationale "second mDNS responder collided with resolved". With resolved mDNS=no this closes multicast discovery entirely. Confirm no host dependency (printer discovery, `.local` peers) and that unit+socket is the complete closure (no D-Bus activation path resurrects it).
- **mt7925e disable_aspm=1 (relocated, value unchanged):** confirm still the correct MT7925 mitigation (coredump/BT-reconnect/assoc) and whether an upstream mt76 fix has landed (→ prefer a floor, flag the option removable — the file comment itself says "drop when upstream fixes"); redundancy vs `pcie_aspm.policy=performance` on this device.
- **amdxdna blacklist (NEW, gated):** confirm `-EINVAL (ret -22)` is the real probe failure under `amd_iommu=off` on current kernels; blacklist-vs-alternatives; the coupling refusal (`BLACKLIST_AMDXDNA=false` + `amd_iommu=off` ⇒ exit 3) is the right fail-closed shape. **Leftover check SETTLED (second pass): the superseded filenames have ZERO in-script references** — no verify/deploy path surfaces a pre-7.99 `60-ry-mt7925e.conf` or `60-ry-blacklist-amdxdna.conf`; the README `sudo rm` note is the ONLY guard. ADD-check (Low/Low) stands as a **confirmed gap**: a stale `60-ry-blacklist-amdxdna.conf` keeps the NPU blacklisted after a user opts back in, and nothing detects it.
- NM backend wpa_supplicant vs iwd parity today; wifi.powersave=2 sufficiency for mt76 latency; no dangling iwd reference (iwd opt-in intact).
- bbr + fq current; BBRv3 status. Dual 10 GbE: netdev_budget/usecs sizing (now entangled with the §7 comment conflict); tcp_rmem/wmem or ring tuning for line rate.
- tcp_notsent_lowat=16384, tcp_slow_start_after_idle=0 rationale holds.
- mDNS off + plaintext DNS: confirm CachyOS ships an encrypted-DNS default this overrides; flag the privacy reduction (§10). Basename-override caution (§B5/B6) stands.
- regdom US fixed: TX power/channel set for MT7925 on current wireless-regdb; 6 GHz AFC posture; non-US hand-edit; "3 dBm readout cosmetic" README claim.
- Sources: docs.kernel.org/networking, wiki.archlinux.org/Sysctl + NetworkManager + Wireless + IPv6, git.kernel.org wireless-regdb + mt76, gitlab.freedesktop.org (avahi), man.archlinux.org avahi-daemon.

### 9. systemd units, time-sync

Mask (**12**): ananicy-cpp, power-profiles-daemon, NetworkManager-wait-online, ufw, modemmanager, **avahi-daemon.service, avahi-daemon.socket (NEW v7.96/97)**, sleep/suspend/hibernate/hybrid-sleep/suspend-then-hibernate targets. Enable (5): fstrim.timer, NetworkManager, cpupower, nftables, bluetooth. Not enabled: systemd-oomd (intentional), NetworkManager-dispatcher (socket-activated), rtkit-daemon (socket-activated). iwd.service untouched (opt-in). ufw flushed after nftables live. **NTP remediation is UNCONDITIONAL (v7.96/97 — `RY_NO_NTP_REMEDIATION` REMOVED)** with a NEW conflict guard: `_ry_check_time_sync` scans `chronyd.service`/`ntpd.service` first and REFUSES to enable systemd-timesyncd when either is enabled/active ("two NTP clients would conflict; repair <unit> manually" — warn, return 1); otherwise an unsynced clock enables timesyncd, re-checks after 2 s, and on sync runs `_ry_rtc_writeback` (`hwclock --systohc --utc`; skipped with an INFO when `RTCInLocalTZ=yes` — defer to systemd — or hwclock absent).

- For each mask, confirm safe and beneficial on CachyOS: ananicy-cpp + ppd (§6); modemmanager (no cellular HW); avahi pair (§8 — printer/`.local` dependency check); sleep/suspend masked = no suspend at all (always-on mini-PC).
- **NTP conflict guard (NEW):** the prior "does auto-enable fight an existing client" concern is resolved by design for chrony/ntpd. Confirm the two-unit scan is sufficient — `ntpsec` ships `ntpd.service` (covered); `openntpd` ships `openntpd.service` (NOT scanned — flag as a gap if openntpd is plausible on CachyOS, else note-and-close). Confirm the removal of the opt-out env is acceptable (remediation is warn-path-only and non-fatal; a user who wants no timesyncd can mask it — state that escape).
- **RTC write-back:** confirm `--systohc --utc` safe; the `RTCInLocalTZ=yes` defer branch correct; skewed-RTC-poisons-`Persistent=true`-timer-stamps rationale real; no ownership conflict with timesyncd.
- bluetooth.service enabled (main.conf §12): AutoEnable=true posture + BlueZ key currency.
- nftables enabled as the firewall; ufw-flush-then-mask leaves no unfirewalled window (mask skipped if nft not confirmed live); nftables.service oneshot judged by live ruleset (`_vrsv_chk_nftables`), and the managed file is now `nft -c`-gated at deploy AND at `_post_nft` reload (v7.96/97).
- fstrim.timer vs continuous discard; cpupower vs CachyOS freq management (the generated conf's header now names the consumer script — §B8); oomd stays disabled; dispatcher/rtkit stay socket-activated.
- logind Handle*Key=ignore (8 keys incl LongPress): intended, no lockout risk.
- Sources: man.archlinux.org (systemd.unit, logind.conf, hwclock, systemd-timesyncd, avahi-daemon), wiki.archlinux.org (Bluetooth, System time, Avahi), wiki.cachyos.org.

### 10. Security and safety (cross-cutting)

nftables **IPv4-only** default-deny-inbound (ufw masked; `ipv6.disable=1` §3): input policy drop, loopback accept (first), ct established/related accept, ct invalid drop, IPv4 ICMP `{ echo-request, destination-unreachable, time-exceeded, parameter-problem }` accept (inbound ping ALLOWED), forward drop, output accept. No ICMPv6/NDP rules. `RY_REMOTE_PLAY_PORTS` gate (default false) appends TCP `{ 47984, 47989, 48010, 27036, 27037 }` + UDP `{ 47998-48010, 27031-27036 }`. **amd_iommu=off (AMD-Vi DISABLED) — now with the amdxdna/NPU consequence made explicit and validator-paired.** clearcpuid=umip (UMIP off, renamed). split_lock_detect=off. **NEW hard gate: the rendered ruleset must pass `nft -c -f` before commit; `_post_nft` re-validates before reload (v7.96/97).**

- **amd_iommu=off (the #2 open reduction):** quantify DMA-isolation loss (USB4/Thunderbolt, NVMe, NIC). NEW: the trade now has a named functional cost — the **XDNA 2 NPU is blacklisted** (amdxdna `-EINVAL` without the IOMMU). The opt-back-in is a single validator-enforced pair (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`) restoring DMA isolation AND the NPU together — confirm the coupling direction is complete (can a user set `amd_iommu=on` while leaving `BLACKLIST_AMDXDNA=true`? Yes — blacklist-with-IOMMU is valid, only false-without-IOMMU refuses; confirm that asymmetry is intended). ROCm on gfx1151 unaffected.
- **IPv6 disabled + inbound ping accepted:** quantify both directions; the `_ir_validate_keys` coupling (nftables managed ⇒ `ipv6.disable=1` required) holds. NEW counterweight: avahi masked (unit+socket) removes the second mDNS responder — net LAN-discoverability delta now = +ping −mDNS; state it.
- **RY_REMOTE_PLAY_PORTS set:** validate TCP 47984/47989/48010 (Sunshine), 27036/27037 (Steam Remote Play) + UDP ranges against current Sunshine/Moonlight/Steam; flag missing/stale; default-OFF right; the gated block now carries its own marker comment (§B3). **Toggle mechanics (second pass): this and every other profile toggle (`BLACKLIST_AMDXDNA`, `NM_WIFI_BACKEND`, `COUNTRY`, `GPU_DPM_LEVEL`, `EPP_PREFERENCE`) is an EMBEDDED script scalar with NO `set -q` env guard — the unconditional `set -g` clobbers any exported environment value.** Opting in means editing the script, not exporting a variable; the ONLY runtime env inputs are `RY_RUN_TIMEOUT`, `RY_INSTALL_SKIP_HARDWARE_CHECK`, `NO_COLOR`. Confirm the env-proof-determinism posture is intended and documented.
- **Inbound ping is a REGRESSION GUARD:** `_vss_nft` hard-fails if `echo-request` accept is MISSING; `_vrsv_nft_assert_ping` warns live. `destination-unreachable` accept preserves IPv4 PMTUD.
- **`nft -c` gate (NEW):** confirm the pre-commit check + post-hook validate close the malformed-ruleset window, and that a `nft -c` pass guarantees `nft -f` load on the same nft version (same binary — yes; state it). Restart failure leaves the validated file to apply at boot — confirm that degraded path is surfaced (warn) not silent.
- `flush ruleset` blast radius: wiping ALL nft tables vs docker/libvirt/podman on this host.
- clearcpuid=umip headline reduction (§3); split_lock_detect=off residual exposure; `ct state invalid drop` ordering after lo+established cannot drop valid traffic.
- Produce the ordered security-delta (above): umip #1, amd_iommu=off+NPU #2, IPv6-off/ping-on/avahi-masked #3.
- Sources: wiki.archlinux.org (nftables, Security, IOMMU, IPv6), docs.kernel.org (split lock, UMIP, AMD-Vi), github Sunshine/Moonlight, man.archlinux.org nft(8).

### 11. Known issues and DKMS currency

MES page faults → the label "MES 0x86" is now **script-only** (README Known-Issues table DROPPED v7.98.x; §1 headline conflict). RTL8127 throughput + suspend/shutdown hang → in-tree r8169 (`f24f7b2f3af9`, `ae1737e7339b`) — the script now states these land **below** the 6.19 floor (direction flipped; verify the actual landing kernels, §1); no DKMS. MT7925 panics/deauth/coredump → mitigated via `disable_aspm=1` (now in the merged `60-ry-modules.conf`) + `btusb.enable_autosuspend=n` + wpa_supplicant; the file's own comment says "drop when upstream fixes" — check whether that has happened. **amdxdna probe failure (NEW known-issue surface):** under `amd_iommu=off` the XDNA driver probes `-EINVAL (ret -22)` every boot; the profile blacklists it (default) rather than shipping the error — confirm the errno/ret pair against current amdxdna, and that no future kernel makes the NPU IOMMU-optional (which would obsolete the blacklist). Strix Halo ACP → still open upstream (no ACP70 internal-mic ASoC machine driver / UCM profile as of mid-2026); internal mic undetected; nothing the profile can ship; installer does not surface it. Install pacman-only.

- **MES firmware — RESOLVE THE LABEL (script-only now).** Determine the shipping gfx1151 GC 11.5.1 MES revision vs the script's "0x86"; report disagreement + trusted source (git.kernel.org/kernel-firmware, ROCm #5724, Launchpad #2129150).
- **RTL8127 floor claim flipped — verify.** Prior: fixes guaranteed ≥6.19. Now: "already present below this floor." Cite the exact mainline releases carrying `f24f7b2f3af9` and `ae1737e7339b`; if either actually landed at 6.19+, the script's new claim is wrong (FIX-doc) and the floor rationale quietly regains a second leg.
- **MT7925:** fixes landed 6.17+, covered by the floor; drop-in stays defensive; wpa_supplicant vs iwd wash. Improving, not closed.
- **ACP/internal mic:** still open; alsa-ucm-conf has no acp70 profile. Document as a known gap; upstream board report remains the action.
- Recommend a kernel/firmware floor over DKMS for any landed fix.
- Sources: gitlab.freedesktop.org/drm/amd, git.kernel.org linux-firmware + r8169 + mt76, bugzilla.kernel.org, discuss.cachyos.org, docs.kernel.org accel/amdxdna.

### 12. MangoHud, Bluetooth, and hygiene

**MangoHud.conf (19 active + 1 commented, 0600, fps/frametime first) — composition UNCHANGED from v7.91.0:** horizontal, legacy_layout=0, position=top-left, toggle_hud=Shift_R+F12, fps, frametime, frame_timing, gpu_stats, gpu_temp, gpu_core_clock, gpu_power, cpu_stats, `# cpu_temp intentionally disabled — enable if you want CPU temperature in the HUD` (**comment TEXT expanded** — was bare `# cpu_temp`), cpu_mhz, cpu_power, vram, ram, font_size=20, text_outline, background_alpha=0.4. Enabled via MANGOHUD=1. bluetooth main.conf: FastConnectable=true, AutoEnable=true, ReconnectAttempts=3 (BlueZ default backoff). USER_DESTINATIONS = 2.

- **`cpu_power` remains the live-verification target:** confirm it populates from the Zen 5 RAPL/`power1_average` hwmon on the installed MangoHud + ≥6.19 kernel under Wayland; blank/zero ⇒ FIX-to-investigate. `cpu_temp` stays dormant (re-enabling is a user edit that re-trips MangoHud #1794 and may need the sensor key — the expanded in-file comment now partially serves the ADD-doc note the prior audit requested; confirm whether a #1794 warning belongs in the comment or README).
- **Byte-exact validation note:** any checksum-class comparison of the MangoHud body must use the EXPANDED comment string; `grep -c '^# cpu_temp'` (prefix) still returns 1.
- Confirm all 19 active directives valid for current MangoHud; gpu_power + cpu_power each populate from their sensor paths; frame_timing (graph) vs frametime (numeric) both current; toggle_hud=Shift_R+F12 valid; gpu_temp/gpu_core_clock/vram/cpu_mhz populate from amdgpu under Wayland; vram+ram right for unified memory; overhead near-zero with gamescope/GameMode.
- **Bluetooth main.conf:** BlueZ keys/sections current; ReconnectAttempts=3 + default backoff sane; AutoEnable=true fixes adapter-off-at-boot; FastConnectable=true no meaningful downside; complementary (not redundant) with `btusb.enable_autosuspend=n`.
- **Naming FIX RESOLVED (v7.92/93):** `PROFILE_NAME=gtr9_pro` — the v7.91.0 "gtr_pro vs GTR9 Pro" finding is closed; drop it.
- **Removed surfaces (do NOT audit):** baloofilerc, `_kb_*` family, `_ry_check_umip_disabled` (lineage 6). The stale-description ledger moved to §C (3 fixed, 1 new).
- Sources: github flightlessmango/MangoHud (config ref, #1794, #1825), wiki.archlinux.org/MangoHud + Bluetooth, man.archlinux.org bluetooth main.conf.

## 13. Candidate enhancements (absent knobs — gaming-first; ADD-opt-in vs KEEP-omitted)

Additive: each item is a knob the profile does NOT set. Anchor every call to gfx1151 / Zen 5 / RDNA 3.5 / current Mesa+Proton-CachyOS. Reserve ADD-as-default for a clear, low-risk frametime/throughput win. Never invent a flag — cite upstream or mark UNCERTAIN. (Carried from prior rounds; v7.92–7.99.1 added NONE of these — re-confirm calls against current upstream.)

### 13a. Kernel cmdline candidates

- **`mitigations=off` — KEEP-omitted.** Zen 5 unaffected by Inception/SRSO; no measured gaming benefit; residual mitigations HW/microcode-handled. Re-open as ADD-opt-in only on a published gfx1151 Proton frametime delta > ~2%. IMPACT Low · RISK Med (security).
- **`amdgpu.ppfeaturemask=0xffffffff` — KEEP-omitted.** Undervolt/OC not implemented on gfx1151 (overdrive/power-cap "Not supported", ROCm #5750); package cap shared. CPU undervolt via ryzenadj is the real lever (out of scope). IMPACT Low · RISK Med.
- **`preempt=full` — KEEP-omitted, redundant.** CachyOS desktop kernel boot-defaults full (`CONFIG_PREEMPT_DYNAMIC=y`); INFO-only check correct. IMPACT none · RISK none.
- **`nvme_core.io_timeout` / `pcie_port_pm=off` — KEEP-omitted.** Redundant beside ps_max_latency=0 + aspm.policy=performance. IMPACT Low · RISK Low.

### 13b. RADV / Mesa env candidates

- **`RADV_PERFTEST` — KEEP-omitted (gpl/sam) / UNCERTAIN (nggc).** gpl default-on since Mesa 23.1; sam auto-on when all VRAM CPU-visible (APU). nggc: no gfx1151 benchmark → UNCERTAIN, do not ADD on a guess. rtwave64 hurts RDNA2; ignore. IMPACT Low · RISK Low.
- **`RADV_DEBUG` correctness toggles — KEEP-omitted** unless a live gfx1151 rendering bug requires one; flag any open issue a toggle works around at audit time.
- **`MESA_VK_WSI_PRESENT_MODE` / `vblank_mode` — KEEP-omitted (per-game).** Document the per-title pattern.
- **`mesa_glthread=true` — KEEP-omitted.** GL-only benefit; Proton is Vulkan-dominant. IMPACT Low · RISK Low.

### 13c. DXVK / VKD3D-Proton candidates

- **dxvk.conf — KEEP-omitted (auto optimal).** GPL default-on; numCompilerThreads=0 auto-detects. Legacy DXVK_ASYNC superseded (gplAsyncCache removed in DXVK 2.7) — never recommend the old async patch. IMPACT Low · RISK Low.
- **Upscaler envs beyond §5 — KEEP-omitted.** PROTON_FSR4_RDNA3_UPGRADE=1 (+ per-title DXIL_SPIRV_CONFIG workaround) is the shipped scope; keep the rest per-title.
- **`VKD3D_CONFIG` — KEEP-omitted (per-game).** IMPACT Low · RISK Low.

### 13d. Firmware / platform (verify-only)

- **Resizable BAR / SAM — verify-only, auto-on.** All VRAM CPU-visible on Strix Halo; RADV auto-enables sam. Optional INFO via rocminfo pool sizes / lspci BAR / amdgpu dmesg. Low priority.
- **BIOS UMA carveout vs GTT — KEEP-omitted (gaming).** Default GTT (~62 GiB) never bottlenecks a game; carveout note is compute-oriented.
- **BIOS power ceiling (NEW context, verify-only):** the README now prescribes `SPL=fPPT=sPPT=85 W` + `TjMax 90` (gains flatten past ~85 W). Installer-external; the audit's only action is consistency — every §6/§13 power statement must name its assumed budget (85 W README vs 140 W stock), and any DPM=`high`/EPP=`performance` re-evaluation must use the 85 W case if the user follows the README. IMPACT doc-only · RISK none.

### 13e. Scheduler / memory (only the gaps)

- **`read_ahead_kb` / `nr_requests` — KEEP-omitted, defaults optimal** absent game-load/LLM-read evidence (§7). IMPACT Low · RISK Low.
- **`vm.max_map_count` — KEEP (sufficient).** MAX_INT−5 (SteamOS value) satisfies Proton/anti-cheat.
- **CPU isolation (`isolcpus`, `nohz_full`, `rcu_nocbs`) — KEEP-omitted (wrong here).** Hurts a 16C/32T gaming desktop (scheduler/thread-placement). IMPACT Low · RISK Med (if added).

For each 13a–13e item: ITEM · PRESENT?(no) · CALL(ADD-default / ADD-opt-in / KEEP-omitted) · IMPACT · RISK · EVIDENCE. Bias toward KEEP-omitted; the profile is intentionally lean.

---

## Scope and non-goals

- Recommendations only — do not emit a modified script.
- Out of scope: dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS, laptops, UKI, BIOS flashing (README link-out only).
- Per-game Proton tuning is secondary; prioritize system-wide config.
- **Reinstatement rule.** Items deliberately removed/disabled — do not recommend reinstating unless current upstream directly contradicts the rationale (then flag, not FIX): `amdgpu.ppfeaturemask`, `--country` flag, TTM/GTT cap, RADV drirc, MangoHud `fps_metrics`, `vm.page-cluster`/`vm.vfs_cache_pressure` (vendor-provided), ntsync autoload conf (assert-only), baloofilerc, the `_kb_*` subs + `_ry_check_umip_disabled`, ICMPv6/NDP rules (do NOT re-add without restoring IPv6), the linux-firmware version advisory, **`RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` (removed v7.98.x — do not recommend a floor bypass)**, **`RY_NO_NTP_REMEDIATION` (removed v7.96/97 — the escape is masking timesyncd)**, **`60-ry-mt7925e.conf`/`60-ry-blacklist-amdxdna.conf` as standalone files (merged v7.99.0)**, **`clearcpuid=514` numeric form (renamed v7.94/95 — never revert to the bit index)**. Live config to evaluate as KEEP-or-FIX-to-remove (not protected): PROTON_FSR4_RDNA3_UPGRADE, MangoHud gpu_power/text_outline/toggle_hud/cpu_power, `ipv6.disable=1`, inbound-ping accept, **`archlinux-contrib` (NEW — nothing invokes it)**, **`BLACKLIST_AMDXDNA=true` default (NEW — evaluate the NPU-off default, not the mechanism)**. `cpu_temp` stays a user opt-in, not a shipped directive.
- **IOMMU special case:** ships `amd_iommu=off`. Do NOT recommend `iommu=pt`/`amd_iommu=on` as a default unless ROCm on gfx1151 is proven to require it (it is not) OR a DMA-isolation requirement is established. The opt-in (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`) is per-user and validator-enforced.
- **IPv6/nftables special case:** ships `ipv6.disable=1` + IPv4-only ruleset accepting inbound ping. Do NOT flag ping-accept as a regression (asserted regression-guard); do NOT re-add ICMPv6/NDP without restoring IPv6.
- **Governor/EPP special case:** powersave + balance_performance is the EPP-honoring config under amd_pstate=active (`dynamic_epp=disabled` asserted) — do not flag powersave without proving `performance` would honor the hint.
- **GPU_DPM_LEVEL special case:** `auto` is deliberate. Do not flag without gfx1151 frametime/1%-low evidence for `high` under the shared package budget — and remember the rule only began firing at v7.94/95 (pre-fix observations are void).

---

# Deep-pass appendix (exact generated bodies + full verify surface)

The §1–§13 investigation is value-level. This appendix is artifact-level: the exact strings the script writes, the complete verify subsystem, and the install-phase model. Validate the **rendered file content**, not a paraphrase. Every block below is quoted from the generator functions in `ry-install.fish` v7.99.1 (UUIDs/joins resolved at runtime). **Global body change (v7.9x): every generator now emits a leading `#` header-comment line** — byte-exact/checksum comparisons must include it.

## A. Install-phase model (`_RY_PHASE_NAMES`)

Six ordered phases; recommendations must respect this sequence:

```
1 Preflight     _install_preflight     — _ir_* gates (counts=21, keys incl BLACKLIST_AMDXDNA + charsets/metachar, kernel floor NO-OVERRIDE, post-hooks, root UUID); mesa soft-floor (vercmp output-validated)
2 Packages      _install_packages      — mkinitcpio.conf pre-deployed (tagged pre-Syu seed log line) → pacman -Syu; PKGS_ADD re-marked -D --asexplicit post-Syu (orphan-proof vs Phase-4 -Rns); chwd Vulkan
3 Configuration _install_system_files  — render+deploy all managed files (atomic tmp+rename); format-validate before write; nftables.conf additionally nft -c pre-validated
4 Services      _install_configure_services — fstab rewrite + resolved + PKGS_DEL (-Rns) + mask (nft-first, then ufw flush; MASK 12 incl avahi pair) + iwd handoff + enable (EXPECTED_SERVICES span Phase 4/6) + regdom + NTP (always; chronyd/ntpd conflict guard) + RTC write-back
5 Boot          _install_rebuild_boot  — taint-gate → mkinitcpio -P + sdboot-manage gen/update (gated on boot-critical writes)
6 Finalize      _install_finalize      — user daemon-reload + paccache (-rk2 and -ruk0 separate) + NetworkManager restart
```

- Confirm the firewall handoff lives in Phase 4 (nftables live before ufw flushed/masked) and boot-critical regeneration (Phase 5) fires only when a `_RY_BOOT_CRITICAL_DSTS` member changed. Flag any recommendation moving a cmdline/mkinitcpio change outside the Phase-5 gate.
- `_RY_BOOT_CRITICAL_DSTS` (4): `/boot/loader/loader.conf`, `/etc/kernel/cmdline`, `/etc/sdboot-manage.conf`, `/etc/mkinitcpio.conf`. **`_RY_BACKUP_TARGETS` = the SAME 4 (derived, v7.96/97)** — all four get `.ry.bak` + post-write verify/restore (plus fstab during its rewrite). The prior "cmdline is regenerable so only 2 are backed" posture is superseded; confirm the derived-equality is asserted (count tripwire 4) so the sets cannot silently diverge.
- **`_RY_POST_HOOKS` (17 entries, 16 distinct tags):** `/boot/*|loader`, `/etc/kernel/cmdline|cmdline`, `/etc/sdboot-manage.conf|boot`, `/etc/mkinitcpio.conf|boot`, `*/resolved.conf.d/*|resolved`, `*/logind.conf.d/*|logind`, `*/NetworkManager-dispatcher.service.d/*|nmdispatch`, `*/NetworkManager/conf.d/*|nm`, `/etc/iw-regdomain|regdom`, `/etc/bluetooth/main.conf|bluetooth`, `/etc/nftables.conf|nft`, `/etc/default/cpupower-service.conf|cpupower`, `*/sysctl.d/*|sysctl`, `/etc/udev/rules.d/*|udev`, `*/modprobe.d/*|modprobe`, `*/environment.d/*|envd`, `*/MangoHud/MangoHud.conf|mangohud`. `_ir_validate_post_hooks` refuses deploy on any tag lacking `_post_<tag>` (empty-tag entries also refuse). **Dispatch mechanics (second pass): `_post_hook_for_target` is FIRST-MATCH-WINS by list order** — pattern ordering is load-bearing for overlapping globs; a recommendation reordering or adding an entry must preserve it. The boot family shares one body, **`_post_boot_apply <target> <skip_mki>`** (taint gate + cascade + entry verify + sanity): `_post_boot` passes `skip_mki=false` (full `mkinitcpio -P`), while `_post_cmdline`/`_post_loader` pass `true` (sdboot regen only — cmdline/loader.conf are not initramfs inputs). Notify-only handlers state their rationale in-code: `_post_logind` (restarting logind kills all sessions → reboot), `_post_modprobe` (load-time options can't live-apply to a loaded module → reboot), `_post_envd`/`_post_mangohud` (session/next-launch). **`_post_nm` DEFERS the NetworkManager restart when WiFi is the active route** (don't cut the session mid-run) — confirm the deferred restart still lands (Phase 6 restarts NM) and the deferral is surfaced. `--install-file` values and ALL managed destinations are canonicalized via `realpath -m` at load (`_RY_CANON_SYSTEM_DSTS`/`_RY_CANON_USER_DSTS`, index-aligned; realpath-fail warns and falls back to the literal path) — managed-file matching is canonical-path based, so a symlinked alias of a destination resolves correctly. Unmatched patterns log `POST_HOOK_NONE` (file deployed, live-apply skipped) vs `POST_HOOK_SKIP_UNCHANGED` (bytes identical); `_post_udev` runs `udevadm verify` (systemd ≥254) before reload+retrigger (block AND cpu).

## B. Exact rendered file bodies (validate content, not summary)

### B1. `/etc/kernel/cmdline` + `/etc/sdboot-manage.conf` + `/etc/mkinitcpio.conf`

```
rw root=UUID=<_ROOT_UUID> 8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0
```
sdboot-manage.conf (header comment first, then keys in this order):
```
# sdboot-manage configuration — changes require: sudo sdboot-manage gen && sudo sdboot-manage update
LINUX_OPTIONS="<KERNEL_PARAMS join>"
LINUX_FALLBACK_OPTIONS="quiet"
DEFAULT_ENTRY="manual"
REMOVE_EXISTING="yes"
OVERWRITE_EXISTING="yes"
REMOVE_OBSOLETE="yes"
```
mkinitcpio.conf:
```
# mkinitcpio configuration — changes require: sudo mkinitcpio -P && sudo sdboot-manage update
MODULES=(amdgpu)
BINARIES=()
FILES=()
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
COMPRESSION="zstd"
COMPRESSION_OPTIONS=(-1 -T0)
```
- **`clearcpuid=umip` ships in BOTH bootloader paths** (`/etc/kernel/cmdline` + `LINUX_OPTIONS`). Confirm the two are not simultaneously active in conflict; state which CachyOS drives and whether maintaining both is redundant or a divergence risk.
- `LINUX_FALLBACK_OPTIONS="quiet"` strips ALL params from the fallback entry: a fallback boot runs kernel-default IOMMU (AMD-Vi ON → amdxdna would probe... and hit its blacklist, which is a modprobe.d file, still active — the NPU stays off in fallback; note the asymmetry inverts vs the main entry) AND IPv6 ENABLED with the IPv4-only ruleset not covering it. Confirm the fallback-only IPv6 exposure window is accepted or flag it.
- mkinitcpio now emits explicit `BINARIES=()`/`FILES=()` — byte-exact validation includes them; the KEY set is otherwise unchanged.

### B2. `/boot/loader/loader.conf`
```
# systemd-boot loader configuration
default @saved
timeout 0
console-mode keep
editor no
```
- `@saved` + timeout 0 + editor no: confirm `@saved` resolves; failed-boot menu reachability vs live-USB recovery (README posture). loader.conf changes regenerate sdboot entries only.

### B3. `/etc/nftables.conf` (exact IPv4-only ruleset — validate rule-by-rule)
```
#!/usr/bin/nft -f
# ry-install: default-deny-inbound, IPv4-only (ufw masked; ipv6.disable=1). Add inbound ports below.
flush ruleset
table inet filter {
    chain input {
        type filter hook input priority filter; policy drop;
        iif "lo" accept
        ct state established,related accept
        ct state invalid drop
        # IPv4 ICMP: inbound ping (echo-request) + error/PMTUD types (replies match ct established)
        icmp type { echo-request, destination-unreachable, time-exceeded, parameter-problem } accept
        # [IFF RY_REMOTE_PLAY_PORTS=true — the gated block ships with its own marker comment:]
        # ry-install: remote-play inbound (RY_REMOTE_PLAY_PORTS=true)
        tcp dport { 47984, 47989, 48010, 27036, 27037 } accept
        udp dport { 47998-48010, 27031-27036 } accept
    }
    chain forward { type filter hook forward priority filter; policy drop; }
    chain output { type filter hook output priority filter; policy accept; }
}
```
- **Deploy-time `nft -c -f <tmpfile>` gate (NEW v7.96/97):** a rendered ruleset failing syntax check refuses deploy with live+installed unchanged (`NFT_PREVALIDATE_FAIL`); `_post_nft` re-validates the installed file before `systemctl restart nftables` and downgrades a restart failure to "applies at next boot" (warn). Confirm the check-then-load pair is same-binary-consistent and the degraded path is surfaced.
- IPv4-ONLY: no `ip6`/`icmpv6` types; safe only under the `ipv6.disable=1` coupling (holds). Rule order lo → established/related → invalid-drop cannot drop valid loopback/established. `echo-request` accept is the regression guard; `destination-unreachable` preserves PMTUD; no echo-reply rule (ct established covers). `flush ruleset` blast radius vs docker/libvirt/podman. No ICMP/new-conn rate-limit — acceptable on a trusted LAN?

### B4. udev `99-ry-perf.rules` (exact — 3 rules + per-rule comments; GPU matcher FIXED v7.94/95)
```
# ry-install: udev performance rules (managed file, do not edit by hand)
# NVMe scheduler none (lowest tail latency; diverges from CachyOS kyber default)
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
# AMD P-State EPP balance_performance (perf-leaning CPPC hint)
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="<EPP_PREFERENCE>"
# GPU performance level (gfx1151 clock-floor; optional)
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="<GPU_DPM_LEVEL>"
```
- **GPU rule: bare `DEVTYPE==` → `ENV{DEVTYPE}==` (v7.94/95).** Bare `DEVTYPE` is not a udev match key; the prior rule NEVER APPLIED (net-nil only because auto = kernel default). Confirm the rule now matches at enumeration; it remains `add`-only (no re-assert after GPU reset — a robustness argument for `auto`).
- **EPP rule value from `$EPP_PREFERENCE`** (enum-gated `_RY_EPP_LEVELS`; unquoted ATTR interpolation bounded — same pattern as `GPU_DPM_LEVEL`/`_RY_DPM_LEVELS`). Rule matcher unchanged since the v7.85 fix; `_post_udev` retriggers cpu+block so it live-applies; `udevadm verify` gates the reload on systemd ≥254.
- Filename 99- sorts after vendor 60-ioschedulers.rules (last ATTR wins); the NVMe comment now names the vendor divergence (kyber) — confirm CachyOS still defaults kyber.

### B5. `/etc/systemd/resolved.conf.d/99-cachyos-resolved.conf`
```
# systemd-resolved: plaintext DNS, mDNS/LLMNR off (diverges from CachyOS DoH default)
[Resolve]
MulticastDNS=no
LLMNR=no
DNSOverTLS=no
DNSSEC=allow-downgrade
```
- Same-basename replace (not merge) caution stands: if CachyOS ships its own `99-cachyos-resolved.conf`, this REPLACES it — confirm intended, not a clash a vendor update re-overwrites. Restarts skipped when bytes unchanged. The header now self-documents the DoH divergence (§10 privacy flag stands).

### B6. NetworkManager `99-cachyos-nm.conf` + dispatcher `logging.conf`
```
# NetworkManager configuration - wpa_supplicant backend
[device]
wifi.backend=wpa_supplicant

[connection]
wifi.powersave=2

[logging]
level=WARN
```
dispatcher: `# LogLevelMax drops info-level dispatcher lines (journald-logged; StandardError=null ineffective)` + `[Service]` + `LogLevelMax=notice`.
- Same basename-override caution as B5. Confirm LogLevelMax=notice remains the correct journald-noise fix.

### B7. `/etc/iw-regdomain` + persistence path
```
# ry-install: wireless regulatory domain (managed file, do not edit by hand)
COUNTRY=US
```
- **Persistence dependency (single most version-fragile external):** confirm `cachyos-iw-set-regdomain` (or its successor) still exists in current CachyOS and reads this file at boot; if dropped, the file is inert and the profile must switch mechanisms. Reserved/user-assigned ISO codes rejected at preflight.

### B8. `/etc/bluetooth/main.conf` + `/etc/default/cpupower-service.conf`
```
# ry-install: BlueZ daemon config (managed file, do not edit by hand)
[General]
FastConnectable=true

[Policy]
AutoEnable=true
ReconnectAttempts=3
```
cpupower-service.conf:
```
# cpupower-service.conf — sourced by /usr/lib/systemd/scripts/cpupower (cpupower.service)
GOVERNOR='powersave'
```
- **The consumer path is now stated in-file** (`/usr/lib/systemd/scripts/cpupower`). The prior open question narrows to a single verify: confirm that script path + `GOVERNOR` var name on current CachyOS cpupower packaging (if moved, the file is inert and the governor falls to kernel default; the udev EPP rule still applies). `_vrsv_chk_cpupower_governor` asserts the running governor.

### B9. `~/.config/MangoHud/MangoHud.conf` (exact — 19 active + 1 commented + 1 file-header, ordered)
```
# ry-install: MangoHud readout-only HUD (managed file, do not edit by hand)
horizontal
legacy_layout=0
position=top-left
toggle_hud=Shift_R+F12
fps
frametime
frame_timing
gpu_stats
gpu_temp
gpu_core_clock
gpu_power
cpu_stats
# cpu_temp intentionally disabled — enable if you want CPU temperature in the HUD
cpu_mhz
cpu_power
vram
ram
font_size=20
text_outline
background_alpha=0.4
```
- **Delta vs v7.91.0: the commented line TEXT only** (was `# cpu_temp`); composition/order identical. Prefix greps unchanged; byte-exact checks must use the new string. `cpu_power` remains the live target (§12); the file-header is not a directive.

### B10. `/etc/modprobe.d/60-ry-modules.conf` (NEW merged destination, v7.99.0)
```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# disable PCIe ASPM on MT7925 (coredump/BT-reconnect/assoc mitigation; drop when upstream fixes)
options mt7925e disable_aspm=1
# [IFF BLACKLIST_AMDXDNA=true (default):]
# blacklist amdxdna: XDNA NPU needs IOMMU, probes -EINVAL (ret -22) under amd_iommu=off
blacklist amdxdna
```
- Replaces `60-ry-mt7925e.conf` + the interim `60-ry-blacklist-amdxdna.conf`; pre-7.99 installs carry stale unmanaged copies until the README's one-time `sudo rm` — confirm whether any check surfaces the leftovers (§8; ADD-check candidate if silent). The conditional block renders ONLY under the default-true toggle; `BLACKLIST_AMDXDNA=false` (validator-coupled to the IOMMU being ON) yields a 3-line file. `_vss_modprobe` asserts the static content; `_vrkm_blacklist_modprobe` asserts amdxdna is NOT loaded (v7.98.x).

## C. Full verify subsystem (`--verify`) — orchestrator families

The top VERIFY block is the user-facing command set; the script's `--verify` runs orchestrators across sub-families. Recommendations that change a value MUST state which sub asserts it (hard-fail vs warn). Architecture re-derived from **v7.99.1** (289 fns). **v7.98.x additions marked ⊕; the v7.90.0 removals (⊘) remain removed.** The three v7.91.0-flagged stale `--description` strings are FIXED; ONE new one found (runtime-session, below).

**Static (on-disk content) — orchestrators + syntax + checksum:**
- `_verify_static_boot` → `_vsb_loader` · `_vsb_sdboot` (LINUX_OPTIONS token set + keys) · `_vsb_cmdline` (`/etc/kernel/cmdline` token set — `amd_iommu=off`/`ipv6.disable=1`/`clearcpuid=umip`/`tsc=reliable` byte-asserted here) · `_vsb_mkinitcpio` (HOOKS/MODULES **⊕ + LIVE `COMPRESSION=` value + `COMPRESSION_OPTIONS` tokens** via `_ry_mkinitcpio_array` — multi-line `KEY=( )` join, LAST assignment wins with a multiple-assignment warn) · `_vsb_entries` (generated BLS entries + count). All hard-fail static.
- `_verify_static_system` (description FIXED — now names exactly: resolved, logind, NM, regdom, bluetooth, cpupower-service.conf, sysctl, udev, modprobe, nftables) → `_vss_logind` · `_vss_nmdispatch` · `_vss_nm` · `_vss_sysctl` · `_vss_regdom` · `_vss_bluetooth` · resolved (inline `_chk_grep` loop) · cpupower-service.conf (inline `_chk_grep GOVERNOR`) · `_vss_udev` (all 3 rules; EPP value from `$EPP_PREFERENCE`; GPU_DPM_LEVEL-aware) · `_vss_modprobe` (**merged file: mt7925e `disable_aspm=1` + optional amdxdna blacklist**) · `_vss_nft` (hard-fail on missing `echo-request` — inbound-ping regression guard; IPv4-only).
- `_verify_static_user` — ENV_VARS (env.d, `_chk_grep` per var) + MangoHud (inline `_chk_file` + `_chk_grep "fps"` existence probe). ⊕ **`_chk_grep` is now comment-proof:** awk strips inline comments and skips comment-only lines before the `grep -wF` match — a commented-out `key=value` can no longer satisfy presence; "file has no non-comment lines" is a distinct FAIL; a mid-read sudo lapse is a distinct warn.
- `_verify_static_packages` → `_vsp_required` (PKGS_ADD **19** + the folded Vulkan pkgs, with the pacman-db-lock guard) · `_vsp_removed` · `_vsp_pacman_conf` (**⊕ sudo-read fallback** for a 0600-hardened pacman.conf; grep rc>1 → warn-skip; rc 1 + sudo-lapse → warn-skip — never a false "not set") · `_verify_static_services` (MASK 12 state) · `_verify_static_syntax` (live mkinitcpio HOOKS presence, multi-line tolerated) · `_verify_static_checksum` → `_vsc_check_one` per-file (embedded SHA256 == installed; graceful skip on `EXIT_GEN_NOUUID`). **Checksum note: every body now carries a header-comment line and mkinitcpio carries `BINARIES=()`/`FILES=()` — embedded and installed move together, but any out-of-band byte comparison must use the §B bodies.**
- **Config-format validators (`_ry_validate_configs` → `_rvc_dispatch` per destination, pre-deploy):** `_grep_kv` (loader/sdboot), `_grep_kparam` (cmdline), `_grep_sysctl_kv`, `_grep_modprobe_entry`, `_grep_regdomain_entry`, `_grep_udev_entry`, `_grep_nft_entry`, `_grep_envd_entry`, `_grep_cpupower_entry`, `_grep_mangohud_entry` (bareword OR key=value OR `#`-comment), `_grep_ini_header` fallback; the mkinitcpio case now REQUIRES `MODULES=(`, `HOOKS=(`, **⊕ and `COMPRESSION="`** lines. `_ry_validate_mkinitcpio_hooks` (leaves `_vmh_existence_only` + `_vmh_order_checks`) + `_ry_validate_mkinitcpio_modules` validate the arrays — `_vmh_*` remain mkinitcpio validators, NOT MangoHud (prior misattribution stays corrected). ⊕ nftables.conf additionally passes `nft -c -f` on the rendered tmpfile before commit (§B3).

**Runtime-kernel — `_verify_runtime_kparams`** (description FIXED — "/proc/cmdline, hardware state, module params, blacklist"):
- Preemption INFO scaffold: caches `sudo -n dmesg` once (`_RY_DMESG_LINES`/`_RY_DMESG_PREEMPT`), INFO-only, erased after.
- `_vrk_cmdline` — generic loop asserting EVERY KERNEL_PARAMS token + `rw` in `/proc/cmdline` (auto-asserts `clearcpuid=umip` post-rename) + preemption INFO.
- `_vrk_gpu_state` — `power_dpm_force_performance_level == $GPU_DPM_LEVEL` across `card*` (**⊕ comparison QUOTED** — empty sysfs reads can no longer mis-evaluate).
- `_vrk_cpu_state` — scaling_driver=`$EXPECTED_SCALING_DRIVER` · scaling_governor=`$CPUPOWER_GOVERNOR` · EPP=`$EPP_PREFERENCE` · amd_pstate status · `dynamic_epp=disabled` · prefcore · boost=1 (expectations now hoisted globals, ⊕).
- `_vrk_module_state` → `_vrkm_amdgpu` (hex-aware; no-ops without `amdgpu.*`) · `_vrkm_blacklist` (`module_blacklist=` cmdline scan — currently no-op, no such token) · **⊕ `_vrkm_blacklist_modprobe` (NEW): parses `blacklist <mod>` entries from the MANAGED modprobe.d content (generator-sourced), normalizes `-`→`_`, `lsmod`-checks each — amdxdna LOADED ⇒ FAIL; `lsmod` absent ⇒ warn; generator failure defers to checksum verify**; + usbcore.autosuspend, nvme_core ps_max_latency, zswap.enabled, nmi_watchdog, NVMe `[none]`. ⊘ `_vrkm_iommu`, `_vrk_clocksource` remain removed (directives still cmdline-asserted).

**Runtime-services — `_verify_runtime_services`:** `_vrsv_chk_active_enabled` · `_vrsv_nft_assert_ping` (live input chain accepts inbound IPv4 ping; warn-only) · `_vrsv_chk_nftables` (oneshot judged by live ruleset policy-drop) · `_vrsv_chk_resolved` · `_vrsv_chk_cpupower_governor` · `_vrsv_sys_units` · `_vrsv_wifi_nm_backend` · `_vrsv_wifi` · `_vrsv_masked_inactive` (now covers the avahi pair). **SETTLED (second pass): `_vrsv_wifi_iwd_proc` is a genuine REMOVAL, not a fold** — `_vrsv_wifi` contains no iwd path at all: it skips entirely when `_RY_PROFILE_USES_WIFI_BACKEND=false` (NM not managed), detects the wlan iface via `/sys/class/net/*/wireless`, calls `_vrsv_wifi_nm_backend`, reads nmcli radio/device state, and closes with a **firewall-posture INFO line (`ufw` active-state + live nft rule count via `# handle` grep; `n/a`=nft absent, `unknown`=sudo lapse)**. Residual iwd coverage = the backend compare only (an iwd-configured host surfaces as an NM backend mismatch); an iwd opt-in host has LOST the process cross-check — flag as a narrow, deliberate coverage reduction (Low/Low) rather than an open question.

**Aggregation (`_ry_verify_all` / `_verify_summary`):** static runs first (`_ry_verify_static`: boot → system → user → packages → services → syntax → checksum, then `_verify_summary`); runtime second; each stage prints its own `_verify_summary` (Results: N OK[, N WARN][, N FAIL][, N GEN_FAIL] from the VERIFY_OK/FAIL/WARN/GEN_FAIL counters) and `_ry_verify_all` sums the counters for the combined footer. **A runtime preflight bail (sudo lapse) restores the static totals, and a static FAIL outranks the runtime bail code** — the exit reflects the worse finding. Confirm no path zeroes the static counters after a runtime bail (the snapshot/restore is explicit).

**Runtime-env — `_verify_runtime_env`** (description FIXED — "ENV_VARS, sysctl, fstab, ntsync, regdom runtime"): `_vre_envvars` (`systemctl --user show-environment`) · `_vre_sysctl_runtime` (`/proc/sys`) · `_vre_fstab` (ext4 `noatime,lazytime,commit=10`) · `_vre_ntsync` (state dispatch — survives) · `_vre_regdom` (`iw reg get`). ⊘ `_vre_tcp`/`_vre_zram` remain removed.

**Runtime-session — `_verify_runtime_session`:** `_vrs_nm_perms` (system-connections 0600 root:root) · `_vrs_installed_file_perms` (system 0644 / user 0600) · `_vrs_parent_dirs` → `_vpd_dir_perm_check` (0755 system / 0700 user). **`_vrs_vfat_skip` guards BOTH loops** (installed-file perms AND parent-dirs — vfat/undetermined `$BOOT` paths are counted-skipped with an INFO, not silently passed). **⊕ NEW FIX-doc (Low/Low): the orchestrator's `--description` still ends "…, Vulkan packages"** — the body calls no Vulkan check (folded into `_vsp_required` v7.90.0). Same drift class as the three fixed strings, one release behind; trim it.

Actionable for §C:
- Confirm `_vrkm_blacklist_modprobe` closes the amdxdna live gap correctly: generator-sourced (so it checks the INTENDED blacklist, not a possibly-stale on-disk file) — which means a pre-7.99 leftover drop-in is invisible to it (§8/§B10 — a CONFIRMED gap, second pass); confirm the `-`→`_` lsmod normalization and the warn-on-no-lsmod are right.
- Confirm the live COMPRESSION/_OPTIONS compare tolerates vendor multi-line arrays (join) and duplicate assignments (last-wins warn) without false FAILs.
- Comment-strip false-negative closure (settled): no managed value contains ` #` and the boot-scalar metachar gate FORBIDS `#` — the greedy strip cannot bite a legitimate value; state the closure, re-check if a future value adds `#`.
- Confirm the removed effect-asserts still leave no coverage gap that matters (directive-level coverage per lineage 5); the iwd narrowing above is deliberate and flagged.
- Trim the runtime-session description (FIX-doc); re-scan all `--description` strings for the same class after any future sub removal.
- `_vsb_entries` canonicalization: loader-entry `linux` paths resolve via `realpath -m` with a WARNED textual-join downgrade when realpath is absent ("symlink escapes undetectable") — confirm the downgrade is loud enough and realpath's presence is effectively guaranteed by the optional-tools warn list.

## D. fstab rewrite (`_install_fstab_opts`) — normalization, not just append

Mechanics unchanged from v7.91.0; the ext4 row filters are now hoisted globals (`_RY_AWK_EXT4_FILTER`, `_RY_AWK_EXT4_MALFORMED_FILTER` — cosmetic):
- Adds `noatime,lazytime,commit=10` to ext4 entries (field 4 only); every other column and non-ext4 row byte-preserved. **The awk rewrite passes through any ext4 row whose options field is purely numeric (`$4 ~ /^[0-9]+$/`) untouched** — a malformed-column guard; confirm such a row is then caught by the malformed-filter/refusal path rather than silently shipped.
- Strips conflicting tokens — the verify-side conflict list is exact: **`defaults`, `relatime`, `atime`, `strictatime`** (presence = "installer removes it — rewrite pending" FAIL); an existing `commit=` is rewritten to `commit=10`, and **non-10 values are tracked in `_RY_FSTAB_COMMIT_OVERRIDES`** (surfaced, not silently replaced — confirm the surfacing).
- Gates: line-count parity + size floor + mandatory `findmnt --verify`. Refused (not corrected): symlinked or whitespace-split/malformed `/etc/fstab` (dedicated malformed-row filter).

Confirm: (a) ext4-only (not the vfat ESP, not btrfs/xfs); (b) idempotent; (c) atomic (tmp+rename, `.ry.bak` taken); (d) `commit=10` durability vs every-boot forced fsck (§3) interaction.

## E. Preflight gate ordering (`_init_runtime` / `_install_preflight` / `_ir_*`)

Order matters for exit-code semantics. **Init-time capability probes now run BEFORE everything else (v7.94–7.99.1):** `id(1)` is the FIRST external command (hard-require; non-numeric `id -u` refuses); `timeout(1)` probed for `--foreground --kill-after`; `find(1)` probed for `-maxdepth`/`-printf`; **`mv -T` live-probed via two mktemp files**; `stat(1)`; `date(1)` `%z`-probed — each rejects busybox/uutils explicitly (exit 3). `TMPDIR` erased (tmp pinned `/tmp`); umask set as the variable directly; `--check` silence pinned pre-argparse. The dependency gate additionally hard-requires a **33-command GNU set** (pacman, systemctl, mkinitcpio, sdboot-manage, findmnt, sha256sum, timeout, mktemp, awk, grep, curl, getent, id, sudo, head, df, mv, tee, stat, find, cp, chmod, chown, install, cat, rm, date, wc, tail, basename, dirname, mkdir, rmdir, touch, env, sleep, cmp) plus a `df --output` capability probe, systemd ≥ 250, and warn-lists optional tools (bootctl, journalctl, dmesg, modinfo, pgrep, free, uptime, zcat, tput, swapon, zramctl, lsmod, modprobe, pkill, nmcli, ping, realpath, ip, lspci, kill). **All managed destinations are canonicalized at load** (`realpath -m` → `_RY_CANON_SYSTEM_DSTS`/`_RY_CANON_USER_DSTS`, index-aligned; failure falls back to the literal path) — `--install-file` matching is canonical.

- `_ir_resolve_root_uuid` → `EXIT_GEN_NOUUID 12` sentinel if cmdline render finds no UUID. **Mode-scoped handling (second pass):** `--install-file` is FATAL only when the (canonicalized) target IS `/etc/kernel/cmdline` (the sole UUID-embedding file) — any other target warn-continues; `--verify` warn-continues with a generic root=UUID presence check; other modes log non-fatally.
- Hardware gate (CPU match; override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed on unreadable model; `--verify` warns, deploy exits 3).
- `_ir_validate_kernel_floor` (3) — ≥ **6.19**, **NO OVERRIDE** (removed v7.98.x); fail-closed on unreadable `uname -r`; `--verify` warns and continues; deploy AND `--check` refuse.
- `_ir_validate_counts` (3) — **21** tripwires (header; incl `_RY_ARGPARSE_SPEC:7`, `_RY_BACKUP_TARGETS:4`, `MKINITCPIO_COMPRESSION_OPTIONS:2`, `PKGS_ADD:19`, `MASK:12`).
- `_ir_validate_keys` (3) — bool: BT_AUTO_ENABLE/BT_FAST_CONNECTABLE/RY_REMOTE_PLAY_PORTS/**BLACKLIST_AMDXDNA**; yes|no: SDBOOT_*/RESOLVED_MDNS/LLMNR/DOT; int: LOADER_TIMEOUT/NM_WIFI_POWERSAVE/BT_RECONNECT_ATTEMPTS; ISO-3166 COUNTRY incl reserved-range rejection; GPU_DPM_LEVEL ∈ `_RY_DPM_LEVELS`; **EPP_PREFERENCE ∈ `_RY_EPP_LEVELS`**; **CPUPOWER_GOVERNOR `^[a-z][a-z0-9_-]*$`** (aligned to `_grep_cpupower_entry`); the **nftables↔`ipv6.disable=1`** coupling; the **`BLACKLIST_AMDXDNA=false`↔IOMMU-required** coupling; non-empty scalar set (incl `EXPECTED_SCALING_DRIVER`); **boot-scalar metachar gate** (PCRE class with `\x27`) on MKINITCPIO_COMPRESSION/SDBOOT_DEFAULT_ENTRY/LOADER_*/CPUPOWER_GOVERNOR; **`MKINITCPIO_COMPRESSION_OPTIONS` token charset `^-?[A-Za-z0-9]+$`**; **`KERNEL_PARAMS` token charset `^[A-Za-z0-9._,=-]+$`**. **NOTE: every validated scalar above is an EMBEDDED script value set unconditionally (`set -g`, no `set -q` guard)** — exported environment variables of the same name are clobbered; the toggles are script-edits by design (§10).
- `_ir_validate_post_hooks` (3) — every tag has a `_post_<tag>` handler; empty tags refuse.
- Generator sentinels: 11/12/13/14; 250/251/255 never reach a process exit (footer `gen_fail`).
- Advisories (non-fatal): mesa < 26.0 soft floor — `vercmp` presence-checked and its output validated `^-?\d+$` before compare (garbage → logged skip, never `test(1)`).

Confirm: (a) counts/keys/floor run BEFORE any disk write; (b) **the documented bypass inventory is exactly ONE** (`RY_INSTALL_SKIP_HARDWARE_CHECK`) — counts/keys/floor cannot be bypassed; (c) `PACTREE_TIMEOUT_S` (60), `BOOT_SPACE_CRIT/WARN` (200/500 MB), `ROOT_AVAIL_CRIT/WARN` (2/5) remain sane — ESP thresholds vs multiple kernels + fallback + the `-1` zstd image (§4).

## F. Deeper-pass investigation deltas (updated actionable items)

1. **MES-0x86 label (HIGHEST PRIORITY, now script-only):** README corroboration dropped (v7.98.x); reconcile the script's label against current upstream; report disagreement + trusted source.
2. **RTL8127 floor claim FLIPPED:** the script now says the r8169 fixes land BELOW 6.19 (was: guaranteed ≥6.19). Cite the exact landing kernels for `f24f7b2f3af9` + `ae1737e7339b`; wrong direction ⇒ FIX-doc + the floor regains a second leg.
3. **`clearcpuid=umip` (renamed):** confirm name-form support on the floor kernel; carry the §3/§10 evaluation unchanged; never recommend the numeric form back.
4. **amdxdna blacklist + `BLACKLIST_AMDXDNA` + IOMMU coupling (NEW):** confirm `-EINVAL (ret -22)` current; blacklist-vs-alternatives; coupling asymmetry intended (true+IOMMU-on valid); NPU-off default acceptable (§10 #2).
5. **modprobe merge migration gap — CONFIRMED (second pass):** the superseded filenames have 0 in-script references; pre-7.99 leftovers are invisible to `_vrkm_blacklist_modprobe` (generator-sourced) and to every verify path; the README `sudo rm` note is the only guard. ADD-check (Low/Low) stands.
6. **PKGS_ADD 19:** pacman-contrib KEEP (script invokes pactree/paccache); **archlinux-contrib DECIDE** (installed, never invoked — KEEP-documented vs FIX-to-remove); post-`Syu` `-D --asexplicit` re-mark is orphan-protection for pre-installed-as-dependency members (§2/§L).
7. **MASK 12 (avahi unit+socket):** confirm no host dependency; unit+socket = complete closure; net §10 discoverability delta (+ping −mDNS).
8. **Kernel-floor override REMOVED:** unconditional floor posture; bypass inventory = 1; assess recoverability trade (§1).
9. **NTP unconditional + conflict guard:** chronyd/ntpd scan sufficient? (`openntpd.service` not scanned — gap or note-and-close); the no-timesyncd escape is masking the unit.
10. **udev GPU rule `ENV{DEVTYPE}` fix:** rule fired for the FIRST time at v7.94/95 — pre-fix DPM observations void; confirm live match; EPP ATTR now from `$EPP_PREFERENCE` (enum-bounded).
11. **Backup surface = the 4 boot-critical files (derived):** post-write verify/restore across all four; non-boot files stay detect-only, nftables now `nft -c`-gated — re-state the §G risk posture.
12. **`nft -c` pre-commit + post-hook validate:** malformed-ruleset window closed; degraded restart path surfaced (warn, applies at boot).
13. **Long-op timeout FLOOR 7200 s (inverted from exemption):** pacman/mkinitcpio/sdboot-manage/paccache/updatedb/pkgfile PATH-resolved and floored; `RY_RUN_TIMEOUT=0` disables; confirm 2 h covers worst-case `-Syu` and a cap-kill stays fatal-with-rollback.
14. **Verify additions (v7.98.x):** `_vrkm_blacklist_modprobe`; live COMPRESSION/_OPTIONS compare (multi-line join, last-wins); comment-proof `_chk_grep`; quoted GPU compare; `_vsp_pacman_conf` sudo-fallback + rc gates — confirm each behaves as specified (§C actionables).
15. **Data hoists:** `EPP_PREFERENCE`/`_RY_EPP_LEVELS`/`EXPECTED_SCALING_DRIVER`; GOVERNOR charset aligned; any EPP TUNE is now a one-global change.
16. **(NEW FIX-doc) `_verify_runtime_session` description** still names "Vulkan packages" — trim (the only stale string left).
17. **(NEW source conflict) SYSCTL `netdev=2.5GbE` comment** vs dual-10GbE platform (§7/§8) — resolve with lspci/driver evidence.
18. **cpupower consumer path now in-file** (`/usr/lib/systemd/scripts/cpupower`): single-verify against current CachyOS packaging (§B8).
19. **`_vrsv_wifi_iwd_proc` — SETTLED as a genuine removal (second pass):** `_vrsv_wifi` contains no iwd path; residual coverage = NM backend compare only, plus a new firewall-posture INFO (ufw state + nft rule count). An iwd opt-in host loses the process cross-check — deliberate narrow reduction, flagged Low/Low (§C).
20. **Docs surface:** README BIOS 85 W flat ceiling (state assumed budget in every power call, §6/§13d); Known-Issues/Benign-Log tables dropped; env table = 3 vars; root `--check` exit-3 + TTY `sudo -v` contract documented. Removed-surfaces recap: lineage item 6 — none exist in v7.99.1; do not verify them.

---

# Deepest-pass appendix (§G–§L) — robustness & correctness audit surface

§1–§13 audit *what the profile configures (and omits)*; §A–§F audit *what the script writes and asserts*. This layer audits *whether the installer is safe to run at all*. These are correctness questions, not tuning. Confirm each guarantee on current fish (3.6 floor) / CachyOS; flag any TOCTOU, fail-open, or partial-write window. Every mechanism below is quoted from `ry-install.fish` v7.99.1.

## G. Atomic-write guarantees (`_awf_*`)

Write path per managed file: `_awf_render_to_tmp` → (nftables only: `nft -c -f` tmpfile gate, v7.96/97) → `_awf_symlink_check` → `_awf_finalize_mv`; **backup targets (now the 4 boot-critical files)** add `_awf_make_backup` (pre) + `_awf_postwrite_verify_restore` (post).

- **tee-to-tmp with `$pipestatus`:** generator piped into `_as $use_sudo tee -- "$tmpfile"`; `pipestatus[1]` (generator) / `[2]` (tee) checked separately, generator failures mapped to `EXIT_GEN_*`. Byte-read probes read `pipestatus[1]` only per contract; **builtin→pipe captures were dropped (SIGPIPE risk, v7.96/97)** — confirm no probe still pipes a fish builtin into an external reader.
- **Post-write symlink-swap probe:** rc 0/1/2 = symlink/not/sudo-lapse, abort on swap. Confirm the check→`mv -T` window is closed.
- **`mv -T` atomic rename:** chmod tmp → `sudo -n true` re-assert → `mv -T`. Tmp created in dst's parent (same-FS rename); **the `mv -T` capability is now live-probed at init** — confirm `/boot` vfat rename semantics still hold (probe runs on /tmp, not vfat; the vfat question stands). `_ry_mkdir_0755` caps parent-dir creation at umask 0022.
- **Post-write byte re-read + restore:** re-runs the generator, compares installed bytes (tri-state rc), restores `.ry.bak` via `mv -T` on mismatch — **now across all 4 boot-critical files** (cmdline + sdboot gained backups, v7.96/97). Confirm generator determinism for all four (cmdline is UUID-dependent — `EXIT_GEN_NOUUID` path skips gracefully; sysctl/envd side-effect guards unchanged); `string collect --no-trim-newlines --allow-empty` preserves trailing-newline comparisons.
- **Non-backup files (nftables.conf et al.):** detect-only on post-write mismatch (no auto-restore) — the firewall's risk is now bounded by the `nft -c` gate (a syntactically-broken ruleset cannot deploy) but a semantically-wrong one still can; confirm that residual posture is stated and accepted.

Actionable: confirm the symlink-check→mv window; `/boot` vfat rename atomicity; generator determinism across the 4 backup targets; no remaining builtin→pipe probe.

## H. Instance lock & PID-recycle TOCTOU (`_acquire_lock*`, `_lock_pid_started_after`)

Lock = atomic `mkdir "$LOCK_DIR"` (state dir under umask 0077, 0700 contract) + pidfile via mktemp+`mv -Tf` (0600). Stale reclaim bounded (3 attempts), fail-closed. **The mkdir-success flag is set beside the rc capture (v7.96/97; `_RY_LOCK_MKDIR_OK`)** — no window where the dir exists but ownership is unrecorded.

- **PID-recycle detection:** `/proc/PID/stat` field 22 (starttime) + `/proc/stat` btime; **tick divisor = `getconf CLK_TCK`, falling back to USER_HZ=100 when absent/garbage (v7.94/95)** — correct by ABI (`starttime` is USER_HZ, fixed 100 on Linux, NOT the kernel tick rate; the old `/proc/config.gz` CONFIG_HZ recovery was wrong in principle and is GONE from the lock path — the `_kconfig_cache` helper survives for other probes, e.g. the `CONFIG_NTSYNC=y` membership check in `_ntsync_state`, list-based to avoid a builtin→pipe SIGPIPE). Reclaim only if holder start > pidfile mtime + 2 s; any unparseable field ⇒ treat live, refuse. **Empty/garbage pidfile: settle 0.2 s, re-read, still garbage ⇒ refuse reclaim.** Confirm (a) the `^.*\) ` comm-strip survives `) ` in comm; (b) field-22 indexing after the strip; (c) +2 s slack right-sized.
- **Re-read-before-rm TOCTOU guard:** pidfile re-read vs decision-time value before `rm -rf`; changed ⇒ abort. Symlinked `$LOCK_DIR` ⇒ refuse; `--preserve-root`. `kill -0` EPERM ⇒ `/proc` liveness branch, not reclaimed.

Actionable: comm-parsing robustness; fail-closed on every btime/starttime/CLK_TCK failure (incl the 100 fallback path logging `LOCK_CLK_TCK_FALLBACK`); re-read-before-rm window closed.

## I. Privilege handling (`_as`, `_run`, `_is_symlink`, `_installed_bytes`)

- **`sudo -n` everywhere; interactive prompt exactly once, TTY-gated:** a cold cache on a TTY (stdin AND stderr) runs `sudo -v` once; non-TTY refuses with "pre-cache via 'sudo -v'" (no mid-run hang possible). The lapse-mitigation banner names `Defaults timestamp_timeout=60`, a `sudo -v` keepalive, or a SCOPED NOPASSWD drop-in (pacman/mkinitcpio/sdboot-manage/systemctl — avoid ALL). Credential re-asserted before each critical write; mid-run expiry fails safe.
- **Tri-state rc 0/1/2 (drift vs sudo-lapse):** `_is_symlink`, `_installed_bytes`, `_ry_content_bytes` return 2 for lapse; every caller must branch on rc 2 (a 2→1 collapse misreports lapse as drift). Spot-check across callers — including the NEW `_vsp_pacman_conf` gates (grep rc>1 warn-skip; rc 1 + lapsed warn-skip).
- **`_run` timeout enforcement (POLICY CHANGED):** default `RY_RUN_TIMEOUT` 3600 s; >9-digit values clamped to 2147483647; invalid → default; `0` disables. **Long ops (pacman, mkinitcpio, sdboot-manage, paccache, updatedb, pkgfile — PATH-resolved basename) are NO LONGER EXEMPT: the effective timeout is FLOORED to `_RY_LONGOP_HARD_CAP=7200 s`** (raised when below, logged `TIMEOUT_LONGOP_CAP`; "short SIGKILL would bypass rollback"; `0` still disables). The effective-command resolver skips the **explicit sudo value-flag set `-h -u -g -p -C -D -R -T -U --user --group --prompt --close-from --chdir --chroot --command-timeout --other-user --host`** (`-h` = host form, flag+value skipped), bare dash flags, `env`, and `VAR=` prefixes before naming the command. Every prior "exempt" statement is retired. `PACTREE_TIMEOUT_S` (60) governs pactree only. **Capture hygiene (second pass):** logged `_run` argv redacts `/tmp/ry-*` paths to `[REDACTED]`; on overflow, the analysis is inlined into JSONL — bytes + sha256 of the full capture, then an awk scan of ONLY the elided middle region for diagnostic keywords (error/fatal/fail/warning/cannot/denied/conflict/corrupt/missing/unable), ≤10 hits, each capped at 2000 chars; nothing retained on disk.

Actionable: audit every rc-2 caller; confirm the TTY-once prompt cannot recur mid-run; confirm 7200 s covers worst-case `-Syu` and a cap-kill is fatal-with-rollback (not a silent skip).

## J. Boot-wipe gate & boot-critical rollback (`_irb_taint_gate`, `_install_rebuild_boot`)

Mechanics unchanged from v7.91.0 — re-confirm on v7.99.1:
1. `_irb_taint_gate`: `_RY_BOOT_TAINTED=true` OR failed mkinitcpio.conf revert ⇒ SKIP `mkinitcpio -P`, `EXIT_BOOT_CRIT` (4). Taint is set through the shared `_taint` helper (flags `INSTALL_HAD_ERRORS` + `_RY_BOOT_TAINTED` together) — confirm every boot-critical write-failure path routes through it.
2. `mkinitcpio -P` failure ⇒ abort remaining, exit 4.
3. `$BOOT` resolved BEFORE the sdboot vfat gate; unresolved OR non-vfat `/boot` + `REMOVE_EXISTING=yes` ⇒ refuse the wipe (exit 4) — never run `sdboot-manage` against an unverified target.
4. `_preflight_boot_sanity`: vmlinuz + initramfs + entries must exist or exit 4.

Confirm: (a) no path to `REMOVE_EXISTING=yes` with unverified `$BOOT`; (b) the Phase-3 cmdline-write vs Phase-5 rebuild window — a boot with new cmdline (`clearcpuid=umip`) + old initramfs is covered by the param-stripped fallback (which also re-enables IPv6 and kernel-default IOMMU; the modprobe amdxdna blacklist REMAINS active in fallback — B1 asymmetry); (c) `EXIT_BOOT_CRIT` is terminal (no Finalize after).

**mkinitcpio rollback:** pre-`Syu` snapshot to `/run/ry-install` (0700, mktemp; the seed is a tagged log line, v7.98.x); on pacman failure `_mkinitcpio_revert` restores via same-`/etc`-FS mktemp + byte-exact `cmp` + atomic mv (`--reference` perms). Duplicate `KEY=` resolves last (matches `_ry_mkinitcpio_array` semantics — confirm the two agree). Snapshot in tmpfs is same-boot-only (acceptable). Flag if a pacman partial transaction can desync mkinitcpio.conf from installed modules without triggering revert.

## K. Signal & exit teardown (`_cleanup`, `_teardown`, `_do_cleanup`)

- **Signal handlers:** INT/TERM/HUP/QUIT/ABRT with 128+N via exec re-raise; idempotent (`_CLEANUP_DONE`); SIGPIPE marks output broken, JSONL-only continues; `fish_exit` teardown prefers `_INTENDED_EXIT_CODE` → `_RY_INSTALL_LAST_EXIT` → `$status`. Pre-init tmpfile-cleanup `_log` guarded. All JSONL `ts` fields from the single `_RY_TS_FMT` global (`+%Y-%m-%dT%H:%M:%S%z`) via `command date`. **NEW (v7.99.1): the `--check` stderr-silence contract holds through the PRE-ARGPARSE window** (`_RY_ARGV_CHECK_ONLY` scanned from raw argv before MODE exists; root + `--check`-only argv ⇒ silent exit 3; `-V`/glued `-VV` compatible; other flags restore exit-2 parity) — confirm no early-init path (dep probes, log-dir failures) can print to stderr under a pure `--check` argv. **The umask is set as the VARIABLE directly (v7.99.1)** — the autoloaded `umask` function could leak "Unknown command" on a mid-autoload signal; confirm the variable form is honored by fish ≥3.6 for child processes and `mkdir`.
- **Cleanup order (`_do_cleanup`):** kill children → mkinitcpio revert → tmpfile sweep → filesystem sweep → lock release LAST → erase globals. **Child reap (second pass): TERM to `-P $fish_pid` descendants only, then a poll loop — 0.5 s grace normally (5 × 0.1 s), 10 s (100 × 0.1 s) when `/var/lib/pacman/db.lck` exists (mid-transaction pacman gets time to release the db) — then KILL, again descendants-only; a missing pgrep degrades to a flat 0.5 s sleep.** The signal→exit map is explicit (HUP:129 INT:130 QUIT:131 TERM:143 ABRT:134; unknown logs and falls back to 130). `_RY_TMPDIR_GLOBS` (6, PID-scoped `ry-*.$fish_pid.*`): ry-sudo-err, ry-tee-err, ry-run, ry-argparse-err, ry-fstab-tee-err, ry-fstab-awk-err — confirm the glob set exactly matches the created set (a non-PID-scoped tmpfile would leak; an over-broad glob would touch a peer). `TMPDIR` is erased at init (children cannot be redirected out of `/tmp` — the sweep target and the create target cannot diverge; confirm). Boot-taint entry point is the shared `_taint` helper (sets `INSTALL_HAD_ERRORS` + `_RY_BOOT_TAINTED` together — §J).
- **Log lifecycle:** rename via `mv -T` with **`cp -pT` + `rm` recovery** (dir-squat safe); both-fail keeps logging at the old path (warn); pre-existing symlinked LOG_FILE removed, re-created 0600 (umask 0177 window). Root-guard refusal emits **one `@@LEFT@@` line per leftover positional** (display sentinel, prefix-stripped) + `@@IF@@` for the install-file value — confirm the sentinel lines never leak into user-visible output unstripped.

- **Exit-code contract (14 distinct — audit for discipline, no bare `exit 1`):**

```
║ CODE ║ NAME              ║ RAISED WHEN                        ║
║──────║───────────────────║────────────────────────────────────║
║ 0    ║ EXIT_OK           ║ success                            ║
║ 1    ║ EXIT_FAIL         ║ generic failure                    ║
║ 2    ║ EXIT_USAGE        ║ CLI misuse                         ║
║ 3    ║ EXIT_PREFLIGHT    ║ preflight gate abort               ║
║ 4    ║ EXIT_BOOT_CRIT    ║ boot-critical rollback path        ║
║ 5    ║ EXIT_LOCK         ║ instance lock contention           ║
║ 10   ║ EXIT_DRIFT        ║ --check found drift                ║
║ 11   ║ EXIT_GEN_NOFN     ║ generator fn not defined           ║
║ 12   ║ EXIT_GEN_NOUUID   ║ root UUID unresolved in generator  ║
║ 13   ║ EXIT_GEN_SYSCTL   ║ sysctl gen printed-count mismatch  ║
║ 14   ║ EXIT_GEN_ENVD     ║ env.d gen printed-count mismatch   ║
║ 250  ║ EXIT_AS_MISUSE    ║ _as w/o cmd or non-bool use_sudo   ║
║ 251  ║ EXIT_RUN_TMPFAIL  ║ _run tmpfile create failed         ║
║ 255  ║ EXIT_RUN_MISUSE   ║ _run no-args or dash-prefixed arg  ║
```

- **`_run` timeout:** default 3600 s; **long pkg/boot/db ops floored to 7200 s (see §I — the exemption is GONE)**; a timeout surfaces as fatal, not a silent skip. Codes 11–14 internal (checksum verify maps NOUUID to graceful skip); 250/251/255 are BUG asserts that must never fire — a recommendation tripping one is by definition wrong.

Actionable: confirm SIGKILL is the only cleanup bypass and stale-reclaim (§H) recovers a SIGKILLed holder; glob set == created set; no path collapses a structured code (3/4/5/10) into bare 1; `--check` silence holds across every pre-argparse error path.

## L. pacman transaction safety (`_ip_pacman_invoke`)

- **Full `-Syu --needed` only:** first pass `-Syu --needed --noconfirm`, retry `-Syyu --needed --noconfirm` (forced db re-sync; the warn states it "will not resolve pkg conflicts"); second failure fatal and surfaced. `SYSTEM_UPGRADED` from a `pacman -Q | sha256sum` fingerprint (identical pre/post ⇒ false + `PKG_STATE_UNCHANGED`; empty fingerprint fails open to true). **Post-`Syu`, present PKGS_ADD members are re-marked `pacman -D --asexplicit` (idempotent)** — this is the "explicit post-Syu" semantic: install-REASON marking so the Phase-4 `-Rns` cannot orphan-cascade a member that pre-existed as a dependency; a `-D` failure warns (`PKG_ASEXPLICIT_FAIL`, "orphan-removal exemption not guaranteed"). Confirm no `-S <pkg>` ever runs without `-yu` context.
- **db.lck pre-check:** refuses if `/var/lib/pacman/db.lck` exists ("another pacman may be running, or stale lock from a crashed run — remove the lock file manually if no pacman process is active"); never removes it. **The teardown child-reaper honors an in-flight transaction: `-P $fish_pid`-scoped TERM with a 10 s grace when db.lck is present (0.5 s otherwise) before KILL (§K)** — a peer pacman is untouchable by scope. Confirm checked before any package op.
- **PKGS_DEL via `-Rns` (rdep-aware):** external dependants skip + log (`_RY_PKG_REMOVE_SKIPS`); no cascade, no partial-upgrade state. `paccache -rk2` and `-ruk0` separate. NOTE: pactree/paccache providers are now declared (PKGS_ADD pacman-contrib, §2) — but PKGS_DEL runs in Phase 4 AFTER Phase-2 installs, so the tool is guaranteed present by its own transaction; confirm that ordering (a pactree-absent first run pre-Phase-2 only warns).

Actionable: retry-then-fatal semantics; no self-removal of db.lck; `-Rns` cannot induce partial-upgrade; `-P $fish_pid` scoping complete; the 7200 s floor (§I) covers the `-Syu` worst case.

## Robustness verdict (required, separate from §1–§13)

Add a final **ROBUSTNESS** verdict block:
- For each of §G–§L: PASS (guarantee holds) / GAP (window or fail-open found) / UNCERTAIN (cannot confirm against current fish/kernel without testing).
- Any GAP in §G/§H/§J (atomic-write, lock, boot-wipe) is release-blocking and outranks every tuning finding — surface it first regardless of IMPACT/RISK on config items.
- This layer is correctness, not preference: there is no "deliberate trade-off" defense for a partial-write window or fail-open lock. FIX applies normally here (flag-don't-FIX is for config values, not safety invariants).

Sources for §G–§L: man7.org (mkdir(2), rename(2)/mv -T, proc(5) stat fields + USER_HZ, sysconf CLK_TCK), fishshell.com/docs (`$pipestatus`, `string collect`, `--on-signal`, `--on-event fish_exit`, variable-umask), wiki.archlinux.org (pacman partial-upgrade policy, mkinitcpio, systemd-boot, nftables), docs.kernel.org (/proc/stat btime, USER_HZ, accel/amdxdna), man.archlinux.org nft(8).
