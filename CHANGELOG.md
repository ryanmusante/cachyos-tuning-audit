CachyOS tuning profile — Beelink GTR9 Pro
==========================================

Newest first. Versions correspond to the ry-install.fish release that
applies the profile. Format: - subsystem: imperative summary.

# 7.135.1 - 2026-07-25

  Rebase across 5 upstream releases (7.130.0 -> 7.135.1). NO TUNING
  VALUE CHANGED. All 21 count tripwires, all 13 profile scalars and
  all 17 generated file bodies are byte-identical to the 7.130.0 pin,
  confirmed by diffing the previous edition's rendered Appendix B
  bodies against fresh generator output (18 of 20 fences matched
  exactly; the 2 misses are the size table and the remote-play diff
  fragment, both correct by construction). Everything in this edition
  is structural, verification-surface, or robustness. Re-derived from
  the 7.135.1 script by live evaluation; nothing carried forward
  (Rule 7).

  - docs: re-pin to ry-install 7.135.1 — 4963 lines / 292 functions /
    sha256 509c41bc on the attached archive; harness cut at the
    ARGPARSE banner (L4834); README and CHANGELOG re-pinned in
    lockstep
  - verify: the modprobe-leftover gap is CLOSED after four
    generations. _ry_stale_ry_dropins is one shared helper with two
    callers — _vss_modprobe warns, _check_record_orphans records
    MODPROBE_STALE_DROPIN to JSONL. Old P0 #15 is retired, not
    carried
  - verify: the stale-mask gap is CLOSED for detection.
    _ry_orphan_masked_units diffs the live masked set against MASK;
    _vss_orphan_masks reports it as INFO because mask ownership is
    unattributable, and --check records MASK_ORPHAN
  - design: neither orphan class sets DRIFT and neither is
    self-healing. A re-run cannot clear either, so exit 10 would go
    permanently non-zero and train the operator to ignore it. New
    P0 #21 evaluates that trade rather than treating it as a gap
  - packages: PKGS_ADD orphans remain undetectable BY DESIGN — the
    reasoning (a persisted manifest would add its own drift surface)
    is now on file and documented upstream, so it is recorded as a
    stated position in the protected list, not an oversight
  - preflight: a fourth validator, _ir_validate_sets, refuses deploy
    on PKGS_ADD n PKGS_DEL, MASK n EXPECTED_SERVICES, and
    MASK n _RY_PKG_MANAGED_SERVICES. All three shipped intersections
    are empty. Any candidate that adds a package or unit must clear
    all three
  - robustness: the one-time .ry.orig first-adoption preserve was
    DEAD CODE from 7.109.0 through 7.135.0 — a fish set -l inside an
    if block is not visible after it, so the branch never ran and 13
    of 17 destinations were overwritten unbacked. Fixed at 7.135.1.
    Appendix F1 is rewritten around it and new P0 #15 assesses the
    residual exposure and the primitive itself
  - robustness: backup coverage restated as two different guarantees.
    4 boot files get .ry.bak plus post-write verify/restore; the
    other 13 get a one-time .ry.orig and nothing more. Do not
    describe the 13 as backed up on every write
  - research: P0 #2 is re-framed from open design question to
    upstream verification. The profile now states in writing that the
    performance governor pins the EPP hint to its maximum and rejects
    any other value, retracting the older claim that powersave was
    required to honor EPP. The rejection half is new and testable —
    if it holds, the udev EPP rule issues a write that cannot land
  - research: a live source conflict is recorded rather than resolved
    in-brief. This document's own section 5 says
    pcie_aspm.policy=performance disables ASPM; the profile now says
    it biases links away from ASPM and to confirm with lspci -vv.
    Both agree that pcie_aspm=off does not disable ASPM. Settle it
    and name the trusted side; host audit A1 is still unrun
  - research: P0 #4 gains a testable claim — the profile now asserts
    that recent Proton-CachyOS builds copy the FSR4 DLL
    automatically, so PROTON_FSR4_UPGRADE mainly pins a version
  - errata: the stderr-write census is RECONCILED, not drifting. 78
    raw >&2 occurrences on 74 lines whole-file, of which 43 sit
    inside 17 function bodies and 35 are top-level pre-init
    preflight. Both figures are correct under their own scoping; the
    invariant means single authority for leveled user-facing output,
    not sole writer to fd 2
  - counts: functions 287 -> 292, verify functions 59 -> 60 (12
    orchestrators + 48 subs), sub markers 91 -> 93, preflight
    validators 3 -> 4. _msg_print moved L1105 -> L1117; the udev
    generator comments moved L870/875/877 -> L882/887/889
  - counts: a full perf-value change now touches 13 sites, up from
    10. The three new ones are README Service Keys rows that carry
    the governor, DPM and EPP values directly. Every line number in
    the old 10-site list is stale
  - method: determinism must be compared per file by content hash or
    with a sorted manifest — a concatenated hash over the output
    directory is not comparable between passes because glob order
    depends on harness filenames
  - method: build the harness by cutting at the ARGPARSE banner AND
    deleting the line-3 source guard, then source it as a non-root
    user. Census functions by set difference before and after
    sourcing, never by parsing
  - method: verify every before column against the old edition's
    RENDERED BODY, not against recollection. A drift row asserting a
    change that never happened passes every count check and every
    byte check

# 7.130.0 - 2026-07-20

  Rebase across 8 upstream releases (7.122.0 -> 7.130.0). The profile
  moved to a maximum-performance power posture and closed five of the
  previous edition's open research questions by code change. Every
  number in this edition was re-derived from the 7.130.0 script by
  live evaluation; nothing was carried forward from the 7.122.0
  edition (Rule 7).

  - docs: re-pin to ry-install 7.130.0 — 4915 lines / 287 functions /
    sha256 59e387af on the attached archive; harness cut at the
    ARGPARSE banner (L4786); README and CHANGELOG re-pinned in
    lockstep
  - power: CPUPOWER_GOVERNOR powersave -> performance,
    EPP_PREFERENCE balance_performance -> performance, GPU_DPM_LEVEL
    auto -> high — the three raises land together as one posture
    change; udev comments restated as "maximum CPPC hint" and
    "gfx1151 clock-floor; forced high"
  - kernel: KERNEL_PARAMS 14 -> 15 — mt7925e.disable_aspm=1 RE-ADDED,
    now as a cmdline module parameter rather than a modprobe drop-in;
    pairs with the retained pcie_aspm.policy=performance
  - env: ENV_VARS 11 -> 10 — FSR4_UPGRADE renamed to
    PROTON_FSR4_UPGRADE (closes old #2 in favor of the prefixed
    name); VKD3D_CONFIG=descriptor_heap removed, deferring to
    vkd3d-proton's own default
  - sysctl: SYSCTL_VALUES 10 -> 11 — kernel.nmi_watchdog=0 added at
    priority 95, which resolves the 7.122.0 assert/token asymmetry:
    the runtime assert is now backed by a value the profile actually
    sets
  - network: systemd-resolved gains explicit upstreams —
    DNS=94.140.14.14 94.140.15.15 (AdGuard ad-block tier), and the
    header text changes from "plaintext DNS, mDNS/LLMNR off
    (diverges from CachyOS DoT default)" to "AdGuard upstreams,
    plaintext, mDNS/LLMNR off". DNSOverTLS=no and DNSSEC=no were
    ALREADY the values at 7.122.0 and did NOT change in this window —
    do not read this entry as a DoT regression. Matches
    the router's own upstream posture; recorded as a closed decision,
    not a finding
  - modules: 60-ry-modules.conf false-state comment updated —
    "MT7925 ASPM handled on the kernel command line"
  - errata: amdxdna probe failure corrected in-script to -ENODEV
    (ret -19); the 7.122.0 brief said -EINVAL / -22, which was wrong.
    Recorded as a correction, not a delta
  - research: P0 queue rebuilt to 20 questions and renumbered fresh —
    #1 max-perf stacking under the 85 W ceiling, #2 whether a
    performance governor changes anything once EPP is pinned under
    amd_pstate=active, #3 the ASPM policy + module-parameter pair
  - research: five 7.122.0 questions CLOSED by code change — old #2
    (FSR4 name), #3 (AMD_VULKAN_ICD), #4 (VKD3D_CONFIG), #7
    (nmi_watchdog), #12 (MT7925 mitigation); tabulated in the header
    so a reader of the old edition sees them before the queue
  - research: old #13 and #25 are void for this host after the
    2026-07-19 fresh reinstall, but both code gaps stand and are kept
    as gaps rather than host findings
  - rules: add Rule 8 — a question closed by a code change is closed;
    do not re-litigate a decision the maintainer already executed
  - appendix: re-render all 17 generator bodies plus both toggle
    states (BLACKLIST_AMDXDNA=false NPU path, RY_REMOTE_PLAY_PORTS=
    true) by executing the 7.130.0 generators offline — 19 bodies,
    5093 B total, udev 639 B; determinism confirmed by 3x re-render
    (concatenated sha256 9b89bcc3d518a40e)
  - appendix: rename the trailing robustness appendix block from the
    nonexistent "G-L" heading carried by the 7.122.0 document to
    F1-F6; the appendix set is and remains A-E plus F
  - verify: 59 verify functions unchanged (12 orchestrators + 47
    subs); _vre_envvars iterates $ENV_VARS dynamically, so both env
    changes propagated with no verifier edit and no stale
    VKD3D_CONFIG assert exists
  - verify (report-level): §15 VERIFY block rewritten for the new
    baseline — governor/EPP/DPM asserted at their raised values,
    PROTON_FSR4_UPGRADE asserted present, VKD3D_CONFIG asserted
    absent, nmi_watchdog read from /proc and cross-checked against
    sysctl, mt7925e.disable_aspm asserted on the cmdline; adds an
    idle-floor turbostat measurement block feeding P0 #1
  - audit: re-verify the §6 modprobe-leftover gap at this baseline —
    60-ry-mt7925e.conf and 60-ry-blacklist-amdxdna.conf still have
    zero references in the script and zero in the README; the finding
    stands open across four generations
  - unchanged: PKGS_ADD 16, PKGS_DEL 9, MASK 11, the 8 logind keys,
    the mkinitcpio HOOKS/MODULES/COMPRESSION arrays, the 4 backup
    targets, 17 managed files, and the MangoHud body (byte-identical
    7.106 -> 7.130) all carry forward intact — 18 of the 21 oracle
    counts did not move

# 7.122.0 - 2026-07-19

  Rebase across 17 upstream releases (7.105.9 -> 7.122.0). This is not
  a version-string bump: 26 profile values changed. Every number in
  this edition was re-derived from the 7.122.0 script by live
  evaluation; nothing was carried forward from the prior edition.

  - docs: re-pin to ry-install 7.122.0 — 4906 lines / 287 functions /
    sha256 e46777cf on the attached archive; README and CHANGELOG
    re-pinned in lockstep
  - audit: add a "Baseline delta vs the 7.105.9 brief" section as the
    first section of the reference — 26-row matrix of every changed
    value, placed before Mission so a reader of the old edition sees
    the invalidated values before anything else
  - kernel: KERNEL_PARAMS 17 -> 14 — pcie_aspm=off REVERTED to
    pcie_aspm.policy=performance; nowatchdog, tsc=reliable and
    8250.nr_uarts=0 removed
  - env: ENV_VARS 12 -> 11 — PROTON_FSR4_RDNA3_UPGRADE renamed to
    FSR4_UPGRADE; AMD_VULKAN_ICD and DXVK_LOG_PATH removed;
    POWERDEVIL_NO_DDCUTIL=1 added
  - packages: PKGS_ADD 18 -> 16 — ddcutil and git-delta dropped
    (ddcutil removal pairs with the new PowerDevil DDC/CI opt-out)
  - services: MASK 12 -> 11 — modemmanager.service dropped;
    ufw.service is now a MASK member, i.e. neutralized by mask rather
    than by removal
  - network: systemd-resolved DNSSEC=allow-downgrade -> DNSSEC=no;
    header text corrected from "CachyOS DoH default" to "DoT default"
  - network: nftables rule order changed — ct state invalid drop now
    precedes established/related accept and the loopback accept
    (previously loopback first); appendix body updated to the new
    order
  - network: NetworkManager dispatcher logging drop-in is now its own
    managed destination
    (/etc/systemd/system/NetworkManager-dispatcher.service.d/logging.conf)
  - gates: record that the profile now carries NO version gates —
    MESA_MIN removed entirely, and the advisory kernel-floor comment
    is gone alongside the previously-removed hard validator; README
    Requirements table and kernel badge updated accordingly
  - verify: 59 verify functions (^function _v*) = 12 _verify_*
    orchestrators + 47 subs, every one cited by name in Appendix C;
    document the new _vrsv_user_units
    sub (asserts plasma-powerdevil.service not failed, warns on any
    failed systemctl --user unit) — reads as the in-script remediation
    for the F-7 PowerDevil crash-loop
  - appendix: re-render all 17 generator bodies plus both toggle
    states (BLACKLIST_AMDXDNA=false NPU path, RY_REMOTE_PLAY_PORTS=
    true) by executing the 7.122.0 generators offline; determinism
    confirmed by 3x re-render with identical sha256 for all 17
  - appendix: byte-verify every appendix fence against the captured
    generator output — 16/16 EXACT, 0 mismatches
  - appendix: correct _RY_ARGPARSE_SPEC to 6 entries (the 7.105.9
    brief recorded 7 — that was wrong, not a delta) and list all
    entries; dependency roster stated as 37 hard + 20 warn-only
  - research: P0 queue 19 -> 24 questions, reordered — the ASPM
    link-state audit and the FSR4 variable-name correctness question
    take the top two slots; the rename toward LESS specificity is
    flagged as the highest-value single question in the brief
  - research: add the nmi_watchdog assert/token asymmetry as a new
    finding — nowatchdog was dropped from KERNEL_PARAMS but --verify
    still hard-asserts nmi_watchdog=0, so either CachyOS yields it by
    default or --verify fails on a clean 7.122.0 boot; cheapest
    testable item in the queue
  - research: add the RADV-selection question — with AMD_VULKAN_ICD
    gone, _vre_envvars no longer asserts any ICD selection
  - audit: re-verify the §6 modprobe-leftover gap at this baseline —
    60-ry-mt7925e.conf and 60-ry-blacklist-amdxdna.conf still have
    zero references in the script and zero in the README; the finding
    stands open across three generations
  - rules: add Rule 7 — do not carry values forward from any
    pre-7.122.0 edition of this brief; re-deriving a removed token is
    a stale-source error, not a finding
  - protected: extend the do-not-recommend list with all 7.106-7.122
    removals and the maintainer's closed candidate ledger (WINEDEBUG,
    BlueZ AutoEnable, NM wifi.backend, cachyos-gaming-meta,
    MODULES=(amdgpu), lib32-mesa all stay); add "do not recommend
    returning ufw to PKGS_DEL"
  - verify (report-level): §15 VERIFY block rewritten for the new
    baseline — ASPM token matched as pcie_aspm\S* rather than =off,
    removed tokens asserted absent, nmi_watchdog read from /proc, ufw
    asserted masked, ddcutil asserted absent, FSR4_UPGRADE and
    POWERDEVIL_NO_DDCUTIL asserted present
  - sysctl/hud/logind/mkinitcpio: re-confirmed unchanged — all 10
    sysctl values, the MangoHud body (byte-identical 7.106 -> 7.122),
    the 8 logind keys, and the mkinitcpio HOOKS/MODULES/COMPRESSION
    arrays carry forward intact

  Second pass (same date, same source v7.122.0 — independent
  re-derivation rather than re-reading the first pass):

  - audit: re-render all 17 generator bodies from a clean harness run
    and byte-compare against the appendices — 16/16 EXACT; the two
    user-scope files differ only in captured FILENAME ($HOME vs
    /home/ryan), content byte-identical
  - audit: re-derive the full 21-entry count oracle by live eval —
    every count matches the shipped figures; note that `eval echo
    \$\$name` collapses arrays to one element and silently reports
    every count as 1, so array counts require `eval set vals \$\$name`
  - fix: correct the verify-function figure — the brief claimed 57
    with no valid basis. Live enumeration of `^function _v*` gives 59
    = 12 _verify_* orchestrators + 47 subs; corrected in the delta
    table, Appendix C, and the README count matrix
  - appendix: close the Appendix C citation gap — _verify_static_packages,
    _vmh_existence_only and _vmh_order_checks were defined in the
    script but uncited; all 59 verify functions are now named, and
    Appendix C citation coverage is verified 59/59 with zero ghost
    references
  - research: NEW FINDING P0 #25 — stale-mask gap. The script never
    unmasks: _configure_services_mask only adds, and both
    _verify_static_services and _vrsv_masked_inactive iterate $MASK.
    A unit DROPPED from MASK therefore stays masked forever on an
    already-deployed host and falls outside all verify coverage.
    modemmanager.service is masked-and-orphaned on any host deployed
    at <=7.121; §6, §9 and the §15 VERIFY line corrected accordingly
    (the prior edition wrongly implied it would read as unmasked)
  - protected: state the removal-reconciliation asymmetry as a general
    rule — values inside generated files self-heal (generators rewrite
    wholesale), values in external system state do not (a package
    dropped from PKGS_ADD stays installed; a unit dropped from MASK
    stays masked). ddcutil is the live example: dropped from PKGS_ADD
    but still referenced by the L4455 i2c-group hint
  - verify (report-level): script re-measured at 4906 lines / 287
    functions with 287/287 carrying --description and zero duplicate
    names; 37 hard + 20 warn-only dep roster re-counted exact; all 17
    destinations resolve to both a content generator and a post-hook;
    P0 queue re-checked continuous 1..25 with all cross-references in
    range; 3 box matrices re-checked non-ragged; 25 fences balanced

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
