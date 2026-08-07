Changes for cachyos-tuning-audit
================================

Newest first. Versions track the pinned ry-install.fish release.


7.158.0
-------

  - docs: re-pin 7.155.0 -> 7.158.0 (4915 lines, 294 functions,
    sha256 13a467b8; zip cc97b5a5; harness cut L4786, unmoved)
  - surface: no perf value changed, now twenty-eight releases (7.130.0
    -> 7.158.0). All four perf scalars identical
  - surface: five generated bodies changed; 17-file total 4,949 ->
    4,858 B
  - surface: mkinitcpio.conf changed content at ZERO byte delta,
    COMPRESSION_OPTIONS -1 -> -3; only the fence diff catches it
  - surface: kernel/cmdline 352 -> 351 B and sdboot-manage.conf 544 ->
    543 B after fsck.mode=force -> auto
  - surface: 95-ry-overrides.conf 441 -> 376 B; both netdev_budget
    keys removed at 7.157.0
  - surface: 10-environment.conf 306 -> 282 B; PROTON_ENABLE_WAYLAND
    removed after 7.155.0
  - gates: all eight T0 observations returned. T0 is now a results
    table, not an action list
  - gates: idle floor 21.33 / 21.69 W package, 3.93 / 4.20 W core at
    Busy% 0.13 / 0.24; max_cstate=1 retired to KEEP
  - gates: every PCIe link reads LnkCtl ASPM Disabled, so
    pcie_aspm.policy=performance does real work on this board
  - gates: root filesystem is ext4, k10temp exposes only Tctl
    (temp1_input), and energy_uj is mode 400
  - gates: softnet squeezed is 0 on all 32 CPUs; both 10 GbE links are
    down and wlan0 carries the default route
  - errata: T3-3 was framed against a Btrfs root. The root is ext4, so
    fsck.mode=force forced a full check on every boot
  - errata: two oracle values moved, ENV_VARS 10 -> 9 and
    SYSCTL_VALUES 11 -> 9; the never-moved claim is retired
  - errata: T2-3 is declined on PLATYPUS grounds, not scope; relaxing
    energy_uj re-opens the RAPL side channel
  - errata: the 278 expected verify OK count is stale and now UNKNOWN;
    three assertion counts fell with no verify-function edit
  - errata: line anchors did not move this rebase, the first time.
    Always locate the symbol regardless
  - scope: T1-2 closed by the artifact, the second such retirement in
    three rebases; T2-1 closed on measurement
  - scope: T3-1 and T3-2 retired to KEEP by maintainer decision;
    record the trade rather than re-proposing removal
  - scope: add T4-7 icmpv6 coupling and T4-8 /boot post-hook glob as
    tracked LOW and INFO items
  - scope: add a tier-3 removal row; an ENV_VARS drop persists in a
    live systemd --user session until logout
  - research: re-fix the upstream version set on 2026-08-07; mainline
    7.2.0-rc6, linux-cachyos 7.1.6-1, Mesa 26.3.0-devel, all unmoved
  - research: MangoHud #1794 and systemd #33579 both still open


7.155.0
-------

  - docs: re-pin 7.141.0 r2 -> 7.155.0 (4915 lines, 294 functions,
    sha256 0f65a126; zip ebd5fc13; harness cut L4786)
  - surface: no perf value changed, now twenty-five releases (7.130.0
    -> 7.155.0). 21 tripwires and 4 perf scalars identical
  - surface: three generated bodies DID change and none moved a count;
    17-file total 5,089 -> 4,949 B
  - surface: 99-cachyos-resolved.conf 154 -> 90 B after the 7.147.0
    DNS= cut and the 7.148.0 DNSOverTLS/DNSSEC cut
  - surface: 99-cachyos-nm.conf 219 -> 148 B; 7.147.0 removed the
    global-dns and global-dns-domain-* blocks
  - surface: 10-environment.conf 311 -> 306 B; 7.154.0 swapped
    PROTON_FSR4_UPGRADE=1 for FSR4_WATERMARK=1
  - errata: the T5 DNS item was wrong on both halves. The host pins
    nothing and the router has run DoT to AdGuard since 2026-07-31
  - errata: T1-3 is closed by the artifact, not withdrawn on evidence.
    There is no global FSR4 flag left for the RDNA3 workaround
  - errata: a perf-value change touches 11 sites, not 13. The udev
    generator description and the README prose were never edit sites
  - errata: sixteen service keys, not nineteen. The old figure listed
    RESOLVED_DOT, removed at 7.148.0
  - errata: BOOT_SPACE_CRIT/WARN is MiB, not MB, and the 278 verify OK
    count is empirical rather than a script literal
  - errata: the sysctl drop-in does NOT annotate 3 of 11 keys at 64
    chars. It carries one header comment and never annotated keys
  - errata: the udev comment lines are sites 4 and 5 of the 11-site
    list; the stale back-reference said 5 and 6 of 13
  - anchors: every line number moved; the script lost 75 lines to
    helper packing and the DNS removals
  - anchors: perf globals shift -1 to 585/587/589/590/591 and MASK to
    609; harness cut moves backwards 4861 -> 4786
  - anchors: _ir_validate_post_hooks 4591 -> 4524,
    _check_record_orphans 2632 -> 2589, _msg_print 1109 -> 1087
  - anchors: README perf sites 84/86/189/193/194 -> 96/98/201/205/206
  - research: re-fetch fourteen settled claims on 2026-08-05; mainline
    is 7.2.0-rc6 and linux-cachyos 7.1.6-1
  - research: clearcpuid still absent from kernel-parameters.txt at
    rc6; dynamic_epp still defaults false at v7.1
  - research: MangoHud #1794 and systemd #33579 both still open
  - scope: add T0-8, read per-link DNS off the running system; nothing
    in the verify surface asserts it any more


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
