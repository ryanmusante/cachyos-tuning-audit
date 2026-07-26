CachyOS tuning profile — Beelink GTR9 Pro
==========================================

Newest first. Versions correspond to the ry-install.fish release the
brief is pinned to. Format: - subsystem: imperative summary.
Blocks below the top two are merged ranges; per-release detail for
those windows lives in the ry-install CHANGELOG.


# 7.139.0 - 2026-07-26

  Rebase 7.135.1 -> 7.139.0 r3, plus a full restructure. NO TUNING
  VALUE CHANGED — that is now nine releases (7.130.0 -> 7.139.0) with
  a frozen surface: 21 count tripwires, every profile scalar and all
  17 generated bodies byte-identical at 5,093 B, determinism 3/3.
  The document was reorganised from subsystem order into
  implementation-safety order and trimmed from 1,140 lines / 156,506 B
  to 970 lines / 65,401 B by folding settled research into conclusions
  and dropping fifteen duplicated file bodies.

  - docs: re-pin to ry-install 7.139.0 r3 — 4974 lines / 293 functions
    / sha256 bcbb695f; zip ec54f631; harness cut at the ARGPARSE
    banner (L4845). Three artifacts self-report v7.139.0 — the pin is
    r3, identified by zip or CHANGELOG hash, never by --version
  - structure: replace §1-§15 subsystem order with a single action
    queue ordered by blast radius, T0 (observation only) through T5
    (closed, do not touch). Four of the six T0 items gate a decision
    in a lower tier, so they run first
  - structure: fold the 21-question P0 queue into two sections — a
    settled table (closed by upstream evidence, do not re-research)
    and the open items, which now live inside their own tier
  - research: P0 #2 CLOSED — v6.18 amd-pstate.c returns -EBUSY for any
    epp>0 under the performance governor and forces epp 0. Writing
    "performance" maps to epp 0 and IS accepted, so the udev EPP write
    lands as a redundant no-op. Rejection is pr_debug, never in default
    dmesg. The profile README is correct on both halves
  - research: P0 #3 CLOSED from the primary source — Kconfig
    PCIEASPM_PERFORMANCE disables ASPM L0s and L1 even where the BIOS
    enabled them. The source conflict recorded at 7.135.1 is resolved
    in favour of the brief's original reading; pcie_aspm=off still does
    NOT disable ASPM. Host audit A1 (lspci LnkCtl) remains unrun and is
    now T0-2
  - research: P0 #4 CLOSED — proton-cachyos 11.0-20260702 copies the
    FSR4 DLL automatically and PROTON_FSR4_UPGRADE version-pins.
    KEEP-with-note; the README needs no edit
  - research: close #6, #9, #10, #12, #14, #19 and the PowerDevil and
    descriptor-heap items into the settled table with citations
  - scope: RY_REMOTE_PLAY_PORTS is GONE (ry-install 7.137.0). The gate,
    the Sunshine/Steam port sets and the 916 B ruleset variant no
    longer exist; the nftables body is 729 B in a single form. Old §11
    item 8 and the 5353/udp mDNS note are retired, not carried
  - scope: the preemption-model advisory is GONE (7.139.0 r2), with its
    dmesg fetch, both cache globals and the dmesg optional dependency.
    Residue confirmed 0. Optional deps 20 -> 19
  - errata: the 7.135.1 Appendix B B10 fence was captured from the
    BLACKLIST_AMDXDNA=false variant (177 B) while the size table
    correctly recorded the shipped default (183 B). Corrected
  - errata: every script line anchor in the previous edition is off by
    one to two lines. Perf globals L586/L588/L590/L591, MASK L610,
    _msg_print L1109, validator chain L779-782. All re-derived
  - verify: 60 -> 61 verify functions (12 orchestrators + 49 subs);
    the addition is _vsb_entry_options, which asserts every
    non-fallback BLS entry carries all 15 KERNEL_PARAMS tokens
  - verify: record the 7.139.0 additions — full cpufreq-policy
    uniformity sweep in _vrk_cpu_state, resolved unit-file state
    (enabled|static), admin-scope-only orphan-mask filtering, root FS
    type and ext4 entry count, absent-vs-unreadable sysctl split,
    millisecond JSONL timestamps, CHECK_GREP key=value form
  - method: add the writable-$HOME harness trap — without it the source
    aborts part-way through log-directory init and every count reads 0,
    the same symptom as a missing L3 source-guard deletion but a
    different cause
  - method: record the toggle-based fence walker as the technique that
    found the B10 defect, and the generator glob filter
    (_content__* + _content_HOME*) that excludes the dispatcher


# 7.135.1 - 2026-07-25

  Rebase across five upstream releases (7.130.0 -> 7.135.1). No tuning
  value changed. Structural, verification-surface and robustness only.

  - verify: the modprobe-leftover gap CLOSED after four generations —
    _ry_stale_ry_dropins is one shared helper with two callers,
    _vss_modprobe (WARN) and _check_record_orphans (JSONL)
  - verify: the stale-mask gap CLOSED for detection —
    _ry_orphan_masked_units diffs the live masked set against MASK;
    reported as INFO because mask ownership is unattributable
  - design: neither orphan class sets DRIFT and neither is
    self-healing; exit 10 would go permanently non-zero and train the
    operator to ignore it
  - preflight: fourth validator added — _ir_validate_sets refuses
    PKGS_ADD n PKGS_DEL, MASK n EXPECTED_SERVICES and
    MASK n _RY_PKG_MANAGED_SERVICES
  - robustness: the .ry.orig first-adoption preserve was DEAD CODE from
    7.109.0 through 7.135.0 — a fish set -l inside an if block is not
    visible after it, so 13 of 17 destinations were overwritten with no
    backup of any kind and no forensic trace. Fixed at 7.135.1
  - packages: PKGS_ADD orphans remain undetectable by design; a
    persisted manifest would add its own drift surface


# 7.130.0 - 2026-07-20

  The last release that moved a tuning value.

  - power: CPUPOWER_GOVERNOR powersave -> performance, EPP
    balance_performance -> performance, GPU_DPM_LEVEL auto -> high
  - kernel: KERNEL_PARAMS 14 -> 15 — mt7925e.disable_aspm=1 re-added
    alongside pcie_aspm.policy=performance
  - env: ENV_VARS 11 -> 10 — FSR4_UPGRADE renamed PROTON_FSR4_UPGRADE
  - sysctl: SYSCTL_VALUES 10 -> 11 — kernel.nmi_watchdog=0 added at
    priority 95, resolving the older assert/token asymmetry without
    restoring the nowatchdog boot token
  - network: systemd-resolved gains explicit AdGuard upstreams
  - errata: the amdxdna probe failure is -ENODEV (ret -19)


# 7.105.8 - 7.122.0 - 2026-07-14 .. 2026-07-19

  - kernel: pcie_aspm=off reverted to pcie_aspm.policy=performance;
    all version gates removed (hard kernel floor, then the Mesa soft
    warning) and none have returned
  - env: VKD3D_CONFIG=descriptor_heap added then dropped;
    PROTON_FSR4_RDNA3_UPGRADE renamed
  - sysctl: vm.watermark_boost_factor=0 added
  - packages: ddcutil and git-delta dropped; modemmanager.service
    dropped from MASK
  - network: DNSSEC allow-downgrade -> no; nftables rule order changed
    so ct state invalid drop precedes the loopback accept; the
    dispatcher logging drop-in became its own managed file
  - docs: the reference was rewritten as a deep-research brief;
    Rule 7 added (do not carry values forward from an older edition)
  - audit: the removal-reconciliation asymmetry stated as a general
    principle; the stale-mask gap first raised


# 7.88.2 - 7.101.0 - 2026-07-03 .. 2026-07-12

  - kernel: IPv6 disabled system-wide with an IPv4-only nftables
    ruleset, coupled at preflight; clearcpuid 514 renamed to the
    version-stable umip string form
  - modprobe: standalone drop-ins merged into 60-ry-modules.conf
  - services: avahi-daemon service and socket masked (second mDNS
    responder); NTP remediation made unconditional
  - gpu: udev EPP and GPU matchers fixed — the prior GPU rule never
    applied, voiding any earlier high-vs-auto observation
  - backup: .ry.bak plus post-write verify/restore across all four
    boot-critical destinations
  - verify: tcp, zram, ntsync-module, iommu and clocksource asserts
    retired; Vulkan package check folded into the required-package check
  - packages: archlinux-contrib dropped


# 7.70.0 - 7.77.1 - prior .. 2026-06-28

  - net: IOMMU inverted to amd_iommu=off (AMD-Vi off; the XDNA NPU is
    the named casualty); Wi-Fi backend set to wpa_supplicant
  - gpu: RADV drirc and TTM/GTT module params removed; ntsync reduced
    from managed autoload to assert-only
  - mem: vm.page-cluster and vm.vfs_cache_pressure dropped as vendor
    duplicates
  - hud: cpu_temp commented out; cpu_custom_temp_sensor key corrected
  - bt: main.conf added (FastConnectable, AutoEnable, ReconnectAttempts)
