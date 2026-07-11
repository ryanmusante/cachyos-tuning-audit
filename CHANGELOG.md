CachyOS tuning profile — Beelink GTR9 Pro
==========================================

Newest first. Versions correspond to the ry-install.fish release that
applies the profile. Format: - subsystem: imperative summary.

# 7.99.1 - 2026-07-11

  - cmdline: rename clearcpuid 514 -> umip (version-stable string form;
    KERNEL_PARAMS stays 17)
  - modprobe: merge drop-ins into 60-ry-modules.conf; blacklist amdxdna
    by default (BLACKLIST_AMDXDNA toggle; false requires amd_iommu=on
    iommu=pt — validator-paired)
  - packages: PKGS_ADD 17 -> 19 (pacman-contrib, archlinux-contrib)
  - services: MASK 10 -> 12 (avahi-daemon service + socket; second mDNS
    responder collided with resolved)
  - preflight: kernel-floor override removed (6.19 unconditional; floor
    rationale re-scoped to gfx1151 only); count tripwires 19 -> 21;
    KERNEL_PARAMS charset + boot-scalar metachar gates; id/find/timeout/
    mv -T/stat/date capability probes; EPP enum-gated (hoisted
    EPP_PREFERENCE + _RY_EPP_LEVELS)
  - ntp: remediation unconditional (opt-out env removed); chronyd/ntpd
    conflict guard never stacks timesyncd
  - udev: GPU rule DEVTYPE -> ENV{DEVTYPE} (prior rule never applied);
    EPP ATTR from the hoisted global
  - backup: .ry.bak + post-write verify/restore across all 4
    boot-critical files (targets derived from _RY_BOOT_CRITICAL_DSTS)
  - net: nft -c pre-validates the rendered ruleset before commit and
    before every reload
  - run: pkg/boot/db ops floored to a 7200 s hard cap (timeout
    exemption removed; RY_RUN_TIMEOUT=0 still disables)
  - verify: lsmod-check managed modprobe blacklists; live COMPRESSION=
    compare (multi-line join, last-wins); comment-proof _chk_grep;
    quoted GPU_DPM compare; pacman.conf sudo-read fallback; three stale
    orchestrator descriptions fixed (one new found: runtime-session)
  - hud: expand the cpu_temp comment text (composition unchanged:
    19 active + 1 commented)
  - naming: PROFILE_NAME gtr_pro -> gtr9_pro (v7.91.0 finding resolved)
  - docs: MES-0x86 note now script-only (README tables dropped); BIOS
    85 W flat-ceiling guidance; env table trimmed to 3 vars; pre-7.99
    modprobe-migration note
  - size: script 4951 -> 4995 lines (288 -> 289 functions)

  Second pass (same date, same source v7.99.1 — verification-hardening):

  - audit: ground the identifier surface (147 cited, 0 unexpected-absent;
    58/58 verify-family functions covered incl. _verify_summary)
  - packages: correct "explicit post-Syu" to its real semantic — pacman
    -D --asexplicit orphan-protection, not ordering
  - settle: modprobe-leftover gap CONFIRMED (0 in-script references;
    README rm note is the only guard); iwd probe is a genuine removal
    (residual = backend compare + new firewall-posture INFO)
  - clarify: all profile toggles are embedded scalars (unconditional
    set -g clobbers exported env); runtime env inputs are exactly
    RY_RUN_TIMEOUT / RY_INSTALL_SKIP_HARDWARE_CHECK / NO_COLOR
  - appendix: post-hook dispatch is first-match-wins via
    _post_boot_apply skip_mki split; _post_nm defers on active WiFi
    route; destinations + --install-file canonicalized (realpath -m);
    NOUUID is mode-scoped (fatal only for the cmdline target)
  - appendix: 33-command GNU dep gate enumerated; sudo value-flag skip
    list; _run argv redaction + elided-region overflow scan; child-reap
    grace 0.5 s / 10 s under db.lck; explicit 128+N signal map; kconfig
    helper survives for CONFIG_NTSYNC (lock path uses USER_HZ=100)
  - appendix: fstab verify conflict list exact (defaults/relatime/
    atime/strictatime); commit!=10 overrides tracked; numeric-$4 rows
    pass through to the malformed guard; _vrs_vfat_skip guards both
    perm loops; _ry_verify_all static/runtime counter aggregation

# 7.91.0 - 2026-07-04

  - hud: re-comment cpu_temp; add cpu_power readout; reorder gpu_temp
    before gpu_core_clock (19 active directives + 1 commented)

# 7.90.0 - 2026-07-04

  - verify: fold the Vulkan-package check into required-package check
    (ports the pacman-db-lock guard); drop the standalone Vulkan
    runtime-session leaf
  - verify: drop tcp, zram, ntsync-modules, iommu, and clocksource
    leaves plus the dmesg TSC cache
  - verify: retire the IOMMU 0-groups and clocksource/TSC-demotion
    correlations; directives still asserted at cmdline + on-disk layers
  - size: script 5080 -> 4951 lines (294 -> 288 functions)

# 7.89.0 - 2026-07-04

  - args: root guard defers to argparse; invalid args exit 2 with the
    same usage message whether or not the run is rooted
  - docs: sync version pins; note argument-message parity

# 7.88.3 - 2026-07-03

  - logging: hoist the JSONL ISO-8601 timestamp to a single format
    global (DRY refactor; no tuning-value or log-schema change)

# 7.88.2 - 2026-07-03

  - kernel: raise minimum to 6.19 (gfx1151 amdgpu; RTL8127
    suspend/shutdown-hang fix, r8169)
  - net: disable IPv6 system-wide (ipv6.disable=1); rewrite nftables
    IPv4-only, accept inbound IPv4 ping
  - net: refuse deploy when nftables is managed without ipv6.disable=1
  - gpu: fix udev EPP matcher to cpu[0-9]*; GPU matcher to card[0-9]*
    plus drm_minor
  - hud: enable cpu_temp (19 active MangoHud directives)
  - net: add remote-play TCP port 27037
  - cpu: assert amd_pstate dynamic_epp=disabled
  - cpu: validate GPU_DPM_LEVEL against the accepted-level set
  - env: add RY_NO_NTP_REMEDIATION opt-out
  - boot: remove the linux-firmware version advisory
  - session: drop baloofilerc management (user-managed files 3 -> 2)
  - preflight: remove the UMIP-disabled check (clearcpuid=514 retained)

# 7.77.1 - 2026-06-28

  - net: invert IOMMU - iommu=pt -> amd_iommu=off (AMD-Vi off; ROCm
    unaffected on gfx1151)
  - gpu: ntsync managed -> assert-only (PROTON_NO_NTSYNC per-title
    opt-out)
  - hud: comment out cpu_temp
  - services: add RTC write-back
  - hud: fix MangoHud sensor key cpu_custom_temp_sensor ->
    cpu_temp_sensor
  - kernel: re-anchor floor rationale to gfx1151 stability

# 7.75.1 - 2026-06-27

  - kernel: KERNEL_PARAMS 12 -> 16; env vars 10 -> 11; sysctl values
    11 -> 9; 6.18 hard floor
  - gpu: GPU clock floor high -> auto
  - mem: drop vm.page-cluster, vm.vfs_cache_pressure (vendor
    duplicates)
  - net: add mt7925e disable_aspm drop-in; remote-play port gating
  - gpu: add ntsync autoload management (3 confs)

# 7.70.1 - 2026-06-26

  - cpu: pin scaling_governor=powersave, EPP balance_performance

# 7.70.0 - prior

  - bt: add main.conf (FastConnectable, AutoEnable,
    ReconnectAttempts=3)
  - services: mask rtkit, modemmanager
  - mesa: soft floor 25.3 -> 26.0
  - net: amd_iommu=off -> iommu=pt (re-inverted to amd_iommu=off in
    7.77.1)
  - net: Wi-Fi backend -> wpa_supplicant
  - net: remove second regdomain file
  - gpu: remove RADV drirc; TTM/GTT module params
