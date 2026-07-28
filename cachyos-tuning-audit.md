# CachyOS Tuning Audit — Beelink GTR9 Pro (gfx1151)

Pinned to `ry-install.fish` **7.141.0 r2**. Deep-research brief: actionable items only,
**ordered by implementation safety** rather than by subsystem.

---

## 0. Provenance

Every number below was re-derived by live evaluation of the attached archive. Nothing was
carried forward from the 7.139.0 edition unaudited.

```
║ ARTIFACT           ║ SHA256   ║ SIZE               ║
║────────────────────║──────────║────────────────────║
║ zip                ║ 76c47d95 ║ 327,084 B          ║
║ ry-install.fish    ║ f8210c9e ║ 4,990 L / 295,103 B║
║ README.md          ║ ab63febc ║   291 L /  21,911 B║
║ CHANGELOG.md       ║ 79661db3 ║   183 L /   8,313 B║
║ LICENSE            ║ 2e1e7c8a ║    21 L /   1,069 B║
```

Archive layout: 5 entries, `zip -0 -X` Stored, topdir `ry-install-v7_141_0`, script 0755,
docs 0644. All three content files self-report `7.141.0`.

**Disambiguate by zip, README or CHANGELOG hash — never by `--version`, and not by script
hash either.** 7.141.0 shipped twice; r1 and r2 carry the *same* script (`f8210c9e`,
r2 being documentation-only), so the script hash separates 7.141.0 from 7.140.0 but does
not separate r1 from r2. Superseded r1: zip `b85911f4` / README `9e62d633` /
CHANGELOG `c3129862`. This brief audits **r2**.

Upstream reference points fixed at audit time (2026-07-27):

```
║ COMPONENT              ║ VERSION AT AUDIT ║ SOURCE                            ║
║────────────────────────║──────────────────║───────────────────────────────────║
║ Linux mainline         ║ 7.2.0-rc5        ║ torvalds/linux Makefile           ║
║ linux-cachyos          ║ 7.1.5-2          ║ CachyOS/linux-cachyos PKGBUILD    ║
║ Mesa main              ║ 26.3.0-devel     ║ mesa/mesa VERSION                 ║
║ proton-cachyos         ║ 11.0-20260703    ║ CachyOS/proton-cachyos releases   ║
║ linux-firmware MES     ║ 2026-05-07 tag   ║ gc_11_5_1_mes_2.bin log           ║
```

---

## 1. Delta vs the 7.139.0 edition

### 1a. The tuning surface still has not moved

**Not one tuning value changed 7.139.0 → 7.141.0.** All 21 count tripwires and all four
perf scalars are identical; 16 of 17 generated bodies are byte-identical. Determinism 3/3
renders, sorted-manifest sha `5be1a60220ba4330` (harness-filename dependent — NOT
comparable to the `6a64251d19c450dc` of the previous edition; per-file content hashes are
the safe form).

**That is now eleven releases (7.130.0 → 7.141.0) with a frozen tuning surface.** A
recommendation asserting that any tuning value moved in this window is a stale-source
error, not a finding.

The single byte movement in the window is **not** a tuning value:

```
║ FILE                 ║ 7.139.0 ║ 7.141.0 ║ CAUSE                               ║
║──────────────────────║─────────║─────────║─────────────────────────────────────║
║ /etc/mkinitcpio.conf ║   280 B ║   276 B ║ redundant -T0 dropped from          ║
║                      ║         ║         ║ MKINITCPIO_COMPRESSION_OPTIONS      ║
║ 17-file total        ║ 5,093 B ║ 5,089 B ║ same edit, oracle 2 -> 1            ║
```

`zstd -T0` means "use all cores"; mkinitcpio already threads by default, so the flag was
redundant, not a behaviour change. **Do not report a compression regression.**

### 1b. What did change

```
║ AREA          ║ 7.139.0 r3 ║ 7.141.0 r2 ║ NOTE                                 ║
║───────────────║────────────║────────────║──────────────────────────────────────║
║ script        ║ 4,974 L    ║ 4,990 L    ║ +16                                  ║
║ functions     ║ 293        ║ 294        ║ +_vsb_sdboot_dropins (L2151)         ║
║ verify fns    ║ 61 (12+49) ║ 62 (12+50) ║ orchestrators unchanged at 12        ║
║ sub: markers  ║ 94         ║ 95         ║ all PARENT_OK                        ║
║ hard deps     ║ 37         ║ 37         ║ unchanged                            ║
║ optional deps ║ 19         ║ 15         ║ dead tokens cut (7.140.0 r2)         ║
║ banners       ║ 90         ║ 90         ║ unchanged                            ║
║ generators    ║ 17         ║ 17         ║ Sigma 5,093 -> 5,089 B               ║
║ oracle        ║ 21         ║ 21         ║ one VALUE moved: COMPRESSION_OPTIONS ║
║ harness cut   ║ L4845      ║ L4861      ║ ALWAYS locate, never hardcode        ║
║ README        ║ 318 L      ║ 291 L      ║ restructured at 7.140.0 r3 / 7.141.0 ║
║ CHANGELOG     ║ 197 L      ║ 183 L      ║ blocks merged 12 -> 11               ║
```

**New verify coverage — `_vsb_sdboot_dropins` (L2151).** Enumerates
`/usr/lib/sdboot-manage.conf.d` and `/etc/sdboot-manage.conf.d` for `*.conf` at depth 1.
Zero drop-ins is `_ok`; any drop-in is a **WARN** plus a `SDBOOT_DROPIN_PRESENT` JSONL key,
because sdboot-manage sources drop-in directories *after* `/etc/sdboot-manage.conf`, so a
drop-in silently outranks the managed `LINUX_OPTIONS`. It does not set DRIFT. This closes
the last silent-override path in the boot chain that `_vsb_cmdline` could not see.

### 1c. Removals that must not be re-derived from a stale source

Four features are gone. Each is a **retired question**, not an open gap:

```
║ REMOVED                       ║ AT       ║ CONSEQUENCE FOR FINDINGS             ║
║───────────────────────────────║──────────║──────────────────────────────────────║
║ RY_REMOTE_PLAY_PORTS + gate,  ║ 7.137.0  ║ nftables body is 729 B, SINGLE form. ║
║ Sunshine/Steam port sets,     ║          ║ No port set to open or close. The    ║
║ the 916 B ruleset variant     ║          ║ 5353/udp mDNS finding is retired     ║
║ Preemption-model advisory,    ║ 7.139.0  ║ Profile never pinned preempt=.       ║
║ dmesg fetch, both cache       ║ r2       ║ Do NOT report a missing preempt      ║
║ globals, dmesg optional dep   ║          ║ check. Residue 0                     ║
║ Redundant -T0 compression flag║ 7.140.0  ║ Oracle 2 -> 1. Do NOT report this as ║
║                               ║          ║ a lost threading option              ║
║ 4 dead optional-dep tokens    ║ 7.140.0  ║ Optional deps 19 -> 15. The 15 that  ║
║                               ║ r2       ║ remain are all live call sites       ║
```

---

## 2. Action queue — ordered by implementation safety

The operative section. Tiers ascend by blast radius: T0 changes nothing, T5 must not be
touched. **Work top-down; do not act on a lower tier while a higher tier that gates it is
unrun.** Every recommendation must carry a tier.

### T0 — Observation only. No system change, no reboot, no risk.

Every item is a read. Five of the seven gate a decision in a lower tier.

```
║ ID   ║ ACTION                            ║ GATES             ║ STATUS       ║
║──────║───────────────────────────────────║───────────────────║──────────────║
║ T0-1 ║ turbostat idle-floor capture      ║ every T3 power    ║ UNRUN —      ║
║      ║ (60 s idle, before/after)         ║ decision          ║ 11 releases  ║
║ T0-2 ║ lspci -vv LnkCtl ASPM per link    ║ T3-4 ASPM pair    ║ UNRUN        ║
║ T0-3 ║ findmnt root FS type              ║ T3-3 fsck.mode    ║ UNRUN        ║
║ T0-4 ║ k10temp hwmon + energy_uj mode    ║ T1-1 / T2-3 HUD   ║ UNRUN        ║
║ T0-5 ║ softnet_stat squeezed column      ║ T2-1 netdev budget║ UNRUN — NEW  ║
║ T0-6 ║ amd_pstate dynamic_epp state      ║ T5 EPP no-op call ║ UNRUN — NEW  ║
║ T0-7 ║ .ry.orig / .ry.bak inventory      ║ nothing — forensic║ UNRUN        ║
```

Advisory reads with no gate: linux-firmware MES revision, installed Proton-CachyOS build.

**T0-1 is the single highest-value action in this brief and has never been run.** The
profile is pinned at maximum on three perf axes simultaneously — governor `performance`,
EPP `performance`, DPM `high` — while `processor.max_cstate=1` already blocks deep CPU
idle, under an 85 W BIOS ceiling shared between CPU and GPU. Three of those four raise the
**idle floor** rather than the load ceiling, and the 85 W PPT caps peak power regardless.
The profile CHANGELOG concedes the direction of the trade (peak unchanged, idle up); that
is a stated expectation, not a measurement. Quantify idle package power and whether
sustained-load clocks *drop* because budget is burned at idle. Name the assumed budget
(85 W README ceiling vs 140 W stock) in every power statement.

**T0-5 is new and it is the correct gate for T2-1.** Both Red Hat and ESnet make
`netdev_budget` tuning conditional on observed budget exhaustion, which is column 3
("squeezed") of `/proc/net/softnet_stat`. If that column is flat under load, the profile's
raised values are inert and the whole T2-1 question is moot.

**T0-6 is new and it is a live-behaviour question, not a version question.** See §3:
`dynamic_epp` is no longer master-only — it ships in the kernel CachyOS is currently
building. If it ever reads `enabled`, every manual EPP write returns `-EBUSY`.

Commands for all seven are in §6.

### T1 — User/session scope. No root, no reboot, reversible by editing one file.

- **T1-1 · MangoHud `cpu_custom_temp_sensor` → k10temp.** The option is current upstream
  and takes a `<hwmon-name>,<input>` pair — documented form
  `cpu_custom_temp_sensor=cpuss0_2,temp3_input`. It points the HUD at an explicit hwmon
  node and fixes the asusec mispick on this board. **Gated on T0-4** for the actual hwmon
  name and temp input index — do not guess them.
  **Do not enable `cpu_temp` as part of this.** MangoHud #1794 (cpu_power reads 0 on Zen 5
  while cpu_temp is active) was re-confirmed **OPEN** on 2026-07-27, so enabling `cpu_temp`
  still costs `cpu_power`. The custom-sensor option is the only path that might allow both,
  and that is a hypothesis to test, not a known fix. IMPACT Low · RISK Low.
- **T1-2 · `PROTON_ENABLE_WAYLAND=1` should move from the global env file to per-game Steam
  launch options.** Upstream documents it under "Pass in the following environment
  variables **to your game**", aliased `PROTON_USE_WAYLAND=1`, alongside a note that Steam
  must be launched with `-steamos3` for Steam Input to work under winewayland. Shipping it
  in `~/.config/environment.d/` applies it to every Proton title and taints every future
  bug report against proton-cachyos. Removal drops ENV_VARS 10 → 9 and is Tier-1 under
  §7e (self-heals; nothing can orphan). IMPACT Med · RISK Low. **This is the strongest
  single T1/T2 candidate in the brief.**
- **T1-3 · `PROTON_FSR4_UPGRADE=1` is under-specified for RDNA 3.5, not obsolete.**
  Upstream: RDNA3-class hardware runs FSR4 in FP16 mode **with graphical glitches** unless
  `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` is also set, and FSR 4.0.1 on RDNA3 carries a
  large performance penalty versus 4.0.0. gfx1151 is RDNA 3.5. The profile sets the upgrade
  flag globally with neither the workaround nor a version pin. Two coherent options: (a)
  pair the workaround per-game and leave the global flag, or (b) move both per-game.
  Verification exists upstream: `FSR4_WATERMARK=1` renders a corner watermark when FSR4 is
  actually active. IMPACT Med · RISK Low.
- **T1-4 · Both user files have a working first-adoption preserve.** As of 7.135.1,
  `~/.config/environment.d/10-environment.conf` and `~/.config/MangoHud/MangoHud.conf` get
  a one-time `.ry.orig`. Note this when recommending any user-scope change: the *first*
  hand-edit is preserved, every subsequent one is silently overwritten by design. Both land
  0600 by design (`_ry_install_file` L2084 sets 0644, then 0600 when `use_sudo` is false).
  Do not raise the mode as drift.

### T2 — Managed config values. Root, no reboot, self-heals on the next deploy.

Everything here is Tier-1 under the removal-reconciliation model (§7e): the generator
rewrites the file wholesale, so a bad value cannot orphan.

- **T2-1 · `netdev_budget_usecs=5000` — the attribution in the previous edition was wrong,
  and the correct authorities disagree with each other.** Kernel defaults are 300 / 2000.
  The 600 / 4000 pairing is **Red Hat's** documented guidance (RHEL 8, 9 and 10 all publish
  the same "double the current values" recipe), **not ESnet's**. ESnet's 100 G "Other
  Tuning" page says the opposite: leave the `net.core.netdev` settings at their defaults,
  and cautions that changing them has been seen to decrease throughput considerably. The
  profile ships budget 600 (an exact doubling, matching Red Hat) with usecs 5000 (2.5x,
  matching nobody). **Gated on T0-5** — every authority makes the change conditional on
  softnet_stat column 3 rising under load. If squeezed is flat, recommend reverting both to
  defaults, or state plainly that the values are inert. IMPACT Low · RISK Low.
- **T2-2 · nftables rule order — resolve to KEEP, downgrade from open question.** The
  profile places `ct state invalid drop` before `iif "lo" accept`. That ordering is the one
  used by the Gentoo wiki's reference workstation ruleset and by the widely redistributed
  "early drop of invalid packets" template; the nftables.org quick reference and the
  ArchWiki dispatcher example put loopback first. **There is no upstream rule requiring
  loopback-first**, and no documented case of legitimate loopback traffic being classified
  `invalid` on a host that is not doing NAT or policy routing — this host does neither.
  KEEP with a note. If a FIX is nonetheless argued, it is a pure order swap, `nft -c`
  gated, re-validated by `_post_nft`. IMPACT Low · RISK Low.
- **T2-3 · `energy_uj` world-readability for MangoHud `cpu_power`.** If T0-4 shows the
  powercap node is root-only, the fix is a permission drop-in — which would be an
  **18th managed file**, a scope addition, not a value change. Weigh a new managed
  destination against leaving `cpu_power` degraded. Note it would also change three oracle
  counts (`SYSTEM_DESTINATIONS` 15→16, `_RY_POST_HOOKS` 17→18 if it needs a hook, managed
  files 17→18) and every count is a hard deploy gate. IMPACT Low · RISK Low-Med.
- **T2-4 · `amd_pstate=active` restates the CachyOS compiled default.** linux-cachyos sets
  `CONFIG_X86_AMD_PSTATE_DEFAULT_MODE=3`, and `drivers/cpufreq/Kconfig.x86` defines 3 as
  **Active (EPP)**. The cmdline token therefore changes nothing on the kernel this host
  actually runs. **KEEP** — the profile does not own the kernel package, the mode is a
  build-time choice that a CachyOS rebuild could flip, and the token costs one cmdline slot.
  But the previous edition's framing that it is "the root of the whole CPU story" needs
  qualifying: on *this* kernel it is belt-and-braces. Contrast with T3-4, where the
  equivalent config option is explicitly **not** set and the cmdline token is load-bearing.
- **T2-5 · `VKD3D_CONFIG=descriptor_heap` removal — validated, close it.** Mesa main
  (26.3.0-devel) documents `RADV_DEBUG=noheap` as the switch that *disables*
  `VK_EXT_descriptor_heap`, and `heap` no longer appears in the `RADV_EXPERIMENTAL` list.
  Default-on is confirmed; the removal is correct on any Mesa this profile will meet.
  Residual check only: confirm vkd3d-proton has no separate opt-in the removal also dropped.

### T3 — Kernel command line. Root + reboot. Reversible, but costs a boot cycle.

All four are **open maintainer trade-offs, not defects.** Flag and quantify; do not
auto-FIX. The cmdline is charset-gated at L731 (`^[A-Za-z0-9._,=-]+$`) and count-asserted at 15 —
any add or removal updates both, `_vsb_entry_options` (L2219) asserts every non-fallback
BLS entry carries every token, and `_vsb_sdboot_dropins` (L2151) now warns when a drop-in
could override `LINUX_OPTIONS` behind all of it.

- **T3-1 · `processor.max_cstate=1` — the highest-leverage single token in the set.** It
  blocks deep CPU idle, costs idle power and boost residency under the 85 W ceiling, and
  compounds directly with DPM pinned `high`. **Gated on T0-1**; do not decide without the
  measurement. Higher leverage than all three perf globals combined.
- **T3-2 · `clearcpuid=umip` — kernel tainted, and the documentation is gone.**
  Re-confirmed 2026-07-27 against mainline 7.2.0-rc5: `clearcpuid` has **zero occurrences**
  in `Documentation/admin-guide/kernel-parameters.txt` while the code still parses it. That
  is undocumented-but-live: no deprecation warning, no removal notice, and no doc contract
  either. On a rolling host that is standing deprecation pressure and the **strongest
  removal candidate in the cmdline set**. The trade is descriptor-table-base leak plus
  support posture against `umip_printk` stutter. Quantify the taint consequence (does any
  subsystem refuse to load?). The string form is deliberate — CPUID bit numbers shift
  between kernels, the name does not; do not propose the numeric `clearcpuid=514`.
- **T3-3 · `fsck.mode=force` + `fsck.repair=yes`.** Largely **inert on a Btrfs root**.
  **Gated on T0-3** — establish the actual root filesystem before costing this. The fstab
  rewrite path is ext4-only (`_vre_fstab` L3096 enumerates every ext4 entry), so an ext4
  data volume and a Btrfs root can coexist here and a live verify log showing ext4 checks
  passing does **not** prove the root is ext4. Neither `fsck.mode` nor `fsck.repair` is
  documented in mainline kernel-parameters.txt — they are systemd-side, parsed by
  `systemd-fsck`, so cite systemd documentation, not the kernel. Quantify boot cost against
  `commit=10` durability; state the filesystem assumption explicitly in any finding.
- **T3-4 · The ASPM pair — `pcie_aspm.policy=performance` + `mt7925e.disable_aspm=1`.**
  **Settled and load-bearing.** Three facts, all re-verified 2026-07-27:
  (a) `drivers/pci/pcie/Kconfig` `PCIEASPM_PERFORMANCE` disables ASPM L0s and L1 even where
  the BIOS enabled them; (b) linux-cachyos ships `CONFIG_PCIEASPM_DEFAULT=y` with
  `PCIEASPM_PERFORMANCE` **explicitly unset**, so the built-in policy is `default` (BIOS)
  and the cmdline token is what actually switches it — unlike T2-4, this token does real
  work; (c) `mt7925e.disable_aspm` is a live writable module param (perm 0644) in mainline
  mt76 that calls `mt76_pci_disable_aspm(pdev)` at probe and suppresses `aspm_supported`.
  The pair is **complementary, not redundant** — global policy governs link state, the
  module option disables at the endpoint. **Do not describe the module option as omitted**
  (true only between 7.102.x and 7.129.x) and **do not simplify the pair away** until T0-2
  confirms per-link state on this board. `pcie_aspm=off` is documented as "don't touch ASPM
  configuration at all" and does NOT disable it — never propose it as an equivalent.
  `pcie_port_pm=off` stays KEEP-omitted unless T0-2 shows port PM active in a way that
  matters. Related mainline detail worth knowing: mt76 now force-disables ASPM on MT7927
  hardware regardless of the param; MT7925 is not covered by that quirk.

### T4 — Boot chain, firewall handoff, and detector severity. Reboot + recovery exposure.

- **T4-1 · Fallback-entry exposure.** `LINUX_FALLBACK_OPTIONS="quiet"` strips all 15
  params, so the fallback boots with IOMMU **on**, IPv6 **enabled** under an IPv4-only
  ruleset, and ASPM at firmware default with the MT7925 endpoint option absent — while the
  modprobe amdxdna blacklist *remains* active, because it is a file rather than a cmdline
  token. That last asymmetry is the sharp edge: the fallback gets IOMMU back but keeps the
  NPU blacklisted, which is the one combination the validator refuses to *deploy*.
  Confirm the window is accepted or flag it. Note `_vsb_entry_options` deliberately skips
  `*-fallback.conf` (upstream sdboot-manage filters the same way), so verify will never
  surface this.
- **T4-2 · mkinitcpio `COMPRESSION_OPTIONS=(-1)`.** Now a single flag after the 7.140.0
  `-T0` cut. Quantify boot decompress against default `-3` (sub-100 ms class on NVMe) and
  image size against the ESP budget (`BOOT_SPACE_CRIT/WARN` L613, 200/500 MiB) with multiple
  kernels plus fallback. TUNE to `-3` only if size threatens the budget. Any change here
  moves the oracle for `MKINITCPIO_COMPRESSION_OPTIONS` and the 5,089 B total.
- **T4-3 · `timeout 0` + `DEFAULT_ENTRY manual` + `REMOVE_EXISTING=yes`** wipes foreign
  BLS entries (EFI-resident loaders untouched). Recovery path is live-USB → chroot. With
  `timeout 0` and no saved EFI variable, a fresh ESP falls back to sd-boot's own sort order
  until the menu is used once. UKI is out of scope. **New adjacency:** `_vsb_sdboot_dropins`
  now warns on any `*.conf` under `/usr/lib/sdboot-manage.conf.d` or
  `/etc/sdboot-manage.conf.d` — a packaged drop-in from a future CachyOS update would
  silently outrank the managed `LINUX_OPTIONS`, and the WARN is the only signal.
- **T4-4 · ufw masked, not removed — confirm the nftables-first gate closes the window.**
  `_csm_enable_nftables_first` (L3901) is gated on `contains ufw.service $MASK` and confirms
  a live default-deny ruleset before anything touches ufw; `_csm_prepare_ufw_masking`
  (L3915) returns non-zero on an unconfirmed ruleset and `_configure_services_mask` (L3939)
  then withholds `ufw.service` from the safe-mask set for that run. Rationale: `mask --now`
  stops ufw and `ufw-init stop` flushes its rules, so masking before default-deny is live
  would open an unfirewalled window. Validate on a host where ufw is installed *and*
  active; confirm a withheld mask leaves ufw's own rules intact rather than half-flushed;
  confirm `nftables.service` being a oneshot (unit state reads inactive after a clean load)
  does not defeat the liveness check — the script judges by live policy-drop, which is
  correct but should be validated against current nftables packaging. Log keys:
  `UFW_MASK_DEFERRED`, `UFW_RULE_FLUSH_OK|FAIL|SKIP`, `SECURITY_POSTURE`.
- **T4-5 · Orphan-detector severity — a design question, not a gap.** Three detectors ship
  and none sets DRIFT: `_vss_modprobe` (L2315) WARNs on unmanaged `60-ry-*.conf` drop-ins,
  `_vss_orphan_masks` (L2459) INFOs on masked units absent from `$MASK`, and
  `_vsb_sdboot_dropins` (L2151) WARNs on sdboot-manage drop-ins. `--check` records
  `MODPROBE_STALE_DROPIN` / `MASK_ORPHAN` / `SDBOOT_DROPIN_PRESENT` to JSONL only. The
  reasoning on file: a re-run cannot clear any of them, so exit 10 would go permanently
  non-zero and train the operator to ignore exit 10 entirely. Mask ownership is genuinely
  **unattributable** — there is no `60-ry-`-style namespace for systemd units — which is why
  it is `_info`, not `_warn`. **Evaluate the trade.** The counter-argument worth testing: an
  INFO in a long `--verify` run is easy to miss and JSONL is only read deliberately. Any
  recommendation to promote one to DRIFT must address the desensitization argument
  explicitly, not around it, and must say which of the three it applies to — they are not
  equally attributable.
- **T4-6 · `_check_record_orphans` (L2632) sits after the sudo/systemctl preflight bail.**
  A stale drop-in is detectable with `find` alone — no privilege — yet the record is
  withheld when `sudo -n` is not cached. Consistent with `--check` being sudo-gated by
  contract, but it is a privilege-free signal withheld for a privilege reason. Assess
  whether the sweep should be hoisted above the bail. Note its *placement inside the mode*
  is already correct and deliberate: it runs before the phase loop, because at the tail of
  `_check_phase_*` it was skipped whenever preflight bailed. Do not re-propose that move.

### T5 — Closed and protected. Do not recommend changing these.

Flag a direct upstream contradiction as a **note**; never as a FIX.

- **Plaintext DNS on both host and router.** AdGuard's ad-block tier
  (94.140.14.14 / 94.140.15.15), `DNSOverTLS=no`, `DNSSEC=no`, and the router left on the
  plaintext picker. Rationale on file: filtering is identical either way, DoT buys only ISP
  query-name privacy, and router-side DoT in Strict mode fails closed — one TLS endpoint
  becomes a single point of failure for every LAN device. Uninterrupted connectivity for
  all devices is the stated priority. Quantify the exposure in §5; do not re-recommend DoT
  or DNSSEC. `DNSSEC=no` additionally matches systemd's own default.
- **`GPU_DPM_LEVEL=high`, not `profile_peak`.** `high` forces the highest power state with
  clock and power gating still active. `profile_peak` adds mclk/pcie forcing and disables
  gating, but kernel documentation scopes `profile_*` to measurement work, ArchWiki and
  amdgpu-clocks document `auto|low|high|manual` as the primary set, ROCm warns STABLE_PEAK
  is ASIC-specific and unverified on gfx1151, and Phoronix found forced `high` vs `auto`
  differs only in select cases. Evaluate `high` vs `auto` on frametime evidence only. The
  `profile_peak` variant measures udev 647 B / total 5,101 B at the pre-7.140.0 baseline —
  recorded so it is never re-measured.
- **The udev EPP write is a redundant no-op and that is fine.** Under
  `CPUPOWER_GOVERNOR=performance` the driver returns `-EBUSY` for any `epp > 0` and only
  offers "performance" in `energy_performance_available_preferences`; writing "performance"
  maps to epp 0 and IS accepted. Do not file the redundancy as a defect — it is a
  hotplug-safe assertion of a state the governor already imposes, and it is the only
  mechanism that survives a CPU hotplug event.
- **No cuts from the script.** `WINEDEBUG=-all`, BlueZ `AutoEnable`, NM `wifi.backend`,
  `cachyos-gaming-meta`, `MODULES=(amdgpu)`, `lib32-mesa` all stay.
- **The PKGS_ADD orphan is not to be "fixed" with a state file.** A package dropped from
  `PKGS_ADD` stays installed forever and nothing can see it. Detection needs a persisted
  manifest of what earlier versions installed, which would add its own drift surface. It is
  documented rather than built. A recommendation to add such a manifest must argue against
  that reasoning explicitly.
- **No version gates, by design.** `KERNEL_MIN` + `_ir_validate_kernel_floor` (removed
  7.105.x) and the Mesa soft `vercmp` warning are both gone; no advisory in-script kernel
  comment survives. Deploy / `--check` / `--verify` run on any kernel and any Mesa. Do not
  treat "no floor" as an oversight; treat it as a posture to evaluate.
- **Removed and still absent — each is a validation question, never a candidate:**
  `nowatchdog`, `tsc=reliable`, `8250.nr_uarts=0` (cmdline); `AMD_VULKAN_ICD`,
  `DXVK_LOG_PATH`, `VKD3D_CONFIG` (env); `ddcutil`, `git-delta` (packages);
  `modemmanager.service` (mask); `RY_REMOTE_PLAY_PORTS` and its port sets (7.137.0);
  the preemption advisory (7.139.0 r2); the redundant `-T0` (7.140.0); `pcie_aspm=off`;
  `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK`; `RY_NO_NTP_REMEDIATION`; `clearcpuid=514` numeric
  form; `archlinux-contrib`; the standalone `60-ry-blacklist-amdxdna.conf` and
  `60-ry-mt7925e.conf` drop-ins (both filenames are now actively swept for).
  **Standing precedent:** `mt7925e.disable_aspm=1` was removed at 7.102.x and re-added at
  7.129.0 — removals are not permanent, but a re-add needs the same evidence bar as a FIX.
- **Do not re-add** ICMPv6/NDP rules without restoring IPv6; do not flag inbound-ping
  accept (`_vss_nft` hard-fails on its absence, `_vrsv_nft_assert_ping` warns live); do not
  propose sleep-hook re-assert workarounds (all five sleep targets are masked, so there is
  no resume path and the udev `ACTION=="add"` rule is the only event that matters); do not
  flag a low MangoHud `vram` reading on UMA (it reports the BIOS carveout only — `ram`
  carries the shared pool); do not propose annotating more sysctl keys (the comment is
  selective by design, 3 of 11, at 64 chars against a 66-char cap).

---

## 3. Settled — do not re-research

Confirmed from primary sources. Re-verify only if a citation is challenged. The **VERIFIED**
column is the date the claim was last checked against the live source.

```
║ CLAIM                          ║ VERDICT / SOURCE                    ║ VERIFIED ║
║────────────────────────────────║─────────────────────────────────────║──────────║
║ Does the performance governor  ║ YES. amd-pstate.c mainline:         ║ 26-07-27 ║
║ reject a non-max EPP write?    ║ epp > 0 && policy ==                ║          ║
║                                ║ CPUFREQ_POLICY_PERFORMANCE returns  ║          ║
║                                ║ -EBUSY. Writing "performance" maps  ║          ║
║                                ║ to epp 0 and IS accepted, so the    ║          ║
║                                ║ udev rule lands as a redundant      ║          ║
║                                ║ no-op. Rejection is pr_debug —      ║          ║
║                                ║ never in default dmesg; the         ║          ║
║                                ║ observable is the udev write error  ║          ║
║ Available-preferences readout  ║ Under the performance policy the    ║ 26-07-27 ║
║ under performance policy       ║ sysfs file emits ONLY "performance" ║          ║
║ dynamic_epp availability       ║ CLOSED — WAS "master-only,          ║ 26-07-27 ║
║ (was UNCHECKED for CachyOS)    ║ backport unchecked". dynamic_epp    ║          ║
║                                ║ ships since 7.1 and linux-cachyos   ║          ║
║                                ║ builds 7.1.5, so it IS present on   ║          ║
║                                ║ this host. Kernel default is        ║          ║
║                                ║ FALSE (static bool dynamic_epp),    ║          ║
║                                ║ matching the profile's "disabled"   ║          ║
║                                ║ assertion. When enabled it blocks   ║          ║
║                                ║ ALL manual EPP writes with -EBUSY   ║          ║
║ amd_dynamic_epp= boot param    ║ NEW, DOCUMENTED in mainline         ║ 26-07-27 ║
║                                ║ kernel-parameters.txt: disable |    ║          ║
║                                ║ enable. Profile does not set it and ║          ║
║                                ║ should not. If a future default     ║          ║
║                                ║ flips, the udev EPP write breaks    ║          ║
║ amd_pstate default mode on     ║ Kconfig.x86 X86_AMD_PSTATE_DEFAULT_ ║ 26-07-27 ║
║ CachyOS                        ║ MODE: 3 = Active (EPP). CachyOS     ║          ║
║                                ║ ships =3, so amd_pstate=active      ║          ║
║                                ║ RESTATES the compiled default       ║          ║
║ pcie_aspm.policy=performance   ║ DISABLES. drivers/pci/pcie/Kconfig  ║ 26-07-27 ║
║ semantics                      ║ PCIEASPM_PERFORMANCE: "Disable PCI  ║          ║
║                                ║ Express ASPM L0s and L1, even if    ║          ║
║                                ║ the BIOS enabled them." Cite the    ║          ║
║                                ║ Kconfig — the "since 4.2" figure on ║          ║
║                                ║ kernelconfig.io is a database-floor ║          ║
║                                ║ artifact                            ║          ║
║ Is the ASPM cmdline token      ║ YES. CachyOS sets PCIEASPM_DEFAULT= ║ 26-07-27 ║
║ load-bearing on CachyOS?       ║ y and PCIEASPM_PERFORMANCE is NOT   ║          ║
║                                ║ set — the token does real work      ║          ║
║ pcie_aspm=off semantics        ║ "Don't touch ASPM configuration at  ║ 26-07-27 ║
║                                ║ all. Leave any configuration done   ║          ║
║                                ║ by firmware unchanged." Does NOT    ║          ║
║                                ║ disable ASPM                        ║          ║
║ clearcpuid documentation state ║ ZERO occurrences in mainline        ║ 26-07-27 ║
║                                ║ kernel-parameters.txt while the     ║          ║
║                                ║ code still parses it                ║          ║
║ mt7925e.disable_aspm exists    ║ YES. mt76/mt7925/pci.c module_param ║ 26-07-27 ║
║                                ║ _named, perm 0644, calls            ║          ║
║                                ║ mt76_pci_disable_aspm() at probe    ║          ║
║                                ║ and suppresses aspm_supported.      ║          ║
║                                ║ MT7927 is force-disabled by a       ║          ║
║                                ║ separate quirk; MT7925 is not       ║          ║
║ RTL8127 in r8169               ║ Present in mainline r8169_main.c;   ║ 26-07-27 ║
║                                ║ first landed v6.16, absent at v6.15 ║          ║
║ PROTON_FSR4_UPGRADE currency   ║ CORRECT AND CURRENT — NOT obsolete. ║ 26-07-27 ║
║                                ║ Upstream FSR4.md: "Just set         ║          ║
║                                ║ PROTON_FSR4_UPGRADE=1, the FSR      ║          ║
║                                ║ 4.0.0 DLL will be automatically     ║          ║
║                                ║ downloaded." Also aliased as        ║          ║
║                                ║ PROTON_ADD_CONFIG=fsr4. RDNA3-class ║          ║
║                                ║ hardware additionally needs         ║          ║
║                                ║ DXIL_SPIRV_CONFIG=wmma_rdna3_       ║          ║
║                                ║ workaround; prefer FSR 4.0.0 over   ║          ║
║                                ║ 4.0.1 on RDNA3. FSR4_WATERMARK=1    ║          ║
║                                ║ verifies it is live                 ║          ║
║ PROTON_ENABLE_WAYLAND scope    ║ PER-GAME by upstream framing.       ║ 26-07-27 ║
║                                ║ Aliased PROTON_USE_WAYLAND=1 /      ║          ║
║                                ║ PROTON_ADD_CONFIG=wayland; Steam    ║          ║
║                                ║ needs -steamos3 for Steam Input     ║          ║
║                                ║ under winewayland                   ║          ║
║ ntsync currency                ║ CONFIG_NTSYNC=m in linux-cachyos    ║ 26-07-27 ║
║                                ║ (module, not builtin). ntsync is    ║          ║
║                                ║ the default; PROTON_NO_NTSYNC=1 is  ║          ║
║                                ║ the opt-out. Profile neither sets   ║          ║
║                                ║ nor checks it, correctly            ║          ║
║ RADV descriptor heap           ║ DEFAULT-ON. Mesa 26.3.0-devel docs: ║ 26-07-27 ║
║                                ║ RADV_DEBUG=noheap "disable VK_EXT_  ║          ║
║                                ║ descriptor_heap"; heap is gone from ║          ║
║                                ║ the RADV_EXPERIMENTAL list          ║          ║
║ MESA_SHADER_CACHE_MAX_SIZE     ║ Documented; number + K/M/G suffix.  ║ 26-07-27 ║
║ accepts 16G                    ║ Default 1 GB if unset               ║          ║
║ MangoHud cpu_custom_temp_      ║ CURRENT. Form is <hwmon>,<input>,   ║ 26-07-27 ║
║ sensor                         ║ e.g. cpuss0_2,temp3_input           ║          ║
║ MangoHud #1794                 ║ STILL OPEN — cpu_power reads 0 when ║ 26-07-27 ║
║                                ║ cpu_temp is active on Zen 5         ║          ║
║ netdev_budget guidance         ║ Kernel defaults 300 / 2000. The     ║ 26-07-27 ║
║ attribution                    ║ 600/4000 pair is RED HAT's (RHEL    ║          ║
║                                ║ 8/9/10), NOT ESnet's. ESnet's 100 G ║          ║
║                                ║ page recommends DEFAULTS and warns  ║          ║
║                                ║ changes can cut throughput. Both    ║          ║
║                                ║ gate on softnet_stat column 3       ║          ║
║ nftables invalid-before-       ║ NO UPSTREAM RULE either way.        ║ 26-07-27 ║
║ loopback ordering              ║ Gentoo's reference ruleset uses the ║          ║
║                                ║ profile's order; nftables.org and   ║          ║
║                                ║ ArchWiki put loopback first. KEEP   ║          ║
║ NM vs resolved DNS precedence  ║ systemd #33973 closed-completed —   ║ 26-07-27 ║
║                                ║ per-link DHCP DNS outranks global   ║          ║
║                                ║ DNS=. Domains=~. is NOT the fix     ║          ║
║                                ║ (#33579, re-confirmed OPEN:         ║          ║
║                                ║ dual-query leak). NM                ║          ║
║                                ║ [global-dns-domain-*] is the        ║          ║
║                                ║ correct mechanism and ships         ║          ║
║ MES firmware timeline          ║ 2025-11-19 update -> 2025-12-01     ║ 26-07-27 ║
║ (gfx1151 hang)                 ║ REVERT -> 2026-02-25 re-land (=0x86)║          ║
║                                ║ -> 2026-05-07 further update, the   ║          ║
║                                ║ current head. First tag shipping    ║          ║
║                                ║ 0x86 = 20260309. 0x7f is the        ║          ║
║                                ║ lr_compute_wa dmesg gate. Given the ║          ║
║                                ║ revert precedent, advise "newest    ║          ║
║                                ║ tag", never a minimum               ║          ║
║ POWERDEVIL_NO_DDCUTIL=1        ║ EXACT VARIABLE PowerDevil reads.    ║ 26-07-25 ║
║                                ║ Not a silent no-op                  ║          ║
║ nmi_watchdog vendor            ║ CachyOS 70-cachyos-settings.conf    ║ 26-07-25 ║
║ interaction                    ║ already sets nmi_watchdog=0 and     ║          ║
║                                ║ swappiness 100; a zram-generator    ║          ║
║                                ║ config ships, so swappiness=150 is  ║          ║
║                                ║ coherent. Priority 95 loads after   ║          ║
║                                ║ 70 — correct ORDERING, not merely   ║          ║
║                                ║ presence. KEEP the redundancy:      ║          ║
║                                ║ dropping it would make the value    ║          ║
║                                ║ depend on a package ry does not own ║          ║
║ ROCm #6165                     ║ CLOSED (an earlier pass said open)  ║ 26-07-25 ║
║ fish set -l block scoping      ║ Documented and stable — "erased     ║ 26-07-25 ║
║                                ║ when the block ends"; dates to fish ║          ║
║                                ║ 3.3, well below the 3.6 floor       ║          ║
```

**Known-stale sources — do not cite:** `wireless.docs.kernel.org` for `pcie_aspm` semantics
(carries pre-6.9 text; Helgaas's v6.9 doc fix, commit 2e0239d47d75, is the correction);
kernelconfig.io for Kconfig introduction versions; any source attributing the 600/4000
netdev pairing to ESnet.

---

## 4. Corrections this rebase makes to the 7.139.0 edition

1. **The ESnet attribution was wrong.** `netdev_budget 600 / netdev_budget_usecs 4000` is
   Red Hat's published recipe, not ESnet's. ESnet's own 100 G page recommends leaving these
   at defaults. T2-1 is rewritten around the correct authorities and now carries a
   measurement gate (T0-5).
2. **`PROTON_FSR4_UPGRADE=1` is NOT near-obsolete.** The previous research line held that
   the DLL is auto-copied and the variable "survives only to version-pin". Upstream FSR4.md
   says the opposite: setting the variable is what *triggers* the automatic download. The
   real gap for this hardware is the missing `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround`
   pairing — now T1-3. **Withdraw the obsolescence finding; do not re-raise it.**
3. **The dynamic_epp CachyOS backport question is CLOSED, not UNCHECKED.** linux-cachyos
   builds 7.1.5 and the feature ships since 7.1, so it is present on this host. The
   previous edition's "master-only at the time of check" verdict is superseded.
4. **`amd_pstate=active` is not "the root of the whole CPU story" on this kernel.** CachyOS
   compiles Active/EPP as the default mode, so the token restates it. Reframed as T2-4;
   contrast drawn against T3-4, where the equivalent option is unset and the token matters.
5. **The nftables rule-order question is resolved to KEEP.** The previous edition left it
   open pending "nftables documentation, not folklore". The documentation is split and
   neither side prescribes; the profile's order matches the most widely redistributed
   reference ruleset.
6. **Line anchors and counts re-derived.** Perf globals remain at L586/L588/L590/L591 and
   MASK at L610 — unchanged since 7.139.0 — but `_ir_validate_post_hooks` moved 4575→4591,
   `_check_record_orphans` 2616→2632, `_ry_stale_ry_dropins` 2586→2602,
   `_vss_orphan_masks` 2443→2459, and the harness cut 4845→4861. All six README perf sites
   moved (100/102/205/209/210/216 → 84/86/189/193/194/200). §7b carries the current list.
7. **`_vsb_entry_options` is a sub of `_vsb_entries`, not of `_verify_static_boot`.** The
   previous edition's ownership table implied the latter. Its `sub:` marker names
   `_vsb_entries`.

---

## 5. Security posture — quantify only, ordered by exposure

No auto-FIX in this section.

1. **UMIP off** (`clearcpuid=umip`) — descriptor-table base leak, kernel tainted, and now
   undocumented upstream while still parsed. Headline open reduction (T3-2).
2. **AMD-Vi fully disabled** (`amd_iommu=off`) — no DMA isolation or remapping; USB4/TB,
   NVMe and NIC DMA unmediated. Named casualty: the XDNA 2 NPU, blacklisted. Opting back in
   is one validator-enforced pair (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`),
   restoring isolation and the NPU together. The coupling asymmetry is intentional:
   `amd_iommu=on` with blacklist true is valid; blacklist false without the IOMMU refuses
   to deploy. **The fallback entry inverts this pairing** (IOMMU on, blacklist still
   active) — see T4-1.
3. **Plaintext DNS with no validation** — observable and spoofable on the wire, no
   authentication of answers. State the exposure and stop; the decision is closed (T5).
4. **IPv6 disabled + inbound IPv4 ping accepted** — net LAN delta is `+ping −mDNS`. Avahi
   is masked (unit *and* socket) and resolved has `MulticastDNS=no`, so multicast discovery
   is fully closed. Ping-accept is an asserted regression guard, not a defect. Latent
   coupling: `table inet` + `policy drop` covers IPv6, but the ruleset carries **no
   `icmpv6` accept rule**, so if `ipv6.disable=1` were ever removed, NDP / RA / MLD would be
   silently dropped and IPv6 would fail with no diagnostic. Nothing enforces the coupling.
   Inert while IPv6 is off; a preflight assert or an inert `icmpv6` accept line would close
   it. LOW.
5. **`split_lock_detect=off`** — a misbehaving application can degrade the whole system.
6. **ufw masked rather than removed** — the package stays installed and could be unmasked
   and started, at which point two firewall managers contend for the same netfilter tables.
   Quantify that against the benefit (reversibility, no package churn); the nftables-first
   gate (T4-4) is the only thing standing between a mis-sequenced run and an unfirewalled
   window.
7. **sdboot-manage drop-in override path** — a packaged drop-in can replace `LINUX_OPTIONS`
   entirely, and until 7.140.0 nothing looked. Now WARN-only (T4-3 / T4-5).
8. **No sleep path** — all five sleep/suspend/hibernate targets masked. An always-on box
   never gets the "locked on resume" checkpoint. Deliberate for a headless-adjacent mini-PC.
9. **Historical, not live: the `.ry.orig` dead-code window.** From 7.109.0 through 7.135.0
   the first-adoption preserve never executed, so 13 of 17 destinations were overwritten
   with **no backup of any kind** — no `.ry.orig`, no `.ry.bak`. Security-relevant members
   of that set: `/etc/nftables.conf` and `/etc/NetworkManager/conf.d/99-cachyos-nm.conf`.
   Fixed at 7.135.1 and void for a host first deployed at ≥7.135.1. **There is no forensic
   trace** — the JSONL never emitted `PREEXISTING_PRESERVED`, so absence of the key does not
   distinguish "nothing to preserve" from "preserve skipped". T0-7 confirms the window on a
   given host. **The generalizable lesson: presence of a guard in source is not evidence the
   guard runs.** Any claim of the form "X is protected because the code does Y" needs a
   behavioural probe, not a read.

**Default-deny-inbound ships and is net positive.** Residual notes: `flush ruleset` blast
radius against docker/libvirt/podman; no ICMP or new-connection rate limit (trusted-LAN
assumption — state it).

---

## 6. Verify block

Grouped to match the tiers. T0 items first. All commands are reads unless marked.

**T0-1 — idle-floor measurement (the headline; idle desktop, 60 s, before and after any
perf change):**

```fish
sudo turbostat --quiet --show PkgWatt,Busy%,Bzy_MHz,PkgTmp --interval 5 --num_iterations 12
cat /sys/class/drm/card*/device/hwmon/hwmon*/power1_average   # GPU package draw at idle
rg . /sys/devices/system/cpu/cpu*/cpuidle/state*/name         # C-state depth under max_cstate=1
```

**T0-2 … T0-7 — the remaining observations:**

```fish
sudo lspci -vv | rg 'LnkCtl:.*ASPM'                     # T0-2 per-link ASPM state
cat /sys/module/mt7925e/parameters/disable_aspm         # Y/1 — endpoint option live
findmnt -no FSTYPE,OPTIONS /                            # T0-3 root FS before costing fsck.mode
findmnt -no OPTIONS -t ext4                             # noatime,lazytime,commit=10
cat /sys/class/hwmon/hwmon*/name | rg -c '^k10temp$'    # T0-4 sensor presence
for d in /sys/class/hwmon/hwmon*; echo $d (cat $d/name); end   # hwmon index for cpu_custom_temp_sensor
sensors 2>/dev/null | rg -i -A1 '^k10temp'              # does Tctl populate?
stat -c '%a' /sys/class/powercap/*/energy_uj            # world-readable? gates cpu_power
awk '{for(i=1;i<=NF;i++) printf strtonum("0x" $i) (i==NF?"\n":" ")}' /proc/net/softnet_stat \
  | awk '{print $3}'                                    # T0-5 squeezed; flat under load = T2-1 moot
cat /sys/devices/system/cpu/amd_pstate/dynamic_epp      # T0-6 expect: disabled
rg -c 'amd_dynamic_epp' /proc/cmdline                   # 0 — profile must not set it
ls -l /etc/*.ry.orig /etc/**/*.ry.orig ~/.config/**/*.ry.orig 2>/dev/null   # T0-7
ls -l /etc/kernel/cmdline.ry.bak /etc/sdboot-manage.conf.ry.bak \
      /boot/loader/loader.conf.ry.bak /etc/mkinitcpio.conf.ry.bak 2>/dev/null
rg -c 'PREEXISTING_PRESERVED|PREEXISTING_PRESERVE_FAIL' ~/ry-install/logs/**/*.jsonl   # 0 = window confirmed
rg -c 'MODPROBE_STALE_DROPIN|MASK_ORPHAN|SDBOOT_DROPIN_PRESENT' ~/ry-install/logs/**/*.jsonl
pacman -Q linux-firmware mesa linux-cachyos             # advisory only, no gate exists
```

**CPU / GPU state (T2/T3 baseline):**

```fish
cat /proc/cmdline
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver                  # amd-pstate-epp
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor                # performance
cat /sys/devices/system/cpu/cpu0/cpufreq/energy_performance_preference   # performance
cat /sys/devices/system/cpu/cpu0/cpufreq/energy_performance_available_preferences # performance only
cat /sys/devices/system/cpu/amd_pstate/status                            # active
cat /sys/devices/system/cpu/amd_pstate/prefcore                          # enabled
cat /sys/devices/system/cpu/cpufreq/boost                                # 1
cat /sys/class/drm/card*/device/power_dpm_force_performance_level        # high
cat /sys/block/nvme0n1/queue/scheduler                                   # [none] (adjust node)
```

**Cmdline tokens and their removals:**

```fish
rg -o 'amd_iommu=\S+|ipv6\.disable=\S+|clearcpuid=\S+|pcie_aspm\S*|mt7925e\S*' /proc/cmdline
rg -o 'processor.max_cstate=\S+|fsck\S+|amd_pstate=\S+' /proc/cmdline
rg -c 'nowatchdog|tsc=reliable|8250' /proc/cmdline    # 0 — the three removals still hold
rg -c 'preempt=' /proc/cmdline                        # 0 — never pinned; advisory removed 7.139.0
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l   # 0 — INFORMATIONAL
sudo dmesg | rg -i 'AMD-Vi|DMAR'                      # expect NO "AMD-Vi: Enabled"
```

**Boot chain (includes the 7.140.0 drop-in surface):**

```fish
ls /usr/lib/sdboot-manage.conf.d /etc/sdboot-manage.conf.d 2>/dev/null   # expect: absent or empty
rg -c 'LINUX_OPTIONS' /etc/sdboot-manage.conf.d/*.conf 2>/dev/null       # any hit outranks the managed conf
rg 'options' /boot/loader/entries/*.conf | rg -v 'fallback'              # 15 tokens per entry
rg 'options' /boot/loader/entries/*fallback*.conf                        # "quiet" only — T4-1 window
bootctl status | rg -i 'default|timeout|entry'
```

**Memory, network, DNS:**

```fish
sysctl kernel.nmi_watchdog net.ipv4.tcp_congestion_control net.core.default_qdisc \
       net.core.netdev_budget net.core.netdev_budget_usecs vm.max_map_count \
       vm.compaction_proactiveness vm.swappiness vm.watermark_boost_factor
systemd-analyze cat-config sysctl.d | rg -n 'nmi_watchdog'   # 95-ry wins over vendor 70
resolvectl status | rg -i 'DNS Servers|DNSSEC|DNSOverTLS'
resolvectl query example.com                                  # AdGuard pair, not a per-link server
rg -n 'global-dns-domain' /etc/NetworkManager/conf.d/99-cachyos-nm.conf
swapon --show; zramctl                                        # zram advisory, not managed
iw reg get | rg -i country                                    # US
```

**Firewall and units:**

```fish
sudo nft list chain inet filter input   # policy drop; invalid-drop FIRST, then est/rel, then lo
sudo nft -c -f /etc/nftables.conf
systemctl is-enabled ufw.service        # masked — NOT "not installed"
systemctl is-enabled sleep.target suspend.target hibernate.target \
                     hybrid-sleep.target suspend-then-hibernate.target   # masked x5
systemctl list-unit-files --state=masked,masked-runtime --no-legend --plain  # compare vs MASK 11
systemctl is-enabled avahi-daemon.service avahi-daemon.socket bluetooth.service
systemctl is-enabled systemd-resolved.service          # enabled or static
systemctl --user is-failed plasma-powerdevil.service   # NOT "failed"
systemctl --user --failed --plain --no-legend          # empty
stat -c '%a %U:%G' /etc/NetworkManager/system-connections/*   # 0600 root:root
```

**Userspace / HUD:**

```fish
lsmod | rg -c '^amdxdna'                                  # 0 — loaded = verify FAIL
ls /etc/modprobe.d/60-ry-*.conf                           # ONLY 60-ry-modules.conf
ls -l /dev/ntsync                                         # present (assert-only)
vulkaninfo | rg -i 'driverName|deviceName'                # RADV / Radeon 8060S, no ICD pin
systemctl --user show-environment | rg 'PROTON_FSR4_UPGRADE|MANGOHUD|POWERDEVIL_NO_DDCUTIL'
systemctl --user show-environment | rg -c 'VKD3D_CONFIG'  # 0 — nonzero = stale session
grep -c '^cpu_power' ~/.config/MangoHud/MangoHud.conf     # 1
grep -c '^cpu_temp' ~/.config/MangoHud/MangoHud.conf      # 0 — deliberate (#1794)
grep -c '^[a-z]' ~/.config/MangoHud/MangoHud.conf         # 19 active directives
```

**Per-game, for T1-3 only (not a system check):** launch one FSR4 title with
`FSR4_WATERMARK=1` and confirm the corner watermark renders; A/B with and without
`DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` for the RDNA3 glitch class.

**`rg -c` exits 1 with no output on zero matches.** Any zero-hit assertion above must be
read as "exit 1 or `0`", not as a failure. Sandbox rg 14.1.0 additionally carries a
false-negative bug — re-confirm zero-hit results with `grep -P` or `python3`.

---

## 7. Reference data

All values live-evaluated from the 7.141.0 script.

### 7a. Count oracle — 21 tripwires, asserted by `_ir_validate_counts` (L665)

```
║ KERNEL_PARAMS            15 ║ PKGS_ADD                 16 ║ _RY_BOOT_CRITICAL_DSTS  4 ║
║ MKINITCPIO_HOOKS         11 ║ PKGS_DEL                  9 ║ _RY_PHASE_NAMES         6 ║
║ MKINITCPIO_MODULES        1 ║ MASK                     11 ║ _RY_BACKUP_TARGETS      4 ║
║ MKINITCPIO_COMPRESSION_   1 ║ EXPECTED_VULKAN_PKGS      2 ║ _RY_TMPDIR_GLOBS        6 ║
║   OPTIONS  (was 2)          ║ EXPECTED_SERVICES         5 ║ SYSTEM_DESTINATIONS    15 ║
║ LOGIND_IGNORE_KEYS        8 ║ _RY_PKG_MANAGED_SERVICES  1 ║ USER_DESTINATIONS       2 ║
║ ENV_VARS                 10 ║ _RY_POST_HOOKS           17 ║ _RY_ARGPARSE_SPEC       6 ║
║ SYSCTL_VALUES            11 ║                             ║                           ║
```

`RESOLVED_DNS_SERVERS` (2) and `_RY_EPP_LEVELS` (5) are deliberately **not** in the oracle —
both are value-bearing, not count invariants. Managed files = 17 (15 system + 2 user),
recomputed at load; a mismatch refuses with exit 3. **Count these by live fish eval, never
by text parsing.**

### 7b. Perf scalars and their maxima

```
║ GLOBAL                  ║ L   ║ VALUE          ║ CEILING                          ║
║─────────────────────────║─────║────────────────║──────────────────────────────────║
║ CPUPOWER_GOVERNOR       ║ 586 ║ performance    ║ AT MAX (regex ^[a-z][a-z0-9_-]*$)║
║ GPU_DPM_LEVEL           ║ 588 ║ high           ║ CLOSED at high (T5), not max     ║
║ EPP_PREFERENCE          ║ 590 ║ performance    ║ AT MAX (_RY_EPP_LEVELS, 5)       ║
║ EXPECTED_SCALING_DRIVER ║ 591 ║ amd-pstate-epp ║ NO MAX — verify-only, tunes      ║
║                         ║     ║                ║ NOTHING; follows a cmdline       ║
║                         ║     ║                ║ amd_pstate= change, never leads  ║
║ BLACKLIST_AMDXDNA       ║ 592 ║ true           ║ boolean; false REQUIRES          ║
║                         ║     ║                ║ amd_iommu=on iommu=pt            ║
```

`_RY_EPP_LEVELS` (L590, same line): `default performance balance_performance
balance_power power`. Changing `EXPECTED_SCALING_DRIVER` makes the verifier assert a driver
the kernel never loaded, producing a false `_chk_eq` failure on every `--verify`. Domain
validation lives in `_ir_validate_keys` (L693): DPM at L705, EPP at L706, governor regex at
L707. Negative tests return rc **3** (`EXIT_PREFLIGHT`), and `_err_loud` **exits** rather
than returns — each negative case needs its own subprocess.

**A full perf-value change touches 13 sites. Enumerate all of them in any TUNE.** Every
line number below was re-derived at 7.141.0; the six README anchors all moved this rebase.

```
║ #  ║ FILE      ║ LINE ║ CARRIES                                              ║
║────║───────────║──────║──────────────────────────────────────────────────────║
║ 1  ║ script    ║  586 ║ set -g CPUPOWER_GOVERNOR performance                 ║
║ 2  ║ script    ║  588 ║ set -g GPU_DPM_LEVEL high + trailing comment         ║
║ 3  ║ script    ║  590 ║ set -g EPP_PREFERENCE performance (+ _RY_EPP_LEVELS) ║
║ 4  ║ script    ║  874 ║ udev generator --description names "EPP performance" ║
║ 5  ║ script    ║  879 ║ "# AMD P-State EPP performance (maximum CPPC hint)"  ║
║ 6  ║ script    ║  881 ║ "# GPU performance level (gfx1151 clock-floor;       ║
║    ║           ║      ║ forced high)"                                        ║
║ 7  ║ README    ║   84 ║ managed-files row: governor (`performance`)          ║
║ 8  ║ README    ║   86 ║ managed-files row: NVMe none, P-State EPP, DPM `high`║
║ 9  ║ README    ║  189 ║ Service Keys row: CPUPOWER_GOVERNOR | performance    ║
║ 10 ║ README    ║  193 ║ Service Keys row: GPU_DPM_LEVEL | high               ║
║ 11 ║ README    ║  194 ║ Service Keys row: EPP_PREFERENCE | performance       ║
║ 12 ║ README    ║  200 ║ CPU/GPU prose (EPP-restates-governor + the accepted- ║
║    ║           ║      ║ value-list sentence, one paragraph)                  ║
║ 13 ║ CHANGELOG ║   7+ ║ new entry inserted after the preamble                ║
```

**Grep traps:** `powersave` has 4 hits but only L586 is in scope — the rest are
`NM_WIFI_POWERSAVE` plumbing. `balance_performance` has exactly 1 hit, the surviving
`_RY_EPP_LEVELS` member. Do not edit the phrase "clock-floor" — it is accurate under `high`
and consistent across the generator and verify paths. `grep -ci 'audit\|spec'` reports
false hits from `_RY_ARGPARSE_SPEC` and from "inspect"/"inspection"; real audit refs are
**0** in all shipped files.

**Measured deltas from the last perf change** (7.128.0, powersave/auto/balance_performance →
max): udev 657→639 (−18), cpupower 113→115 (+2), 17-file total 5,109→5,093 (−16); the split
was value substitution −6 and comment edits −10. A spec revision predicting −6 by omitting
the comment edits was wrong. **On a re-apply, use verbatim strings — never paraphrase edit
wording from memory; the acceptance test is reproducing the predicted SHAs, not a passing
functional battery.**

### 7c. Configured values

**KERNEL_PARAMS (15, sorted as emitted):** `amd_iommu=off` `amd_pstate=active`
`btusb.enable_autosuspend=n` `clearcpuid=umip` `fsck.mode=force` `fsck.repair=yes`
`ipv6.disable=1` `mt7925e.disable_aspm=1` `nvme_core.default_ps_max_latency_us=0`
`pcie_aspm.policy=performance` `processor.max_cstate=1` `quiet` `split_lock_detect=off`
`usbcore.autosuspend=-1` `zswap.enabled=0`.

Documentation status in mainline `kernel-parameters.txt`, checked 2026-07-27 — a token
being absent is not a defect, but it changes which source to cite:

```
║ TOKEN                              ║ IN kernel-parameters.txt ║ CITE INSTEAD       ║
║────────────────────────────────────║──────────────────────────║────────────────────║
║ amd_pstate=                        ║ YES                      ║ —                  ║
║ amd_iommu=                         ║ YES                      ║ —                  ║
║ processor.max_cstate=              ║ YES                      ║ —                  ║
║ split_lock_detect=                 ║ YES                      ║ —                  ║
║ usbcore.autosuspend=               ║ YES                      ║ —                  ║
║ pcie_aspm= (policy= is a modparam) ║ pcie_aspm= only          ║ pcie/Kconfig       ║
║ clearcpuid=                        ║ NO — code still parses   ║ arch/x86 source    ║
║ fsck.mode= / fsck.repair=          ║ NO — systemd-side        ║ systemd-fsck(8)    ║
║ ipv6.disable=                      ║ NO                       ║ networking docs    ║
║ zswap.enabled=                     ║ NO                       ║ admin-guide/mm     ║
║ nvme_core.default_ps_max_latency_us║ NO — module param        ║ drivers/nvme       ║
║ btusb.enable_autosuspend=          ║ NO — module param        ║ drivers/bluetooth  ║
║ mt7925e.disable_aspm=              ║ NO — module param        ║ mt76/mt7925/pci.c  ║
```

**ENV_VARS (10, `~/.config/environment.d/10-environment.conf`, 0600):** `DXVK_LOG_LEVEL=none`
`MANGOHUD=1` `MESA_SHADER_CACHE_MAX_SIZE=16G` `POWERDEVIL_NO_DDCUTIL=1`
`PROTON_ENABLE_WAYLAND=1` `PROTON_FSR4_UPGRADE=1` `PROTON_LOCAL_SHADER_CACHE=1`
`VKD3D_DEBUG=none` `VKD3D_SHADER_DEBUG=none` `WINEDEBUG=-all`. No drirc, no ttm/amdgpu
module params, no ICD pin. `_vre_envvars` (L3060) iterates the array dynamically, so the
verifier follows any ENV_VARS edit with no verifier change.

**SYSCTL_VALUES (11, `/etc/sysctl.d/95-ry-overrides.conf`, priority 95):**
`kernel.nmi_watchdog=0` `net.core.default_qdisc=fq` `net.core.netdev_budget=600`
`net.core.netdev_budget_usecs=5000` `net.ipv4.tcp_congestion_control=bbr`
`net.ipv4.tcp_notsent_lowat=16384` `net.ipv4.tcp_slow_start_after_idle=0`
`vm.compaction_proactiveness=0` `vm.max_map_count=2147483642` `vm.swappiness=150`
`vm.watermark_boost_factor=0`. Stored `k=v`, emitted `k = v` — any parity check must
normalise whitespace. The annotation comment is selective by design (3 of 11 keys) at 64
characters against a ≤66 cap; do not propose annotating more. **Known non-defect:** on a
non-initial network namespace the four `net.core.*` keys fail with ENOENT because
`net_core_table` is init_net only — that is a container signature, not a host fault.

**MASK (11, L610):** `ananicy-cpp.service` `power-profiles-daemon.service`
`NetworkManager-wait-online.service` `avahi-daemon.service` `avahi-daemon.socket`
`ufw.service` `sleep.target` `suspend.target` `hibernate.target` `hybrid-sleep.target`
`suspend-then-hibernate.target`.

**EXPECTED_SERVICES (5):** `fstrim.timer` `NetworkManager.service` `cpupower.service`
`nftables.service` `bluetooth.service`. **`_RY_PKG_MANAGED_SERVICES (1):**
`NetworkManager.service` — it is in both sets; strip it from EXPECTED_SERVICES first when
isolating the intersection gates.

**PKGS_ADD (16):** nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa,
mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, lm_sensors, rtkit,
realtime-privileges, nftables, pacman-contrib. **PKGS_DEL (9, `-Rns`, rdep-aware via
pactree):** plymouth, cachyos-plymouth-bootanimation, cachyos-plymouth-theme,
breeze-plymouth, plymouth-kcm, micro, cachyos-micro-settings, cachy-update, kdeconnect.
**AUR: none** — 13 resolve in Arch extra/multilib/core and 3 in `[cachyos]`, so the
pacman-only `-Syu --needed` path works with no AUR helper. Vulkan via chwd, verify-only:
vulkan-radeon + lib32-vulkan-radeon.

**LOGIND_IGNORE_KEYS (8):** `HandlePowerKey` `HandlePowerKeyLongPress` `HandleSuspendKey`
`HandleSuspendKeyLongPress` `HandleHibernateKey` `HandleHibernateKeyLongPress`
`HandleRebootKey` `HandleRebootKeyLongPress` — emitted as `<key>=ignore`. The 4 uncovered
`Handle*` keys are the 3 lid-switch ones plus `HandleSecureAttentionKey`, correctly out of
scope for a mini-PC.

**MKINITCPIO:** `MODULES=(amdgpu)` (early KMS); HOOKS (11) base systemd autodetect microcode
modconf kms keyboard sd-vconsole block filesystems fsck — **byte-identical to Arch
mkinitcpio 41's shipped default**; COMPRESSION zstd, `COMPRESSION_OPTIONS=(-1)`; explicit
`BINARIES=()` / `FILES=()`.

**Nineteen service keys** are documented in the profile README. Only two reach their
destination under their own name (`COUNTRY`; `LOGIND_IGNORE_KEYS` → 8 `Handle*Key=ignore`
lines). The other seventeen are **renamed on the way out** — `RESOLVED_DOT`→`DNSOverTLS`,
`BT_AUTO_ENABLE`→`AutoEnable`, `EPP_PREFERENCE`→udev
`ATTR{cpufreq/energy_performance_preference}`, `GPU_DPM_LEVEL`→`ATTR{device/
power_dpm_force_performance_level}`, `CPUPOWER_GOVERNOR`→`GOVERNOR=` — which is why
grepping a generated file for the script's variable name returns nothing.
`EXPECTED_SCALING_DRIVER` writes nothing at all.

### 7d. Generated-file byte anchors

```
║ FILE                                              ║ 7.139 ║ 7.141 ║
║───────────────────────────────────────────────────║───────║───────║
║ /boot/loader/loader.conf                          ║    89 ║    89 ║
║ /etc/kernel/cmdline                               ║   352 ║   352 ║
║ /etc/sdboot-manage.conf                           ║   544 ║   544 ║
║ /etc/mkinitcpio.conf                              ║   280 ║   276 ║
║ /etc/systemd/resolved.conf.d/99-cachyos-resolved  ║   154 ║   154 ║
║ /etc/systemd/logind.conf.d/99-cachyos-logind      ║   292 ║   292 ║
║ NetworkManager-dispatcher.service.d/logging.conf  ║   127 ║   127 ║
║ /etc/NetworkManager/conf.d/99-cachyos-nm.conf     ║   219 ║   219 ║
║ /etc/iw-regdomain                                 ║    88 ║    88 ║
║ /etc/bluetooth/main.conf                          ║   147 ║   147 ║
║ /etc/nftables.conf                                ║   729 ║   729 ║
║ /etc/default/cpupower-service.conf                ║   115 ║   115 ║
║ /etc/sysctl.d/95-ry-overrides.conf                ║   441 ║   441 ║
║ /etc/udev/rules.d/99-ry-perf.rules                ║   639 ║   639 ║
║ /etc/modprobe.d/60-ry-modules.conf                ║   183 ║   183 ║
║ ~/.config/environment.d/10-environment.conf       ║   311 ║   311 ║
║ ~/.config/MangoHud/MangoHud.conf                  ║   383 ║   383 ║
║───────────────────────────────────────────────────║───────║───────║
║ TOTAL (17 managed files)                          ║ 5,093 ║ 5,089 ║
```

The udev rule at **639 B** and the **5,089 B** total are the anchors for any perf-value
change — a value substitution plus its comment edits moves both.

### 7e. Generated bodies — all seventeen, byte-exact

Rendered from the 7.141.0 generators under the §9 harness, 3/3 deterministic. Reproduce
before diffing; do not diff against a prior edition's fences without re-rendering.

**`/boot/loader/loader.conf`** (89 B):

```
# systemd-boot loader configuration
default @saved
timeout 0
console-mode keep
editor no
```

**`/etc/kernel/cmdline`** (352 B; the UUID below is a 36-char stub — a real render is the
same length):

```
rw root=UUID=12345678-90ab-cdef-1234-567890abcdef amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0
```

**`/etc/sdboot-manage.conf`** (544 B):

```
# sdboot-manage configuration — changes require: sudo sdboot-manage gen && sudo sdboot-manage update
LINUX_OPTIONS="amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0"
LINUX_FALLBACK_OPTIONS="quiet"
DEFAULT_ENTRY="manual"
REMOVE_EXISTING="yes"
OVERWRITE_EXISTING="yes"
REMOVE_OBSOLETE="yes"
```

**`/etc/mkinitcpio.conf`** (276 B — was 280 B before the `-T0` cut):

```
# mkinitcpio configuration — changes require: sudo mkinitcpio -P && sudo sdboot-manage update
MODULES=(amdgpu)
BINARIES=()
FILES=()
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
COMPRESSION="zstd"
COMPRESSION_OPTIONS=(-1)
```

**`/etc/systemd/resolved.conf.d/99-cachyos-resolved.conf`** (154 B):

```
# systemd-resolved: AdGuard upstreams, plaintext, mDNS/LLMNR off
[Resolve]
DNS=94.140.14.14 94.140.15.15
MulticastDNS=no
LLMNR=no
DNSOverTLS=no
DNSSEC=no
```

**`/etc/systemd/logind.conf.d/99-cachyos-logind.conf`** (292 B):

```
# systemd-logind configuration — desktop power handling
[Login]
HandlePowerKey=ignore
HandlePowerKeyLongPress=ignore
HandleSuspendKey=ignore
HandleSuspendKeyLongPress=ignore
HandleHibernateKey=ignore
HandleHibernateKeyLongPress=ignore
HandleRebootKey=ignore
HandleRebootKeyLongPress=ignore
```

**`/etc/systemd/system/NetworkManager-dispatcher.service.d/logging.conf`** (127 B):

```
# LogLevelMax drops info-level dispatcher lines (journald-logged; StandardError=null ineffective)
[Service]
LogLevelMax=notice
```

**`/etc/NetworkManager/conf.d/99-cachyos-nm.conf`** (219 B) — `[global-dns-domain-*]` is
the mechanism that actually beats per-link DHCP DNS:

```
# NetworkManager configuration — wpa_supplicant backend
[device]
wifi.backend=wpa_supplicant

[connection]
wifi.powersave=2

[global-dns]

[global-dns-domain-*]
servers=94.140.14.14,94.140.15.15

[logging]
level=WARN
```

**`/etc/iw-regdomain`** (88 B):

```
# ry-install: wireless regulatory domain (managed file, do not edit by hand)
COUNTRY=US
```

**`/etc/bluetooth/main.conf`** (147 B):

```
# ry-install: BlueZ daemon config (managed file, do not edit by hand)
[General]
FastConnectable=true

[Policy]
AutoEnable=true
ReconnectAttempts=3
```

**`/etc/nftables.conf`** (729 B, single form — no remote-play variant exists):

```
#!/usr/bin/nft -f
# ry-install: default-deny-inbound, IPv4-only (ufw masked; ipv6.disable=1). Add inbound ports below.
flush ruleset
table inet filter {
    chain input {
        type filter hook input priority filter; policy drop;
        ct state invalid drop # early drop of invalid connections
        ct state established,related accept
        iif "lo" accept
        # IPv4 ICMP: inbound ping (echo-request) + error/PMTUD types (replies match ct established)
        icmp type { echo-request, destination-unreachable, time-exceeded, parameter-problem } accept
    }
    chain forward { type filter hook forward priority filter; policy drop; }
    chain output { type filter hook output priority filter; policy accept; }
}
```

**`/etc/default/cpupower-service.conf`** (115 B) — uppercase `GOVERNOR=` is exactly what
cpupower 7.1.5's service script reads; notes citing lowercase `governor=` are stale:

```
# cpupower-service.conf — sourced by /usr/lib/systemd/scripts/cpupower (cpupower.service)
GOVERNOR='performance'
```

**`/etc/sysctl.d/95-ry-overrides.conf`** (441 B):

```
# ry-install sysctl tunables (priority 95 — loaded after CachyOS vendor 70-cachyos-settings.conf)
kernel.nmi_watchdog = 0
net.core.default_qdisc = fq
net.core.netdev_budget = 600
net.core.netdev_budget_usecs = 5000
net.ipv4.tcp_congestion_control = bbr
net.ipv4.tcp_notsent_lowat = 16384
net.ipv4.tcp_slow_start_after_idle = 0
vm.compaction_proactiveness = 0
vm.max_map_count = 2147483642
vm.swappiness = 150
vm.watermark_boost_factor = 0
```

**`/etc/udev/rules.d/99-ry-perf.rules`** (639 B) — the perf anchor; lines 4-6 and 7-9 are
sites 5 and 6 of the 13-site list:

```
# ry-install: udev performance rules (managed file, do not edit by hand)
# NVMe scheduler none (lowest tail latency; diverges from CachyOS kyber default)
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
# AMD P-State EPP performance (maximum CPPC hint)
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="performance"
# GPU performance level (gfx1151 clock-floor; forced high)
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="high"
```

**`/etc/modprobe.d/60-ry-modules.conf`** (183 B, shipped default `BLACKLIST_AMDXDNA=true`).
**The `false` variant renders 177 B without the last two lines — do not capture the fence
from a toggled state:**

```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# blacklist amdxdna: XDNA NPU needs IOMMU, probes -ENODEV (ret -19) under amd_iommu=off
blacklist amdxdna
```

**`~/.config/environment.d/10-environment.conf`** (311 B, mode 0600):

```
# Environment for systemd --user services and graphical sessions (Plasma, Flatpak, D-Bus apps)
DXVK_LOG_LEVEL=none
MANGOHUD=1
MESA_SHADER_CACHE_MAX_SIZE=16G
POWERDEVIL_NO_DDCUTIL=1
PROTON_ENABLE_WAYLAND=1
PROTON_FSR4_UPGRADE=1
PROTON_LOCAL_SHADER_CACHE=1
VKD3D_DEBUG=none
VKD3D_SHADER_DEBUG=none
WINEDEBUG=-all
```

**`~/.config/MangoHud/MangoHud.conf`** (383 B, mode 0600, 19 active directives) — `cpu_temp`
is commented out on purpose, see T1-1:

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

### 7f. Removal reconciliation — state the tier for any removal you recommend

```
║ TIER ║ CLASS                        ║ ON REMOVAL         ║ DETECTION            ║
║──────║──────────────────────────────║────────────────────║──────────────────────║
║ 1    ║ value in a generated file    ║ self-heals on next ║ n/a — cannot orphan  ║
║      ║ (KERNEL_PARAMS, ENV_VARS,    ║ deploy; generator  ║                      ║
║      ║ SYSCTL_VALUES, …)            ║ rewrites wholesale ║                      ║
║ 2    ║ masked unit dropped from MASK║ stays masked       ║ DETECTED — INFO +    ║
║      ║                              ║ forever            ║ MASK_ORPHAN JSONL    ║
║ 2    ║ stale 60-ry-*.conf drop-in   ║ stays on disk      ║ DETECTED — WARN +    ║
║      ║                              ║                    ║ MODPROBE_STALE_DROPIN║
║ 2    ║ external sdboot-manage       ║ n/a — never ours   ║ DETECTED — WARN +    ║
║      ║ drop-in (NEW 7.140.0)        ║                    ║ SDBOOT_DROPIN_PRESENT║
║ 3    ║ package dropped from         ║ stays installed    ║ NOT DETECTED —       ║
║      ║ PKGS_ADD                     ║ forever            ║ by design (T5)       ║
```

Tier 2 is reportable but never self-healing and never sets DRIFT — the operator must act by
hand. Tier 3 is invisible.

### 7g. Gates, exits, backups

- **Preflight validator chain, four deep, called from `_init_runtime` (L747) at L779-782:**
  `_ir_validate_counts` (L665, 21 tripwires) → `_ir_validate_keys` (L693, domains and
  charsets, incl. the nftables↔`ipv6.disable` coupling and the
  `BLACKLIST_AMDXDNA=false`↔IOMMU-on coupling) → `_ir_validate_sets` (L734) →
  `_ir_validate_post_hooks` (L4591). **There is no `_ry_validate_keys`,
  `_ry_validate_counts` or `_ry_validate_post_hooks`** — those names return rc 127, which
  looks like a signal and is only an unknown command. Real sibling validators:
  `_ry_validate_mkinitcpio_hooks`, `_ry_validate_mkinitcpio_modules`, `_rvc_dispatch`,
  `_ry_validate_configs`.
- **`_ir_validate_sets` refuses three intersections**, each `_err_loud` + exit 3:
  `PKGS_ADD ∩ PKGS_DEL` (phase 2 installs what phase 4 removes), `EXPECTED_SERVICES ∩ MASK`
  (phase 4 masks before it enables), `_RY_PKG_MANAGED_SERVICES ∩ MASK` (a masked unit cannot
  be package-managed). All three shipped intersections are empty. **Any recommendation that
  adds a package or unit must be checked against all three — a contradiction is a hard
  deploy refusal, not a silent misbehaviour.**
- **Phases (6, `_RY_PHASE_NAMES`):** Preflight → Packages → Configuration →
  Services (fstab → resolved → pkg-remove → mask → enable → regdom) → Boot → Finalize,
  mirrored 1:1 across the array, `_progress`, the orchestrator, the log headers and the
  README. `_PROG_STEPS` is derived from the array, so phase order cannot drift.
- **Hardware gate:** CPU model match on `EXPECTED_CPU_MATCH` = `Ryzen AI Max` (L614); sole override
  `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed on an unreadable model; `--verify` warns,
  deploy and `--check` exit 3.
- **Runtime env inputs are exactly three:** `RY_RUN_TIMEOUT`,
  `RY_INSTALL_SKIP_HARDWARE_CHECK`, `NO_COLOR` (plus multiplexer detection). Every profile
  toggle is an embedded scalar set unconditionally with `set -g`, so an exported env var of
  the same name is clobbered — opting in means editing the script. `NO_COLOR` needs a
  **non-empty** value to fire; `TERM=dumb` disables colour independently. `RY_RUN_TIMEOUT=0`
  disables the timeout; package and boot operations floor at 7200 s
  (`_RY_LONGOP_HARD_CAP`).
- **Dependencies:** 37 hard-required commands (missing any → rc 1) + **15** warn-only
  optional tools, plus a `df --output` probe and systemd ≥ 250. Optional list at L1580:
  bootctl journalctl modinfo pgrep zcat tput lsmod modprobe pkill nmcli ping realpath ip
  lspci kill. Profile-installed tools (`nft`, `pactree`, `paccache`, `sysctl`, `udevadm`,
  `bluetoothctl`, `timedatectl`, `ufw`, `iw`) are call-site `command -q` guarded and
  deliberately undeclared — do not "fix" the asymmetry.
- **Exit model, 14 constants:** 0 OK · 1 FAIL · 2 USAGE · 3 PREFLIGHT · 4 BOOT_CRIT ·
  5 LOCK · 10 DRIFT; internal sentinels 11 GEN_NOFN, 12 GEN_NOUUID, 13 GEN_SYSCTL,
  14 GEN_ENVD, 250 AS_MISUSE, 251 RUN_TMPFAIL, 255 RUN_MISUSE — function returns only,
  never a process exit. Signals map to 128+N. **No orphan class reaches EXIT_DRIFT.**
  Root `--check` → rc 3, silent, no JSONL (the root guard precedes LOG_DIR init); all other
  root modes → rc 2, loud, 92 B. Lock dir `$HOME/ry-install/.lock` with a `pid` file inside
  (L335); `--verify` and `--check` are lock-free. Expected live `--verify` OK count **278** —
  an empirical figure from a clean host run, not a literal in the script; re-baseline it
  after any verify-surface change.
- **JSONL logs land under `$HOME/ry-install/logs/<YYYY-MM-DD>/<mode>-<timestamp>-<pid>.jsonl`**
  (L182 / L199), not in the profile root. A glob that misses the `logs/<date>/` level returns
  zero hits silently, which reads as "the key was never emitted" — the exact false negative
  that the `.ry.orig` forensics in T0-7 and §5 item 9 depend on not making.
- **Argument parsing (`_RY_ARGPARSE_SPEC`, L14, 6 entries):**
  `--exclusive=verify,check,install-file` `h/help` `v/version` `verify` `check`
  `install-file=`. No positional arguments; `--` ends option parsing and anything after it
  exits 2. Glued short flags early-intercept with first char winning (`-hv`/`-hV` → help,
  `-vh` → version); `-h`/`-v` are honored before the root guard.
- **Backups are two different guarantees.** The 4 boot-critical destinations
  (`/boot/loader/loader.conf`, `/etc/kernel/cmdline`, `/etc/sdboot-manage.conf`,
  `/etc/mkinitcpio.conf` — `_RY_BACKUP_TARGETS` and `_RY_BOOT_CRITICAL_DSTS` are the same
  four) get `.ry.bak` **plus post-write re-read and restore**. The other 13 get a one-time
  `.ry.orig` on **first adoption only** — captured once, then every subsequent overwrite is
  silent by design. `/etc/fstab` is not a managed destination and has always had its own
  `.ry.bak` during rewrite. **Do not describe the 13 as backed up on every write, and do not
  recommend `.ry.orig` for the boot four — it is weaker than what they have.** No verify
  path asserts `.ry.orig` exists and none can: absence is indistinguishable from "the
  destination had no pre-existing content", which is the common case.

---

## 8. Verify-surface ownership

**62 verify functions = 12 `_verify_*` + 50 subs** (`_vsb` 7 · `_vss` 10 · `_vsp` 3 ·
`_vsc` 1 · `_vrk` 4 · `_vrkm` 3 · `_vrsv` 10 · `_vre` 5 · `_vrs` 4 · `_vpd` 1 · `_vmh` 2).
A recommendation that changes a value must state which **sub** asserts it, and whether that
sub hard-fails or warns.

```
║ ORCHESTRATOR             ║ SUBS THAT MATTER FOR TUNING FINDINGS             ║
║──────────────────────────║──────────────────────────────────────────────────║
║ _verify_static_boot      ║ _vsb_loader (2115) · _vsb_sdboot (2120) ·        ║
║ (2263)                   ║ _vsb_sdboot_dropins (2151, NEW — WARN) ·         ║
║                          ║ _vsb_cmdline (2165, all 15 tokens + root= + rw) ·║
║                          ║ _vsb_mkinitcpio (2192; live COMPRESSION/_OPTIONS,║
║                          ║ multi-line join, last-wins) · _vsb_entries (2236)║
║                          ║ -> _vsb_entry_options (2219)                     ║
║ _verify_static_system    ║ _vss_logind (2274) · _vss_nmdispatch (2280) ·    ║
║ (2324)                   ║ _vss_nm (2281) · _vss_sysctl (2288, 11 keys) ·   ║
║                          ║ _vss_regdom (2294) · _vss_bluetooth (2295) ·     ║
║                          ║ _vss_udev (2302; 3 rules, EPP + DPM aware) ·     ║
║                          ║ _vss_modprobe (2315; blacklist + stale-dropin    ║
║                          ║ sweep, WARN) · _vss_nft (2310; HARD-FAILS on a   ║
║                          ║ missing echo-request accept)                     ║
║ _verify_static_user      ║ inline: ENV_VARS + MangoHud directives           ║
║ (2348)                   ║                                                  ║
║ _verify_static_packages  ║ _vsp_required (2360; PKGS_ADD 16 + Vulkan 2) ·   ║
║ (2425)                   ║ _vsp_removed (2383) · _vsp_pacman_conf (2393)    ║
║ _verify_static_services  ║ MASK 11 inline + _vss_orphan_masks (2459; INFO,  ║
║ (2435)                   ║ admin-scope /etc + /run only)                    ║
║ _verify_static_syntax    ║ live mkinitcpio HOOKS presence, multi-line       ║
║ (2466)                   ║ tolerated                                        ║
║ _verify_static_checksum  ║ _vsc_check_one (2480; expected vs installed      ║
║ (2509)                   ║ SHA256 per destination)                          ║
║ _verify_runtime_kparams  ║ _vrk_cmdline (2681; every token + rw) ·          ║
║ (2849)                   ║ _vrk_gpu_state (2700; QUOTED compare vs          ║
║                          ║ $GPU_DPM_LEVEL) · _vrk_cpu_state (2722; cpu0     ║
║                          ║ detail + FULL cpufreq-policy uniformity sweep) · ║
║                          ║ _vrk_module_state (2824) -> _vrkm_amdgpu (2767), ║
║                          ║ _vrkm_blacklist (2787), _vrkm_blacklist_modprobe ║
║                          ║ (2803; amdxdna LOADED = FAIL)                    ║
║ _verify_runtime_services ║ _vrsv_sys_units (2937) -> _vrsv_chk_active_      ║
║ (3049)                   ║ enabled (2857), _vrsv_chk_nftables (2880; judged ║
║                          ║ by live policy-drop, not unit state),            ║
║                          ║ _vrsv_chk_resolved (2910; enabled|static),       ║
║                          ║ _vrsv_chk_cpupower_governor (2922) ·             ║
║                          ║ _vrsv_nft_assert_ping (2872; WARN) ·             ║
║                          ║ _vrsv_wifi (2975) -> _vrsv_wifi_nm_backend (2954)║
║                          ║ · _vrsv_masked_inactive (3015; iterates $MASK) · ║
║                          ║ _vrsv_user_units (3032)                          ║
║ _verify_runtime_env      ║ _vre_envvars (3060; dynamic over $ENV_VARS) ·    ║
║ (3166)                   ║ _vre_sysctl_runtime (3078; 11, absent-vs-        ║
║                          ║ unreadable split, WARNs on an absent knob) ·     ║
║                          ║ _vre_fstab (3096; every ext4 entry) ·            ║
║                          ║ _vre_ntsync (3127) · _vre_regdom (3149)          ║
║ _verify_runtime_session  ║ _vrs_nm_perms (3175) · _vrs_installed_file_perms ║
║ (3282)                   ║ (3201) -> _vrs_vfat_skip (3190) ·                ║
║                          ║ _vrs_parent_dirs (3248) -> _vpd_dir_perm_check   ║
║                          ║ (3229)                                           ║
║ _verify_summary (1198)   ║ pass/fail/warn tally — not an assertion path     ║
```

**Attribution traps:**

- `_vrkm_blacklist_modprobe` is **generator-sourced** — it checks intended content, not
  on-disk extras. Attribute the drop-in sweep to `_vss_modprobe` / `_ry_stale_ry_dropins`
  (L2602), never to it.
- `_vrsv_masked_inactive` asserts every *declared* mask is present and inactive. The reverse
  direction (masked units not declared) is `_vss_orphan_masks`, which lives on the **static**
  services path. Do not conflate the two.
- `_vsb_entry_options` is a sub of **`_vsb_entries`**, not of `_verify_static_boot`. It
  skips `*-fallback.conf` by design, which is why T4-1 is invisible to verify.
- `_vss_nft` hard-fails on a missing inbound echo-request accept; `_vrsv_nft_assert_ping`
  only warns. A recommendation to drop inbound ping must address the hard-fail.
- `_ry_orphan_masked_units` (L2606) filters to **admin scope only** (`/etc`, `/run`) —
  vendor `/usr/lib` masks and `Alias=` cascades are deliberately dropped. `Alias=` matters
  here: masking `avahi-daemon.service` makes its upstream alias read masked too.
- **Removed asserts — do NOT verify:** `_vrkm_iommu`, `_vrk_clocksource`, `_vre_zram`,
  `_vre_tcp` (all gone since 7.90.0); kernel-floor and Mesa-floor checks; the preemption
  advisory (7.139.0 r2). No THP, KSM, `ttm.*`, drirc, `iommu=pt`, ICMPv6/NDP or baloo assert
  exists. No `VKD3D_CONFIG` assert exists — the variable left ENV_VARS and `_vre_envvars`
  followed automatically.
- **Sandbox artifact, not a regression:** `_ry_validate_mkinitcpio_hooks` returns rc 1 and
  `_ry_validate_configs` returns rc 3 in a container because `/etc/mkinitcpio.conf` is
  absent. Always A/B a nonzero validator rc against the previous release.

**Output-channel invariant — reconciled; do not re-file as drift.** Every leveled
user-facing message funnels through `_msg_print` (**L1109**), honouring QUIET /
`_RY_OUTPUT_BROKEN` / `_RY_NO_COLOR` / isatty(2). Raw `>&2` counts of ~78 (whole-file) and
~43 (inside function bodies) are **both correct under their own scoping**; the remainder are
top-level pre-init preflight writes made before `_msg_print` is defined. The invariant means
"single authority for leveled user-facing output", not "sole writer to fd 2" — the latter
reading generates a false finding every time. stdout carries only `--help` and `--version`.

---

## 9. Reproduction method and its traps

Recorded so the next rebase does not re-pay these costs.

- **Harness.** Cut the script just before the `# ── MAIN: ARGPARSE` banner —
  **L4861 at 7.141.0**, L4845 at 7.139.0, L4834 at 7.135.1. **Always locate the banner,
  never hardcode.** Then delete the L3 source guard and `source` the result as a non-root
  user with a **writable `$HOME`**:
  `sed -n '1,4860p' ry-install.fish | sed '3d' > harness.fish`. Without the L3 deletion the
  guard fires on `source` and every count silently reads 0. Without a writable `$HOME` the
  log-directory init aborts the source part-way and counts read 0 the same way — a different
  cause with an identical symptom. Pre-create `$HOME/.config/fish` and
  `$HOME/.local/share/fish`.
- **Shadow `exit` before sourcing** (`function exit; end`, then `functions -e exit`
  afterwards) so the fallen-through top level runs to completion; oracle counts, scalars and
  all 17 generators then probe in ONE shell. TRAP: that flow calls `_ry_erase_handlers`,
  which `functions -e`'s `_cleanup*` and `_progress_on_winch`, so the live function table
  UNDER-COUNTS handler families — take the census from `^function ` at column 0 in source
  (**294** at 7.141.0), not from a before/after `functions --all --names` diff.
- **`test -w /tmp` at L161 bails rc 3 before anything else.** Any sandbox reproduction needs
  a non-root user *and* a writable `/tmp`.
- **Array counts by live fish eval, never text parsing.** `eval echo \$$name` collapses a
  fish array and reports every count as 1 — use `eval "set vals \$$name"` then
  `count $vals`. Continuation regexes truncate multi-line declarations, `set -g --` evades
  awk, and several service keys share one `set -g` line (which also breaks any
  line-anchored `^set -g NAME` scalar extraction — use a non-anchored `finditer` and
  de-duplicate preserving order; strip trailing `;` from values).
- **Filter generators to `_content__*` + `_content_HOME*`.** A bare `_content_*` glob also
  catches the `_content_fn_for` dispatcher (L935) and returns 18. Fish function names
  contain dots (`_content_HOME_.config_MangoHud_MangoHud.conf`), so a `[A-Za-z0-9_-]*`
  charset truncates them and fabricates duplicates — match `\S+` after `function`.
- **Set `_ROOT_UUID`** (single underscore, not `_RY_ROOT_UUID`) with any 36-char valid-shape
  UUID, or the cmdline generator returns 12 / `GEN_NOUUID` and `_ry_validate_configs`
  returns rc 3, which reads as a regression. With the stub, the render reproduces the host
  byte count exactly.
- **Generated bytes must be measured as WRITTEN FILES** (`$fn > tmp; stat -c %s`), never
  `(cmd | string collect)` — collect strips each trailing newline and the 17-file total
  reads 5,072 B instead of 5,089 B, a phantom deficit that looks like anchor drift.
  `string length` counts characters and under-reports for the same reason.
- **Determinism must be compared per-file by content hash, or with a SORTED manifest.** A
  concatenated hash over the output directory is not comparable between passes because the
  glob order depends on harness filenames. The manifest sha changes between editions even
  when every body is identical — that is expected, not drift.
- **Verify every "before" column against the OLD EDITION'S RENDERED BODY, not memory.** The
  old script is not in the archive; only its brief is. A drift row asserting a change that
  never happened passes every count check and every byte check — only an old-vs-new body
  diff catches it. Extract the previous edition's fences with a **toggle-based** fence
  walker; a naive `re.findall` on triple-backticks consumes alternating pairs and
  under-reports (it returned 0 nft candidates on a brief that contains one). **That walker
  is how the modprobe-fence defect in the 7.135.1 edition was found** — a body captured from
  the `BLACKLIST_AMDXDNA=false` variant (177 B) while the size table correctly carried the
  shipped 183 B default. §7e now embeds all 17 bodies precisely so this class cannot recur
  silently.
- **Fence-aware heading checks are mandatory.** A naive `^# ` scan false-reports h1
  violations on the `#` comments inside the nft, udev, sysctl and config fences.
- **Byte-vs-character length.** Banner and line-length checks must count CHARACTERS; `awk
  length` under a C locale counts bytes and falsely reports the U+2500/U+2192 box-drawing
  content over any character cap.
- **CRLF and banner false positives.** A byte-level `b'\r' in raw` test reports dozens of
  hits that all sit inside string literals; test `line.endswith(b'\r')` instead — 0 lines
  actually end with CR. And a "contains U+2500" banner-width check flags a one-line function
  whose box characters are in runtime `_echo` output, not a comment banner; split the
  classes.
- **Verify the upload before trusting it.** A fresh upload can be behind what is recorded as
  shipped. Hash the archive against §0 first. Collision precedent is routine: 7.141.0
  shipped twice with an identical script hash, 7.140.0 shipped nine times with only two
  distinct script hashes. Read the `VERSION` line plus a structural count, then confirm with
  the zip/README/CHANGELOG hash.
- **Sandbox limits.** Target host paths do not exist, so only sudo-fail and preflight paths
  are exercisable and a full install cannot complete by design. `_err_loud` **exits** rather
  than returns — run each negative test in its own subprocess. Useful shims: a 3-line `sudo`
  wrapper (strip `-n`/`--`, exec) makes `_as true` paths runnable unprivileged;
  `functions FN | string replace -a /sys/... /tmp/fx/... | source` re-binds hardcoded sysfs
  paths for fixture tests. fish `count` counts ARGUMENTS, not stdin — `… | count` is always
  0; use `count (cmd)`. `find -type d` returns 0 hits for cpufreq because
  `/sys/devices/system/cpu/cpuN/cpufreq` is a symlink — use `-xtype d` or a glob + `test -d`.

---

## 10. Scope and output contract

**In scope:** recommendations only — do not emit a modified script. Hardware-anchored to
gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE / 85 W BIOS ceiling.

**Out of scope:** dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS,
laptops, UKI, BIOS flashing. Per-game Proton tuning is secondary to system-wide config
except where the brief explicitly moves an item there (T1-2, T1-3).

**Rules:**

1. Respect deliberate trade-offs — flag and quantify, do not auto-FIX. Reserve FIX for
   incorrect, superseded, deprecated, or harmful values.
2. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is
   non-trivial.
3. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN.
4. Flag every source conflict and name the trusted side. Two live conflicts are already
   named here: netdev budget guidance (Red Hat vs ESnet) and nftables rule order (Gentoo vs
   nftables.org/ArchWiki). Do not resolve either silently.
5. Give exact versions (kernel / Mesa / linux-firmware / proton-cachyos / package) and exact
   before→after, mapped to the in-script global.
6. **Do not carry values forward from any pre-7.141.0 edition.** A recommendation that
   re-derives a removed token, or that asserts a tuning value *changed* in the 7.130.0 →
   7.141.0 window, is a stale-source error rather than a finding.
7. A question closed by a code change is closed. A question closed by upstream evidence
   (§3) is closed. Cite them as settled and evaluate their *design*, not their existence.
8. A correction in §4 is binding. Do not re-raise a finding this edition withdrew.

**Required output:**

- **Findings matrix** (box-drawn, code-fenced, grouped by tier): ITEM · CURRENT · CALL
  (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE.
- **Before→after** for each TUNE/FIX, naming the in-script global and — for any perf value —
  all 13 sites from §7b.
- **Tier placement** for every recommendation, and the removal tier (§7f) for any removal.
- **Oracle delta** for anything that changes an array length, plus the affected byte anchors
  from §7d.
- **Security delta** (§5, ordered).
- **Verdict** per tier plus overall (PASS / PASS-WITH-FIXES / FAIL).
- **Methodology:** source list with access dates and versions; unknowns marked UNCERTAIN.

---

## Sources

**Primary, accessed 2026-07-27:** git.kernel.org torvalds/linux at 7.2.0-rc5 —
`Documentation/admin-guide/kernel-parameters.txt`, `drivers/pci/pcie/Kconfig`,
`drivers/cpufreq/amd-pstate.c`, `drivers/cpufreq/Kconfig.x86`,
`drivers/net/wireless/mediatek/mt76/mt7925/pci.c`,
`drivers/net/ethernet/realtek/r8169_main.c`, `Makefile` · git.kernel.org linux-firmware
`amdgpu/gc_11_5_1_mes_2.bin` history · github.com CachyOS/linux-cachyos PKGBUILD + config
(7.1.5-2) · gitlab.freedesktop.org mesa/mesa `VERSION` (26.3.0-devel) + `docs/envvars.rst` ·
github.com CachyOS/proton-cachyos releases (11.0-20260703) and the Proton-EM `em-10` docs it
designates authoritative (`FSR4.md`, `EM-ADDITIONS.md`) · github.com flightlessmango/MangoHud
README + issue #1794 · github.com systemd/systemd #33579 · docs.redhat.com RHEL 8/9/10
network performance tuning · fasterdata.es.net 100 G "Other Tuning" · wiki.gentoo.org
Nftables/Examples · wiki.nftables.org quick reference · wiki.archlinux.org Nftables.

**Standing reference:** docs.kernel.org (PCIe/ASPM, amd-pstate, sysctl/vm, sysctl/kernel,
networking, block, ext4, UMIP, AMD-Vi, accel/amdxdna, amdgpu
`power_dpm_force_performance_level`) · docs.mesa3d.org · wiki.archlinux.org (AMDGPU, IOMMU,
fsck, Gaming, Zram, SSD, Ext4, Btrfs, Sysctl, NetworkManager, Wireless,
Uncomplicated_Firewall, Mkinitcpio, systemd-boot, Bluetooth, CPU_frequency_scaling,
MangoHud, pacman) · wiki.cachyos.org · discuss.cachyos.org · man.archlinux.org (nft,
avahi-daemon, systemd.unit, logind.conf, resolved.conf, NetworkManager.conf, systemd-fsck) ·
invent.kde.org powerdevil · fishshell.com/docs · strixhalo.wiki (Power Modes, C-States) ·
amd.com ROCm · mangohud-gtr9-pro v1.17.0 companion archive.

**Do not cite** wireless.docs.kernel.org for `pcie_aspm` semantics (pre-6.9 text);
kernelconfig.io for Kconfig introduction versions; any source attributing the 600/4000
netdev pairing to ESnet. Cite access dates and exact versions in the methodology block.
