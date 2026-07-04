#  cachyos-tuning-audit — ry-install Tuning (CachyOS · Beelink GTR9 Pro)

Target: `ry-install.fish` **v7.88.3** (attached). Source of truth: script > README > CHANGELOG.

**Platform:** Beelink GTR9 Pro · Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤96 GB as VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP · CachyOS · systemd-boot.

**Counts (from `_ir_validate_counts`, all 19 hard-asserted):** KERNEL_PARAMS **17** · MKINITCPIO_HOOKS 11 · MKINITCPIO_MODULES 1 · LOGIND_IGNORE_KEYS 8 · ENV_VARS 11 · SYSCTL_VALUES 9 · PKGS_ADD 17 · PKGS_DEL 9 · MASK 10 · EXPECTED_VULKAN_PKGS 2 · EXPECTED_SERVICES 5 · _RY_PKG_MANAGED_SERVICES 1 · _RY_POST_HOOKS **17** · _RY_BOOT_CRITICAL_DSTS 4 · _RY_PHASE_NAMES 6 · _RY_BACKUP_TARGETS 2 · _RY_TMPDIR_GLOBS 6 · SYSTEM_DESTINATIONS 15 · USER_DESTINATIONS **2**. Managed files = **17** (15 system + 2 user; `_RY_MANAGED_FILE_COUNT=17`). MangoHud.conf = **19 active directives, 0 commented** (`cpu_temp` is now LIVE/uncommented; the v7.77.1 "17 active + 1 commented `# cpu_temp`" body is superseded). NTSYNC autoload confs = **0** (de-managed v7.76.1; ntsync is assert-only).

**Hard floors:** KERNEL_MIN **6.19** (preflight hard-fail, `_ir_validate_kernel_floor`; override `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK=1`; verify warns only) — the script's own rationale anchors the floor to **gfx1151 "MES-0x86" amdgpu** stability AND the **RTL8127 suspend/shutdown-hang fix + r8169 support** (both land ≥ the floor). · CPU gate `Ryzen AI Max` (override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`). · soft Mesa < 26.0 warn (vercmp, stderr silenced). · **The version-specific linux-firmware preflight advisory was REMOVED (v7.81.0/v7.82.0)** — the profile no longer hard-warns on `20251125*` or soft-warns below `20260110`; firmware currency is now handled by the kernel floor + `mkinitcpio-firmware` package + the README "resolved" note (MES 0x86), not a runtime blob-version gate.

> **SCRIPT-vs-PRIOR-AUDIT CONFLICT — MES version label (resolve this first).** The v7.77.1 prompt asserted the "MES 0x86" label was *stale* and that the relevant microcode versions were 0x83 (bad, `20251125`) vs 0x80 (good, `20251111` / `20260110` revert). **The current source of truth contradicts that:** `ry-install.fish` v7.88.3 line 14 sets `KERNEL_MIN 6.19 # floor: gfx1151 MES-0x86 amdgpu`, the kernel-floor refusal message (`_ir_validate_kernel_floor`) cites "gfx1151 MES-0x86 firmware needs >=$KERNEL_MIN amdgpu", and README line 208 lists MES page faults as "resolved (MES 0x86; current `linux-firmware` + shipped `mkinitcpio-firmware`)". Per the stated precedence (script > README > CHANGELOG), **the label in force is "MES 0x86"** and the firmware fix is now framed as *kernel/firmware-currency-resolved*, not a version-pinned advisory. Re-verify the exact gfx1151 MES microcode revision against current upstream (git.kernel.org/kernel-firmware, ROCm #5724, Launchpad #2129150) and state plainly which label (0x86 vs 0x83/0x80) matches the shipping GC 11.5.1 blob; flag the disagreement rather than silently carrying either. This is the single most important factual reconciliation in this re-audit.

## What changed since v7.77.1 (re-audit focus)

The prior prompt targeted v7.77.1. Between v7.77.1 and v7.88.3 the script moved substantially. The following are NEW, INVERTED, or REMOVED and need fresh evidence — do NOT carry forward the old conclusions on these:

1. **KERNEL_MIN RAISED 6.18 → 6.19** (v7.79.0). The integer floor is now 6.19, and the script's rationale is stated as gfx1151 MES-0x86 amdgpu + the RTL8127 suspend/shutdown-hang fix (`ae1737e7339b`) + r8169 support (`f24f7b2f3af9`) — all guaranteed at ≥6.19. Every "6.18 floor" statement in the prior prompt is now wrong. Re-confirm 6.19 is the correct minimum and whether the true gfx1151-stability floor sits above it.
2. **IPv6 DISABLED SYSTEM-WIDE + nftables now IPv4-ONLY** (v7.87.x). NEW cmdline param `ipv6.disable=1` (KERNEL_PARAMS 16 → **17**). The nftables ruleset was rewritten: **all ICMPv6/NDP accept rules were DROPPED** (the ruleset is IPv4-only), and **inbound IPv4 ping (`echo-request`) is now ACCEPTED** (previously deliberately blocked). A hard preflight coupling (`_ir_validate_keys`) refuses to deploy if `/etc/nftables.conf` is managed but `ipv6.disable=1` is absent from KERNEL_PARAMS. Every prior "no inbound IPv4 ping", "ICMPv6 NDP/PMTUD accept", "IPv6 pingable / IPv4 unpingable asymmetry", and "nd-neighbor-solicit break-glass gate" statement is now retired. The `_vss_nft` / `_vrsv_nft_assert_*` subs now assert the **ping accept** (`echo-request`), not NDP.
3. **`amd_iommu=off` unchanged** (still v7.77.0's inversion; `iommu=pt` remains gone). AMD-Vi stays fully disabled; `_vrkm_iommu` (off-derivation → expect 0 iommu_groups, hard-fail if present) is intact. Re-evaluate on its own merits exactly as before (latency vs DMA-isolation cost); the README VFIO/SR-IOV escape hatch (`amd_iommu=on iommu=pt`) is still documented. ROCm on gfx1151 is unaffected (`IOMMU Support: None`).
4. **MangoHud `cpu_temp` is now LIVE (UNCOMMENTED)** — the profile ships `cpu_temp` as an active directive (line 994, bareword, between `cpu_stats` and `cpu_mhz`). MangoHud.conf is now **19 active directives, 0 commented**. The entire prior "cpu_temp COMMENTED OUT / 17 active + 1 commented / re-enable via cpu_temp_sensor=k10temp" framing is obsolete. Re-verify `cpu_temp` populates from `k10temp`/Tctl on Zen 5 as shipped, and note the still-open MangoHud #1794 caveat (enabling `cpu_temp` can zero the `cpu_power` readout on Zen 5) now applies to the SHIPPED config, not a hypothetical opt-in — confirm whether `cpu_power` (still present) and `cpu_temp` coexist correctly on the installed MangoHud + kernel.
5. **`_kb_*` known-benign INFO subsystem REMOVED entirely** (v7.77.0: "drop baloofilerc, _post_baloo, _kb_*, umip check fn"). There is no `_vss_known_benign`, no `_kb_modemmanager_masked` / `_kb_acp70_no_machine_driver` / `_kb_thunderbolt_nhi_unknown` / `_kb_no_battery_backlight` / `_kb_usb_mic_volume_curve`, and no `VERIFY-STATIC: KNOWN-BENIGN ADVISORIES` banner. §12's "5 `_kb_*` INFO subs" surface no longer exists — drop all of it. The underlying hardware facts (ACP70 mic gap, etc.) remain real but are no longer surfaced by the installer.
6. **baloofilerc DE-MANAGED** (v7.77.0). `~/.config/baloofilerc` is no longer written; there is no `_post_baloo`. USER_DESTINATIONS dropped 3 → **2** (environment.d + MangoHud). Every baloo/Indexing-Enabled reference is obsolete.
7. **UMIP check function REMOVED** (v7.77.0: "umip check fn" dropped). `_ry_check_umip_disabled` no longer exists; only the `clearcpuid=514` cmdline param survives. Any "`_ry_check_umip_disabled` runs in preflight" statement is now false — the param is asserted only generically by `_vrk_cmdline` (every KERNEL_PARAMS token present in `/proc/cmdline`).
8. **linux-firmware version advisory REMOVED** (v7.81.0/7.82.0: "remove firmware soft-floor advisory"). No `20251125`/`20260110` preflight strings remain. Drop the firmware-blob advisory from §1/§11/§E and the `pacman -Q linux-firmware` known-good/known-bad annotation from the VERIFY block (keep it only as an informational currency check, not a hard/soft warn the script emits).
9. **NEW verify assert: `amd_pstate/dynamic_epp == disabled`** (v7.79.0). `_vrk_cpu_state` now asserts `/sys/devices/system/cpu/amd_pstate/dynamic_epp` equals `disabled` (else an EPP pin returns `-EBUSY`; the node is absent pre-6.16). Add this to the hard-assert inventory.
10. **NEW remote-play port** (v7.79/7.80: "add TCP 27037"). The gated `RY_REMOTE_PLAY_PORTS=true` TCP set is now `{ 47984, 47989, 48010, 27036, 27037 }` (added 27037); the UDP set is unchanged `{ 47998-48010, 27031-27036 }`. Re-validate the full port set against current Sunshine/Moonlight + Steam Remote Play.
11. **udev rule matchers CORRECTED** — the EPP rule "never fired" and was fixed to `KERNEL=="cpu[0-9]*"` (v7.85; earlier `SUBSYSTEM=="cpu", DEVPATH=="*/cpufreq"` did not match at enumeration); the GPU rule is now `KERNEL=="card[0-9]*"` + `DEVTYPE=="drm_minor"` (v7.79). The prior appendix rule strings (B4) are stale. `_post_udev` now retriggers **both** block and cpu devices so the EPP rule live-applies.
12. **GPU_DPM_LEVEL accepted set hoisted to `_RY_DPM_LEVELS`** (v7.83/7.84) and validated against the dpm-level enum (`_ir_validate_keys`). Default remains `auto`. COUNTRY now also rejects ISO-3166-1 reserved/user-assigned codes (v7.81/7.82).

Non-behavioral doc/robustness churn folded in v7.83–v7.88.3 (verify banner glyph normalization, comment trims, `_run` overflow handling moved inline with sha256/bytes diag, tmpfiles PID-scoped for peer-run safety, lock state-dir under `umask 0077`, `--check` arg-handling exit-code fixes, 3.6 version-gate `$()` replacement, `install-file` format-validation before write, `SYSTEM_UPGRADED` from pacman `-Q` fingerprint, `/dev/stdin`+fd-0 piped-stdin refusal, mesa soft-floor vercmp stderr silenced, pre-init-signal `_cleanup_tmpfiles` `_log` guard, and — the sole v7.88.3 change — the JSONL ISO-8601 timestamp format hoisted to a single `_RY_TS_FMT` global (`+%Y-%m-%dT%H:%M:%S%z`) consumed by the footer, `_log`, and the header emitter). These do not change tuning values but several touch §A–§L (the `_RY_TS_FMT` hoist touches only §K logging) — appendix updated accordingly.

> **Verification note (2026-07-03).** This revision re-derives every value directly from `ry-install.fish` v7.88.3 (extracted array-by-array), not from the prior prompt. Where the script and the v7.77.1 prompt disagree, the script wins and the disagreement is flagged (headline: the **MES 0x86 label** conflict above, and the **inbound-IPv4-ping / IPv6-disabled** network inversion). Live-upstream anchors carried forward from Round 2 that the script did NOT contradict (ntsync CONFIG_NTSYNC=y, amd_iommu=off not breaking ROCm, governor/EPP semantics, §13 candidate calls) are retained; anchors the script *did* contradict or remove (firmware blob advisory, cpu_temp-commented, `_kb_*`, baloo, 6.18 floor, NDP ruleset, no-inbound-ping) are corrected or deleted. **Deeper pass (293 fns / 180 globals censused):** the `--verify` leaf subs (`_vsb_*`, `_vsp_*`, `_vsc_check_one`, `_vmh_*`, `_vpd_dir_perm_check`) are now mapped leaf-to-orchestrator in §C, and the full 14-code exit contract + `_run` 1 h timeout are documented in §K. Internal-state globals (progress-bar, matrix counters, boot-enum scratch) are deliberately out of scope — implementation scaffolding, not tuning targets.

## Mission

Evaluate every config decision against current upstream sources for this exact silicon. Return a prioritized, evidence-backed tuning report. The profile deliberately trades PCI-passthrough capability, power-saving, IPv6, and host inbound-firewalling-of-ping for performance, latency, and a simpler IPv4-only ruleset; confirm each choice is current and correct, surface anything superseded or harmful, and quantify safety deltas without second-guessing intentional design.

## Rules

1. Item-by-item, hardware-anchored to gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE.
2. Respect deliberate trade-offs: **flag and quantify, do not auto-FIX.** Reserve FIX for incorrect, superseded, deprecated, or harmful values.
3. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is non-trivial.
4. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN.
5. Flag every source conflict and state which is trusted (the MES-0x86 label conflict is pre-flagged above).
6. Give exact versions (kernel / Mesa / linux-firmware / pkg) and exact before→after, mapped to the in-script global.

## Output

- **Findings matrix** (box-drawn Unicode, code fence, grouped by section): ITEM · CURRENT (v7.88.3 value) · CALL (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE (URL + version/date/commit).
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
cat /sys/.../cpu0/cpufreq/scaling_driver                    # amd-pstate-epp
cat /sys/.../cpu0/cpufreq/scaling_governor                  # powersave (EPP-honoring under pstate=active)
cat /sys/.../cpu0/cpufreq/energy_performance_preference     # balance_performance
cat /sys/.../amd_pstate/status                              # active
cat /sys/.../amd_pstate/dynamic_epp                         # disabled (NEW assert; else EPP pin -EBUSY; absent pre-6.16)
cat /sys/.../amd_pstate/prefcore                            # enabled
cat /sys/.../cpufreq/boost                                  # 1
cat /sys/.../clocksource0/current_clocksource               # tsc (HPET → fail; check dmesg TSC-demotion)
cat /sys/block/nvme0n1/queue/scheduler                      # [none] (adjust node)
cat /sys/class/drm/card*/device/power_dpm_force_performance_level   # auto (DPM floor un-pinned; GPU_DPM_LEVEL)
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l   # 0 (amd_iommu=off → no groups)
cat /proc/cmdline | rg -o 'amd_iommu=\S+'                   # amd_iommu=off (AMD-Vi disabled)
cat /proc/cmdline | rg -o 'ipv6\.disable=\S+'              # ipv6.disable=1 (IPv6 OFF system-wide — NEW)
cat /proc/cmdline | rg -o 'clearcpuid=\S+'                  # clearcpuid=514 (UMIP off)
cat /proc/cmdline | rg -o 'processor.max_cstate=\S+'        # 1 (C-state cap)
cat /proc/cmdline | rg -o 'fsck\S+'                         # fsck.mode=force fsck.repair=yes
ls -l /dev/ntsync                                           # present (assert-only; autoload conf dropped v7.76.1)
sudo dmesg | rg -i 'AMD-Vi|DMAR'                            # expect NO "AMD-Vi: Enabled" (amd_iommu=off)
ip -6 addr                                                  # expect no IPv6 addresses (ipv6.disable=1) — NEW
cat /etc/modprobe.d/60-ry-mt7925e.conf                      # options mt7925e disable_aspm=1
pacman -Q linux-firmware                                    # currency check only (script no longer hard/soft-warns on version)
vulkaninfo | rg -i 'driverName|deviceName'                 # RADV / Radeon 8060S; confirm uma heap
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc vm.max_map_count vm.compaction_proactiveness vm.swappiness
findmnt -no OPTIONS /                                       # noatime,lazytime,commit=10
swapon --show; zramctl                                      # zram active (advisory; profile does not manage zram)
iw reg get | rg -i country                                 # US
cat /etc/iw-regdomain                                       # COUNTRY=US
sudo nft list chain inet filter input                      # policy drop + lo + established/related + IPv4 ICMP incl echo-request (inbound ping ALLOWED); +remote-play ports IFF RY_REMOTE_PLAY_PORTS=true. NO ICMPv6/NDP rules (IPv4-only).
stat -c '%a %U:%G' /etc/NetworkManager/system-connections/* # 0600 root:root
systemctl is-enabled bluetooth.service                     # enabled
printenv MANGOHUD                                          # 1
grep -c '^cpu_temp' ~/.config/MangoHud/MangoHud.conf       # 1 (cpu_temp is now LIVE/uncommented)
```

Hard `--verify` asserts (mismatch → exit 1/3): every `KERNEL_PARAMS` token present in `/proc/cmdline` + `rw` (generic loop in `_vrk_cmdline`, so `amd_iommu=off` AND `ipv6.disable=1` are both auto-asserted); scaling_driver=amd-pstate-epp, scaling_governor=powersave, EPP=balance_performance, **amd_pstate `dynamic_epp=disabled` (NEW)**, amd_pstate status/prefcore, boost, clocksource=tsc; GPU `power_dpm_force_performance_level=auto`; **IOMMU effect (`_vrkm_iommu`): with `amd_iommu=off`, 0 iommu_groups required — hard-fail if groups present**; usbcore.autosuspend=-1, nvme_core.default_ps_max_latency_us=0, zswap.enabled∈{N,0}, nmi_watchdog=0, NVMe sched `[none]`; regdom (`/etc/iw-regdomain`); **nftables IPv4 ping accept (`echo-request`) present (`_vss_nft`; regression guard — the ruleset MUST keep inbound ping enabled)**; mt7925e `disable_aspm=1`; NM system-connections 0600 root:root (`_vrs_nm_perms`). Warn-level: GPU DPM not-`auto`, ZRAM state, live nftables ping cross-check (`_vrsv_nft_assert_ping`, warn-only), ntsync runtime device state, iwd runtime state (opt-in), TSC demotion advisory. No THP, KSM, `ttm.*`, drirc, `radv_enable_unified_heap_on_apu`, `iommu=pt`, ICMPv6/NDP, baloo, or `_kb_*` assert exists — do not verify them. NOTE: the IOMMU assert is `off`/0-groups; the nftables assert is now the **ping accept**, NOT an NDP rule; the firmware-version advisory no longer exists.

### Security delta (ordered)

1. **UMIP off** (`clearcpuid=514`) — descriptor-table base leak, kernel tainted; headline open reduction.
2. **AMD-Vi fully disabled** (`amd_iommu=off`) — no DMA isolation/remapping; any DMA-capable device (USB4/Thunderbolt, NVMe, NIC) can in principle DMA over system RAM unmediated. Quantify the exposure and confirm it is an accepted trade for a no-passthrough single-user desktop. Open reduction.
3. **IPv6 disabled + inbound IPv4 ping now allowed (NEW, net wash-to-slight-reduction).** `ipv6.disable=1` removes the entire IPv6 attack surface (no IPv6 stack at all) — a reduction in exposure, but also a loss of IPv6 reachability/functionality; simultaneously the IPv4 ruleset now **accepts inbound `echo-request` (ping)**, a small increase in host discoverability on the LAN. Net effect on a trusted dual-10GbE LAN is minor; quantify both directions. Confirm disabling IPv6 system-wide (vs firewalling it) is the intended posture and that no LAN service (mDNS already off, SLAAC, link-local) is silently broken. The README documents dropping `ipv6.disable=1` + restoring IPv6 firewall rules to return to dual-stack.
4. **split_lock_detect=off** — a misbehaving app can degrade the system.
5. **Plaintext DNS** (`DNSOverTLS=no`, `DNSSEC=allow-downgrade`) reverting the CachyOS DoH default — DNS observable and spoofable on-path.
6. **Optional inbound remote-play ports** (`RY_REMOTE_PLAY_PORTS`, default OFF) — when enabled, opens Sunshine/Steam stream ports (TCP 47984/47989/48010/27036/27037 + UDP 47998-48010/27031-27036); quantify the exposure of the opt-in.
7. **Firewall default-deny-inbound ships** (nftables, IPv4-only; lo + established/related + IPv4 diagnostic ICMP incl. inbound ping; all else dropped) — net positive.

## Investigation (§1–§12 ordered by installer phase; §13 = candidate enhancements)

### 1. Platform baseline and version floors

Current: **hard kernel floor KERNEL_MIN 6.19** (preflight `_ir_validate_kernel_floor`, override `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK=1`, fail-closed on unreadable `uname -r`; `--verify` warns and continues) — the script's rationale (line 14 + the refusal message) anchors the floor to **gfx1151 "MES-0x86" amdgpu** stability plus the **RTL8127 suspend/shutdown-hang fix + r8169 support** (both land ≥6.19); CPU gate `Ryzen AI Max` (override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`, fail-closed on unreadable model); soft Mesa < 26.0 warn (vercmp, stderr silenced on the compare). **The version-specific linux-firmware advisory was REMOVED** (v7.81/7.82) — no `20251125`/`20260110` gate remains.

- **6.19 floor — RE-VERIFY.** Confirm 6.19 mainline is the correct minimum for this silicon: (a) which RTL8127 r8169 commits (`f24f7b2f3af9` support, `ae1737e7339b` suspend/shutdown-hang fix) actually gate 6.19 vs land earlier and are merely guaranteed by it; (b) whether the true gfx1151-stability floor sits above 6.19 (the prior prompt cited a <6.18.4 gfx1151 bug and 6.18.6+ recommended — re-check whether a higher point-release floor is warranted at audit time). State the real per-subsystem floors and whether the integer 6.19 is conservative, exact, or insufficient.
- **MES-0x86 label — RESOLVE THE CONFLICT (see headline note).** The script + README now use "MES 0x86" as the resolved-firmware label; the v7.77.1 prompt called that label stale (0x83 bad / 0x80 good). Determine, against current upstream (git.kernel.org/kernel-firmware GC 11.5.1 history, ROCm #5724, Launchpad #2129150, Debian firmware-nonfree changelog), which MES revision the shipping gfx1151 blob actually reports and whether "0x86" is (i) the current-correct label after further firmware churn, (ii) a typo/carry-over in the script, or (iii) a distinct third revision. Do NOT silently adopt either; report the disagreement and the trusted source.
- **linux-firmware — advisory removed.** The profile no longer emits a firmware-version warn. Treat firmware currency as satisfied by the kernel floor + `mkinitcpio-firmware` package; keep `pacman -Q linux-firmware` in VERIFY as an informational currency check only. If a specific gfx1151 MES-regressed release is still shipping in current repos, flag it as advisory context, but note the script intentionally no longer gates on it.
- Confirm the soft Mesa 26.0 floor matches current RADV guidance; enumerate open gfx1151 RADV issues and state whether 26.0 is still the right threshold.
- Confirm gfx1151 reports `uma:1` natively, so the removed drirc override is genuinely redundant on current Mesa.
- Sources: wiki.cachyos.org, docs.kernel.org gpu/amdgpu, gitlab.freedesktop.org/mesa (gfx1151), git.kernel.org linux-firmware + r8169.

### 2. Packages

PKGS_ADD (17): nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa, mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, git-delta, lm_sensors, rtkit, realtime-privileges, ddcutil, nftables. PKGS_DEL (9, `-Rns`, rdep-aware): plymouth stack (plymouth, cachyos-plymouth-bootanimation, cachyos-plymouth-theme, breeze-plymouth, plymouth-kcm) + micro + cachyos-micro-settings + cachy-update + kdeconnect. AUR: none. Vulkan (chwd): vulkan-radeon, lib32-vulkan-radeon. (Unchanged since v7.77.1.)

- Confirm cachyos-gaming-meta and -applications supply RADV/Proton/gamescope/MangoHud/GameMode, and that the meta pulls MangoHud (the profile also ships MangoHud.conf — confirm no conflict). **GameMode integration gap — KEEP the omission.** `gamemode`/`lib32-gamemode` are NOT in PKGS_ADD explicitly and the profile ships no `gamemode.ini` / `gamemoderun` wrapper / GameMode env. Feral GameMode's primary effect is switching the CPUFreq governor to `performance` while a game runs — but this profile already pins governor/EPP/DPM profile-wide, so GameMode's governor switch is redundant and its nice/ioprio hints marginal. KEEP the omission; document why. IMPACT Low · RISK Low.
- `rtkit`: confirm RealtimeKit is correct alongside realtime-privileges for PipeWire thread priority; is `rtkit-daemon.service` socket-activated (not in EXPECTED_SERVICES)?
- Confirm `lib32-mesa` is still needed alongside `lib32-vulkan-radeon`, or now redundant.
- Confirm plymouth, micro, cachy-update, kdeconnect removal has no dependency fallout (`-Rns` skips + logs a package with an external dependant rather than cascading; skips tracked in `_RY_PKG_REMOVE_SKIPS`).
- Advisory one-liner (script no longer probes repo tier as of v7.74.0): state whether znver/x86-64-v4 (AVX-512) repos benefit this build over v3.
- Sources: wiki.cachyos.org, wiki.archlinux.org/Gaming + PipeWire + RealtimeKit.

### 3. Kernel cmdline (17)

`8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=514 fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0`

NEW/changed since v7.77.1 — audit fresh:

- **`ipv6.disable=1` (NEW, v7.87.x):** disables the kernel IPv6 stack system-wide. This is the enabling half of the IPv4-only nftables ruleset (§8/§10) and is HARD-COUPLED: `_ir_validate_keys` refuses to deploy if `/etc/nftables.conf` is managed without this token. Evaluate: (a) is fully disabling IPv6 (vs firewalling it with an IPv6 policy-drop chain) the right posture for this LAN — does it break SLAAC/link-local dependencies, any LAN peer, or future IPv6-only services? (b) confirm no application on this gaming/LLM desktop needs IPv6 (Steam, Proton netcode, and most services fall back to IPv4, but confirm); (c) the README opt-back-in (drop the token + restore IPv6 firewall rules + re-run) is the correct dual-stack path. IMPACT Low (functionality) · RISK Low-Med (silent IPv6-dependency breakage).
- **`amd_iommu=off` (unchanged):** AMD-Vi fully off; no `iommu=pt`, no `amd_iommu=on`. **`amd_iommu=off` does NOT break ROCm** — `rocminfo` reports `IOMMU Support: None` on gfx1151 and the Strix Halo LLM-toolbox community *recommends* `amd_iommu=off` for the 128 GB unified pool (no SVM/large-allocation penalty). Not a compute regression. Re-evaluate the remaining parts: (a) the gaming/compute latency win of disabling vs `iommu=pt` translation-bypass on Zen 5 (likely marginal); (b) the DMA-isolation loss (§10); (c) the README opt-back-in (`amd_iommu=on iommu=pt`, re-run, `_vrkm_iommu` asserts groups populated). Net: accepted single-user-desktop trade-off, not a free latency win, not a compute regression.

Carry-forward params (still validating):

- **processor.max_cstate=1:** capping C-states trades idle power for wake-latency/jitter. Confirm beneficial on Strix Halo for frametime consistency; quantify idle-power/thermal cost on a 140 W package; confirm it does not fight amd_pstate=active or starve boost headroom. Is `1` the right cap?
- **btusb.enable_autosuspend=n:** confirm disabling BT USB autosuspend is the correct fix for MT7925/BT 5.4 reconnect/stutter (alongside the mt7925e ASPM drop-in §8/§11). Does it overlap with `usbcore.autosuspend=-1`, making one redundant?
- **fsck.mode=force + fsck.repair=yes:** forcing fsck every boot on dual NVMe ext4 — confirm safe and intended (boot-time cost, dirty/large-fs behavior, interaction with systemd fsck units and the `fsck` mkinitcpio hook). Auto-repair without prompt: confirm no ext4 data-loss risk. Is "force every boot" the right durability posture, or should it be periodic?
- **clearcpuid=514 (UMIP off):** benefit (avoiding umip_printk stutter under anti-cheat/Wine) vs security cost (descriptor-table leak, taint). Confirm whether current Proton/EAC/BattlEye actually trip UMIP emulation on Zen 5. Recommend dropping if no stutter observed. **Note: the dedicated `_ry_check_umip_disabled` preflight function was REMOVED (v7.77.0)** — the param is now asserted only generically (every KERNEL_PARAMS token present in `/proc/cmdline`, `_vrk_cmdline`); there is no UMIP-specific check.
- **amd_pstate=active:** confirm recommended on Zen 5; interaction with powersave governor + balance_performance EPP (§6) and the new `dynamic_epp=disabled` assert.
- **split_lock_detect=off:** perf vs stability; current default; blast radius.
- No `preempt=` pinned: `_vrk_cmdline` reads the runtime model from dmesg (`Dynamic Preempt: <mode>`) and only INFOs it; `_ok` only when the string contains `full`. **KEEP-omitted — pinning is redundant.** The CachyOS default desktop kernel ships `CONFIG_PREEMPT_DYNAMIC=y` with boot default = full (only `-server` defaults to lazy), so `preempt=full` is a no-op. The INFO-only check is correct.
- Zero amdgpu/ttm module params: confirm hands-off GPU-param posture is correct (`_vrkm_amdgpu` is hex-aware but no-ops when KERNEL_PARAMS carries no `amdgpu.*`).
- Validate the rest: tsc=reliable, nowatchdog, 8250.nr_uarts=0, usbcore.autosuspend=-1, nvme_core.default_ps_max_latency_us=0, pcie_aspm.policy=performance, zswap.enabled=0.
- Sources: docs.kernel.org kernel-parameters + pm/amd-pstate + x86 UMIP + admin-guide (processor.max_cstate, fsck) + IOMMU/AMD-Vi + networking/ipv6-sysctl, wiki.archlinux.org/AMDGPU + IOMMU + fsck + IPv6, amd.com ROCm (IOMMU/SVM).

### 4. Bootloader and initramfs

loader.conf: default @saved, timeout 0, console-mode keep, editor no. sdboot-manage: DEFAULT_ENTRY manual, OVERWRITE/REMOVE_EXISTING/REMOVE_OBSOLETE yes, LINUX_FALLBACK_OPTIONS "quiet". mkinitcpio: MODULES=(amdgpu), HOOKS (11) = base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck, COMPRESSION zstd, COMPRESSION_OPTIONS=(-1 -T0). `mkinitcpio.conf` is pre-deployed in Phase 2 so `pacman -Syu` triggers exactly one initramfs rebuild.

- Verify HOOKS order with the systemd hook (microcode/kms/sd-vconsole/block placement); confirm amdgpu + kms is recommended early-KMS for this GPU. With `fsck.mode=force` on the cmdline (§3), confirm the `fsck` hook + fsck.repair handshake produces no boot prompt or hang.
- **COMPRESSION_OPTIONS=(-1 -T0):** the profile pins zstd level **-1** (negative = fastest/lowest-ratio, not the zstd default of 3) with `-T0` (all threads). Quantify: (a) is the decompression-time delta between `-1` and default-3 measurable at boot on this NVMe (likely sub-100ms), and (b) does the larger `-1` image risk the ESP `BOOT_SPACE_*` gates (§E, 200/500 MB) with multiple kernels + fallback resident? If size cost threatens the ESP budget but the boot-time win is negligible, TUNE toward default-3 or `-T0` alone. Confirm `-T0` actually parallelizes at this level.
- **`install-file` now format-validates content before write (v7.87.x)** and loader.conf/cmdline changes regenerate sdboot entries only (not a full initramfs rebuild). Confirm the Phase-3 pre-deploy + single-rebuild model still holds and that `--install-file /boot/loader/loader.conf` no longer triggers `mkinitcpio -P`.
- timeout 0 + DEFAULT_ENTRY manual + REMOVE_EXISTING=yes wipes foreign BLS entries (EFI loaders untouched); confirm current and intended.
- Confirm sdboot-manage is current and maintained vs kernel-install/UKI (UKI out of scope).
- Sources: wiki.archlinux.org/Mkinitcpio + systemd-boot, sdboot-manage upstream.

### 5. GPU / Vulkan / gaming

No drirc shipped (gfx1151 reports uma:1 natively). No ttm/modprobe params (kernel auto-sizes GTT). ENV_VARS (11): AMD_VULKAN_ICD=RADV, DXVK_LOG_LEVEL=none, DXVK_LOG_PATH=none, MANGOHUD=1, MESA_SHADER_CACHE_MAX_SIZE=16G, PROTON_ENABLE_WAYLAND=1, PROTON_FSR4_RDNA3_UPGRADE=1, PROTON_LOCAL_SHADER_CACHE=1, VKD3D_DEBUG=none, VKD3D_SHADER_DEBUG=none, WINEDEBUG=-all. (Unchanged since v7.77.1.)

- **ntsync (assert-only, autoload conf dropped v7.76.1 — unchanged):** the profile ships no `modules-load.d` autoload; ntsync is asserted in preflight + verify only (`_ntsync_state` 4-state machine builtin|loaded|loaded_nodev|missing survives; `_vss_ntsync_modules` + `_vre_ntsync`). README documents mainline ≥ 6.14 + per-title opt-out `PROTON_NO_NTSYNC=1`. Confirm: (a) ntsync is the current Wine-sync mechanism vs esync/fsync; (b) the CachyOS kernel ships `CONFIG_NTSYNC=y` (builtin) so `/dev/ntsync` exists without any autoload — dropping the conf is correct; (c) the `loaded_nodev` state is a real-enough failure mode to keep the explicit handling; (d) Proton consumes `/dev/ntsync` and improves frametimes on 16C/32T; (e) `PROTON_NO_NTSYNC=1` is the current correct per-title escape. The 6.19 floor (§1) already satisfies ntsync's ≥6.14 requirement — no separate ntsync floor needed.
- **PROTON_FSR4_RDNA3_UPGRADE=1:** confirm current Proton / Proton-CachyOS actually consume this variable to upgrade FSR3.1→FSR4 on RDNA 3.5 (gfx1151), the minimum Proton-CachyOS version, and whether it is a no-op or harmful on titles without FSR. README claims FSR4 reached RDNA3/3.5 via Proton-CachyOS — verify against current releases (on RDNA3.5 also needs `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` to avoid broken visuals). If unverified upstream, flag FIX-to-remove; if verified, KEEP and cite.
- **RADV unified heap (drirc removed):** confirm current RADV on gfx1151 reports uma:1 and treats the heap as unified without the override. If not, flag a regression.
- **GTT sizing (ttm removed):** confirm the installed kernel auto-sizes GTT sensibly (~62 GiB ceiling). README directs >~62 GiB single allocations (ROCm/llama.cpp) to the BIOS UMA carveout (up to 96 GB), not deprecated `amdgpu.gttsize`, verified via `cat /sys/module/ttm/parameters/pages_limit`. `amd_iommu=off` does NOT change the usable GTT/SVM ceiling or break large ROCm allocations (`IOMMU Support: None`). Confirm this is the current correct mechanism and removing the cap does not under-provision compute.
- PROTON_ENABLE_WAYLAND=1: maturity and fallback on current Proton.
- AMD_VULKAN_ICD=RADV: confirm it reliably forces RADV vs VK_DRIVER_FILES.
- MESA_SHADER_CACHE_MAX_SIZE=16G + PROTON_LOCAL_SHADER_CACHE=1: confirm no conflict, sane sizing.
- MANGOHUD=1 global enable: confirm low-overhead vs per-launch, clean with gamescope/GameMode.
- Sources: docs.mesa3d.org (RADV, APU heap), gitlab.freedesktop.org/mesa + drm/amd (GTT auto-sizing kernel version), github Proton/Proton-CachyOS (FSR4, ntsync), amd.com ROCm, wiki.cachyos.org.

### 6. CPU performance and power

amd_pstate=active; governor **powersave** (honors EPP under active mode; sourced from `/etc/default/cpupower-service.conf` `GOVERNOR`); EPP **balance_performance** via udev (`ACTION=="add|change"`, `KERNEL=="cpu[0-9]*"` — corrected v7.85, the rule previously never fired; re-asserts after AC/DC); **GPU clock-floor GPU_DPM_LEVEL=auto** (parameterized; add-only udev rule; `KERNEL=="card[0-9]*"` + `DEVTYPE=="drm_minor"`, v7.79). Masked: power-profiles-daemon, ananicy-cpp, modemmanager. Installed: realtime-privileges, rtkit. **NEW verify assert: `amd_pstate/dynamic_epp == disabled`.**

- **governor=powersave + EPP=balance_performance (LIVE, special case):** under `amd_pstate=active` only the `powersave`/`performance` pseudo-governors exist; `powersave` honors the EPP hint (dynamic scaling), `performance` pins max and ignores EPP. The active + powersave + balance_performance triple IS the documented EPP-honoring max-perf config on Zen 5. Do not flag powersave; do not flip the governor. The `dynamic_epp=disabled` assert (NEW) confirms the kernel accepts the udev EPP pin (a non-disabled dynamic_epp returns `-EBUSY` on write; the node is absent pre-6.16, so the 6.19 floor guarantees it). The `balance_performance` → `performance` EPP question (does mid-bias leave 1%-lows on the table?) is **UNCERTAIN** — no gfx1151/Zen-5 gaming frametime comparison between the two EPP values exists. Mark the EPP=performance opt-in UNCERTAIN; if such a comparison appears, the change would be the udev EPP ATTR only (`CPUPOWER_GOVERNOR` stays powersave). IMPACT UNCERTAIN · RISK Low.
- **GPU_DPM_LEVEL=auto:** the profile does not pin SCLK; accepted set is `_RY_DPM_LEVELS` (auto low high manual profile_standard profile_min_sclk profile_min_mclk profile_peak perf_determinism), validated by `_ir_validate_keys`. Confirm `auto` is the right default — (a) does leaving DPM at `auto` cost frametime/1%-lows on gfx1151 vs pinning `high`, or was pinning only burning idle power? (b) on a shared 140 W package with EPP=balance_performance, does `auto` correctly let firmware arbitrate CPU↔GPU power? (c) any title class where `high` still wins enough to warrant a documented opt-in? The udev rule is `add`-only (one-shot at enumeration): if a reviewer recommends `high`, the rule would NOT re-assert after a GPU reset — a reason `auto` is the more robust default.
- Confirm udev add|change EPP pinning is robust vs one-shot; `_post_udev` now retriggers both block and cpu devices so the EPP rule live-applies (v7.87.x). prefcore=enabled + boost=1 correct on Strix Halo.
- Mask power-profiles-daemon: confirm no needed platform_profile path lost; does CachyOS expect ppd? Consider tuned.
- Mask ananicy-cpp: confirm net win for gaming.
- Mask modemmanager: confirm no cellular HW, masking loss-free. (Note: the prior `_kb_modemmanager_masked` benign-INFO sub was REMOVED — the masked-ModemManager D-Bus log line is no longer surfaced by the installer.)
- Sources: docs.kernel.org pm/amd-pstate (active + EPP + governor + dynamic_epp), wiki.archlinux.org/CPU_frequency_scaling + AMDGPU (power_dpm_force_performance_level), freedesktop ppd.

### 7. Memory and storage

zswap.enabled=0; NVMe scheduler none (udev `99-ry-perf.rules`, sorts after vendor 60-ioschedulers.rules); **SYSCTL_VALUES (9):** net.core.default_qdisc=fq, net.core.netdev_budget=600, net.core.netdev_budget_usecs=5000, net.ipv4.tcp_congestion_control=bbr, net.ipv4.tcp_notsent_lowat=16384, net.ipv4.tcp_slow_start_after_idle=0, vm.compaction_proactiveness=0, vm.max_map_count=2147483642, vm.swappiness=150 (priority 95 `95-ry-overrides.conf` after vendor 70-cachyos-settings.conf); fstab ext4 noatime,lazytime,commit=10; THP/KSM/systemd-oomd left to CachyOS. `vm.page-cluster` and `vm.vfs_cache_pressure` remain DROPPED (vendor duplicates). (Unchanged since v7.77.1.)

- **Confirm the vendor-duplicate drop is still a no-op:** confirm CachyOS `70-cachyos-settings.conf` (or another vendor sysctl.d file) actually sets `vm.page-cluster=0` and `vm.vfs_cache_pressure=50` — the profile's removal must leave the same effective values. If the vendor default differs, the removal is a SILENT change for zram readahead and cache reclaim — flag it. List exact vendor values.
- **zram pair:** confirm the running kernel accepts swappiness > 100; is 150 gratuitous on 128 GB RAM, or does it help large-alloc/LLM reclaim? The priority-95 file overrides just swappiness/compaction/max_map_count from vendor 70-cachyos — confirm justified.
- zswap.enabled=0: confirm CachyOS uses zram (not zswap) by default — no double-compression conflict.
- NVMe scheduler none: confirm best practice vs mq-deadline/kyber on this kernel. **`nr_requests` and `read_ahead_kb` are NOT set by the profile** (only `scheduler=none`). Determine whether tuning either materially helps game-load/asset-streaming or large-sequential LLM-weight reads, and if so propose concrete udev ATTR values; else state kernel defaults are optimal. Confirm `99-ry-perf.rules` sorts after vendor `60-ioschedulers.rules` (last-matching ATTR assignment wins).
- fstab noatime,lazytime,commit=10: confirm noatime and lazytime coexist; weigh commit=10 durability (esp. with `fsck.mode=force` every boot §3); confirm fstrim.timer over continuous discard.
- vm.max_map_count near 2^31: confirm appropriate for Proton/anti-cheat. compaction_proactiveness=0: confirm right for gaming + large unified allocs.
- Confirm not enabling systemd-oomd (kernel OOM + zram) is right on 128 GB.
- Sources: docs.kernel.org (block, sysctl/vm), wiki.archlinux.org/Zram + SSD + Ext4, wiki.cachyos.org (zram + sysctl defaults).

### 8. Network and latency

sysctl net: default_qdisc=fq, netdev_budget=600, netdev_budget_usecs=5000, tcp_congestion_control=bbr, tcp_notsent_lowat=16384, tcp_slow_start_after_idle=0. **IPv6 disabled system-wide (`ipv6.disable=1`, §3); the nftables ruleset is IPv4-only (§10).** NM: wifi.backend=wpa_supplicant (NM default; iwd opt-in via NM_WIFI_BACKEND), wifi.powersave=2 (off), logging WARN. `/etc/modprobe.d/60-ry-mt7925e.conf` → `options mt7925e disable_aspm=1`. resolved: MulticastDNS=no, LLMNR=no, DNSOverTLS=no, DNSSEC=allow-downgrade (plaintext; diverges from CachyOS DoH default). regdom: COUNTRY=US fixed (`/etc/iw-regdomain`; reserved/user-assigned ISO codes now rejected). Masked: NetworkManager-wait-online, modemmanager. Enabled: NetworkManager.

- **IPv6 fully disabled (NEW):** with `ipv6.disable=1` the kernel brings up no IPv6 stack. Evaluate the LAN impact (§3, §10) — confirm no dependency on SLAAC/link-local/IPv6-only services on this dual-10GbE network; confirm mDNS-off + IPv6-off do not compound to break any discovery the user relies on. This is why the nftables ruleset dropped all ICMPv6/NDP rules.
- **mt7925e disable_aspm=1:** confirm disabling PCIe ASPM on MT7925 is the current correct mitigation for coredump/BT-reconnect/assoc-fail, and whether an upstream mt76 fix has landed (→ if landed, prefer a kernel/firmware floor and flag the drop-in as removable). Confirm it does not fight `pcie_aspm.policy=performance` (§3) — are both saying "no ASPM" on this device, making one redundant?
- **NM backend wpa_supplicant:** verify current Arch/CachyOS guidance for MT7925/mt76 — is wpa_supplicant the more stable backend today, or has iwd matured to parity? Confirm wifi.powersave=2 alone fully disables the mt76 software power-save latency issue under wpa_supplicant, and that backend + ASPM drop-in + btusb.enable_autosuspend=n (§3) together close the MT7925 stability gap. Confirm no dangling iwd reference.
- bbr + fq: confirm still recommended; clarify BBRv3 status in current kernels.
- Dual 10 GbE: validate netdev_budget/usecs for 10G; assess tcp_rmem/wmem or NIC ring tuning for line-rate (RTL8127, §11).
- tcp_notsent_lowat=16384, tcp_slow_start_after_idle=0: confirm rationale holds.
- mDNS off (consistent with the IPv4-only default-deny §10); plaintext DNS reverting DoH — confirm CachyOS ships an encrypted-DNS default this overrides; flag the privacy reduction (§10).
- regdom US fixed: confirm correct max TX power and channel set for MT7925 on current cfg80211/wireless-regdb. Flag if 6 GHz Wi-Fi 7 needs a specific AFC posture beyond a country code; note non-US deployers must hand-edit COUNTRY. (README notes the `3 dBm` TX-power readout is cosmetic — correct power applied; confirm.)
- Sources: docs.kernel.org/networking (bbr, fq, tcp, ipv6-sysctl), wiki.archlinux.org/Sysctl + NetworkManager + Wireless/MediaTek + IPv6, git.kernel.org wireless-regdb + mt76.

### 9. systemd units, time-sync

Mask (10): ananicy-cpp, power-profiles-daemon, NetworkManager-wait-online, ufw, modemmanager, sleep/suspend/hibernate/hybrid-sleep/suspend-then-hibernate targets. Enable (5): fstrim.timer, NetworkManager, cpupower, nftables, bluetooth. Not enabled: systemd-oomd (intentional), NetworkManager-dispatcher (socket-activated), rtkit-daemon (socket-activated). iwd.service untouched (opt-in). ufw flushed after nftables live. `_ry_rtc_writeback` (`hwclock --systohc --utc`) at both sync-confirmed paths; NTP remediation can be skipped with `RY_NO_NTP_REMEDIATION=1` (v7.85) — an unsynced clock with no NTP client enables `systemd-timesyncd` + RTC writeback.

- For each mask, confirm safe and beneficial on CachyOS: ananicy-cpp + ppd (§6); modemmanager (no cellular HW); sleep/suspend masked = no suspend at all (matches an always-on mini-PC).
- **RTC write-back (`_ry_rtc_writeback`):** confirm `hwclock --systohc --utc` after NTP sync is safe and that the `RTCInLocalTZ=yes` guard (defer to systemd) is the correct branch. Confirm the rationale — a skewed RTC poisoning systemd-timer persistence stamps (`Persistent=true` timers) — is real, and that a UTC-only direct write does not conflict with `systemd-timesyncd` ownership of the RTC. Confirm `RY_NO_NTP_REMEDIATION=1` cleanly disables the timesyncd-enable path (v7.85).
- bluetooth.service enabled (with main.conf §12): confirm AutoEnable=true posture and BlueZ key currency.
- Confirm nftables is enabled as the firewall and that ufw-flush-then-mask leaves no unfirewalled window (script skips the ufw mask if nft is not confirmed live — confirm this handoff). nftables.service is a oneshot with no RemainAfterExit — verify judges it by the live ruleset, not unit-active state.
- fstrim.timer vs continuous discard; cpupower vs CachyOS freq management; confirm oomd stays disabled and dispatcher/rtkit stay socket-activated.
- logind Handle*Key=ignore incl LongPress: confirm intended, no lockout risk.
- Sources: man.archlinux.org (systemd.unit, logind.conf, hwclock, systemd-timesyncd), wiki.archlinux.org (Bluetooth, System time), wiki.cachyos.org.

### 10. Security and safety (cross-cutting)

nftables **IPv4-only** default-deny-inbound (ufw masked; `ipv6.disable=1` §3): input policy drop, loopback accept (first), ct established/related accept, ct invalid drop, **IPv4 ICMP `{ echo-request, destination-unreachable, time-exceeded, parameter-problem }` accept (inbound ping now ALLOWED)**, forward drop, output accept. **No ICMPv6/NDP rules** (IPv4-only). `RY_REMOTE_PLAY_PORTS` gate (default false) appends `tcp dport { 47984, 47989, 48010, 27036, 27037 }` + `udp dport { 47998-48010, 27031-27036 }`. **amd_iommu=off (AMD-Vi DISABLED).** clearcpuid=514 (UMIP off). split_lock_detect=off.

- **IPv6 disabled + inbound IPv4 ping accepted (NEW — the biggest §10 change):** the ruleset was rewritten IPv4-only. Quantify both directions: disabling IPv6 removes an entire stack (reduction) but loses IPv6 functionality; accepting inbound `echo-request` makes the host pingable on the LAN (minor increase in discoverability). Confirm both are intended on a trusted dual-10GbE LAN. The hard coupling (`_ir_validate_keys` refuses deploy if nftables is managed without `ipv6.disable=1`) exists because an IPv4-only ruleset would let ICMPv6/ND hit the policy-drop and break IPv6 neighbor discovery — so IPv6 must be off at the kernel, not merely unfirewalled. The README opt-back-in restores IPv6 + IPv6 firewall rules.
- **amd_iommu=off (the #2 open reduction):** quantify the DMA-isolation loss with AMD-Vi fully off (USB4/Thunderbolt, NVMe, NIC DMA unmediated). Confirm intended for a no-passthrough single-user desktop and that the README escape hatch (`amd_iommu=on iommu=pt`) is correct. The ROCm/SVM concern is resolved — `amd_iommu=off` does not break ROCm on gfx1151 (`IOMMU Support: None`) — so the trade-off is purely security (DMA isolation) vs latency, and it is accepted.
- **RY_REMOTE_PLAY_PORTS port set (now includes 27037):** validate the exact ports against current Sunshine/Moonlight + Steam Remote Play. Confirm `47984/47989/48010` (Sunshine HTTPS/HTTP/RTSP), `27036/27037` (Steam Remote Play), and the UDP ranges are correct and complete — flag any missing port (Sunshine video/control/audio/mic UDP) or stale one. Confirm default-OFF is right and that enabling appends cleanly. (27037 was added v7.79/7.80 — confirm it is the correct additional Steam port.)
- **Inbound IPv4 ping (`echo-request`) is now a REGRESSION GUARD, not a prohibition:** `_vss_nft` hard-fails if the `echo-request` accept is MISSING (the ping accept must stay enabled); `_vrsv_nft_assert_ping` warns if the live chain lacks it. This inverts the prior "no inbound ping" posture. Confirm `destination-unreachable` accept still preserves IPv4 PMTUD (frag-needed).
- Validate the nftables shape is minimal-but-sufficient on a dual-10 GbE LAN; `ct state invalid drop` sits after lo + established — confirm it cannot drop a valid loopback/established packet.
- clearcpuid=514 is the headline open reduction; quantify exposure vs the umip_printk stutter prevented (§3); list first.
- split_lock_detect=off: quantify residual exposure.
- `flush ruleset` blast radius: confirm wiping all nft tables (not just inet filter) is safe vs docker/libvirt/podman on this host.
- Produce the ordered security-delta subsection (above), with amd_iommu=off at position 2 and the IPv6-off/ping-on pair at position 3.
- Sources: wiki.archlinux.org (nftables, Security, IOMMU, IPv6), docs.kernel.org (split lock, UMIP, AMD-Vi, ipv6-sysctl), github Sunshine/Moonlight (port reference).

### 11. Known issues and DKMS currency

MES page faults → framed by the current script/README as **firmware-currency-resolved** via the kernel floor + `mkinitcpio-firmware` (README: "resolved (MES 0x86; current linux-firmware + shipped mkinitcpio-firmware)"). **The version-specific preflight advisory (`20251125*` hard-warn / `<20260110` soft-warn) was REMOVED (v7.81/7.82).** See the headline MES-0x86 conflict note — re-verify the exact shipping gfx1151 MES revision and whether "0x86" is current-correct. RTL8127 throughput + suspend/shutdown hang → resolved in-tree r8169 (`f24f7b2f3af9` support; `ae1737e7339b` suspend/shutdown-hang fix), guaranteed at the ≥6.19 floor; no DKMS. MT7925 panics/deauth/coredump → mitigated via `disable_aspm=1` drop-in (§8) + `btusb.enable_autosuspend=n` (§3) + wpa_supplicant; upstream status improving, not fully closed — the drop-in stays defensive. Strix Halo ACP → still open upstream (no ACP70 internal-mic ASoC machine driver / UCM profile as of mid-2026); internal mic undetected. **Note: the `_kb_acp70_no_machine_driver` benign-INFO sub was REMOVED (v7.77.0)** — the installer no longer surfaces the ACP mic gap; it remains a real hardware gap the profile cannot fix. Install pacman-only.

- **MES firmware — RESOLVE THE LABEL CONFLICT.** Determine against current upstream which MES microcode revision the shipping gfx1151 GC 11.5.1 blob reports and whether the script's "0x86" label is current-correct, a stale carry-over, or a distinct revision. The prior prompt's 0x83-bad/0x80-good framing is NOT authoritative here — the script/README (source of truth) say 0x86-resolved. Report the disagreement + trusted source; do not silently adopt either. Firmware hashes are secondary-corroborated (Launchpad #2129150 / Debian changelog / ROCm #5724); git.kernel.org blocked direct fetch in prior rounds — retry at audit time.
- **MT7925:** mt7925e stability fixes landed 6.17+, already covered by the 6.19 floor; keep the `disable_aspm=1` drop-in defensive (instability still reported on some platforms). wpa_supplicant vs iwd remains a wash — keep wpa_supplicant. Status: improving, not a closed gap.
- **ACP/internal mic — still open upstream.** No ACP70 internal-mic machine driver / UCM profile exists as of mid-2026 (alsa-ucm-conf has no acp70 profile; only ASUS TAS2783 speaker SoundWire quirks landed ~kernel 7.0). Nothing the profile can ship — document as a known gap; the installer no longer flags it (the `_kb_*` surface is gone). Reporting the board model upstream remains the correct action.
- Recommend a kernel/firmware floor over DKMS for any landed fix; confirm any suggested DKMS still builds.
- Sources: gitlab.freedesktop.org/drm/amd, git.kernel.org linux-firmware + r8169 (`f24f7b2f3af9`, `ae1737e7339b`) + mt76, bugzilla.kernel.org, discuss.cachyos.org.

### 12. MangoHud, Bluetooth, and hygiene

**MangoHud.conf (19 active directives, 0 commented, 0600, fps/frametime first):** horizontal, legacy_layout=0, position=top-left, toggle_hud=Shift_R+F12, fps, frametime, frame_timing, gpu_stats, gpu_core_clock, gpu_temp, gpu_power, cpu_stats, **`cpu_temp` (now LIVE/uncommented)**, cpu_mhz, vram, ram, font_size=20, text_outline, background_alpha=0.4. Still fully absent: fps_metrics. Enabled via MANGOHUD=1. bluetooth main.conf: FastConnectable=true, AutoEnable=true, ReconnectAttempts=3 (no explicit ReconnectIntervals — BlueZ default backoff). **baloofilerc DE-MANAGED (v7.77.0) — no longer written; USER_DESTINATIONS = 2.**

- **MangoHud `cpu_temp` is SHIPPED LIVE (the framing inverted since v7.77.1):** the directive is now an active bareword. Verify: (a) `cpu_temp` populates from `k10temp`/Tctl on Zen 5 as shipped (does current MangoHud auto-detect the Zen 5 sensor, or is an explicit `cpu_temp_sensor=k10temp` still needed? — MangoHud #1825 established the key name); (b) the open MangoHud #1794 caveat now applies to the SHIPPED config — enabling `cpu_temp` can zero the `cpu_power` readout on Zen 5, and `cpu_power` is NOT present in this config (only `gpu_power` is), so confirm whether the #1794 interaction is even triggered here or is moot without `cpu_power`. If `cpu_temp` reads blank/zero on the installed MangoHud + kernel, that is a real FIX (add `cpu_temp_sensor=k10temp`); if it populates, KEEP.
- Confirm all 19 active directives are valid for current MangoHud (do not invent). With `gpu_power` present, confirm it populates from amdgpu sensors on gfx1151 under Wayland. Confirm `frame_timing` (graph) vs `frametime` (numeric) are both current and not redundant. Confirm `toggle_hud=Shift_R+F12` is a valid current bind.
- Confirm gpu_temp/gpu_core_clock/vram/cpu_mhz populate from amdgpu sensors under Wayland on this iGPU; confirm vram + ram is the right unified-memory representation.
- Overhead: confirm near-zero, no conflict with the gamescope overlay or GameMode.
- **Bluetooth main.conf:** confirm BlueZ keys/sections current; ReconnectAttempts=3 + BlueZ default backoff sane for paired audio sinks; AutoEnable=true fixes adapter-off-at-boot; FastConnectable=true no meaningful downside on an always-on desktop. Cross-check `btusb.enable_autosuspend=n` (§3) — confirm kernel-level autosuspend-off and BlueZ-level reconnect policy are complementary, not redundant.
- **Removed surfaces (do NOT audit — gone since v7.77.1):** baloofilerc / Indexing-Enabled (de-managed); the 5 `_kb_*` benign-INFO subs (`_kb_modemmanager_masked`, `_kb_acp70_no_machine_driver`, `_kb_thunderbolt_nhi_unknown`, `_kb_no_battery_backlight`, `_kb_usb_mic_volume_curve`) and `_vss_known_benign`; the `_ry_check_umip_disabled` preflight. The underlying hardware facts remain real but the installer no longer surfaces them.
- **Naming:** every human-facing string reads "GTR9 Pro" but the internal PROFILE_NAME and function names carry `gtr_pro` (FIX, Low/Low) — do not change if it breaks function-name refs or log fields; cosmetic.
- Sources: github flightlessmango/MangoHud (config ref, version tags), wiki.archlinux.org/MangoHud + Bluetooth, man.archlinux.org bluetooth main.conf.

## 13. Candidate enhancements (absent knobs — gaming-first; ADD-opt-in vs KEEP-omitted)

Additive: each item is a knob the profile does NOT set, evaluated for whether it should be ADDED (default or documented opt-in) or whether the omission is correct (KEEP). Anchor every call to gfx1151 / Zen 5 / RDNA 3.5 / current Mesa+Proton-CachyOS. Reserve ADD-as-default only for a clear, low-risk frametime/throughput win. Never invent a flag — cite upstream or mark UNCERTAIN. (Carried from v7.77.1 Round 2; the script did not add any of these — re-confirm calls against current upstream.)

### 13a. Kernel cmdline candidates

- **`mitigations=off` — KEEP-omitted.** Zen 5 (Ryzen AI 300 / 9000) is not affected by Inception/SRSO, and `mitigations=off` yields no measured benefit outside synthetic kernel microbenchmarks. Residual default mitigations are HW/microcode-handled. No Proton/Wine gaming delta for this APU. KEEP-omitted; do NOT add to the security delta. Re-open as ADD-opt-in only if a published gfx1151 Proton frametime delta > ~2% appears. IMPACT Low · RISK Med (security).
- **`amdgpu.ppfeaturemask=0xffffffff` — KEEP-omitted.** GPU undervolt/OC is NOT implemented for Strix Halo: `pp_power_profile_mode`/overdrive/power-cap report "Not supported" on gfx1151 even with the full mask (ROCm #5750), and the GPU cannot exceed the shared 140 W package cap. The real power lever is CPU undervolt via `ryzenadj` (out of scope for a kernel cmdline). Keep removed. IMPACT Low · RISK Med.
- **`preempt=full` — KEEP-omitted, redundant.** CachyOS default desktop kernel ships `CONFIG_PREEMPT_DYNAMIC=y` boot-default full (only `-server` = lazy); pinning is a no-op. See §3. IMPACT none · RISK none.
- **`nvme_core.io_timeout` / `pcie_port_pm=off` — KEEP-omitted.** Redundant alongside `nvme_core.default_ps_max_latency_us=0` + `pcie_aspm.policy=performance`; no concrete stutter/latency case. IMPACT Low · RISK Low.

### 13b. RADV / Mesa env candidates

- **`RADV_PERFTEST` — KEEP-omitted (gpl/sam) / UNCERTAIN (nggc).** `gpl` is default-on since Mesa 23.1 (disable via `RADV_DEBUG=nogpl`); `sam` is auto-enabled when all VRAM is CPU-visible (the APU case) — both KEEP-omitted. `nggc` (NGG culling) is plausibly relevant but has no gfx1151 benchmark → UNCERTAIN, do not ADD on a guess. `rtwave64` hurts RDNA2; ignore. IMPACT Low · RISK Low.
- **`RADV_DEBUG=novrsflatshading` / correctness toggles — KEEP-omitted.** Only if a known gfx1151 rendering bug requires one — none currently identified. Flag any open RADV gfx1151 issue a toggle works around at audit time.
- **`MESA_VK_WSI_PRESENT_MODE` / `vblank_mode` — KEEP-omitted (per-game).** Forcing a present mode belongs per-title, not host-wide. Document the per-game pattern.
- **`mesa_glthread=true` — KEEP-omitted.** OpenGL-threading helps only native/older GL titles; most Proton games are Vulkan via DXVK/VKD3D. IMPACT Low · RISK Low.

### 13c. DXVK / VKD3D-Proton candidates

- **DXVK config (`dxvk.conf`) — KEEP-omitted (auto optimal).** `dxvk.enableGraphicsPipelineLibrary=Auto` is default-on (enables GPL when supported) and `dxvk.numCompilerThreads=0` auto-detects all cores — pinning gives no measured stutter improvement on 16C/32T. Legacy `DXVK_ASYNC` is unsupported/superseded by GPL (`gplAsyncCache` removed in DXVK 2.7) — do NOT recommend the old async patch. IMPACT Low · RISK Low.
- **`PROTON_ENABLE_NGX_UPDATER` / upscaler envs — KEEP-omitted beyond §5.** Beyond `PROTON_FSR4_RDNA3_UPGRADE=1` (§5; on RDNA3.5 also needs `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround`), no host-wide upscaler/frame-gen env is frametime-relevant enough to globalize. Keep upscaler scope per-title.
- **`VKD3D_CONFIG` — KEEP-omitted (per-game).** DX12 toggles like `dxr` belong per-title. IMPACT Low · RISK Low.

### 13d. Firmware / platform (verify-only)

- **Resizable BAR / SAM — verify-only, auto-on.** On Strix Halo all VRAM is CPU-visible (rocminfo shows the full unified pool), so RADV auto-enables sam optimizations — no need to force `RADV_PERFTEST=sam`. Optional advisory INFO for ReBAR-off via rocminfo pool sizes / `lspci -vv` BAR sizes / amdgpu dmesg. Low priority.
- **BIOS UMA carveout vs GTT — KEEP-omitted (gaming).** For gaming (not ROCm), the default GTT ceiling (~62 GiB) never bottlenecks a game's VRAM working set on the 128 GB pool. The README carveout note is compute-oriented; gaming needs nothing.

### 13e. Scheduler / memory (only the gaps)

- **`read_ahead_kb` / `nr_requests` — KEEP-omitted, defaults optimal.** Under `none`, smaller read-ahead (128 KB) is the guidance for NVMe random I/O; defaults are appropriate. Game load is CPU-decompression/shader-bound; no game-load or LLM-weight benchmark shows benefit from raising either. Add no knob. IMPACT Low · RISK Low.
- **`vm.max_map_count` — KEEP (sufficient).** 2147483642 (MAX_INT−5, the SteamOS value) satisfies all current Proton/anti-cheat requirements. No change. (Arch default is 1048576; the profile's raised value is the established max-compat value.)
- **CPU affinity / isolation (`isolcpus`, `nohz_full`, `rcu_nocbs`) — KEEP-omitted (wrong here).** Core isolation on a 16C/32T gaming desktop hurts: it removes cores from the scheduler and breaks GameMode/Proton thread placement. IMPACT Low · RISK Med (if added).

For each 13a–13e item: ITEM · PRESENT?(no) · CALL(ADD-default / ADD-opt-in / KEEP-omitted) · IMPACT · RISK · EVIDENCE. Bias toward KEEP-omitted unless the gaming win is concrete and low-risk; the profile is intentionally lean.

---

## Scope and non-goals

- Recommendations only — do not emit a modified script.
- Out of scope: dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS, laptops, UKI.
- Per-game Proton tuning is secondary; prioritize system-wide config.
- **Reinstatement rule.** Items deliberately removed/disabled (do not recommend reinstating unless current upstream directly contradicts the rationale — then flag, not FIX): `amdgpu.ppfeaturemask` (re-evaluate per §13a as an undervolt/OC opt-in, not a silent default), `--country` flag, TTM/GTT cap, RADV drirc, MangoHud `fps_metrics`, `vm.page-cluster`/`vm.vfs_cache_pressure` (vendor-provided), ntsync `modules-load.d` autoload conf (now assert-only), baloofilerc (de-managed v7.77.0), the `_kb_*` benign-INFO subs + `_ry_check_umip_disabled` (removed v7.77.0), ICMPv6/NDP nftables rules (dropped v7.87.x — do NOT re-add without restoring IPv6). Live config to evaluate as KEEP-or-FIX-to-remove (not protected): PROTON_FSR4_RDNA3_UPGRADE, MangoHud gpu_power/text_outline/toggle_hud/**cpu_temp (now live)**, `ipv6.disable=1`, inbound-ping accept.
- **IOMMU special case:** the profile ships `amd_iommu=off` (AMD-Vi fully disabled). Do NOT recommend re-adding `iommu=pt`/`amd_iommu=on` as a default unless ROCm/compute on gfx1151 is proven to require the IOMMU (it is not — `IOMMU Support: None`), OR a DMA-isolation requirement is established. The VFIO/SR-IOV opt-in (`amd_iommu=on iommu=pt`) is the documented per-user override.
- **IPv6/nftables special case (NEW):** the profile ships `ipv6.disable=1` + an IPv4-only ruleset that ACCEPTS inbound ping. Do NOT flag inbound-ping-accept as a regression (it is an asserted regression-guard) and do NOT recommend re-adding ICMPv6/NDP rules without also restoring IPv6 (`ipv6.disable=1` removed). The dual-stack opt-in (drop the token + restore IPv6 firewall rules + re-run) is documented.
- **Governor/EPP special case:** `powersave` + `balance_performance` is the EPP-honoring config under `amd_pstate=active` (with `dynamic_epp=disabled` asserted) — do not flag `powersave` as a regression without proving the `performance` governor would override the EPP hint.
- **GPU_DPM_LEVEL special case:** `auto` is deliberate. Do not flag `auto` as a GPU-perf regression without proving `high` materially improves frametime/1%-lows on gfx1151 without costing CPU boost on the shared 140 W package.

---

# Deep-pass appendix (exact generated bodies + full verify surface)

The §1–§13 investigation is value-level. This appendix is artifact-level: the exact strings the script writes, the complete verify subsystem, and the install-phase model. Validate the **rendered file content**, not a paraphrase. Every block below is quoted from the generator functions in `ry-install.fish` v7.88.3 (UUIDs/joins resolved at runtime).

## A. Install-phase model (`_RY_PHASE_NAMES`)

Six ordered phases; recommendations must respect this sequence:

```
1 Preflight     _install_preflight     — _ir_* gates (counts, keys, kernel floor, post-hooks, root UUID); mesa soft-floor advisory (NO firmware advisory — removed)
2 Packages      _install_packages      — mkinitcpio.conf pre-deployed → pacman -Syu PKGS_ADD (one initramfs rebuild); chwd Vulkan
3 Configuration _install_system_files  — render+deploy all managed files (atomic tmp+rename); install-file format-validates before write
4 Services      _install_configure_services — fstab rewrite + resolved + PKGS_DEL (-Rns) + mask (nft-first, then ufw flush) + iwd handoff + enable + regdom + RTC write-back (skip: RY_NO_NTP_REMEDIATION=1)
5 Boot          _install_rebuild_boot  — taint-gate → mkinitcpio -P + sdboot-manage gen/update (gated on boot-critical writes)
6 Finalize      _install_finalize      — user daemon-reload + paccache (-rk2 and -ruk0 as separate runs, v7.85) + NetworkManager restart
```

- Confirm the **firewall handoff lives in Phase 4** (nftables made live before ufw is flushed/masked) and that boot-critical regeneration (Phase 5) only fires when one of `_RY_BOOT_CRITICAL_DSTS` changed. Flag any recommendation that would move a cmdline/mkinitcpio change outside the Phase-5 gate.
- `_RY_BOOT_CRITICAL_DSTS` (4): `/boot/loader/loader.conf`, `/etc/kernel/cmdline`, `/etc/sdboot-manage.conf`, `/etc/mkinitcpio.conf`. `_RY_BACKUP_TARGETS` (2, `.ry.bak`): `/boot/loader/loader.conf`, `/etc/mkinitcpio.conf` (plus fstab during its rewrite). Confirm the backup set is sufficient (`/etc/kernel/cmdline` is regenerable from `KERNEL_PARAMS`).
- **`_RY_POST_HOOKS` (17 entries, 16 distinct tags — baloo REMOVED):** `/boot/*|loader`, `/etc/kernel/cmdline|cmdline`, `/etc/sdboot-manage.conf|boot`, `/etc/mkinitcpio.conf|boot` (boot appears twice → one distinct tag), `*/resolved.conf.d/*|resolved`, `*/logind.conf.d/*|logind`, `*/NetworkManager-dispatcher.service.d/*|nmdispatch`, `*/NetworkManager/conf.d/*|nm`, `/etc/iw-regdomain|regdom`, `/etc/bluetooth/main.conf|bluetooth`, `/etc/nftables.conf|nft`, `/etc/default/cpupower-service.conf|cpupower`, `*/sysctl.d/*|sysctl`, `/etc/udev/rules.d/*|udev`, `*/modprobe.d/*|modprobe`, `*/environment.d/*|envd`, `*/MangoHud/MangoHud.conf|mangohud`. `_ir_validate_post_hooks` refuses deploy if any tag lacks a `_post_<tag>` handler. **There is no `_post_baloo`** (baloo de-managed v7.77.0). Confirm `--install-file <path>` of any single managed file triggers the correct reload and that modprobe/udev/cmdline handlers correctly defer to reboot.

## B. Exact rendered file bodies (validate content, not summary)

### B1. `/etc/kernel/cmdline` + `/etc/sdboot-manage.conf`

```
rw root=UUID=<_ROOT_UUID> 8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=514 fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0
```
sdboot-manage.conf: `LINUX_OPTIONS` = same KERNEL_PARAMS join; `LINUX_FALLBACK_OPTIONS="quiet"`; `DEFAULT_ENTRY="manual"`; `REMOVE_EXISTING="yes"`; `OVERWRITE_EXISTING="yes"`; `REMOVE_OBSOLETE="yes"`.

- **`amd_iommu=off` AND `ipv6.disable=1` now ship in BOTH bootloader-management paths.** Confirm `/etc/kernel/cmdline` (kernel-install) and `sdboot-manage.conf` `LINUX_OPTIONS` (sdboot-manage) are not simultaneously active in a conflicting way; state which CachyOS actually drives and whether maintaining both is redundant or a divergence risk.
- `LINUX_FALLBACK_OPTIONS="quiet"` strips ALL params from the fallback entry. Confirm a fallback boot with none of `amd_pstate`/`fsck.*`/`amd_iommu=off`/`ipv6.disable=1` is the intended recovery posture (minimal fallback — but the fallback boots with the *kernel default* IOMMU state AND IPv6 ENABLED; with the main-entry ruleset IPv4-only, a fallback boot re-enables the IPv6 stack the firewall does not cover — confirm that asymmetry is harmless on this firmware, or note it as a fallback-only exposure window).

### B2. `/boot/loader/loader.conf`
```
default @saved
timeout 0
console-mode keep
editor no
```
- `default @saved` + `timeout 0` + `editor no`: confirm `@saved` resolves on systemd-boot with timeout 0; does a failed boot still let the user reach the menu, or a recovery dead-end requiring external media? Flag the recovery-ergonomics trade (README points failures to live-USB → arch-chroot). loader.conf changes now regenerate sdboot entries only (not a full initramfs rebuild; v7.87.x).

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
        # [IFF RY_REMOTE_PLAY_PORTS=true:]
        tcp dport { 47984, 47989, 48010, 27036, 27037 } accept
        udp dport { 47998-48010, 27031-27036 } accept
    }
    chain forward { type filter hook forward priority filter; policy drop; }
    chain output { type filter hook output priority filter; policy accept; }
}
```
- **IPv4-ONLY (the major B3 rewrite).** All ICMPv6/NDP accept rules were DROPPED (v7.87.x); the ruleset carries no `ip6`/`icmpv6` types. This is safe ONLY because `ipv6.disable=1` removes the IPv6 stack entirely (`_ir_validate_keys` hard-couples the two). Confirm the coupling holds and that no IPv6 neighbor-discovery breakage results (there is no IPv6 to break).
- **Rule-order:** `iif "lo" accept` is FIRST (moved v7.85), then `established,related`, then `ct state invalid drop`. Confirm this cannot drop a valid loopback or established packet.
- **IPv4 ICMP (4 types) now INCLUDES `echo-request` (inbound ping ALLOWED).** This inverts the prior posture. `_vss_nft` hard-fails if `echo-request` is MISSING (regression guard). Confirm `destination-unreachable` accept preserves IPv4 PMTUD (frag-needed).
- **No echo-reply accept** — dropped v7.87.x (ct established covers replies). Confirm correct.
- **`flush ruleset`:** confirm wiping the entire ruleset (not just inet filter) is safe vs docker/libvirt/podman nat tables on this host.
- **No rate-limiting on ICMP/new-conn:** confirm acceptable on a trusted LAN.

### B4. udev `99-ry-perf.rules` (exact — 3 rules; matchers CORRECTED)
```
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="balance_performance"
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", DEVTYPE=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="<GPU_DPM_LEVEL>"
```
- **EPP rule matcher CORRECTED (v7.85):** now `SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*"` — the earlier `DEVPATH=="*/cpufreq"` form **never fired** at enumeration. `_post_udev` also retriggers cpu (beside block) so the EPP rule live-applies (v7.87.x). Confirm the rule now matches at boot and on change uevents.
- **GPU rule matcher UPDATED (v7.79):** `KERNEL=="card[0-9]*"` + `DEVTYPE=="drm_minor"` (was `card[0-9]` single-digit, no DEVTYPE). `add`-only (one-shot). Confirm no multi-GPU/render-node concern on this single-iGPU host; if a reviewer recommends `high`, the `add`-only rule would NOT re-assert after a GPU reset — a reason `auto` is more robust.
- `<GPU_DPM_LEVEL>` is interpolated UNQUOTED into the ATTR; `_ir_validate_keys` restricts it to `_RY_DPM_LEVELS`, so injection is bounded.
- Filename `99-` sorts after vendor `60-ioschedulers.rules` — confirm 99 wins (last-matching ATTR assignment).

### B5. `/etc/systemd/resolved.conf.d/99-cachyos-resolved.conf`
```
[Resolve]
MulticastDNS=no
LLMNR=no
DNSOverTLS=no
DNSSEC=allow-downgrade
```
- `99-` + basename `cachyos-resolved`: same filename = replace, not merge. If CachyOS ships its OWN `99-cachyos-resolved.conf` (DoH default), this file *replaces* it — confirm intended override, not an accidental basename clash a CachyOS update would re-overwrite. Restarts skipped when drop-in bytes unchanged (v7.87.x).

### B6. NetworkManager `99-cachyos-nm.conf` + dispatcher `logging.conf`
```
[device]
wifi.backend=wpa_supplicant
[connection]
wifi.powersave=2
[logging]
level=WARN
```
dispatcher: `[Service]` `LogLevelMax=notice` (drops info-level nm-dispatcher journal spam).
- Same basename-override caution as B5. Confirm `LogLevelMax=notice` is the correct journald-noise fix and validate against current NM-dispatcher behavior.

### B7. `/etc/iw-regdomain` + persistence path
```
COUNTRY=US
```
- **Persistence mechanism (name it precisely):** confirm `cachyos-iw-set-regdomain` is the CachyOS unit/script that reads `/etc/iw-regdomain` and runs `iw reg set` at boot — and that it still exists in current CachyOS. If dropped, the file is inert and the profile must switch to `wireless-regdb`/`crda` or a systemd unit. **Single most version-fragile external dependency.** COUNTRY now rejects ISO-3166-1 reserved/user-assigned codes (AA, QM-QZ, XA-XZ, ZZ) at preflight (v7.81/7.82).

### B8. `/etc/bluetooth/main.conf` + `/etc/default/cpupower-service.conf`
```
[General]
FastConnectable=true
[Policy]
AutoEnable=true
ReconnectAttempts=3
```
cpupower-service.conf: `GOVERNOR='powersave'` (sourced by the cpupower service).
- Confirm `cpupower.service` on CachyOS actually sources `/etc/default/cpupower-service.conf` for `GOVERNOR` (path + var name). If CachyOS uses a different path/unit, this file is inert and the governor falls to kernel default (the udev EPP rule still applies). `_vrsv_chk_cpupower_governor` asserts the running governor — validate the path.

### B9. `~/.config/MangoHud/MangoHud.conf` (exact — 19 active, 0 commented, ordered)
```
horizontal
legacy_layout=0
position=top-left
toggle_hud=Shift_R+F12
fps
frametime
frame_timing
gpu_stats
gpu_core_clock
gpu_temp
gpu_power
cpu_stats
cpu_temp
cpu_mhz
vram
ram
font_size=20
text_outline
background_alpha=0.4
```
- **`cpu_temp` now ships LIVE (uncommented).** The v7.77.1 "`# cpu_temp` commented" body is stale. Verify `cpu_temp` populates from `k10temp`/Tctl on Zen 5 as shipped (does current MangoHud auto-detect, or is an explicit `cpu_temp_sensor=k10temp` needed? — MangoHud #1825). Note #1794 (cpu_temp can zero `cpu_power` on Zen 5) — but `cpu_power` is NOT in this config (only `gpu_power`), so confirm whether the interaction is even triggered. If `cpu_temp` reads blank on the installed MangoHud+kernel → FIX (add the sensor key); if it populates → KEEP. Confirm `frame_timing` (graph) vs `frametime` (numeric) both valid; confirm `toggle_hud=Shift_R+F12` syntax current.

## C. Full verify subsystem (`--verify`) — orchestrator families

The top VERIFY block is the user-facing command set; the script's actual `--verify` runs a set of orchestrator functions across sub-families. Recommendations that change a value MUST state which sub asserts it (and whether hard-fail or warn). The architecture (v7.88.3):

**Static (on-disk content) — 6 orchestrators + 1 checksum:**
- `_verify_static_boot` → leaves `_vsb_loader` (loader.conf key=values) · `_vsb_sdboot` (sdboot-manage.conf) · `_vsb_cmdline` (`/etc/kernel/cmdline` token set) · `_vsb_mkinitcpio` (HOOKS/MODULES/COMPRESSION incl. `LINUX_FALLBACK_OPTIONS="quiet"`) · `_vsb_entries` (generated BLS entries). All leaves hard-fail static.
- `_verify_static_system` → subs `_vss_ntsync_modules` · `_vss_logind` · `_vss_nmdispatch` · `_vss_nm` · `_vss_sysctl` (key=value) · `_vss_regdom` · `_vss_bluetooth` · `_vss_udev` (all 3 rules, GPU_DPM_LEVEL-aware, EPP `cpu[0-9]*` matcher) · `_vss_nft` (**hard-fail on missing `echo-request` — the inbound-ping regression guard; IPv4-only**) · `_vss_modprobe` (mt7925e disable_aspm=1). **NO `_vss_known_benign` (removed v7.77.0).**
- `_verify_static_user` — ENV_VARS env.d + MangoHud via leaves `_vmh_existence_only` (asserts ≥1 directive + `fps`; `_grep_mangohud_entry` accepts bareword OR key=value OR `#`-comment) and `_vmh_order_checks` (directive ordering). **NO baloo (de-managed v7.77.0).**
- `_verify_static_packages` → leaves `_vsp_required` (PKGS_ADD present) · `_vsp_removed` (PKGS_DEL absent) · `_vsp_pacman_conf` (pacman.conf hooks/opts) · `_verify_static_services` · `_verify_static_syntax` · `_verify_static_checksum` → leaf `_vsc_check_one` per-file (embedded content SHA256 == installed file; **skips gracefully — returns 0 — when the generator yields `EXIT_GEN_NOUUID` and the root UUID is genuinely unresolved**, presence still verified separately).

**NO known-benign INFO family** — the 5 `_kb_*` subs and their aggregator were removed. Do not audit them.

**Runtime-kernel — `_verify_runtime_kparams`:**
- `_vrk_cmdline` — **generic loop asserting EVERY `KERNEL_PARAMS` token + `rw` is present in `/proc/cmdline`** (so `amd_iommu=off` AND `ipv6.disable=1` are auto-asserted); + preemption-model INFO from dmesg `Dynamic Preempt:`.
- `_vrk_gpu_state` — `power_dpm_force_performance_level` == `$GPU_DPM_LEVEL` (auto) across `card*`.
- `_vrk_cpu_state` — scaling_driver=amd-pstate-epp, scaling_governor=$CPUPOWER_GOVERNOR, EPP=balance_performance, amd_pstate status, **`dynamic_epp=disabled` (NEW)**, prefcore, boost=1.
- `_vrk_module_state` → subs `_vrkm_amdgpu` (hex-aware, splits param pairs once since values may contain `:`, v7.87.x; no-ops without amdgpu.*), **`_vrkm_iommu`** (derives off/on from KERNEL_PARAMS; with `amd_iommu=off` hard-fails if iommu_groups present; dmesg AMD-Vi/DMAR correlation), `_vrkm_blacklist` (module_blacklist= scan); + usbcore.autosuspend, nvme_core ps_max_latency, zswap.enabled, nmi_watchdog, NVMe `[none]`.
- `_vrk_clocksource` — clocksource=tsc; HPET → **fail** + dmesg TSC-demotion (`Marking TSC unstable`) correlation.

**Runtime-services — `_verify_runtime_services`:** `_vrsv_chk_active_enabled` · **`_vrsv_nft_assert_ping`** (live input chain accepts inbound IPv4 ping; warn-only — REPLACES the old `_vrsv_nft_assert_ndp`) · `_vrsv_chk_nftables` (oneshot, no RemainAfterExit → judged by live ruleset policy-drop) · `_vrsv_chk_resolved` · `_vrsv_chk_cpupower_governor` · `_vrsv_sys_units` · `_vrsv_wifi_nm_backend` · `_vrsv_wifi_iwd_proc` · `_vrsv_wifi` · `_vrsv_masked_inactive`.

**Runtime-env — `_verify_runtime_env`:** `_vre_envvars` (`systemctl --user show-environment`) · `_vre_sysctl_runtime` (`/proc/sys`) · `_vre_tcp` (tcp_bbr) · `_vre_zram` (zram service + active swap; PASS/WARN — profile does not manage zram but asserts it) · `_vre_fstab` (ext4 has `noatime,lazytime,commit=10`) · `_vre_ntsync` (state dispatch) · `_vre_regdom` (`iw reg get`).

**Runtime-session — `_verify_runtime_session`:** `_vrs_nm_perms` (**system-connections 0600 root:root**) · `_vrs_vfat_skip` (skips perm check on vfat/undetermined boot) · `_vrs_installed_file_perms` (system 0644 / user 0600) · `_vrs_parent_dirs` → leaf `_vpd_dir_perm_check` (0755 system / 0700 user config dirs) · `_vrs_vulkan` (vulkan-radeon + lib32-vulkan-radeon).

Actionable for §C:
- Confirm `_vrkm_iommu`'s 0-groups expectation under `amd_iommu=off` is correct and its dmesg patterns (`AMD-Vi: (Enabled|Found|Interrupt)`, `DMAR: IOMMU enabled`, `Adding to iommu group`) are current; confirm it stays silent when KERNEL_PARAMS carries no iommu directive.
- Confirm `_vss_nft` / `_vrsv_nft_assert_ping` correctly gate the inbound-ping accept (hard-fail static, warn live) and that the IPv4-only ruleset needs no ICMPv6 assert (IPv6 is off).
- Confirm `_vre_zram` PASS/WARN policy (masked-zram → WARN, no-swap → WARN) — reconcile the manages-nothing-but-asserts-something tension.
- Confirm `_vrs_vfat_skip` carve-out is not hiding a real perms regression on `/boot/loader/loader.conf`.
- Confirm `_vrk_clocksource` HPET-fail + TSC-demotion correlation is the right severity given `tsc=reliable` is on the cmdline.

## D. fstab rewrite (`_install_fstab_opts`) — normalization, not just append

The rewrite does more than add tokens:
- Adds `noatime,lazytime,commit=10` to ext4 entries (field 4 only); every other column and every non-ext4 row byte-preserved.
- **Strips conflicting tokens:** redundant `defaults`, `relatime`, `atime`, `strictatime`, and an existing `commit=` (rewritten to `commit=10`, not duplicated). `--verify` flags a `defaults`-bearing or atime-variant ext4 row as a pending rewrite (v7.85/v7.87.x).
- Gates: line-count parity + size floor + mandatory `findmnt --verify`.
- **Refused (not corrected):** a symlinked or whitespace-split (malformed) `/etc/fstab`.

Confirm: (a) the rewrite touches ONLY ext4 (not the vfat ESP, not btrfs/xfs); (b) idempotent (re-run is a no-op on already-correct fstab); (c) atomic (tmp+rename) so a crash mid-rewrite cannot truncate this boot-critical file (also takes a `.ry.bak`); (d) `commit=10` durability is acceptable given `fsck.mode=force` runs every boot (§3) — do the two interact?

## E. Preflight gate ordering (`_init_runtime` / `_install_preflight` / `_ir_*`)

Order matters for exit-code semantics:
- `_ir_resolve_root_uuid` → `EXIT_GEN_NOUUID 12` (sentinel) if cmdline render finds no UUID.
- Hardware gate (CPU match, override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`, fail-closed on unreadable model; in `--verify` the gate warns, deploy exits 3, v7.87.x).
- `_ir_validate_kernel_floor` (EXIT_PREFLIGHT 3) — kernel ≥ **6.19** (override `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK=1`, fail-closed on unreadable `uname -r`; `--verify` warns and continues).
- `_ir_validate_counts` (3) — all 19 array counts (see header; note `_RY_POST_HOOKS=17`, `USER_DESTINATIONS=2`).
- `_ir_validate_keys` (3) — scalar domains (true|false: BT_AUTO_ENABLE/BT_FAST_CONNECTABLE/RY_REMOTE_PLAY_PORTS; yes|no: SDBOOT_*/RESOLVED_MDNS/LLMNR/DOT; int: LOADER_TIMEOUT/NM_WIFI_POWERSAVE/BT_RECONNECT_ATTEMPTS; ISO-3166 COUNTRY incl. reserved-code rejection; GPU_DPM_LEVEL ∈ `_RY_DPM_LEVELS`; **the IPv4-only↔ipv6.disable=1 coupling**; non-empty: LOADER_*/SDBOOT_DEFAULT_ENTRY/RESOLVED_DNSSEC/NM_*/CPUPOWER_GOVERNOR/NM_DISPATCHER_LOGLEVELMAX/MKINITCPIO_COMPRESSION).
- `_ir_validate_post_hooks` (3) — every `_RY_POST_HOOKS` tag has a handler (no baloo).
- Generator sentinels: `EXIT_GEN_NOFN 11`, `EXIT_GEN_NOUUID 12`, `EXIT_GEN_SYSCTL 13`, `EXIT_GEN_ENVD 14`; `250/251/255` never reach a process exit (surface as footer `gen_fail`).
- Advisories (non-fatal): mesa < 26.0 soft-floor (stderr silenced on the vercmp compare). **NO linux-firmware advisory (removed). NO `_ry_check_umip_disabled` (removed).**

Confirm: (a) counts/keys/floor run BEFORE any disk write; (b) the two skip-override env vars are the only documented bypasses, each scoped (kernel-floor and hardware only — counts/keys cannot be bypassed); (c) `PACTREE_TIMEOUT_S` (60), `BOOT_SPACE_CRIT`/`WARN`, `ROOT_AVAIL_CRIT`/`WARN` are sane gates — are the ESP thresholds right for a systemd-boot ESP holding multiple kernels + fallback + the `-1` zstd initramfs (§4)?

## F. Deeper-pass investigation deltas (updated actionable items)

1. **MES-0x86 label conflict (HIGHEST PRIORITY):** the script/README (source of truth) say "MES 0x86 resolved"; the prior prompt said the label was stale (0x83/0x80). Reconcile against current upstream; report the disagreement + trusted source. Do not silently adopt either.
2. **IPv6-off + IPv4-only ruleset + inbound-ping accept (NEW):** confirm `ipv6.disable=1` is the right posture (vs firewalling IPv6), that the hard coupling in `_ir_validate_keys` is correct, and that accepting inbound ping is intended. Highest-priority network change.
3. **IOMMU live-effect assert (`_vrkm_iommu`):** confirm the 0-iommu_groups expectation under `amd_iommu=off`, the off/on derivation, and the dmesg patterns. `amd_iommu=off` does NOT break ROCm (`IOMMU Support: None`) — accepted trade-off.
4. **MangoHud `cpu_temp` now LIVE:** confirm it populates from `k10temp`/Tctl as shipped; assess the #1794 interaction (moot without `cpu_power`?). FIX only if it reads blank.
5. **Firmware advisory REMOVED:** confirm firmware currency is adequately handled by the kernel floor + `mkinitcpio-firmware` without the version gate; keep `pacman -Q linux-firmware` informational only.
6. **`amd_pstate/dynamic_epp=disabled` assert (NEW):** confirm the node exists on the 6.19 kernel and that the EPP pin depends on it being `disabled`.
7. **udev matcher fixes:** confirm the EPP rule (`cpu[0-9]*`, v7.85) now actually fires and the GPU rule (`card[0-9]*` + `drm_minor`, v7.79) matches on this iGPU.
8. **`cachyos-iw-set-regdomain` existence (B7):** verify this external CachyOS unit still exists and reads `/etc/iw-regdomain`. Highest version-fragility in the profile.
9. **`cpupower.service` config path (B8):** verify CachyOS sources `/etc/default/cpupower-service.conf` `GOVERNOR`.
10. **`99-cachyos-*` basename overrides (B5/B6):** confirm same-filename replace (not merge) is intended.
11. **`flush ruleset` blast radius (B3):** confirm wiping all nft tables is safe vs docker/libvirt/podman.
12. **`LINUX_FALLBACK_OPTIONS="quiet"` recovery posture (B1):** confirm a param-stripped fallback (no `amd_iommu=off`, no `fsck.*`, IPv6 RE-ENABLED) is intended, and `timeout 0`+`editor no` (B2) is not a dead-end; note the fallback re-enables the IPv6 stack the firewall does not cover.
13. **Removed surfaces:** confirm the audit does not re-flag baloo, `_kb_*`, `_ry_check_umip_disabled`, ICMPv6/NDP rules, or the firmware advisory — all removed since v7.77.1.
14. **RTC write-back (`_ry_rtc_writeback`) + `RY_NO_NTP_REMEDIATION=1`:** confirm `hwclock --systohc --utc` is safe, the `RTCInLocalTZ=yes` defer-to-systemd guard is correct, and the skip env var cleanly disables the timesyncd-enable path.

---

# Deepest-pass appendix (§G–§L) — robustness & correctness audit surface

§1–§13 audit *what the profile configures (and omits)*; §A–§F audit *what the script writes and asserts*. This final layer audits *whether the installer is safe to run at all* — the atomic-write, locking, privilege, rollback, and signal machinery. These are correctness questions, not tuning. A reviewer must confirm each guarantee holds on current fish (3.6 floor) / CachyOS, and flag any TOCTOU, fail-open, or partial-write window. Every mechanism below is quoted from `ry-install.fish` v7.88.3.

## G. Atomic-write guarantees (`_awf_*`)

Write path per managed file: `_awf_render_to_tmp` → `_awf_symlink_check` → `_awf_finalize_mv`; boot/backup targets add `_awf_make_backup` (pre) + `_awf_postwrite_verify_restore` (post).

- **tee-to-tmp with `$pipestatus`:** content generator piped into `_as $use_sudo tee -- "$tmpfile"`; `$pipestatus[1]` (generator) and `$pipestatus[2]` (tee) checked separately, mapping generator failures to `EXIT_GEN_NOFN/NOUUID/SYSCTL/ENVD`. **Byte-read probes now read `pipestatus[1]` only, per a documented contract (v7.87.x).** Confirm fish `$pipestatus` still distinguishes the two stages and a generator returning non-zero never leaves a partial tmpfile promoted.
- **Post-write symlink-swap probe (`_awf_symlink_check`):** after writing tmp, re-tests whether tmp was replaced by a symlink (rc 0/1/2 = symlink/not/sudo-lapse), aborting on swap. Confirm the probe closes the window (any gap between the check and the `mv -T`?).
- **`mv -T` atomic rename (`_awf_finalize_mv`):** chmod tmp → re-assert `sudo -n true` (credential-lapse guard) → `mv -T -- tmp dst`. Confirm `mv -T` is atomic on the same filesystem (tmp created in dst's parent so rename is same-FS — verify this holds for `/boot` on vfat, where rename atomicity differs). `mkdir` for parent dirs is capped at umask 0022 via `_ry_mkdir_0755` (v7.85).
- **Post-write byte re-read + restore (`_awf_postwrite_verify_restore`):** re-runs the generator, reads installed bytes (`_installed_bytes`, tri-state rc), compares; on mismatch restores `.ry.bak` via `mv -T`. Confirm: (a) generator determinism (the sysctl/environment.d generators reject control chars and side-effect bad-entry globals; preflight already refuses `_RY_BACKUP_TARGETS` members using side-effecting generators — confirm that guard is complete); (b) restore `mv -T` is itself atomic; (c) `string collect --no-trim-newlines --allow-empty` preserves trailing-newline-sensitive comparisons.
- **Backup only for `_RY_BACKUP_TARGETS` (2):** loader.conf + mkinitcpio.conf. Every other managed file relies solely on atomic-write (no .bak). Confirm a post-write mismatch on a NON-backup file (e.g. nftables.conf) is detected but cannot be auto-restored — is that the right risk posture for the firewall ruleset? (`_awf_make_backup` skips `.ry.bak` on an inconclusive probe and drops a stale symlink first, v7.85.)

Actionable: confirm the symlink-check→mv window is non-exploitable; confirm `/boot` vfat rename atomicity; confirm generator determinism for all post-write-verified files.

## H. Instance lock & PID-recycle TOCTOU (`_acquire_lock*`, `_lock_pid_started_after`)

Lock = atomic `mkdir "$LOCK_DIR"` (**state dir created under `umask 0077` for the 0700 contract, v7.87.x**) + chmod 700 + pidfile via `mktemp`+`mv -Tf` (chmod 600). Stale reclaim bounded (3 attempts) and fail-closed.

- **Atomic mkdir as the lock primitive:** confirm race-free vs a competing instance on the same host.
- **PID-recycle detection (`_lock_pid_started_after`):** reads `/proc/PID/stat` field 22 (starttime ticks) + `/proc/stat` btime ÷ USER_HZ (`getconf CLK_TCK`, with CONFIG_HZ recovery from `/proc/config.gz` if getconf absent), reclaims ONLY if holder start > pidfile mtime + 2s. **Fail-closed**: any unparseable field → treat as live, refuse reclaim. **Reclaim is refused on an empty/garbage pidfile and the owner is re-verified (v7.85).** Confirm: (a) the `string replace -r '^.*\) '` correctly handles a `comm` containing `) ` (the `/proc/stat` parsing trap); (b) field-22 indexing survives a comm with spaces/parens; (c) the +2s slack is sufficient vs clock granularity but tight enough to catch a recycled PID.
- **Re-read-before-rm TOCTOU guard:** before `rm -rf` of a stale lock, pidfile is re-read and compared to the decision-time value; changed → abort. Confirm this closes the reclaim race.
- **Symlink refusal:** symlinked `$LOCK_DIR` → reclaim refused; `rm -rf --preserve-root`. Confirm no path-traversal.
- **kill -0 EPERM → /proc branch:** unsignalable-but-alive PIDs (different UID) detected via `/proc/PID` and NOT reclaimed. Confirm correct.

Actionable: confirm `/proc/stat` comm-parsing robustness; confirm fail-closed on every USER_HZ/btime/starttime read failure; confirm the re-read-before-rm window is closed on current kernels.

## I. Privilege handling (`_as`, `_run`, `_is_symlink`, `_installed_bytes`)

- **`_as use_sudo` / `sudo -n` everywhere:** all privileged ops use non-interactive `sudo -n`; credential lapse re-checked immediately before each critical write (`mv`, mkinitcpio revert). Confirm NO interactive sudo prompt mid-run (would hang an unattended install) and that mid-run credential expiry fails safe (aborts the file, no partial-write). The `_run` timeout-bypass logic now skips separated/glued sudo value flags when deciding exemptions (v7.87.x).
- **Tri-state rc 0/1/2 (drift vs sudo-lapse):** `_is_symlink`, `_installed_bytes`, `_ry_content_bytes` return 2 specifically for sudo-cache-lapse so callers distinguish "file differs" from "couldn't read due to expired sudo." Confirm every caller branches on rc 2 (a 2→1 collapse would misreport a sudo lapse as drift). Spot-check across callers.
- **`_run` timeout enforcement:** `_run` wraps commands with logging + capture + timeout (`RY_RUN_TIMEOUT`, default 3600 s, clamped above 9 digits to 2147483647 (v7.85), 0 disables). Confirm pacman/mkinitcpio/sdboot-manage/paccache/updatedb/pkgfile are EXEMPT from the cap (README states they are) so a slow mirror/large initramfs is not killed; `PACTREE_TIMEOUT_S` (60) governs pactree only. Run-overflow output is now analyzed inline (no spill dir; a ≤10-line elided-region diag sample + sha256/bytes is logged, nothing retained on disk — v7.87.x).

Actionable: audit every rc-2 caller for the drift-vs-lapse distinction; confirm no interactive-sudo hang path; confirm the long-op timeout exemptions are complete.

## J. Boot-wipe gate & boot-critical rollback (`_irb_taint_gate`, `_install_rebuild_boot`)

The single most dangerous operation: `SDBOOT_REMOVE_EXISTING=yes` makes `sdboot-manage gen` wipe all `loader/entries/` (foreign BLS included). Gate sequencing:
1. `_irb_taint_gate` → `_check_boot_taint_gate`: if `_RY_BOOT_TAINTED=true` OR the mkinitcpio.conf revert failed, **SKIP `mkinitcpio -P` entirely** and return `EXIT_BOOT_CRIT` (4). Confirm the taint flag is set on any prior boot-critical write failure so a half-written cmdline/mkinitcpio never reaches `mkinitcpio -P`.
2. `mkinitcpio -P` failure → abort remaining steps, `EXIT_BOOT_CRIT`, skip post-mki.
3. **`$BOOT` resolution refusal:** `$BOOT` is resolved BEFORE the sdboot vfat gate (v7.86); if `SDBOOT_REMOVE_EXISTING=yes` AND `_resolve_boot_path` returns empty (bootctl + findmnt both failed AND `/boot` missing), **refuse the boot-wipe gate** — `EXIT_BOOT_CRIT` rather than run `sdboot-manage` against an unresolved `$BOOT`. A non-vfat `/boot` ESP also refuses sdboot (exit 4). This guard prevents wiping the wrong/no target.
4. Post-rebuild sanity (`_preflight_boot_sanity`): vmlinuz + initramfs + entries must exist or `EXIT_BOOT_CRIT`.

Confirm: (a) NO path where `sdboot-manage REMOVE_EXISTING=yes` runs against an unverified `$BOOT`; (b) a mkinitcpio failure cannot leave a new cmdline with a stale/absent initramfs — trace the Phase-3 cmdline-write vs Phase-5 initramfs-rebuild ordering window and confirm `LINUX_FALLBACK_OPTIONS="quiet"` (§B1) covers a boot with new cmdline + old initramfs (note: the fallback entry carries neither `amd_iommu=off` nor `ipv6.disable=1` nor `fsck.*`, so a fallback boot reverts to kernel-default IOMMU AND re-enables IPv6 — confirm benign, or flag the fallback-only IPv6 exposure); (c) `EXIT_BOOT_CRIT` aborts ALL remaining phases (no Finalize after a boot failure).

**mkinitcpio rollback (`_ip_snapshot_mkinitcpio` / `_mkinitcpio_revert`):** before `pacman -Syu`, mkinitcpio.conf is snapshotted to `/run/ry-install` (0700 root:root, mktemp). On pacman failure, `_mkinitcpio_revert` restores via same-`/etc`-FS mktemp + `_mr_copy_cmp_verify` (cp + byte-exact `cmp`) + `_mr_chmod_chown_mv` (atomic mv, `--reference=destination` perms). Duplicate `KEY=` lines in mkinitcpio.conf resolve to last (shell-sourced semantics, v7.85). Confirm the snapshot→revert path is complete (snapshot in `/run` tmpfs lost on reboot — acceptable since revert is same-boot; revert tmp in `/etc` same-FS atomic; symlink-checked). Flag if a pacman partial-transaction could desync mkinitcpio.conf from installed modules without triggering revert.

Actionable: trace the cmdline(Phase 3)-vs-initramfs(Phase 5) ordering window; confirm the `$BOOT`-unresolved + non-vfat refusal is the only path to `sdboot-manage REMOVE_EXISTING`; confirm `EXIT_BOOT_CRIT` is terminal.

## K. Signal & exit teardown (`_cleanup`, `_teardown`, `_do_cleanup`)

- **Signal handlers:** `_cleanup` traps INT/TERM/HUP/QUIT/ABRT with correct **128+N exit codes** — propagated via exec re-raise on INT/TERM/HUP/ABRT (v7.85), idempotent via `_CLEANUP_DONE`. SIGPIPE (`_cleanup_pipe`) marks output broken and continues JSONL-only. `fish_exit` (`_cleanup_on_exit`) ensures teardown on normal exit, preferring `_INTENDED_EXIT_CODE` → `_RY_INSTALL_LAST_EXIT` → `$status`. The tmpfile-cleanup `_log` is guarded for pre-init signals (v7.88.0). Every JSONL record (`header`, `_log` events, and the exit `footer`) stamps its `ts` field from the single `_RY_TS_FMT` global (`+%Y-%m-%dT%H:%M:%S%z`, hoisted v7.88.3) via `command date` — confirm the three emission sites (header/`_log`/footer) share the one format so timestamps stay ISO-8601-uniform across the log stream, and that the hoist is purely a DRY refactor with no field-shape change (a log-schema consumer keys on `ts` as before).
- **Cleanup order (`_do_cleanup`):** kill children → mkinitcpio revert → tmpfile sweep → filesystem sweep → **lock release (sweeps run while lock still held)** → erase globals. Confirm: (a) children reaped BEFORE revert so revert never races a live pacman; (b) lock released LAST so no second instance starts mid-sweep; (c) the `_RY_TMPDIR_GLOBS` (6 patterns) match every tmpfile created — **the globs are now PID-scoped (`ry-*.$fish_pid.*`, v7.87.x) so a sweep never touches a peer run's files**: ry-sudo-err, ry-tee-err, ry-run, ry-argparse-err, ry-fstab-tee-err, ry-fstab-awk-err. A missed glob leaks; an over-broad glob deletes another instance's tmp. Verify the glob set is exactly the created set and the PID-scoping is complete (a non-PID-scoped tmpfile would leak).

- **Exit-code contract (14 distinct — audit for discipline, no bare `exit 1`):** beyond the documented `0/1/2/3+`, the script carries a structured code space. A recommendation touching control flow MUST preserve these.

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

- **`_run` timeout:** `_RY_RUN_TIMEOUT_DEFAULT=3600` (1 h) wraps external calls; clamped + warned when overridden (`_RY_RUN_TIMEOUT_CLAMPED`/`_WARNED`). Confirm the 1 h ceiling covers a worst-case `-Syu` on slow mirrors without false-killing, and that a timeout surfaces as a fatal (not a silent skip). The 11–14 generator codes are internal (consumed by `_verify_static_checksum`, which maps `EXIT_GEN_NOUUID` to a graceful checksum-skip); the 250/251/255 codes are BUG asserts (`_log "BUG: …"`) that must never fire in normal operation — a recommendation that trips one is by definition wrong.

Actionable: confirm SIGKILL (uncatchable) is the only signal bypassing cleanup and the lock's stale-reclaim (§H) recovers from a SIGKILLed holder; confirm the tmpdir glob set exactly matches the created-tmpfile set and every created tmpfile is PID-scoped; confirm no code path collapses a structured exit code (3/4/5/10) into a bare `1`.

## L. pacman transaction safety (`_ip_pacman_invoke`)

- **Full `-Syu --needed` only (no partial upgrades):** first pass `-Syu`, retry `-Syyu` (forced db re-sync for mirror staleness), `--noconfirm`. `SYSTEM_UPGRADED` is derived from a pacman `-Q` fingerprint (not the transaction rc, v7.87.x). Confirm the retry only addresses transient staleness and never masks a real conflict (the second failure must be fatal and surfaced).
- **db.lck pre-check:** refuses if `/var/lib/pacman/db.lck` exists; never removes it itself (instructs the user). Confirm checked before any package op.
- **PKGS_DEL via `-Rns` (rdep-aware):** an external dependant skips + logs (tracked in `_RY_PKG_REMOVE_SKIPS`) rather than cascading. Confirm `-Rns` never triggers a partial-upgrade state and that full `-Syu` is mandatory (script never issues `-S <pkg>` without `-yu`). `paccache` runs `-rk2` and `-ruk0` as separate invocations (v7.85).

Actionable: confirm retry-then-fatal semantics; confirm no self-removal of db.lck; confirm PKGS_DEL `-Rns` cannot induce a partial-upgrade.

## Robustness verdict (required, separate from §1–§13)

Add a final **ROBUSTNESS** verdict block:
- For each of §G–§L: PASS (guarantee holds) / GAP (window or fail-open found) / UNCERTAIN (cannot confirm against current fish/kernel without testing).
- Any GAP in §G/§H/§J (atomic-write, lock, boot-wipe) is release-blocking and outranks every tuning finding — surface it first regardless of IMPACT/RISK on config items.
- This layer is correctness, not preference: there is no "deliberate trade-off" defense for a partial-write window or fail-open lock. FIX applies normally here (flag-don't-FIX is for config values, not safety invariants).

Sources for §G–§L: man7.org (mkdir(2), rename(2)/mv -T atomicity, proc(5) stat fields, sysconf CLK_TCK), fishshell.com/docs (`$pipestatus`, `string collect`, `--on-signal`, `--on-event fish_exit`), wiki.archlinux.org (pacman partial-upgrade policy, mkinitcpio, systemd-boot), docs.kernel.org (/proc/stat btime, USER_HZ).
