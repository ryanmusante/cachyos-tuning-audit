CachyOS tuning profile — Beelink GTR9 Pro
==========================================

Newest first. Versions correspond to the ry-install.fish release that
applies the profile. Format: - subsystem: imperative summary.

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
