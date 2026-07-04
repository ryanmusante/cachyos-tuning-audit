# Changelog

Notable changes to the CachyOS tuning profile for the Beelink GTR9 Pro. Newest first.
Versions correspond to the `ry-install.fish` release that applies the profile.

## [7.88.2] - 2026-07-03

Changed
- kernel: raise minimum to 6.19 (gfx1151 MES-0x86 amdgpu; RTL8127 suspend/shutdown-hang fix, r8169)
- net: disable IPv6 system-wide (`ipv6.disable=1`); rewrite nftables IPv4-only, accept inbound IPv4 ping
- net: refuse deploy when nftables is managed without `ipv6.disable=1`
- gpu: fix udev EPP matcher to `cpu[0-9]*`; GPU matcher to `card[0-9]*` + `drm_minor`
- hud: enable `cpu_temp` (19 active MangoHud directives)

Added
- net: remote-play TCP port 27037
- cpu: assert `amd_pstate` `dynamic_epp=disabled`
- cpu: validate `GPU_DPM_LEVEL` against the accepted-level set
- env: `RY_NO_NTP_REMEDIATION` opt-out

Removed
- boot: linux-firmware version advisory
- session: baloofilerc management (user-managed files 3 → 2)
- preflight: UMIP-disabled check (`clearcpuid=514` retained)

## [7.77.1] - 2026-06-28

Changed
- net: invert IOMMU — `iommu=pt` → `amd_iommu=off` (AMD-Vi off; ROCm unaffected on gfx1151)
- gpu: ntsync managed → assert-only (`PROTON_NO_NTSYNC` per-title opt-out)
- hud: comment out `cpu_temp`
- services: add RTC write-back

Added
- boot: linux-firmware preflight advisory (removed again in 7.88.2)

Fixed
- hud: MangoHud sensor key `cpu_custom_temp_sensor` → `cpu_temp_sensor`
- kernel: floor rationale re-anchored to gfx1151 stability (avoid < 6.18.4)

## [7.75.1] - 2026-06-27

Changed
- kernel: KERNEL_PARAMS 12 → 16; env vars 10 → 11; sysctl values 11 → 9; 6.18 hard floor
- gpu: GPU clock floor `high` → `auto`
- mem: drop `vm.page-cluster`, `vm.vfs_cache_pressure` (vendor duplicates)

Added
- net: mt7925e `disable_aspm` drop-in; remote-play port gating
- gpu: ntsync autoload management (3 confs)

## [7.70.1] - 2026-06-26

Changed
- cpu: pin `scaling_governor=powersave`, EPP `balance_performance`

## [7.70.0] - prior

Added
- bt: `main.conf` (FastConnectable, AutoEnable, ReconnectAttempts=3)
- services: mask rtkit, modemmanager

Changed
- mesa: soft floor 25.3 → 26.0
- bt: ReconnectAttempts 7 → 3

Removed
- net: second regdomain file
- gpu: RADV drirc; TTM/GTT module params

Security
- net: `amd_iommu=off` → `iommu=pt` (re-inverted to `amd_iommu=off` in 7.77.1)
- net: Wi-Fi backend → wpa_supplicant
