Changes for cachyos-tuning-audit
================================

Newest first. Versions track the pinned ry-install.fish release.


7.141.0
-------

  - docs: re-pin 7.139.0 r3 -> 7.141.0 r2 (4990 lines, 294 functions,
    sha256 f8210c9e; zip 76c47d95; harness cut L4861)
  - docs: 7.141.0 shipped twice with an identical script hash;
    disambiguate by zip, README or CHANGELOG hash only
  - surface: no tuning value changed, now eleven releases (7.130.0 ->
    7.141.0). 21 tripwires and 4 perf scalars identical
  - surface: sole byte delta is the redundant -T0 cut from the
    initramfs compression options; 17-file total 5,093 -> 5,089 B
  - structure: embed all 17 generated bodies byte-exact, up from two,
    so a fence cannot disagree with the size table beside it
  - structure: add a corrections section so a withdrawn finding cannot
    be re-raised from an older copy. Section map is now 0-10
  - structure: date every claim in the settled table
  - errata: netdev_budget 600 / usecs 4000 is Red Hat's guidance, not
    ESnet's; ESnet recommends the 300 / 2000 kernel defaults
  - errata: the profile ships 600 / 5000, matching Red Hat on budget
    and neither authority on usecs
  - errata: PROTON_FSR4_UPGRADE=1 is current, not near-obsolete.
    Setting it is what triggers the automatic DLL download
  - errata: the real FSR4 gap on RDNA 3.5 is the missing
    DXIL_SPIRV_CONFIG=wmma_rdna3_workaround pairing
  - errata: amd_pstate=active restates the CachyOS compiled default
    (X86_AMD_PSTATE_DEFAULT_MODE=3 is Active/EPP)
  - errata: pcie_aspm.policy=performance IS load-bearing by contrast;
    PCIEASPM_PERFORMANCE is explicitly unset in the CachyOS config
  - errata: _vsb_entry_options is a sub of _vsb_entries, not of
    _verify_static_boot
  - errata: JSONL lives under ~/ry-install/logs/<date>/, not the
    profile root; a shorter glob returns zero hits silently
  - research: close the CachyOS dynamic_epp question; it ships since
    kernel 7.1 and linux-cachyos builds 7.1.5, so it is present
  - research: dynamic_epp defaults false; when enabled it blocks every
    manual EPP write with -EBUSY
  - research: amd_dynamic_epp= is now a documented kernel parameter
  - research: clearcpuid still has zero occurrences in mainline
    kernel-parameters.txt while the code parses it
  - research: resolve the nftables invalid-before-loopback question to
    KEEP; no upstream rule prescribes either order
  - research: re-verify at mainline 7.2.0-rc5 that the performance
    policy returns -EBUSY for any epp>0
  - research: re-verify that PCIEASPM_PERFORMANCE disables L0s and L1
    while pcie_aspm=off leaves firmware config untouched
  - research: re-verify mt7925e.disable_aspm as a live 0644 module
    param and RTL8127 as present in r8169
  - research: Mesa 26.3.0-devel documents RADV_DEBUG=noheap and drops
    heap from RADV_EXPERIMENTAL, closing the descriptor-heap removal
  - research: MangoHud #1794 and systemd #33579 both remain open
  - scope: add T0 items for softnet_stat pressure and dynamic_epp
    state; T0 is now seven items
  - scope: add T1 items to move PROTON_ENABLE_WAYLAND to per-game
    launch options and to pair the RDNA3 FSR4 workaround
  - scope: record _vsb_sdboot_dropins, which warns on drop-ins that
    are sourced after the managed conf and outrank LINUX_OPTIONS
  - scope: the -T0 removal is not a lost threading option; mkinitcpio
    already threads by default


7.139.0
-------

  - docs: re-pin 7.135.1 -> 7.139.0 r3 (4974 lines, 293 functions,
    sha256 bcbb695f; zip ec54f631; harness cut L4845)
  - structure: replace subsystem order with one action queue ordered
    by blast radius, T0 observation-only through T5 closed
  - structure: fold the 21-question research queue into a settled
    table plus open items placed in their own tier
  - structure: trim 1,140 lines / 156,506 B to 970 lines / 65,401 B
  - research: the performance governor returns -EBUSY for any epp>0
    and forces epp 0, so the udev EPP write is a redundant no-op
  - research: Kconfig PCIEASPM_PERFORMANCE disables L0s and L1 even
    where the BIOS enabled them; pcie_aspm=off does not
  - research: close the PowerDevil, descriptor-heap, ntsync, MES
    firmware and DNS-precedence items with citations
  - scope: RY_REMOTE_PLAY_PORTS is gone (7.137.0); the nftables body
    is 729 B in a single form and the mDNS note is retired
  - scope: the preemption advisory is gone (7.139.0 r2) with its dmesg
    fetch and both cache globals; optional deps 20 -> 19
  - errata: the modprobe fence came from the BLACKLIST_AMDXDNA=false
    variant (177 B); the size table had the shipped 183 B default
  - errata: every script line anchor in the previous edition was off
    by one to two lines; all re-derived


7.135.1
-------

  - docs: rebase across five releases (7.130.0 -> 7.135.1); no tuning
    value changed, structural and verification surface only
  - verify: close the modprobe-leftover gap; _ry_stale_ry_dropins has
    two callers, _vss_modprobe (WARN) and _check_record_orphans (JSONL)
  - verify: close the stale-mask gap; _ry_orphan_masked_units diffs
    live masks against MASK, INFO because ownership is unattributable
  - design: neither orphan class sets DRIFT and neither self-heals;
    exit 10 would go permanently non-zero
  - preflight: add _ir_validate_sets, refusing PKGS_ADD n PKGS_DEL,
    MASK n EXPECTED_SERVICES and MASK n _RY_PKG_MANAGED_SERVICES
  - robustness: the .ry.orig preserve was dead code from 7.109.0 to
    7.135.0; a fish set -l inside an if block is invisible after it
  - robustness: 13 of 17 destinations were overwritten with no backup
    and no forensic trace in that window; fixed at 7.135.1
  - packages: PKGS_ADD orphans stay undetectable by design; a
    persisted manifest would add its own drift surface


7.130.0
-------

  - power: the last release to move a tuning value. Governor and EPP
    powersave/balance_performance -> performance, DPM auto -> high
  - kernel: KERNEL_PARAMS 14 -> 15, mt7925e.disable_aspm=1 re-added
    alongside pcie_aspm.policy=performance
  - env: ENV_VARS 11 -> 10, FSR4_UPGRADE renamed PROTON_FSR4_UPGRADE
  - sysctl: SYSCTL_VALUES 10 -> 11, kernel.nmi_watchdog=0 added at
    priority 95 without restoring the nowatchdog boot token
  - network: systemd-resolved gains explicit AdGuard upstreams
  - errata: the amdxdna probe failure is -ENODEV (ret -19)


7.105.8 - 7.122.0
-----------------

  - kernel: pcie_aspm=off reverted to pcie_aspm.policy=performance;
    all version gates removed and none have returned
  - env: VKD3D_CONFIG=descriptor_heap added then dropped;
    PROTON_FSR4_RDNA3_UPGRADE renamed
  - sysctl: vm.watermark_boost_factor=0 added
  - packages: ddcutil and git-delta dropped; modemmanager.service
    dropped from MASK
  - network: DNSSEC allow-downgrade -> no; ct state invalid drop moved
    ahead of the loopback accept
  - network: the dispatcher logging drop-in became its own managed
    file
  - docs: the reference was rewritten as a deep-research brief; the
    do-not-carry-values-forward rule added
  - audit: state the removal-reconciliation asymmetry as a general
    principle; first raise the stale-mask gap


7.88.2 - 7.101.0
----------------

  - kernel: IPv6 disabled system-wide with an IPv4-only nftables
    ruleset, coupled at preflight
  - kernel: clearcpuid 514 renamed to the version-stable umip form
  - modprobe: standalone drop-ins merged into 60-ry-modules.conf
  - services: avahi-daemon service and socket masked as a second mDNS
    responder; NTP remediation made unconditional
  - gpu: udev EPP and GPU matchers fixed. The prior GPU rule never
    applied, voiding any earlier high-vs-auto observation
  - backup: .ry.bak plus post-write verify and restore across all four
    boot-critical destinations
  - verify: tcp, zram, ntsync-module, iommu and clocksource asserts
    retired; the Vulkan check folded into the required-package check
  - packages: archlinux-contrib dropped


7.70.0 - 7.77.1
---------------

  - kernel: IOMMU inverted to amd_iommu=off, with the XDNA NPU as the
    named casualty; Wi-Fi backend set to wpa_supplicant
  - gpu: RADV drirc and TTM/GTT module params removed; ntsync reduced
    from managed autoload to assert-only
  - mem: vm.page-cluster and vm.vfs_cache_pressure dropped as vendor
    duplicates
  - hud: cpu_temp commented out; cpu_custom_temp_sensor key corrected
  - bt: main.conf added with FastConnectable, AutoEnable and
    ReconnectAttempts
