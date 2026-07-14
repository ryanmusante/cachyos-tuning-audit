CachyOS tuning profile — Beelink GTR9 Pro
==========================================

Newest first. Versions correspond to the ry-install.fish release that
applies the profile. Format: - subsystem: imperative summary.

# 7.105.9 - 2026-07-14

  - docs: re-pin to ry-install 7.105.9 — script delta vs 7.105.8 is
    the version string only (header comment + VERSION, 2 lines);
    4923 lines / 288 functions / fish --no-execute clean re-certified
    on the attached archive; all profile values unchanged
  - audit: re-verify README-derived claims against the restructured
    7.105.9 README (Globals table -> prose; Managed Files split into
    Boot/System/User tables) — BIOS flat 85 W ceiling + TjMax 90,
    MangoHud #1794 note, PCIe-ASPM rationale line, and the advisory
    kernel row all survive; the pre-7.99 modprobe-leftover sudo-rm
    note remains absent, so the §6 unguarded-gap finding stands;
    ASPM rationale now quoted byte-exact

  Second pass (same date, same source v7.105.9 — appendix-body
  completion after the accidental-upload double-check):

  - audit: full re-render pass — all 17 generator outputs (plus both
    modprobe states and both nftables port states) re-derived by
    executing the 7.105.9 generator functions in a sandbox and
    byte-compared against the appendices
  - appendix: expand the B1 LINUX_OPTIONS and B4 EPP/DPM placeholders
    to the literal rendered values; scalar-to-global maps annotated
    under the fences (root=UUID placeholder retained —
    machine-specific)
  - appendix: move the dispatcher logging.conf body into the B6 fence
    (was inline prose)
  - appendix: add B11-B13 exact bodies (logind drop-in, sysctl
    drop-in, environment.d) — every generator-managed file now has a
    byte-exact anchor
  - verify (report-level): P0 queue, cross-references, box-matrix
    widths, CHANGELOG history preservation, README counts, HUD
    companion parity, and installer-README claims all re-checked
    clean; nftables B3 keeps the single annotated body (the
    RY_REMOTE_PLAY_PORTS=true block is comment-labelled in-fence)

  Third pass (same date, same source v7.105.9 — re-certification
  against the ry-install-main archive upload):

  - audit: re-render all 17 generator outputs (both modprobe states,
    both nftables port states) from the uploaded archive and
    byte-compare against Appendix B — 19/19 match; version pins across
    script header / VERSION / README badge / checkout / CHANGELOG,
    4923 lines / 288 functions / fish --no-execute clean re-confirmed
  - audit: verify document structure — both box matrices
    column-aligned, 22 balanced fences, appendix lettering stable,
    MangoHud v1.17.0 references consistent across header / P0 / §12 /
    §14 / B9 / Sources; no corruption or conflict from the HUD
    companion cross-integration
  - p0: complete the #19 row — the trailing "HUD-scoped" now closes
    with "floors only, do not conflate with §10 profile floors"
    (mirrors the §12 statement)

# 7.105.8 - 2026-07-14

  - kernel: swap pcie_aspm.policy=performance -> pcie_aspm=off; drop
    the mt7925e disable_aspm=1 module option (global ASPM-off covers
    the MT7925 mitigation; the modprobe drop-in is now
    amdxdna-blacklist-only and the NPU path renders a comment-only
    file, validator-accepted)
  - env: add VKD3D_CONFIG=descriptor_heap (ENV_VARS 11 -> 12)
  - sysctl: add vm.watermark_boost_factor=0 (SYSCTL_VALUES 9 -> 10)
  - kernel: hard floor removed (_ir_validate_kernel_floor + KERNEL_MIN
    dropped; validators 4 -> 3); 6.18.4 kept as an advisory comment —
    RTL8127 r8169 + suspend-hang regression baseline; gfx1151 hang fix
    re-anchored to firmware (linux-firmware MES 0x86), not kernel
  - audit: flag the modprobe-leftover migration gap as unguarded — the
    ry-install README no longer carries the pre-7.99 sudo rm note and
    no in-script reference remains
  - docs: rewrite the reference as a deep-research brief — prose
    removed, research-priority ordering (P0 queue), per-item search
    anchors and source domains; closed-finding history dropped
  - docs: bump version pins to 7.105.8

  Second pass (same date, same source v7.105.8 — HUD companion
  integration, mangohud-gtr9-pro v1.17.0):

  - hud: cross-audit the embedded MangoHud generator against the
    companion archive — 19/19 active directives identical in set and
    order; byte delta is two comment lines (identity header, cpu_temp
    comment wording); installer confirmed source of truth (repo
    1.14.0 realignment)
  - hud: import repo-history dead ends into the §14 protected list —
    fps_metrics (added 1.10.0, dropped 1.13.0), gpu_junction_temp
    (hotspot mirrors edge), throttling_status(+_graph) (removed
    twice), gpu_mem_clock + swap (shared-memory APU)
  - hud: rewrite P0 #15 — reconcile #1794 (cpu_power reads 0 while
    cpu_temp active) against the repo's k10temp-pickup claim and the
    1.15.0 "sensor not reported" note; add P0 #19 HUD floor currency
    (MangoHud >= 0.8.4, kernel >= 6.14, Mesa 24 — HUD-scoped)
  - scope: add the UMA vram-carveout special case (vram = BIOS
    carveout only; ram is the load-bearing figure — never a finding)
  - verify: add active-directive count (19) plus k10temp hwmon
    presence and sensors-pickup probes (informational)
  - docs: register the companion archive in the header, §12, B9, and
    Sources

# 7.101.0 - 2026-07-12

  - docs: condense reference prose (intro, preflight, BIOS, packages)
    and trim verbose inline notes to vital rationale; no profile-value
    change
  - docs: bump version pins to 7.101.0

# 7.100.0 - 2026-07-11

  - kernel: re-anchor MES floor label to post-0x83 (0x83 reverted
    upstream 2025-12-01); resolves the prior "MES-0x86" script-only
    label conflict
  - packages: drop archlinux-contrib (PKGS_ADD 19 -> 18; nothing in the
    profile invoked it)
  - sysctl: correct the netdev trailing comment 2.5GbE -> 10GbE
    (RTL8127) to match the platform
  - ntp: add openntpd.service to the NTP-client conflict guard scan
    (now chronyd/ntpd/openntpd)
  - verify: trim the stale "Vulkan packages" mention from the
    runtime-session description
  - docs: note the fallback-entry IPv6/IOMMU exposure window; cpu_temp
    #1794 caveat; drop the pre-7.99 modprobe drop-in removal note

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
