# CachyOS Tuning Audit — Beelink GTR9 Pro (gfx1151)

Pinned to `ry-install.fish` **7.139.0 r3**. Deep-research brief: actionable items only,
**ordered by implementation safety** rather than by subsystem.

---

## 0. Provenance

Every number below was re-derived by live evaluation of the attached archive. Nothing was
carried forward from the 7.135.1 edition.

```
║ ARTIFACT           ║ SHA256   ║ SIZE               ║
║────────────────────║──────────║────────────────────║
║ zip                ║ ec54f631 ║ 327,496 B          ║
║ ry-install.fish    ║ bcbb695f ║ 4,974 L / 294,363 B║
║ README.md          ║ 34a8b6d6 ║   318 L /  22,166 B║
║ CHANGELOG.md       ║ c581c726 ║   197 L /   9,210 B║
║ LICENSE            ║ 2e1e7c8a ║    21 L /   1,069 B║
```

Three artifacts self-report `v7.139.0`. **Disambiguate by zip or CHANGELOG hash, never by
`--version`.** This brief audits r3 (`ec54f631` / CHANGELOG `c581c726`).

---

## 1. Delta vs the 7.135.1 edition

### 1a. The tuning surface did not move — again

**Not one tuning value changed 7.135.1 → 7.139.0.** All 21 count tripwires, all profile
scalars and all 17 generated bodies are byte-identical, total **5,093 B**, determinism
3/3 renders (sorted-manifest sha `6a64251d19c450dc` — harness-filename dependent, not
comparable across passes; per-file content hashes are the safe form).

That is now **nine releases** (7.130.0 → 7.139.0) with a frozen tuning surface. A
recommendation asserting that any value moved in this window is a stale-source error.

### 1b. What did change

```
║ AREA          ║ 7.135.1        ║ 7.139.0 r3     ║ NOTE                        ║
║───────────────║────────────────║────────────────║─────────────────────────────║
║ script        ║ 4,963 L        ║ 4,974 L        ║ +11                         ║
║ functions     ║ 292            ║ 293            ║ +_vsb_entry_options         ║
║ verify fns    ║ 60 (12+48)     ║ 61 (12+49)     ║ orchestrators unchanged     ║
║ sub: markers  ║ 93             ║ 94             ║ all PARENT_OK               ║
║ optional deps ║ 20             ║ 19             ║ dmesg dropped (r2)          ║
║ hard deps     ║ 37             ║ 37             ║ unchanged                   ║
║ banners       ║ 90             ║ 90             ║ unchanged                   ║
║ harness cut   ║ L4834          ║ L4845          ║ ALWAYS locate, never hardcode║
```

**Two features were removed and must not be re-derived from a stale source:**

- **`RY_REMOTE_PLAY_PORTS` is gone (7.137.0).** The gate, the Sunshine/Steam TCP/UDP
  rules and the 916 B ruleset variant no longer exist. The nftables body is 729 B, single
  form, no variant. Any finding about remote-play ports, port 47984/27036 sets, or the
  missing 5353/udp mDNS hole is now **out of scope** — there is nothing to open or close.
- **The preemption-model advisory is gone (7.139.0 r2).** The `_vrk_cmdline` tail section,
  the `dmesg` fetch, `DMESG_CACHE_EMPTY`, `_RY_DMESG_LINES`/`_RY_DMESG_PREEMPT` and the
  `dmesg` optional-dep entry were all removed. Residue confirmed **0** (the one surviving
  `dmesg` string is the L2703 amdgpu diagnostic hint, deliberately kept). The profile never
  pinned `preempt=`, so nothing was ever asserted. Do not report a missing preempt check.

**New verify coverage at 7.139.0 (all evaluated below, none changes a tuning value):**

```
║ SUB / BEHAVIOUR                    ║ WHAT IT NOW ASSERTS                        ║
║────────────────────────────────────║────────────────────────────────────────────║
║ _vsb_entry_options (L2204)         ║ every non-fallback BLS entry carries all 15 ║
║                                    ║ KERNEL_PARAMS; *-fallback.conf skipped     ║
║ _vrk_cpu_state (L2706)             ║ full cpufreq-policy sweep for driver /      ║
║                                    ║ governor / EPP uniformity; cpu0 stays the  ║
║                                    ║ representative detail readout              ║
║ _vrsv_chk_resolved (L2894)         ║ resolved unit-file state enabled|static     ║
║ _ry_orphan_masked_units (L2590)    ║ admin-scope masks only (/etc, /run); vendor ║
║                                    ║ /usr/lib masks and Alias= cascades dropped ║
║ fstab / root-fs reporting          ║ root FS type + ext4 fstab entry count       ║
║ sysctl knob handling               ║ absent knob distinguished from unreadable   ║
║ logging                            ║ ms JSONL timestamps; CHECK_GREP key=value;  ║
║                                    ║ nftables verdict names the unit-file state ║
```

### 1c. Corrections this rebase makes to the previous edition

1. **Appendix B B10 was wrong in the 7.135.1 edition.** The default-state modprobe fence
   was captured from the `BLACKLIST_AMDXDNA=false` variant (177 B). The shipped default is
   `true`, rendering 183 B with `blacklist amdxdna`. The size table's 183 B was correct;
   the fence beneath it disagreed with it. Fixed here.
2. **Every script line anchor in the previous edition is off by one to two lines.** The
   perf globals sit at L586/L588/L590/L591 (were 587/589/591/592), `MASK` at L610 (was
   612), `_msg_print` at L1109 (was 1117), the validator chain at **L779-782** (was
   781-784). Re-derived, not shifted.
3. **P0 #2, #3, #4 and several others are closed by upstream evidence** gathered
   2026-07-25 and are recorded in §3 as settled, not re-asked.

---

## 2. Action queue — ordered by implementation safety

This is the operative section. Tiers ascend by blast radius: T0 changes nothing, T5 must
not be touched. **Work top-down; do not act on a lower tier while a higher tier that gates
it is unrun.**

### T0 — Observation only. No system change, no reboot, no risk.

Every one of these is a read. Four of the six gate a decision in a lower tier, which is
why they are first.

```
║ ID ║ ACTION                          ║ GATES              ║ STATUS      ║
║────║─────────────────────────────────║────────────────────║─────────────║
║ T0-1║ turbostat idle-floor capture   ║ every T3 power     ║ UNRUN — 9   ║
║     ║ (60 s idle, before/after)      ║ decision           ║ releases    ║
║ T0-2║ lspci -vv LnkCtl ASPM per link ║ T3-4 ASPM pair     ║ UNRUN       ║
║ T0-3║ findmnt root FS type           ║ T3-3 fsck.mode     ║ UNRUN       ║
║ T0-4║ k10temp hwmon + energy_uj mode ║ T1-1 / T2-3 HUD    ║ UNRUN       ║
║ T0-5║ linux-firmware MES revision    ║ nothing — advisory ║ UNRUN       ║
║ T0-6║ .ry.orig / .ry.bak inventory   ║ nothing — forensic ║ UNRUN       ║
```

**T0-1 is the single highest-value action in this brief and has never been run.** The
profile is pinned at maximum on three perf axes simultaneously — governor `performance`,
EPP `performance`, DPM `high` — while `processor.max_cstate=1` already blocks deep CPU
idle, under an 85 W BIOS ceiling shared between CPU and GPU. Three of those four raise the
**idle floor** rather than the load ceiling, and the 85 W PPT caps peak power regardless.
The profile CHANGELOG concedes the direction of the trade (peak unchanged, idle up) but
that is a stated expectation, not a measurement. Quantify idle package power and whether
sustained-load clocks *drop* because budget is burned at idle. Name the assumed budget
(85 W README ceiling vs 140 W stock) in every power statement.

Commands for all six are in §5.

### T1 — User/session scope. No root, no reboot, reversible by editing one file.

- **T1-1 · MangoHud `cpu_custom_temp_sensor` → k10temp.** Added in MangoHud 0.8.3; points
  the HUD at the k10temp hwmon explicitly and fixes the asusec mispick on this board.
  Gated on T0-4. **Do not enable `cpu_temp` as part of this** — MangoHud #1794 (cpu_power
  reads 0 on Zen 5 while cpu_temp is active) was still **OPEN** at last check (2026-07-25),
  so enabling `cpu_temp` still costs `cpu_power`. The custom-sensor option is the only path
  that might allow both. IMPACT Low · RISK Low.
- **T1-2 · Both user files now have a working first-adoption preserve.** As of 7.135.1,
  `~/.config/environment.d/10-environment.conf` and `~/.config/MangoHud/MangoHud.conf` get
  a one-time `.ry.orig`. Note this when recommending any user-scope change: the *first*
  hand-edit is preserved, every subsequent one is silently overwritten by design.

### T2 — Managed config values. Root, no reboot, self-heals on the next deploy.

Everything in this tier is Tier-1 under the removal-reconciliation model (§6): the
generator rewrites the file wholesale, so a bad value cannot orphan.

- **T2-1 · `netdev_budget_usecs=5000` vs the ES.net 100 G reference (4000).** The profile
  ships budget 600 / usecs 5000; the widely cited ES.net tuning page pairs 600 with 4000.
  Neither is wrong; establish whether the 25 % longer poll window measurably helps or hurts
  RTL8127 dual-10 GbE latency. TUNE only with driver evidence. IMPACT Low · RISK Low.
- **T2-2 · nftables rule order: `ct state invalid drop` precedes `iif "lo" accept`.** Most
  netfilter guidance places the loopback accept first, specifically to avoid conntrack
  classifying legitimate loopback traffic as invalid. This is a legitimate FIX candidate
  *if* any loopback path can be so classified; otherwise KEEP with a note. Cite nftables
  documentation, not folklore. IMPACT Low · RISK Low (order swap, `nft -c` gated,
  re-validated by `_post_nft`).
- **T2-3 · `energy_uj` world-readability for MangoHud `cpu_power`.** If T0-4 shows the
  powercap node is root-only, the fix is a permission drop-in — which would be an
  **18th managed file**, a scope addition, not a value change. Weigh a new managed
  destination against leaving `cpu_power` degraded. IMPACT Low · RISK Low-Med (new
  destination = new drift surface).
- **T2-4 · `PROTON_FSR4_UPGRADE=1` — KEEP-with-note, not FIX.** Confirmed 2026-07-25
  against proton-cachyos release "Version 11.0-20260702": the FSR4 DLL is copied
  automatically and the variable now version-pins (4.0.0 / 4.1.1 only). It is near-inert
  but harmless and correctly named. **The profile README needs no edit** — a previous
  research pass recommended a fix targeting text that does not exist.
- **T2-5 · `VKD3D_CONFIG=descriptor_heap` removal — validated, close it.** Mesa main has
  made the RADV descriptor heap default-on with `RADV_DEBUG=noheap` as the opt-out and the
  experimental flag gone; Mesa 26.1 still gates it behind `RADV_EXPERIMENTAL=heap`.
  Removal is correct on any Mesa the profile will meet. Residual check only: confirm
  vkd3d-proton has no *separate* opt-in that the removal also dropped.

### T3 — Kernel command line. Root + reboot. Reversible, but costs a boot cycle.

All four are **open maintainer trade-offs, not defects.** Flag and quantify; do not
auto-FIX. The cmdline is charset-gated (`^[A-Za-z0-9._,=-]+$`) and count-asserted at 15 —
any add or removal updates both, and `_vsb_entry_options` now additionally asserts every
non-fallback BLS entry carries every token.

- **T3-1 · `processor.max_cstate=1` — the highest-leverage single token in the set.** It
  blocks deep CPU idle, costs idle power and boost residency under the 85 W ceiling, and
  compounds directly with DPM pinned `high`. **Gated on T0-1**; do not decide without the
  measurement. Higher leverage than all three perf globals combined.
- **T3-2 · `clearcpuid=umip` — kernel tainted since 5.19.** Upstream states plainly that
  `clearcpuid` is not for production use. The trade is descriptor-table-base leak +
  support posture against `umip_printk` stutter. Quantify the taint consequence (does any
  subsystem refuse to load?); confirm the `umip` string form is still accepted. The string
  form is deliberate — CPUID bit numbers shift between kernels, the name does not.
  **Watch item:** the master kernel-parameters doc has dropped the `clearcpuid` entry while
  the code still parses it. That is deprecation pressure on a rolling host.
- **T3-3 · `fsck.mode=force` + `fsck.repair=yes`.** Largely **inert on a Btrfs root**.
  **Gated on T0-3** — establish the actual root filesystem before costing this. The fstab
  rewrite path is ext4-only, so an ext4 data volume and a Btrfs root can coexist here.
  Quantify boot cost against `commit=10` durability; state the filesystem assumption
  explicitly in any finding.
- **T3-4 · The ASPM pair — `pcie_aspm.policy=performance` + `mt7925e.disable_aspm=1`.**
  **The source conflict is SETTLED** (§3): `drivers/pci/pcie/Kconfig` PCIEASPM_PERFORMANCE
  states it disables ASPM L0s and L1 even where the BIOS enabled them. The correct reading
  of the pair is complementary, not redundant — the global policy governs link state, the
  module option disables at the endpoint, and coredumps are reported on the Wi-Fi adapter
  without the endpoint option. **Do not describe the module option as omitted** (true only
  between 7.102.x and 7.129.x) and **do not simplify the pair away** until T0-2 confirms
  per-link state on this board. `pcie_port_pm=off` stays KEEP-omitted unless T0-2 shows
  port PM active in a way that matters.

### T4 — Boot chain, firewall handoff, and detector severity. Reboot + recovery exposure.

- **T4-1 · Fallback-entry exposure.** `LINUX_FALLBACK_OPTIONS="quiet"` strips all 15
  params, so the fallback boots with IOMMU **on**, IPv6 **enabled** under an IPv4-only
  ruleset, and ASPM at firmware default with the MT7925 endpoint option absent — while the
  modprobe amdxdna blacklist *remains* active, because it is a file rather than a cmdline
  token. Confirm the window is accepted or flag it. Note `_vsb_entry_options` deliberately
  skips `*-fallback.conf`, so verify will never surface this.
- **T4-2 · mkinitcpio `COMPRESSION_OPTIONS=(-1 -T0)`.** Quantify boot decompress against
  default `-3` (sub-100 ms class on NVMe) and image size against the ESP budget
  (`BOOT_SPACE_CRIT/WARN` 200/500 MB) with multiple kernels plus fallback. TUNE to `-3`
  only if size threatens the budget.
- **T4-3 · `timeout 0` + `DEFAULT_ENTRY manual` + `REMOVE_EXISTING=yes`** wipes foreign
  BLS entries (EFI-resident loaders untouched). Recovery path is live-USB → chroot. Confirm
  sdboot-manage currency; UKI is out of scope.
- **T4-4 · ufw masked, not removed — confirm the nftables-first gate closes the window.**
  `_csm_enable_nftables_first` is gated on `contains ufw.service $MASK` and confirms a live
  default-deny ruleset *before* anything touches ufw; `_csm_prepare_ufw_masking` returns
  non-zero on an unconfirmed ruleset and `_configure_services_mask` then withholds
  `ufw.service` from the safe-mask set for that run. Rationale: `mask --now` stops ufw and
  `ufw-init stop` flushes its rules, so masking before default-deny is live would open an
  unfirewalled window. Validate on a host where ufw is installed *and* active; confirm a
  withheld mask leaves ufw's own rules intact rather than half-flushed; confirm
  `nftables.service` being a oneshot (unit state reads inactive) does not defeat the
  liveness check — the script judges by live policy-drop, which is correct but should be
  validated against current nftables packaging. Log keys: `UFW_MASK_DEFERRED`,
  `UFW_RULE_FLUSH_OK|FAIL|SKIP`, `SECURITY_POSTURE`.
- **T4-5 · Orphan-detector severity — a design question, not a gap.** Both detectors ship
  and neither sets DRIFT: `_vss_modprobe` WARNs on unmanaged `60-ry-*.conf` drop-ins,
  `_vss_orphan_masks` INFOs on masked units absent from `$MASK`, and `--check` records
  `MODPROBE_STALE_DROPIN` / `MASK_ORPHAN` to JSONL only. The reasoning on file: a re-run
  cannot clear either, so exit 10 would go permanently non-zero and train the operator to
  ignore exit 10 entirely. Mask ownership is genuinely **unattributable** — there is no
  `60-ry-`-style namespace for systemd units — which is why it is `_info`, not `_warn`.
  **Evaluate the trade.** The counter-argument worth testing: an INFO in a long `--verify`
  run is easy to miss and JSONL is only read deliberately. Any recommendation to promote
  either to DRIFT must address the desensitization argument explicitly, not around it.
- **T4-6 · `_check_record_orphans` sits after the sudo/systemctl preflight bail (L2616).**
  A stale drop-in is detectable with `find` alone — no privilege — yet the record is
  withheld when `sudo -n` is not cached. Consistent with `--check` being sudo-gated by
  contract, but it is a privilege-free signal withheld for a privilege reason. Assess
  whether the sweep should be hoisted above the bail.

### T5 — Closed and protected. Do not recommend changing these.

Flag a direct upstream contradiction as a **note**; never as a FIX.

- **Plaintext DNS on both host and router.** AdGuard's ad-block tier
  (94.140.14.14 / 94.140.15.15), `DNSOverTLS=no`, `DNSSEC=no`, and the router left on the
  plaintext picker. Rationale on file: filtering is identical either way, DoT buys only
  ISP query-name privacy, and router-side DoT in Strict mode fails closed — one TLS
  endpoint becomes a single point of failure for every LAN device. Uninterrupted
  connectivity for all devices is the stated priority. Quantify the exposure in §4; do not
  re-recommend DoT or DNSSEC.
- **`GPU_DPM_LEVEL=high`, not `profile_peak`.** `high` forces the highest power state with
  clock and power gating still active. `profile_peak` adds mclk/pcie forcing and disables
  gating, but kernel documentation scopes `profile_*` to measurement work, ArchWiki and
  amdgpu-clocks document `auto|low|high|manual` as the primary set, ROCm warns STABLE_PEAK
  is ASIC-specific and unverified on gfx1151, and Phoronix found forced `high` vs `auto`
  differs only in select cases. Evaluate `high` vs `auto` on frametime evidence only; note
  that observations predating the v7.94/95 udev matcher fix are void.
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
  the preemption advisory (7.139.0 r2); `pcie_aspm=off`;
  `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK`; `RY_NO_NTP_REMEDIATION`; `clearcpuid=514` numeric
  form; `archlinux-contrib`; the standalone `60-ry-blacklist-amdxdna.conf` and
  `60-ry-mt7925e.conf` drop-ins (both filenames are now actively swept for).
  **Standing precedent:** `mt7925e.disable_aspm=1` was removed at 7.102.x and re-added at
  7.129.0 — removals are not permanent, but a re-add needs the same evidence bar as a FIX.
- **Do not re-add** ICMPv6/NDP rules without restoring IPv6; do not flag inbound-ping
  accept (`_vss_nft` hard-fails on its absence, `_vrsv_nft_assert_ping` warns live); do not
  propose sleep-hook re-assert workarounds (all five sleep targets are masked, so there is
  no resume path); do not flag a low MangoHud `vram` reading on UMA (it reports the BIOS
  carveout only — `ram` carries the shared pool).

---

## 3. Settled — do not re-research

Confirmed from primary sources on 2026-07-25. Re-verify only if a citation is challenged.

```
║ CLAIM                                    ║ VERDICT / SOURCE                     ║
║──────────────────────────────────────────║──────────────────────────────────────║
║ Does the performance governor reject a   ║ YES. v6.18 amd-pstate.c              ║
║ non-max EPP write?                       ║ store_energy_performance_preference  ║
║                                          ║ returns -EBUSY for any epp>0 under   ║
║                                          ║ CPUFREQ_POLICY_PERFORMANCE and       ║
║                                          ║ forces epp 0. Writing "performance"  ║
║                                          ║ maps to epp 0 and IS accepted, so    ║
║                                          ║ the udev rule lands as a redundant   ║
║                                          ║ no-op. Rejection is pr_debug — never ║
║                                          ║ in default dmesg; the observable is  ║
║                                          ║ the udev write error, not a log line ║
║ README claim "forces max and rejects any ║ CORRECT ON BOTH HALVES. No edit      ║
║ other value" (L216)                      ║ needed                               ║
║ dynamic_epp availability                 ║ MASTER-ONLY at the time of check     ║
║                                          ║ (0 refs in 6.15/6.16/6.18/6.19/7.0). ║
║                                          ║ Script L2745 comment "ships since    ║
║                                          ║ 7.1" is the corrected form;          ║
║                                          ║ _chk_sysfs_eq is silent on a missing ║
║                                          ║ path, so the probe is harmless.      ║
║                                          ║ CachyOS backport status UNCHECKED    ║
║ pcie_aspm.policy=performance semantics   ║ DISABLES. drivers/pci/pcie/Kconfig   ║
║                                          ║ PCIEASPM_PERFORMANCE: disable L0s    ║
║                                          ║ and L1 even if the BIOS enabled      ║
║                                          ║ them. Cite the Kconfig — the         ║
║                                          ║ "since 4.2" figure on kernelconfig   ║
║                                          ║ .io is a database-floor artifact     ║
║ pcie_aspm=off semantics                  ║ Leave firmware config untouched —    ║
║                                          ║ does NOT disable ASPM. Confirmed in  ║
║                                          ║ master kernel-parameters             ║
║ PROTON_FSR4_UPGRADE currency             ║ CORRECT NAME. proton-cachyos         ║
║                                          ║ "Version 11.0-20260702": DLL copied  ║
║                                          ║ automatically, variable now version- ║
║                                          ║ pins (4.0.0/4.1.1 only).             ║
║                                          ║ PROTON_FSR4_RDNA3_UPGRADE removed;   ║
║                                          ║ PROTON_FSR3_UPGRADE renamed          ║
║                                          ║ PROTON_FFX3_UPGRADE; the DXIL wmma   ║
║                                          ║ workaround is no longer required in  ║
║                                          ║ most cases, WITH an MLFG-on-RDNA3    ║
║                                          ║ exception                            ║
║ POWERDEVIL_NO_DDCUTIL=1                  ║ EXACT VARIABLE PowerDevil reads      ║
║                                          ║ (KDE powerdevil README, master +     ║
║                                          ║ Plasma/6.4). Not a silent no-op      ║
║ ntsync currency                          ║ CONFIG_NTSYNC=m in linux-cachyos     ║
║                                          ║ master (module, not builtin).        ║
║                                          ║ PROTON_USE_NTSYNC removed — ntsync   ║
║                                          ║ is the default even in Valve's       ║
║                                          ║ Proton 11; PROTON_NO_NTSYNC is the   ║
║                                          ║ opt-out. Profile neither sets nor    ║
║                                          ║ checks it, correctly                 ║
║ RADV descriptor heap                     ║ DEFAULT-ON in Mesa main              ║
║                                          ║ (RADV_DEBUG=noheap opt-out,          ║
║                                          ║ experimental flag gone); Mesa 26.1   ║
║                                          ║ still needs RADV_EXPERIMENTAL=heap   ║
║ nmi_watchdog vendor interaction          ║ CachyOS 70-cachyos-settings.conf     ║
║                                          ║ ships page-cluster=0,                ║
║                                          ║ vfs_cache_pressure=50,               ║
║                                          ║ nmi_watchdog=0, swappiness 100;      ║
║                                          ║ 30-zram.rules sets swappiness 150 +  ║
║                                          ║ zswap N on zram activation. Priority ║
║                                          ║ 95 loads after 70 — correct ORDERING,║
║                                          ║ not merely presence                  ║
║ NM vs resolved DNS precedence            ║ systemd #33973 closed-completed —    ║
║                                          ║ per-link DHCP DNS outranks global    ║
║                                          ║ DNS=. Domains=~. is NOT the fix      ║
║                                          ║ (#33579, still open: dual-query      ║
║                                          ║ leak). NM [global-dns-domain-*] is   ║
║                                          ║ the correct mechanism and ships      ║
║ MES firmware timeline (gfx1151 hang)     ║ 2025-11-19 update → 2025-12-01 REVERT║
║                                          ║ → 2026-02-25 re-land (=0x86) →       ║
║                                          ║ 2026-05-07 further update. First tag ║
║                                          ║ shipping 0x86 = 20260309.            ║
║                                          ║ 0x7f is the lr_compute_wa dmesg gate.║
║                                          ║ Given the revert precedent, advise   ║
║                                          ║ "newest tag", not a minimum          ║
║ ROCm #6165                               ║ CLOSED (an earlier research pass     ║
║                                          ║ reported it open)                    ║
║ RTL8127 in r8169                         ║ Present at v6.16, absent at v6.15    ║
║ MangoHud #1794                           ║ STILL OPEN — cpu_power reads 0 when  ║
║                                          ║ cpu_temp is active on Zen 5          ║
║ fish set -l block scoping                ║ Documented and stable — "erased when ║
║                                          ║ the block ends"; block scoping dates ║
║                                          ║ to fish 3.3, well below the 3.6 floor║
```

**Known-stale sources — do not cite:** `wireless.docs.kernel.org` for `pcie_aspm`
semantics (carries pre-6.9 text); kernelconfig.io for Kconfig introduction versions.

---

## 4. Security posture — quantify only, ordered by exposure

No auto-FIX in this section. Six deltas remain after the remote-play removal.

1. **UMIP off** (`clearcpuid=umip`) — descriptor-table base leak, kernel tainted since
   5.19, upstream says not for production. Headline open reduction (T3-2).
2. **AMD-Vi fully disabled** (`amd_iommu=off`) — no DMA isolation or remapping; USB4/TB,
   NVMe and NIC DMA unmediated. Named casualty: the XDNA 2 NPU, blacklisted. Opting back in
   is one validator-enforced pair (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`),
   restoring isolation and the NPU together. The coupling asymmetry is intentional:
   `amd_iommu=on` with blacklist true is valid; blacklist false without the IOMMU refuses
   to deploy.
3. **Plaintext DNS with no validation** — observable and spoofable on the wire, no
   authentication of answers. State the exposure and stop; the decision is closed (T5).
4. **IPv6 disabled + inbound IPv4 ping accepted** — net LAN delta is `+ping −mDNS`. Avahi
   is masked (unit *and* socket) and resolved has `MulticastDNS=no`, so multicast discovery
   is fully closed. Ping-accept is an asserted regression guard, not a defect.
5. **`split_lock_detect=off`** — a misbehaving application can degrade the system.
6. **ufw masked rather than removed** — the package stays installed and could be unmasked
   and started, at which point two firewall managers contend for the same netfilter tables.
   Quantify that against the benefit (reversibility, no package churn); the nftables-first
   gate (T4-4) is the only thing standing between a mis-sequenced run and an unfirewalled
   window.
7. **No sleep path** — all five sleep/suspend/hibernate targets masked. An always-on box
   never gets the "locked on resume" checkpoint. Deliberate for a headless-adjacent mini-PC.
8. **Historical, not live: the `.ry.orig` dead-code window.** From 7.109.0 through 7.135.0
   the first-adoption preserve never executed, so 13 of 17 destinations were overwritten
   with **no backup of any kind** — no `.ry.orig`, no `.ry.bak`. Security-relevant members
   of that set: `/etc/nftables.conf` and `/etc/NetworkManager/conf.d/99-cachyos-nm.conf`.
   Fixed at 7.135.1 and void for a host first deployed at ≥7.135.1. **There is no forensic
   trace** — the JSONL never emitted `PREEXISTING_PRESERVED`, so absence of the key does not
   distinguish "nothing to preserve" from "preserve skipped". T0-6 confirms the window on a
   given host. **The generalizable lesson: presence of a guard in source is not evidence
   the guard runs.** Any claim of the form "X is protected because the code does Y" needs a
   behavioural probe, not a read.

**Default-deny-inbound ships and is net positive.** Residual notes: `flush ruleset` blast
radius against docker/libvirt/podman; no ICMP or new-connection rate limit (trusted-LAN
assumption — state it).

---

## 5. Verify block

Grouped to match the tiers. T0 items first.

**T0-1 — idle-floor measurement (the headline; idle desktop, 60 s, before and after any
perf change):**

```fish
sudo turbostat --quiet --show PkgWatt,Busy%,Bzy_MHz,PkgTmp --interval 5 --num_iterations 12
cat /sys/class/drm/card*/device/hwmon/hwmon*/power1_average   # GPU package draw at idle
rg . /sys/devices/system/cpu/cpu*/cpuidle/state*/name         # C-state depth under max_cstate=1
```

**T0-2 … T0-6 — the remaining observations:**

```fish
sudo lspci -vv | rg 'LnkCtl:.*ASPM'                     # T0-2 per-link ASPM state
cat /sys/module/mt7925e/parameters/disable_aspm         # Y/1 — endpoint option live
findmnt -no FSTYPE,OPTIONS /                            # T0-3 root FS before costing fsck.mode
findmnt -no OPTIONS -t ext4                             # noatime,lazytime,commit=10
cat /sys/class/hwmon/hwmon*/name | rg -c '^k10temp$'    # T0-4 sensor presence
sensors 2>/dev/null | rg -i -A1 '^k10temp'              # does Tctl populate?
stat -c '%a' /sys/class/powercap/*/energy_uj            # world-readable? gates cpu_power
pacman -Q linux-firmware mesa                           # T0-5 advisory only, no gate exists
ls -l /etc/*.ry.orig /etc/**/*.ry.orig ~/.config/**/*.ry.orig 2>/dev/null   # T0-6
ls -l /etc/kernel/cmdline.ry.bak /etc/sdboot-manage.conf.ry.bak \
      /boot/loader/loader.conf.ry.bak /etc/mkinitcpio.conf.ry.bak 2>/dev/null
rg -c 'PREEXISTING_PRESERVED|PREEXISTING_PRESERVE_FAIL' ~/ry-install/*.jsonl   # 0 = window confirmed
rg -c 'MODPROBE_STALE_DROPIN|MASK_ORPHAN' ~/ry-install/*.jsonl                 # ≥7.135.0 logs only
```

**CPU / GPU state (T2/T3 baseline):**

```fish
cat /proc/cmdline
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver                  # amd-pstate-epp
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor                # performance
cat /sys/devices/system/cpu/cpu0/cpufreq/energy_performance_preference   # performance
cat /sys/devices/system/cpu/amd_pstate/status                            # active
cat /sys/devices/system/cpu/amd_pstate/prefcore                          # enabled
cat /sys/devices/system/cpu/amd_pstate/dynamic_epp                       # disabled (may be absent)
cat /sys/devices/system/cpu/cpufreq/boost                                # 1
cat /sys/class/drm/card*/device/power_dpm_force_performance_level        # high
cat /sys/block/nvme0n1/queue/scheduler                                   # [none] (adjust node)
```

**Cmdline tokens and their removals:**

```fish
rg -o 'amd_iommu=\S+|ipv6\.disable=\S+|clearcpuid=\S+|pcie_aspm\S*|mt7925e\S*' /proc/cmdline
rg -o 'processor.max_cstate=\S+|fsck\S+' /proc/cmdline
rg -c 'nowatchdog|tsc=reliable|8250' /proc/cmdline    # 0 — the three removals still hold
rg -c 'preempt=' /proc/cmdline                        # 0 — never pinned; advisory removed 7.139.0
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l   # 0 — INFORMATIONAL
sudo dmesg | rg -i 'AMD-Vi|DMAR'                      # expect NO "AMD-Vi: Enabled"
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
                     hybrid-sleep.target suspend-then-hibernate.target   # masked ×5
systemctl list-unit-files --state=masked,masked-runtime --no-legend --plain  # compare vs MASK 11
systemctl is-enabled avahi-daemon.service avahi-daemon.socket bluetooth.service
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
grep -c '^[a-z]' ~/.config/MangoHud/MangoHud.conf         # 19 active
```

**`rg -c` exits 1 with no output on zero matches.** Any zero-hit assertion above must be
read as "exit 1 or `0`", not as a failure. Sandbox rg 14.1.0 additionally carries a
false-negative bug — re-confirm zero-hit results with `grep -P` or `python3`.

---

## 6. Reference data

All values live-evaluated from the 7.139.0 script.

### 6a. Count oracle — 21 tripwires, asserted by `_ir_validate_counts` (L665)

```
║ KERNEL_PARAMS            15 ║ PKGS_ADD                 16 ║ _RY_BOOT_CRITICAL_DSTS  4 ║
║ MKINITCPIO_HOOKS         11 ║ PKGS_DEL                  9 ║ _RY_PHASE_NAMES         6 ║
║ MKINITCPIO_MODULES        1 ║ MASK                     11 ║ _RY_BACKUP_TARGETS      4 ║
║ MKINITCPIO_COMPRESSION_   2 ║ EXPECTED_VULKAN_PKGS      2 ║ _RY_TMPDIR_GLOBS        6 ║
║   OPTIONS                   ║ EXPECTED_SERVICES         5 ║ SYSTEM_DESTINATIONS    15 ║
║ LOGIND_IGNORE_KEYS        8 ║ _RY_PKG_MANAGED_SERVICES  1 ║ USER_DESTINATIONS       2 ║
║ ENV_VARS                 10 ║ _RY_POST_HOOKS           17 ║ _RY_ARGPARSE_SPEC       6 ║
║ SYSCTL_VALUES            11 ║                             ║                           ║
```

`RESOLVED_DNS_SERVERS` (2) is deliberately **not** in the oracle — it is value-bearing,
not a count invariant. Managed files = 17 (15 system + 2 user), recomputed at load;
mismatch refuses with exit 3. **Count these by live fish eval, never by text parsing.**

### 6b. Perf scalars and their maxima

```
║ GLOBAL                  ║ L   ║ VALUE          ║ CEILING                          ║
║─────────────────────────║─────║────────────────║──────────────────────────────────║
║ CPUPOWER_GOVERNOR       ║ 586 ║ performance    ║ AT MAX (regex ^[a-z][a-z0-9_-]*$)║
║ GPU_DPM_LEVEL           ║ 588 ║ high           ║ CLOSED at high (T5), not max     ║
║ EPP_PREFERENCE          ║ 590 ║ performance    ║ AT MAX (_RY_EPP_LEVELS, 5)       ║
║ EXPECTED_SCALING_DRIVER ║ 591 ║ amd-pstate-epp ║ NO MAX — verify-only, tunes      ║
║                         ║     ║                ║ NOTHING; follows a cmdline       ║
║                         ║     ║                ║ amd_pstate= change, never leads  ║
```

Changing `EXPECTED_SCALING_DRIVER` makes the verifier assert a driver the kernel never
loaded, producing a false `_chk_eq` failure on every `--verify`. Domain validation lives in
`_ir_validate_keys` (L693): DPM at L705, EPP at L706, governor regex at L707. Negative
tests return rc **3** (`EXIT_PREFLIGHT`), and `_err_loud` **exits** rather than returns —
each negative case needs its own subprocess.

**A full perf-value change touches 13 sites. Enumerate all of them in any TUNE:**

```
║ #  ║ FILE      ║ LINE ║ CARRIES                                              ║
║────║───────────║──────║──────────────────────────────────────────────────────║
║ 1  ║ script    ║ 586  ║ set -g CPUPOWER_GOVERNOR performance                 ║
║ 2  ║ script    ║ 588  ║ set -g GPU_DPM_LEVEL high + trailing comment         ║
║ 3  ║ script    ║ 590  ║ set -g EPP_PREFERENCE performance (+ _RY_EPP_LEVELS) ║
║ 4  ║ script    ║ 874  ║ udev generator --description names "EPP performance" ║
║ 5  ║ script    ║ 879  ║ "# AMD P-State EPP performance (maximum CPPC hint)"  ║
║ 6  ║ script    ║ 881  ║ "# GPU performance level (gfx1151 clock-floor;       ║
║    ║           ║      ║ forced high)"                                        ║
║ 7  ║ README    ║ 100  ║ managed-files row: governor (`performance`)          ║
║ 8  ║ README    ║ 102  ║ managed-files row: GPU DPM level `high`              ║
║ 9  ║ README    ║ 205  ║ Service Keys row: CPUPOWER_GOVERNOR | performance    ║
║ 10 ║ README    ║ 209  ║ Service Keys row: GPU_DPM_LEVEL | high               ║
║ 11 ║ README    ║ 210  ║ Service Keys row: EPP_PREFERENCE | performance       ║
║ 12 ║ README    ║ 216  ║ CPU/GPU prose (EPP-restates-governor + the accepted- ║
║    ║           ║      ║ value-list sentence, now one paragraph)              ║
║ 13 ║ CHANGELOG ║ 5+   ║ new entry inserted after the preamble                ║
```

**Grep traps:** `powersave` has 4 hits but only L586 is in scope — the rest are
`NM_WIFI_POWERSAVE` plumbing. `balance_performance` has exactly 1 hit, the surviving
`_RY_EPP_LEVELS` member. Do not edit the phrase "clock-floor" — it is accurate under
`high` and consistent across the generator and verify paths. `grep -ci 'audit\|spec'`
reports false hits from `_RY_ARGPARSE_SPEC` and from "inspect"/"inspection"; real audit
refs are **0** in all shipped files.

### 6c. Configured values

**KERNEL_PARAMS (15, sorted as emitted):** `amd_iommu=off` `amd_pstate=active`
`btusb.enable_autosuspend=n` `clearcpuid=umip` `fsck.mode=force` `fsck.repair=yes`
`ipv6.disable=1` `mt7925e.disable_aspm=1` `nvme_core.default_ps_max_latency_us=0`
`pcie_aspm.policy=performance` `processor.max_cstate=1` `quiet` `split_lock_detect=off`
`usbcore.autosuspend=-1` `zswap.enabled=0`. **`amd_pstate=active` is the root of the whole
CPU story** — governor and EPP are downstream of it and meaningless without it.

**ENV_VARS (10, `~/.config/environment.d/10-environment.conf`, 0600):** `DXVK_LOG_LEVEL=none`
`MANGOHUD=1` `MESA_SHADER_CACHE_MAX_SIZE=16G` `POWERDEVIL_NO_DDCUTIL=1`
`PROTON_ENABLE_WAYLAND=1` `PROTON_FSR4_UPGRADE=1` `PROTON_LOCAL_SHADER_CACHE=1`
`VKD3D_DEBUG=none` `VKD3D_SHADER_DEBUG=none` `WINEDEBUG=-all`. No drirc, no ttm/amdgpu
module params, no ICD pin. `_vre_envvars` iterates the array dynamically, so the verifier
follows any ENV_VARS edit with no verifier change.

**SYSCTL_VALUES (11, `/etc/sysctl.d/95-ry-overrides.conf`, priority 95):**
`kernel.nmi_watchdog=0` `net.core.default_qdisc=fq` `net.core.netdev_budget=600`
`net.core.netdev_budget_usecs=5000` `net.ipv4.tcp_congestion_control=bbr`
`net.ipv4.tcp_notsent_lowat=16384` `net.ipv4.tcp_slow_start_after_idle=0`
`vm.compaction_proactiveness=0` `vm.max_map_count=2147483642` `vm.swappiness=150`
`vm.watermark_boost_factor=0`. Stored `k=v`, emitted `k = v` — any parity check must
normalise whitespace. The annotation comment is **selective by design** (3 of 11 keys) at
64 characters against a ≤66 cap; do not propose annotating more.

**MASK (11, L610):** `ananicy-cpp.service` `power-profiles-daemon.service`
`NetworkManager-wait-online.service` `avahi-daemon.service` `avahi-daemon.socket`
`ufw.service` `sleep.target` `suspend.target` `hibernate.target` `hybrid-sleep.target`
`suspend-then-hibernate.target`. **EXPECTED_SERVICES (5):** `fstrim.timer`
`NetworkManager.service` `cpupower.service` `nftables.service` `bluetooth.service`.

**PKGS_ADD (16):** nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa,
mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, lm_sensors, rtkit,
realtime-privileges, nftables, pacman-contrib. **PKGS_DEL (9, `-Rns`, rdep-aware via
pactree):** plymouth, cachyos-plymouth-bootanimation, cachyos-plymouth-theme,
breeze-plymouth, plymouth-kcm, micro, cachyos-micro-settings, cachy-update, kdeconnect.
**AUR: none.** Vulkan via chwd, verify-only: vulkan-radeon + lib32-vulkan-radeon.

**MKINITCPIO:** `MODULES=(amdgpu)` (early KMS); HOOKS (11) base systemd autodetect
microcode modconf kms keyboard sd-vconsole block filesystems fsck; COMPRESSION zstd,
`COMPRESSION_OPTIONS=(-1 -T0)`; explicit `BINARIES=()` / `FILES=()`.

**Nineteen service keys** are documented in the profile README. Only two reach their
destination under their own name (`COUNTRY`; `LOGIND_IGNORE_KEYS` → 8 `Handle*Key=ignore`
lines). The other seventeen are **renamed on the way out** — which is why grepping a
generated file for the script's variable name returns nothing.

### 6d. Generated-file byte anchors

```
║ FILE                                              ║ BYTES ║
║───────────────────────────────────────────────────║───────║
║ /boot/loader/loader.conf                          ║    89 ║
║ /etc/kernel/cmdline                               ║   352 ║
║ /etc/sdboot-manage.conf                           ║   544 ║
║ /etc/mkinitcpio.conf                              ║   280 ║
║ /etc/systemd/resolved.conf.d/99-cachyos-resolved  ║   154 ║
║ /etc/systemd/logind.conf.d/99-cachyos-logind      ║   292 ║
║ NetworkManager-dispatcher.service.d/logging.conf  ║   127 ║
║ /etc/NetworkManager/conf.d/99-cachyos-nm.conf     ║   219 ║
║ /etc/iw-regdomain                                 ║    88 ║
║ /etc/bluetooth/main.conf                          ║   147 ║
║ /etc/nftables.conf                                ║   729 ║
║ /etc/default/cpupower-service.conf                ║   115 ║
║ /etc/sysctl.d/95-ry-overrides.conf                ║   441 ║
║ /etc/udev/rules.d/99-ry-perf.rules                ║   639 ║
║ /etc/modprobe.d/60-ry-modules.conf                ║   183 ║
║ ~/.config/environment.d/10-environment.conf       ║   311 ║
║ ~/.config/MangoHud/MangoHud.conf                  ║   383 ║
║───────────────────────────────────────────────────║───────║
║ TOTAL (17 managed files)                          ║ 5,093 ║
```

The udev rule at **639 B** and the **5,093 B** total are the anchors for any perf-value
change — a value substitution plus its comment edits moves both. Measured deltas from the
last change (7.128.0, powersave/auto/balance_performance → max): udev 657→639 (−18),
cpupower 113→115 (+2), total 5,109→5,093 (−16). The `profile_peak` variant measured
udev 647 / total 5,101 — recorded so it is never re-measured.

The two bodies worth reproducing in full:

**`/etc/kernel/cmdline`** (UUID elided; the real render is 352 B with a 36-char UUID):

```
rw root=UUID=<_ROOT_UUID> amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0
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

The other 15 bodies are reproducible byte-exactly from the harness in §8; they are omitted
here rather than duplicated.

### 6e. Removal reconciliation — state the tier for any removal you recommend

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
║ 3    ║ package dropped from         ║ stays installed    ║ NOT DETECTED —       ║
║      ║ PKGS_ADD                     ║ forever            ║ by design (T5)       ║
```

Tier 2 is reportable but never self-healing and never sets DRIFT — the operator must act by
hand. Tier 3 is invisible.

### 6f. Gates, exits, backups

- **Preflight validator chain, four deep, called from `_init_runtime` at L779-782:**
  `_ir_validate_counts` (L665, 21 tripwires) → `_ir_validate_keys` (L693, domains and
  charsets, incl. the nftables↔`ipv6.disable` coupling and the
  `BLACKLIST_AMDXDNA=false`↔IOMMU-on coupling) → `_ir_validate_sets` (L734) →
  `_ir_validate_post_hooks` (L4575). **There is no `_ry_validate_keys`,
  `_ry_validate_counts` or `_ry_validate_post_hooks`** — those names return rc 127, which
  looks like a signal and is only an unknown command.
- **`_ir_validate_sets` refuses three intersections**, each `_err_loud` + exit 3:
  `PKGS_ADD ∩ PKGS_DEL` (phase 2 installs what phase 4 removes), `EXPECTED_SERVICES ∩ MASK`
  (phase 4 masks before it enables), `_RY_PKG_MANAGED_SERVICES ∩ MASK` (a masked unit cannot
  be package-managed). All three shipped intersections are empty. **Any recommendation that
  adds a package or unit must be checked against all three — a contradiction is now a hard
  deploy refusal, not a silent misbehaviour.**
- **Hardware gate:** CPU model match on `Ryzen AI Max`; sole override
  `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed on an unreadable model; `--verify` warns,
  deploy and `--check` exit 3.
- **Runtime env inputs are exactly three:** `RY_RUN_TIMEOUT`,
  `RY_INSTALL_SKIP_HARDWARE_CHECK`, `NO_COLOR`. Every profile toggle is an embedded scalar
  set unconditionally with `set -g`, so an exported env var of the same name is clobbered —
  opting in means editing the script. `NO_COLOR` needs a **non-empty** value to fire;
  `TERM=dumb` disables colour independently.
- **Dependencies:** 37 hard-required commands (missing any → rc 1) + **19** warn-only
  optional tools, plus a `df --output` probe and systemd ≥ 250.
- **Exit model, 14 constants:** 0 OK · 1 FAIL · 2 USAGE · 3 PREFLIGHT · 4 BOOT_CRIT ·
  5 LOCK · 10 DRIFT; internal sentinels 11 GEN_NOFN, 12 GEN_NOUUID, 13 GEN_SYSCTL,
  14 GEN_ENVD, 250 AS_MISUSE, 251 RUN_TMPFAIL, 255 RUN_MISUSE — function returns only,
  never a process exit. Signals map to 128+N. **Neither orphan class reaches EXIT_DRIFT.**
  Root `--check` → rc 3, silent, no JSONL (the root guard precedes LOG_DIR init); all other
  root modes → rc 2, loud, 92 B.
- **Backups are two different guarantees.** The 4 boot-critical destinations
  (`/boot/loader/loader.conf`, `/etc/kernel/cmdline`, `/etc/sdboot-manage.conf`,
  `/etc/mkinitcpio.conf`) get `.ry.bak` **plus post-write re-read and restore**. The other
  13 get a one-time `.ry.orig` on **first adoption only** — captured once, then every
  subsequent overwrite is silent by design. `/etc/fstab` is not a managed destination and
  has always had its own `.ry.bak` during rewrite. **Do not describe the 13 as backed up on
  every write, and do not recommend `.ry.orig` for the boot four — it is weaker than what
  they have.** No verify path asserts `.ry.orig` exists and none can: absence is
  indistinguishable from "the destination had no pre-existing content", which is the common
  case.

---

## 7. Verify-surface ownership

**61 verify functions = 12 orchestrators + 49 subs.** A recommendation that changes a value
must state which **sub** asserts it, and whether that sub hard-fails or warns.

```
║ ORCHESTRATOR             ║ SUBS THAT MATTER FOR TUNING FINDINGS             ║
║──────────────────────────║──────────────────────────────────────────────────║
║ _verify_static_boot      ║ _vsb_loader · _vsb_sdboot · _vsb_cmdline (all 15 ║
║                          ║ tokens byte-asserted) · _vsb_mkinitcpio (live    ║
║                          ║ COMPRESSION/_OPTIONS, multi-line join, last-wins)║
║                          ║ · _vsb_entries → _vsb_entry_options (NEW)        ║
║ _verify_static_system    ║ _vss_logind · _vss_nm · _vss_nmdispatch ·        ║
║                          ║ _vss_sysctl (11) · _vss_regdom · _vss_bluetooth ·║
║                          ║ _vss_udev (3 rules; EPP + DPM aware) ·           ║
║                          ║ _vss_modprobe (blacklist + stale-dropin sweep,   ║
║                          ║ WARN) · _vss_nft (HARD-FAILS on missing          ║
║                          ║ echo-request)                                    ║
║ _verify_static_services  ║ MASK 11 + _vss_orphan_masks (INFO, admin-scope)  ║
║ _verify_static_packages  ║ _vsp_required (PKGS_ADD 16 + Vulkan) ·           ║
║                          ║ _vsp_removed · _vsp_pacman_conf                  ║
║ _verify_runtime_kparams  ║ _vrk_cmdline (every token + rw) · _vrk_gpu_state ║
║                          ║ (QUOTED compare vs $GPU_DPM_LEVEL) ·             ║
║                          ║ _vrk_cpu_state (cpu0 detail + FULL policy sweep) ║
║                          ║ · _vrkm_blacklist_modprobe (amdxdna LOADED =     ║
║                          ║ FAIL) · usbcore / nvme_core / zswap /            ║
║                          ║ nmi_watchdog / NVMe-none                         ║
║ _verify_runtime_services ║ _vrsv_sys_units · _vrsv_masked_inactive          ║
║                          ║ (iterates $MASK) · _vrsv_chk_resolved (NEW:      ║
║                          ║ enabled|static) · _vrsv_chk_nftables (judged by  ║
║                          ║ live policy-drop, not unit state) ·              ║
║                          ║ _vrsv_nft_assert_ping (warn) · _vrsv_user_units  ║
║ _verify_runtime_env      ║ _vre_envvars (dynamic over $ENV_VARS) ·          ║
║                          ║ _vre_sysctl_runtime (11) · _vre_fstab · _vre_    ║
║                          ║ ntsync · _vre_regdom                             ║
║ _verify_runtime_session  ║ _vrs_nm_perms · _vrs_installed_file_perms ·      ║
║                          ║ _vrs_parent_dirs                                 ║
```

**Attribution traps:**

- `_vrkm_blacklist_modprobe` is **generator-sourced** — it checks intended content, not
  on-disk extras. Attribute the drop-in sweep to `_vss_modprobe` / `_ry_stale_ry_dropins`,
  never to it.
- `_vrsv_masked_inactive` asserts every *declared* mask is present and inactive. The
  reverse direction (masked units not declared) is `_vss_orphan_masks`, which lives on the
  **static** services path. Do not conflate the two.
- **Removed asserts — do NOT verify:** `_vrkm_iommu`, `_vrk_clocksource`, `_vre_zram`,
  `_vre_tcp` (all gone since 7.90.0); kernel-floor and Mesa-floor checks; the preemption
  advisory (7.139.0 r2). No THP, KSM, `ttm.*`, drirc, `iommu=pt`, ICMPv6/NDP or baloo
  assert exists. No `VKD3D_CONFIG` assert exists — the variable left ENV_VARS and
  `_vre_envvars` followed automatically.
- **Sandbox artifact, not a regression:** `_ry_validate_mkinitcpio_hooks` returns rc 1 and
  `_ry_validate_configs` returns rc 3 in a container because `/etc/mkinitcpio.conf` is
  absent. Always A/B a nonzero validator rc against the previous release.

**Output-channel invariant — reconciled; do not re-file as drift.** Every leveled
user-facing message funnels through `_msg_print` (**L1109**), honouring QUIET /
`_RY_OUTPUT_BROKEN` / `_RY_NO_COLOR` / isatty(2). Raw `>&2` counts of 78 (whole-file) and
43 (inside 17 function bodies) are **both correct under their own scoping**; the remaining
35 are top-level pre-init preflight writes made before `_msg_print` is defined. The
invariant means "single authority for leveled user-facing output", not "sole writer to
fd 2" — the latter reading generates a false finding every time. stdout carries only
`--help` and `--version`.

---

## 8. Reproduction method and its traps

Recorded so the next rebase does not re-pay these costs.

- **Harness.** Cut the script just before the `# ── MAIN: ARGPARSE` banner —
  **L4845 at 7.139.0**, L4834 at 7.135.1, L4786 at 7.125.0-7.130.0. **Always locate the
  banner, never hardcode.** Then delete the L3 source guard and `source` the result as a
  non-root user with a **writable `$HOME`**:
  `sed -n '1,4844p' ry-install.fish | sed '3d' > harness.fish`. Without the L3 deletion the
  guard fires on `source` and every count silently reads 0. Without a writable `$HOME` the
  log-directory init aborts the source part-way and counts read 0 the same way — a
  different cause with an identical symptom.
- **`test -w /tmp` at L161 bails rc 3 before anything else.** Any sandbox reproduction needs
  a non-root user *and* a writable `/tmp`.
- **Array counts by live fish eval, never text parsing.** `eval echo \$$name` collapses a
  fish array and reports every count as 1 — use `eval "set vals \$$name"` then
  `count $vals`. Continuation regexes truncate multi-line declarations, `set -g --` evades
  awk, and several service keys share one `set -g` line.
- **Function census by set difference.** `functions --all --names` before and after
  sourcing; the difference is exactly the script's own functions (**293** here).
  `functions --names` alone hides underscore-prefixed names. Hand-written depth parsers
  desync on this script's 21 one-line `function …; …; end` definitions — that is also why
  the 293 `function` keywords pair with only 272 top-level `end`s.
- **Filter generators to `_content__*` + `_content_HOME*`.** A bare `_content_*` glob also
  catches the `_content_fn_for` dispatcher.
- **Set `_ROOT_UUID`** (single underscore, not `_RY_ROOT_UUID`) or the cmdline generator
  returns 12 / `GEN_NOUUID` and `_ry_validate_configs` returns rc 3, which reads as a
  regression.
- **Generated bytes must be measured as WRITTEN FILES** (`$fn > tmp; stat -c %s`), never
  `(cmd | string collect)` — collect strips each trailing newline and the 17-file total
  reads 5,076 B instead of 5,093 B, a phantom 17 B deficit that looks like anchor drift.
- **Determinism must be compared per-file by content hash, or with a SORTED manifest.** A
  concatenated hash over the output directory is not comparable between passes because the
  glob order depends on harness filenames.
- **Verify every "before" column against the OLD EDITION'S RENDERED BODY, not memory.** The
  old script is not in the archive; only its brief is. A drift row asserting a change that
  never happened passes every count check and every byte check — only an old-vs-new body
  diff catches it. Extract the previous edition's fences with a **toggle-based** fence
  walker (a naive `re.findall` on ``` consumes alternating pairs and under-reports).
  **This is exactly how the B10 defect in §1c was found.**
- **Fence-aware heading checks are mandatory.** A naive `^# ` scan false-reports h1
  violations on the `#` comments inside the nft and config fences.
- **Byte-vs-character length.** Banner and line-length checks must count CHARACTERS; `awk
  length` under a C locale counts bytes and falsely reports the U+2500/U+2192 box-drawing
  content over any character cap.
- **Fish function names contain dots** (`_content_HOME_.config_MangoHud_MangoHud.conf`), so
  a `[A-Za-z0-9_-]*` charset truncates them and fabricates duplicates. Match `\S+` after
  `function`.
- **Verify the upload before trusting it.** A fresh upload can be behind what is recorded as
  shipped. Hash the archive against §0 first. Two collision precedents exist: 7.130.0 (two
  artifacts, distinguishable only by CHANGELOG) and 7.139.0 (three artifacts —
  first cut / r2 / r3, distinguishable by zip or script hash).
- **Sandbox limits.** Target host paths do not exist, so only sudo-fail and preflight paths
  are exercisable and a full install cannot complete by design. `_err_loud` **exits** rather
  than returns — run each negative test in its own subprocess.

---

## 9. Scope and output contract

**In scope:** recommendations only — do not emit a modified script. Hardware-anchored to
gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE.

**Out of scope:** dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS,
laptops, UKI, BIOS flashing. Per-game Proton tuning is secondary to system-wide config.

**Rules:**

1. Respect deliberate trade-offs — flag and quantify, do not auto-FIX. Reserve FIX for
   incorrect, superseded, deprecated, or harmful values.
2. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is
   non-trivial.
3. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN.
4. Flag every source conflict and name the trusted side.
5. Give exact versions (kernel / Mesa / linux-firmware / vkd3d-proton / package) and exact
   before→after, mapped to the in-script global.
6. **Do not carry values forward from any pre-7.139.0 edition.** A recommendation that
   re-derives a removed token, or that asserts a value *changed* in this window, is a
   stale-source error rather than a finding.
7. A question closed by a code change is closed. A question closed by upstream evidence
   (§3) is closed. Cite them as settled and evaluate their *design*, not their existence.

**Required output:**

- **Findings matrix** (box-drawn, code-fenced, grouped by tier): ITEM · CURRENT · CALL
  (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE.
- **Before→after** for each TUNE/FIX, naming the in-script global and — for any perf
  value — all 13 sites from §6b.
- **Tier placement** for every recommendation, and the removal tier (§6e) for any removal.
- **Security delta** (§4, ordered).
- **Verdict** per tier plus overall (PASS / PASS-WITH-FIXES / FAIL).
- **Methodology:** source list with access dates and versions; unknowns marked UNCERTAIN.

---

## Sources

docs.kernel.org (kernel-parameters, PCIe/ASPM, amd-pstate, sysctl/vm, sysctl/kernel,
networking, block, ext4, UMIP, AMD-Vi, accel/amdxdna, amdgpu
`power_dpm_force_performance_level`) · git.kernel.org (`drivers/pci/pcie/Kconfig`,
`drivers/cpufreq/amd-pstate.c`, linux-firmware `gc_11_5_1_mes_2.bin` history, r8169, mt76,
wireless-regdb) · gitlab.freedesktop.org (mesa, drm/amd) · docs.mesa3d.org ·
github.com (CachyOS/proton-cachyos releases, CachyOS/linux-cachyos, HansKristian-Work/
vkd3d-proton, flightlessmango/MangoHud #1794, systemd/systemd #33973 + #33579) ·
invent.kde.org (powerdevil) · wiki.archlinux.org (AMDGPU, IOMMU, fsck, Gaming, PipeWire,
Zram, SSD, Ext4, Btrfs, Sysctl, NetworkManager, Wireless, nftables,
Uncomplicated_Firewall, Mkinitcpio, systemd-boot, Bluetooth, System_time,
CPU_frequency_scaling, MangoHud, pacman) · wiki.cachyos.org · discuss.cachyos.org ·
man.archlinux.org (nft, avahi-daemon, systemd.unit, logind.conf, resolved.conf,
NetworkManager.conf, hwclock, timesyncd) · fishshell.com/docs (variable scope) ·
strixhalo.wiki (Power Modes, C-States) · es.net (100 G host tuning) · amd.com ROCm ·
mangohud-gtr9-pro v1.17.0 companion archive.

**Do not cite** wireless.docs.kernel.org for `pcie_aspm` semantics — it carries stale
pre-6.9 text. Cite access dates and exact versions in the methodology block.
