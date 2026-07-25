# cachyos-tuning-audit — ry-install Tuning (CachyOS · Beelink GTR9 Pro)

**Target:** `ry-install.fish` **v7.135.1** (attached, 2026-07-25; 4963 lines, 292 functions, `fish --no-execute` clean; sha256 `509c41bc`).
**Source of truth:** script > README > CHANGELOG. Every value below is re-derived from the v7.135.1 script by live evaluation (truncated-execution harness, cut at the `MAIN: ARGPARSE` banner, **L4834** at this release); where any other source disagrees, the script wins and the disagreement must be flagged.
**Platform:** Beelink GTR9 Pro · Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤96 GB as VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP (README BIOS ceiling 85 W) · CachyOS · systemd-boot.
**Companion source:** `mangohud-gtr9-pro` **v1.17.0** (2026-07-04; MIT) — standalone publication of the embedded HUD config; installer is source of truth (repo CHANGELOG 1.14.0). Cross-audited in §12 / B9; HUD-scoped floors in §12.

Findings closed in-script at ≤7.135.1 are excluded; do not re-open them. The Protected list (§14) is the do-not-recommend inventory.

> **Re-pin note (7.130.0 → 7.135.1).** This is a 5-version rebase across 7.131.1, 7.132.0-7.133.0, 7.134.0, 7.135.0 and 7.135.1. **Not one tuning value moved.** All 21 count tripwires, all 13 profile scalars and all 17 generated file bodies are byte-identical to the 7.130.0 pin — verified by executing every generator and diffing the previous edition's Appendix B fences against fresh output (18 of 20 fences matched exactly; the 2 misses are the byte table and the 3-line remote-play fragment, both correct by construction). Everything that changed is **structural, verification-surface, or robustness**. Two long-standing code gaps this brief has carried for four and two generations respectively are now **CLOSED in code**, one previously-silent data-loss defect was found and fixed, and one of this brief's own top research questions is now answered by the profile's own documentation. The full delta is tabulated immediately below.

---

## Δ. Baseline delta vs the 7.130.0 brief (read first)

Every row is a live-eval or byte-level difference between the two script versions, not a documentation edit.

**Part 1 — the tuning surface did not move.**

```
║ SURFACE                        ║ 7.130.0    ║ 7.135.1    ║ VERDICT              ║
║────────────────────────────────║────────────║────────────║──────────────────────║
║ 21 count tripwires (oracle)    ║ see §Hard  ║ identical  ║ NO DRIFT             ║
║ KERNEL_PARAMS                  ║ 15         ║ 15         ║ same 15 tokens       ║
║ ENV_VARS                       ║ 10         ║ 10         ║ same 10 vars         ║
║ SYSCTL_VALUES                  ║ 11         ║ 11         ║ same 11 keys         ║
║ MASK / PKGS_ADD / PKGS_DEL     ║ 11/16/9    ║ 11/16/9    ║ same members         ║
║ CPUPOWER_GOVERNOR              ║ performance║ performance║ unchanged            ║
║ EPP_PREFERENCE                 ║ performance║ performance║ unchanged            ║
║ GPU_DPM_LEVEL                  ║ high       ║ high       ║ unchanged            ║
║ EXPECTED_SCALING_DRIVER        ║ amd-pstate-║ amd-pstate-║ unchanged            ║
║                                ║ epp        ║ epp        ║                      ║
║ RESOLVED_DOT / _DNSSEC         ║ no / no    ║ no / no    ║ unchanged            ║
║ RESOLVED_DNS_SERVERS           ║ AdGuard x2 ║ AdGuard x2 ║ unchanged            ║
║ BLACKLIST_AMDXDNA              ║ true       ║ true       ║ unchanged            ║
║ RY_REMOTE_PLAY_PORTS           ║ false      ║ false      ║ unchanged            ║
║ 17 generated bodies            ║ 5,093 B    ║ 5,093 B    ║ BYTE-IDENTICAL       ║
║ udev 99-ry-perf.rules          ║ 639 B      ║ 639 B      ║ byte-identical       ║
║ /etc/kernel/cmdline            ║ 352 B      ║ 352 B      ║ byte-identical       ║
║ cpupower-service.conf          ║ 115 B      ║ 115 B      ║ byte-identical       ║
```

**Consequence for the research pass:** no §1-§13 recommendation may cite "changed since the last edition" as evidence. The tuning questions carried forward are carried forward *unchanged* — what moved is the machinery around them.

**Part 2 — what actually changed.**

```
║ ITEM                        ║ 7.130.0 (old brief)      ║ 7.135.1 (this target)              ║ WHERE     ║
║─────────────────────────────║──────────────────────────║────────────────────────────────────║───────────║
║ Script lines / sha          ║ 4915 / 59e387af          ║ 4963 / 509c41bc                    ║ head      ║
║ Script bytes                ║ 289,600                  ║ 293,399                            ║ head      ║
║ Harness cut line            ║ 4786                     ║ 4834 (locate, never hardcode)      ║ method    ║
║ Function count              ║ 287                      ║ 292 (+5, all audit-relevant)       ║ App. C    ║
║ Verify functions            ║ 59 (12 orch + 47 subs)   ║ 60 (12 orch + 48 subs)             ║ App. C    ║
║ `sub:` parent markers       ║ 91                       ║ 93                                 ║ App. C    ║
║ Preflight validators        ║ 3 (L770-772)             ║ 4 (L781-784) — _ir_validate_sets   ║ App. E    ║
║ §6 modprobe-leftover gap    ║ OPEN (4 generations)     ║ **CLOSED** — _ry_stale_ry_dropins  ║ §6/App.C  ║
║ §9 stale-mask gap           ║ OPEN                     ║ **CLOSED (detect)** — _ry_orphan_  ║ §9/App.C  ║
║                             ║                          ║ masked_units                       ║           ║
║ PKGS_ADD orphan             ║ OPEN, undocumented       ║ OPEN, DOCUMENTED not fixed         ║ §8/§14    ║
║ `.ry.orig` first-adoption   ║ present in source but    ║ **LIVE** — fixed at 7.135.1        ║ App. F1   ║
║   preserve                  ║ DEAD CODE (never ran)    ║ (13 non-boot destinations)         ║           ║
║ Backup coverage             ║ 4 of 17 destinations     ║ 17 of 17 (4 .ry.bak + 13 .ry.orig) ║ App. F1   ║
║ Set-contradiction gate      ║ absent                   ║ 3 intersections refused (exit 3)   ║ App. E    ║
║ P0 #2 (governor vs EPP)     ║ OPEN, load-bearing       ║ ANSWERED in-tree — now a VERIFY    ║ §2        ║
║ ASPM doc wording            ║ "disable ASPM + clock PM"║ "biases every link away from ASPM" ║ §5        ║
║ FSR4 doc wording            ║ (none)                   ║ "mainly pins a version"            ║ §1        ║
║ `_msg_print` definition     ║ L1105                    ║ L1117                              ║ App. F    ║
║ udev generator comments     ║ L870 / L875 / L877       ║ L882 / L887 / L889                 ║ §2/B4     ║
║ Perf-value change sites     ║ 10                       ║ **13** (README grew 3 → 6)         ║ §2        ║
║ Perf globals                ║ L587/589/591/592         ║ L587/589/591/592 (unmoved)         ║ §2        ║
║ `_ir_validate_keys`         ║ L695 (checks 707/708/709)║ L695 (checks 707/708/709) unmoved  ║ §2/App.E  ║
║ MASK declaration            ║ L612                     ║ L612 (unmoved)                     ║ §9        ║
║ Deps roster                 ║ 37 hard + 20 warn        ║ 37 hard + 20 warn (unchanged)      ║ App. E    ║
║ Exit-code constants         ║ 14                       ║ 14 (identical values)              ║ App. E    ║
```

**Unchanged and re-confirmed by live eval:** MKINITCPIO_HOOKS 11, MKINITCPIO_MODULES 1 (`amdgpu`), MKINITCPIO_COMPRESSION_OPTIONS 2 (`-1 -T0`), LOGIND_IGNORE_KEYS 8, EXPECTED_VULKAN_PKGS 2, EXPECTED_SERVICES 5, `_RY_PKG_MANAGED_SERVICES` 1, `_RY_POST_HOOKS` 17, `_RY_ARGPARSE_SPEC` 6, `_RY_BOOT_CRITICAL_DSTS` 4, `_RY_PHASE_NAMES` 6, `_RY_BACKUP_TARGETS` 4, `_RY_TMPDIR_GLOBS` 6, SYSTEM_DESTINATIONS 15, USER_DESTINATIONS 2, `_RY_MANAGED_FILE_COUNT` 17, and every scalar (`NM_WIFI_BACKEND=wpa_supplicant`, `NM_WIFI_POWERSAVE=2`, `COUNTRY=US`, `BT_AUTO_ENABLE=true`). Runtime contract re-smoked identical: root `--check` → rc 3 silent 0 B/0 B, root `--verify` → rc 2 / 92 B, `--version` → `v7.135.1`, `fish --no-execute` rc 0.

**CLOSED BY CODE CHANGE — do not re-ask these 7.130.0 P0 questions:**

```
║ OLD # ║ QUESTION                                   ║ HOW IT CLOSED                        ║
║───────║────────────────────────────────────────────║──────────────────────────────────────║
║ #15   ║ Modprobe-leftover gap still unguarded —    ║ _ry_stale_ry_dropins: one shared     ║
║       ║ pre-7.99 drop-ins have 0 in-script refs    ║ helper, 2 callers (verify WARN +     ║
║       ║ and 0 README refs                          ║ check JSONL). README documents it    ║
║ (§9)  ║ Stale-mask code gap — a unit dropped from  ║ _ry_orphan_masked_units diffs live   ║
║       ║ MASK stays masked, invisible to --verify   ║ masked set against $MASK; 2 callers  ║
```

**Two 7.130.0 items are re-framed rather than closed:**

- **Old P0 #2** (does `governor=performance` defeat EPP under `amd_pstate=active`?) is now answered by the profile's own README: the driver forces the EPP hint to its maximum and rejects any other value, so `EPP_PREFERENCE=performance` **restates** what the governor already imposes. That is a documentation claim, not upstream evidence — it is carried below as a **verification** item (new #2), not as an open design question. Do not repeat the 7.122.0-era claim that `powersave` is *required* to honor EPP; the profile has now explicitly retracted it.
- **The PKGS_ADD orphan** (a package dropped from `PKGS_ADD` stays installed) is the one leg of the removal asymmetry that is **still undetectable**, and it was deliberately not built: detection needs a persisted manifest of what earlier versions installed, and a new state file would add its own drift surface. It is now documented in the README rather than fixed. Treat it as a stated design position (§14), not an oversight.

---

## Mission

Deep-research brief: evaluate every configured value against current upstream for this exact silicon and return a prioritized, evidence-backed report targeting **gaming improvements, performance enhancements, and overall system speed gain**. The profile deliberately trades PCI passthrough, the XDNA NPU, power saving, IPv6, and DNS privacy for latency, throughput, and a simpler IPv4-only ruleset — confirm each trade is current and correct, surface anything superseded or harmful, and quantify safety deltas without second-guessing intentional design.

**The central question is unchanged from 7.130.0 and is now a full release-cycle old without an answer:** the profile is pinned at maximum on three perf axes simultaneously (governor `performance`, EPP `performance`, DPM `high`) while `processor.max_cstate=1` already blocks deep CPU idle, under an 85 W BIOS ceiling shared between CPU and GPU. Evaluate the *idle floor and thermal-headroom* consequence of that stacking, not just the load ceiling. Five releases have shipped since the posture change with no measurement taken; that measurement (§15) is the single highest-value action in this brief.

**The secondary question is new and belongs to robustness, not tuning:** every non-boot managed destination was overwritten without a backup from 7.109.0 through 7.135.0 because the first-adoption preserve was dead code. It is fixed, but any host deployed in that window has already lost whatever hand-made content those 13 files held, and no `.ry.orig` exists to recover from. Assess the residual exposure and whether the fixed mechanism is sufficient (Appendix F1).

## Rules

1. Item-by-item, hardware-anchored to gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE.
2. Respect deliberate trade-offs: flag and quantify, do not auto-FIX. Reserve FIX for incorrect, superseded, deprecated, or harmful values.
3. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is non-trivial.
4. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN.
5. Flag every source conflict and name the trusted side. **Two live conflicts are already on file and must be handled, not rediscovered:** the ASPM semantics conflict (§5) and the stderr-census reconciliation (Appendix F).
6. Give exact versions (kernel / Mesa / linux-firmware / vkd3d-proton / pkg) and exact before→after, mapped to the in-script global.
7. **Do not carry values forward from any pre-7.135.1 edition of this brief.** The Δ table above is the authoritative change set. A recommendation that re-derives a removed token (`nowatchdog`, `tsc=reliable`, `8250.nr_uarts=0`, `AMD_VULKAN_ICD`, `DXVK_LOG_PATH`, `VKD3D_CONFIG`, `FSR4_UPGRADE` unprefixed) is a stale-source error, not a finding. Equally, a recommendation asserting that a value *changed* in this window is a stale-source error — nothing did.
8. **Two decisions are CLOSED by the maintainer and are not open questions** (§14): plaintext DNS on both host and router, and `GPU_DPM_LEVEL=high` rather than `profile_peak`. Do not re-litigate either; if upstream evidence directly contradicts the rationale on file, flag it as a note, not a FIX.
9. **A question closed by a code change is closed** (Δ table). The §6 modprobe sweep and the §9 mask-orphan detector both shipped — cite them as present, evaluate their *design*, and do not report either gap as open.

## Output

- **Findings matrix** (box-drawn, code-fenced, grouped by section): ITEM · CURRENT · CALL (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE.
- **Candidate-enhancement matrix** (§13, separate): ITEM · PRESENT?(no) · CALL (ADD-default/ADD-opt-in/KEEP-omitted) · IMPACT · RISK · EVIDENCE.
- **Before→after** for each TUNE/FIX/ADD: exact current string, exact replacement, in-script global. **A perf-value change now touches 13 sites** — enumerate all of them (§2).
- **VERIFY block** (post-reboot commands, §15).
- **Security delta vs CachyOS defaults** (§11, ordered).
- **Verdict:** one per section (OPTIMAL/TUNE/FIX) plus overall (PASS/PASS-WITH-FIXES/FAIL).
- **ROBUSTNESS verdict** (Appendix F, separate from tuning).
- **Methodology:** source list with access dates and versions; unknowns marked UNCERTAIN.

## Hard data (all counts asserted by `_ir_validate_counts`, 21 tripwires)

Live-evaluated from the v7.135.1 script, not transcribed, and identical to the 7.130.0 set:

**KERNEL_PARAMS 15** · MKINITCPIO_HOOKS 11 · MKINITCPIO_MODULES 1 · MKINITCPIO_COMPRESSION_OPTIONS 2 · LOGIND_IGNORE_KEYS 8 · **ENV_VARS 10** · **SYSCTL_VALUES 11** · PKGS_ADD 16 · PKGS_DEL 9 · MASK 11 · EXPECTED_VULKAN_PKGS 2 · EXPECTED_SERVICES 5 · `_RY_PKG_MANAGED_SERVICES` 1 · `_RY_POST_HOOKS` 17 · `_RY_ARGPARSE_SPEC` 6 · `_RY_BOOT_CRITICAL_DSTS` 4 · `_RY_PHASE_NAMES` 6 · `_RY_BACKUP_TARGETS` 4 · `_RY_TMPDIR_GLOBS` 6 · SYSTEM_DESTINATIONS 15 · USER_DESTINATIONS 2.

`RESOLVED_DNS_SERVERS` (2: 94.140.14.14, 94.140.15.15) is deliberately **not** in the count oracle — it is value-bearing, not a count invariant.

Managed files = 17 (15 system + 2 user; `_RY_MANAGED_FILE_COUNT`), recomputed at load; mismatch refuses (exit 3). Generated total = **5,093 B** across all 17, measured as written files (see the byte rule in §Method below), deterministic across three consecutive renders (17/17 identical by per-file content hash).

**Gates (v7.135.1):**
- **No version gates of any kind.** `KERNEL_MIN` + `_ir_validate_kernel_floor` were removed at 7.105.x, the Mesa soft `vercmp` warning is gone, and no advisory in-script kernel comment survives. Deploy / `--check` / `--verify` run on any kernel and any Mesa. Do not describe any floor as enforced and do not recommend restoring a gate (§14 Protected). 6.18.4 is a *research* baseline for this brief only (RTL8127 r8169 + suspend-hang fix `ae1737e7339b`; the gfx1151 hang fix is linux-firmware MES 0x86, not kernel).
- **CPU gate** `Ryzen AI Max` — sole skip env `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed on unreadable model; `--verify` warns, deploy/`--check` exit 3.
- Preflight validators, in `_init_runtime` order: `_ir_validate_counts` → `_ir_validate_keys` → **`_ir_validate_sets`** → `_ir_validate_post_hooks` (**4 total**, script **L781-784** — was 3 at L770-772). There is no `_ry_validate_keys`/`_ry_validate_counts`/`_ry_validate_post_hooks` — those names return rc 127 and look like a signal when they are only an unknown command.
- The only runtime env inputs are `RY_RUN_TIMEOUT`, `RY_INSTALL_SKIP_HARDWARE_CHECK`, `NO_COLOR`. Every profile toggle (`BLACKLIST_AMDXDNA`, `NM_WIFI_BACKEND`, `COUNTRY`, `GPU_DPM_LEVEL`, `EPP_PREFERENCE`, `CPUPOWER_GOVERNOR`, `RY_REMOTE_PLAY_PORTS`) is an embedded scalar set unconditionally (`set -g`) — an exported env var of the same name is clobbered; opting in means editing the script.
- `NO_COLOR` requires a **non-empty** value to disable color (`set -q NO_COLOR; and test -n "$NO_COLOR"`); presence alone does not fire. `TERM=dumb` disables independently.

---

## P0. Research priority queue (search in this order)

The queue is **21 questions** at this pin, renumbered fresh. One question retired by code change (old #15, modprobe gap), one re-framed from open design question to upstream verification (old #2 → new #2), and two new items raised by the 7.135.0/7.135.1 robustness work (#15, #21). Numbering is fresh — do not map these onto the 7.130.0 numbers.

```
║ #  ║ QUESTION (search anchors)                                            ║ SECTION ║
║────║──────────────────────────────────────────────────────────────────────║─────────║
║ 1  ║ MAX-PERF STACKING (still unmeasured after 5 releases) —              ║ §2      ║
║    ║ governor=performance + EPP=performance + DPM=high +                  ║         ║
║    ║ processor.max_cstate=1 under an 85 W shared PPT. Quantify the IDLE   ║         ║
║    ║ floor and the thermal-headroom cost; does pinning cost sustained-    ║         ║
║    ║ load clocks by burning budget at idle? §15 carries the turbostat     ║         ║
║    ║ block that feeds this — it has never been run                        ║         ║
║ 2  ║ VERIFY the profile's own EPP claim against upstream. README L239     ║ §2      ║
║    ║ states that under amd-pstate-epp with CPUPOWER_GOVERNOR=performance  ║         ║
║    ║ "the driver forces the EPP hint to its maximum and REJECTS any       ║         ║
║    ║ other value". Confirm both halves against docs.kernel.org/admin-     ║         ║
║    ║ guide/pm/amd-pstate: (a) does the performance governor pin EPP to    ║         ║
║    ║ its max, and (b) does the driver genuinely REJECT writes to          ║         ║
║    ║ energy_performance_preference while that governor is selected? If    ║         ║
║    ║ (b) holds, the udev EPP rule is a write that CANNOT LAND, which is   ║         ║
║    ║ a different (and more interesting) finding than "inert"              ║         ║
║ 3  ║ pcie_aspm.policy=performance + mt7925e.disable_aspm=1 TOGETHER —     ║ §5/§6   ║
║    ║ global policy vs endpoint option: redundant, complementary, or       ║         ║
║    ║ conflicting? SOURCE CONFLICT ON FILE: this brief's §5 says the       ║         ║
║    ║ policy DISABLES ASPM; the profile README now says it BIASES links    ║         ║
║    ║ away from ASPM and says to confirm with lspci. Settle it, then       ║         ║
║    ║ run host item A1 (lspci -vv LnkCtl) — still unrun                    ║         ║
║ 4  ║ PROTON_FSR4_UPGRADE=1 — is THIS the name current Proton-CachyOS      ║ §1      ║
║    ║ reads? Three names have shipped. NEW CLAIM TO TEST: the profile      ║         ║
║    ║ README now says recent Proton-CachyOS builds copy the DLL            ║         ║
║    ║ automatically so the variable "mainly pins a version" — if true      ║         ║
║    ║ the variable is near-inert; cite source. DXIL_SPIRV_CONFIG           ║         ║
║    ║ companion status; min version for FSR3.1→FSR4                        ║         ║
║ 5  ║ VKD3D_CONFIG=descriptor_heap REMOVED — was it default-enabled        ║ §1      ║
║    ║ upstream (removal correct) or is a capability now lost? Regression   ║         ║
║    ║ risk on RADV/RDNA3.5 titles that relied on the forced path           ║         ║
║ 6  ║ kernel.nmi_watchdog=0 via sysctl.d priority 95 — confirm it loads    ║ §3/§5   ║
║    ║ AFTER vendor 70-cachyos-settings.conf and that runtime value is 0    ║         ║
║    ║ without the nowatchdog boot token                                    ║         ║
║ 7  ║ GPU_DPM_LEVEL=high on gfx1151 — clock/power gating stays ACTIVE      ║ §2      ║
║    ║ under `high`; confirm against amdgpu docs. Frametime/1%-low evidence ║         ║
║    ║ vs auto. Do NOT propose profile_peak (closed, §14)                   ║         ║
║ 8  ║ vm.watermark_boost_factor=0 — reclaim/kcompactd spike removal on     ║ §3      ║
║    ║ 128 GB unified; interaction with compaction_proactiveness=0          ║         ║
║ 9  ║ linux-firmware MES 0x86 (GC 11.5.1) — shipping revision contains     ║ §10     ║
║    ║ the gfx1151 hang fix? min linux-firmware release; kernel-irrelevant  ║         ║
║ 10 ║ ntsync vs fsync on 16C/32T — CONFIG_NTSYNC=y, /dev/ntsync,           ║ §1      ║
║    ║ PROTON_NO_NTSYNC opt-out currency                                    ║         ║
║ 11 ║ RTL8127 r8169 commits f24f7b2f3af9 + ae1737e7339b — exact mainline   ║ §10     ║
║    ║ landing releases (research baseline only; no floor is enforced)      ║         ║
║ 12 ║ netdev_budget 600/5000 + BBR/fq sizing for dual 10 GbE (RTL8127)     ║ §6      ║
║ 13 ║ AdGuard 94.140.14.14/.15.15 ad-block tier + DNSSEC=no + DoT=no —     ║ §6/§11  ║
║    ║ quantify the security delta ONLY. Plaintext is CLOSED (§14): the     ║         ║
║    ║ host and the router both run plaintext by explicit decision          ║         ║
║ 14 ║ NM [global-dns-domain-*] servers= — confirm this is the mechanism    ║ §6      ║
║    ║ that beats per-link DHCP DNS under NetworkManager (systemd #33973);  ║         ║
║    ║ confirm Domains=~. is NOT the fix (#33579 dual-query leak)           ║         ║
║ 15 ║ NEW — THE .ry.orig DEAD-CODE WINDOW. From 7.109.0 to 7.135.0 the     ║ App.F1  ║
║    ║ first-adoption preserve never executed (fish `set -l` inside an      ║         ║
║    ║ `if` is not visible after the block), so 13 of 17 destinations were  ║         ║
║    ║ overwritten UNBACKED on every first deploy. Fixed at 7.135.1.        ║         ║
║    ║ Assess: residual exposure for hosts deployed in that window; is a    ║         ║
║    ║ one-time preserve the right primitive at all, or should the 13       ║         ║
║    ║ follow the boot four onto .ry.bak + post-write verify/restore?       ║         ║
║ 16 ║ ufw MASKED not removed — confirm the nftables-first gate closes the  ║ §9/§11  ║
║    ║ unfirewalled window; mask --now stops ufw and ufw-init stop flushes  ║         ║
║ 17 ║ Every-boot fsck.mode=force + commit=10 — boot cost vs durability;    ║ §4/§5   ║
║    ║ note fsck.mode=force is largely INERT on a Btrfs root (open Stage-3) ║         ║
║ 18 ║ swappiness=150 + CachyOS zram + zswap.enabled=0 coherence            ║ §3      ║
║ 19 ║ MangoHud cpu_custom_temp_sensor (added 0.8.3) as the k10temp fix;    ║ §12     ║
║    ║ #1794 (cpu_power reads 0 when cpu_temp active on Zen 5) is STILL     ║         ║
║    ║ OPEN upstream — the profile README now says so too; treat as         ║         ║
║    ║ unresolved, not fixed                                                ║         ║
║ 20 ║ Candidate knobs (§13) — mitigations, ppfeaturemask, RADV_PERFTEST,   ║ §13     ║
║    ║ read_ahead_kb: KEEP-omitted re-checks                                ║         ║
║ 21 ║ NEW — ORPHAN SEVERITY DESIGN. The two new detectors deliberately     ║ §8/§9   ║
║    ║ never set DRIFT: verify reports WARN (drop-ins) / INFO (masks) and   ║         ║
║    ║ --check writes JSONL only, because a re-run cannot clear either and  ║         ║
║    ║ a permanently non-zero exit 10 would train the operator to ignore    ║         ║
║    ║ it. Evaluate that trade. Separately: the PKGS_ADD orphan remains     ║         ║
║    ║ wholly undetectable by design — is documenting it sufficient?        ║         ║
```

**Retired from the 7.130.0 queue:** old #15 (modprobe-leftover gap — *closed in code*, Δ table). Old #2 survives only as a verification item (new #2), not as an open design question.

---

## §1. GPU · Vulkan · Proton runtime

**ENV_VARS (10, `~/.config/environment.d/10-environment.conf`, 0600) — byte-identical to 7.130.0:** DXVK_LOG_LEVEL=none · MANGOHUD=1 · MESA_SHADER_CACHE_MAX_SIZE=16G · POWERDEVIL_NO_DDCUTIL=1 · PROTON_ENABLE_WAYLAND=1 · **PROTON_FSR4_UPGRADE=1** · PROTON_LOCAL_SHADER_CACHE=1 · VKD3D_DEBUG=none · VKD3D_SHADER_DEBUG=none · WINEDEBUG=-all. No drirc (uma:1 native); no ttm/amdgpu module params; Vulkan stack via chwd (vulkan-radeon + lib32-vulkan-radeon).

**One new documentary claim to test, no value change:**

- **`PROTON_FSR4_UPGRADE=1` — P0 #4, now with an upstream-testable assertion attached.** Three names have shipped across this profile's history: `PROTON_FSR4_RDNA3_UPGRADE` (7.105.x), `FSR4_UPGRADE` (7.122.0), `PROTON_FSR4_UPGRADE` (7.130.0-current). Exactly one is read by the shipping Proton-CachyOS build; determine which, with a citation to the Proton-CachyOS source or release notes. **New at this pin:** the profile README's Tuning Notes now assert that recent Proton-CachyOS builds copy the FSR4 DLL automatically, so the variable "now mainly pins a version". If that is accurate, the variable is close to inert and the correct call is KEEP-with-note rather than FIX; if it is inaccurate, the README is the thing to correct. Either way it is a *documentation* finding, not a value change. Also re-confirm the `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` companion status and the minimum Proton version for FSR3.1→FSR4 upgrade on RDNA 3.5.
- **`VKD3D_CONFIG=descriptor_heap` REMOVED — P0 #5, unchanged since 7.130.0.** The variable is gone from ENV_VARS entirely. Two readings, and the research must distinguish them: (a) current vkd3d-proton enables the mutable-descriptor/descriptor-heap path by default on RADV, making the global force redundant — removal is correct; or (b) the path is still opt-in, in which case removal is a **capability regression** on D3D12 titles that benefited. Enumerate current vkd3d-proton defaults for RADV/RDNA 3.5, and state whether any per-title flag is now the recommended replacement. Do not recommend restoring a global force without evidence that the default is off.

**Carried-forward items (all unchanged in code):**

- **ICD selection with no pin:** `AMD_VULKAN_ICD` has been absent since 7.122.0 and the maintainer has confirmed RADV is the only ICD JSON on this stack. Confirm no gaming meta can install a second ICD (AMDVLK/amdvlk-pro) that would win enumeration; if one can, `VK_DRIVER_FILES` is the non-deprecated successor and this becomes a latent-regression flag. `_vre_envvars` asserts no ICD selection — the only Vulkan coverage left is `_vsp_required`'s package-presence check (§8, Appendix C).
- **ntsync (assert-only, no autoload conf):** `_vre_ntsync`/`_ntsync_state` (builtin|loaded|loaded_nodev|missing). Confirm: ntsync vs fsync currency; CachyOS `CONFIG_NTSYNC=y` (node without autoload); `loaded_nodev` still a real failure; frametime benefit on 16C/32T; `PROTON_NO_NTSYNC=1` opt-out current — the profile neither sets nor checks it, and the README now says so explicitly.
- **RADV heap:** confirm uma:1 on current Mesa (drirc removed by design).
- **GTT:** kernel auto-sizes (~62 GiB); >62 GiB single allocations route to BIOS UMA carveout (≤96 GB), not deprecated `amdgpu.gttsize`; verify `/sys/module/ttm/parameters/pages_limit`. `amd_iommu=off` does not change the ceiling.
- **`POWERDEVIL_NO_DDCUTIL=1`:** confirm the exact variable name PowerDevil reads to disable its ddcutil/DDC-CI backend and that it is consumed from the systemd user-manager environment (where `environment.d` places it). A wrong name is a silent no-op. The maintainer runs this deliberately — Plasma brightness control for the external monitor is intentionally off — so the only question is mechanism correctness, not desirability. Paired runtime health check `_vrsv_user_units` (Appendix C) fails on a failed `plasma-powerdevil.service`.
- PROTON_ENABLE_WAYLAND maturity/fallback on current Proton; MESA_SHADER_CACHE_MAX_SIZE=16G sizing; MANGOHUD=1 overhead with gamescope/GameMode; `PROTON_LOCAL_SHADER_CACHE=1` interaction with the 16G global cache cap (do they contend?).
- **XDNA NPU:** blacklisted by default (§10/§11) — zero gaming impact; LLM/NPU work needs the validator-paired opt-in.
- **Mesa:** no floor is enforced (§10). Evaluate current RADV guidance for gfx1151 on its merits and state a *recommended* version, flagged clearly as advice rather than a restored gate.
- **New at this pin, relevant here:** `~/.config/environment.d/10-environment.conf` and `~/.config/MangoHud/MangoHud.conf` are both in the 13 destinations that gained a working `.ry.orig` preserve at 7.135.1 (Appendix F1). A user's pre-existing environment.d or MangoHud file is now preserved once on first adoption; before 7.135.1 it was silently destroyed. Note this when recommending any user-scope change.
- Sources: github.com/HansKristian-Work/vkd3d-proton (env docs), docs.mesa3d.org (RADV, APU heap), gitlab.freedesktop.org/mesa + drm/amd, github CachyOS/proton-cachyos, amd.com ROCm, docs.kernel.org accel/amdxdna, invent.kde.org powerdevil.

## §2. CPU performance & power

**No value in this section moved since 7.130.0.** What moved is the profile's own explanation of it, and the number of places a change would have to touch.

amd_pstate=active · governor **performance** (`CPUPOWER_GOVERNOR` L587, cpupower-service.conf) · EPP **performance** via udev (`EPP_PREFERENCE` L591, enum `_RY_EPP_LEVELS` = default performance balance_performance balance_power power) · `EXPECTED_SCALING_DRIVER=amd-pstate-epp` (L592, verify-only) · `dynamic_epp=disabled` asserted · **GPU_DPM_LEVEL=high** (L589, enum `_RY_DPM_LEVELS` = auto low high manual profile_standard profile_min_sclk profile_min_mclk profile_peak perf_determinism; add-only udev rule, `ENV{DEVTYPE}=="drm_minor"`) · prefcore + boost=1 asserted. Masked: power-profiles-daemon, ananicy-cpp. All four globals remain at L587/L589/L591/L592 — the lines did not move despite the script growing 48 lines.

**Both EPP and governor are AT their maximum** — `performance` is the top of `_RY_EPP_LEVELS`, and the governor validates against `^[a-z][a-z0-9_-]*$` with `performance` the max meaningful value. There is no remaining headroom on those two axes. `EXPECTED_SCALING_DRIVER` has no maximum and tunes nothing: it is verify-only with zero generator references. Changing it makes the verifier assert a driver the kernel never loaded, producing a false `_chk_eq` failure on every `--verify`. It FOLLOWS a KERNEL_PARAMS `amd_pstate=` change, never leads one.

- **P0 #2 — the question changed shape; read this before researching it.** The 7.122.0 edition of this brief asserted that `powersave` honors EPP while `performance` pins max and *ignores* it. The 7.130.0 edition carried that as an open question. **The profile has since retracted it in writing.** README L239 now states: under `amd-pstate-epp` with `CPUPOWER_GOVERNOR=performance`, the driver forces the EPP hint to its maximum **and rejects any other value**, so `EPP_PREFERENCE=performance` restates what the governor already imposes rather than adding to it. That is a plausible reading of amd-pstate's behavior, but it is the profile's own prose, not upstream evidence, and it carries a second claim the older wording did not: *rejection*. Settle both halves against `docs.kernel.org/admin-guide/pm/amd-pstate`. **If the rejection claim holds, the udev EPP rule is issuing a write that cannot land** — which is materially different from "inert decoration", because a rejected sysfs write may surface as a udev error rather than silently succeeding. Check the udev log path, not just the resulting value.
- **P0 #1 — max-perf stacking under an 85 W ceiling, now five releases old and still unmeasured.** Governor max + EPP max + DPM `high` + `processor.max_cstate=1` all push the same direction, and three of the four raise the *idle* floor rather than the load ceiling. The 85 W PPT caps peak power regardless of any of them, so the marginal gain at full load may be near zero while the idle cost is real and compounding. **Idle is the metric to watch, not the load ceiling.** Quantify: idle package power before/after, and whether sustained-load clocks *drop* because budget is burned on an elevated idle floor. Name the assumed budget in every power call (85 W README ceiling vs 140 W stock). The profile CHANGELOG already concedes the direction of the trade — it records that package power stays capped at 85 W in firmware so peak draw is unchanged, while idle draw rises because clocks no longer scale down. **That is a stated expectation, not a measurement.** §15 carries the turbostat block that would settle it.
- **P0 #7 — `GPU_DPM_LEVEL=high` correctness.** `high` forces clocks to the highest power state with clock and power gating still **active** — the documented distinction from `profile_peak`, which adds mclk/pcie forcing and disables gating. Confirm against amdgpu documentation that gating genuinely remains active under `high` on gfx1151, and gather frametime/1%-low evidence for `high` vs `auto`. **Do not propose `profile_peak`** — that choice is closed (§14). Two inertia facts still apply: the udev rule is add-only (no re-assert after GPU reset) and only began firing at the v7.94/95 matcher fix, so pre-fix "high made no difference" observations are void.
- **No resume path exists on this host.** MASK (L612, unmoved) contains all five sleep/suspend/hibernate targets, so the classic "amdgpu resets `power_dpm_force_performance_level` on resume" concern does not apply. The udev `ACTION=="add"` rule is the only event that matters. **Do not propose systemd sleep-hook workarounds.**
- Confirm EPP live-applies (`_post_udev`: `udevadm verify` ≥254, reload + retrigger cpu/block) and the GPU rule matches at enumeration; `dynamic_epp` node ≥6.16; prefcore/boost on Strix Halo; masks (ppd, ananicy-cpp) safe on current CachyOS. Masking power-profiles-daemon removes the competing governor authority — this matters for any benchmark comparison.
- `processor.max_cstate=1` (§5) is the **higher-leverage variable than the three perf globals** and remains an open maintainer decision: idle power/thermal vs wake-latency/jitter; boost-headroom interaction; is `1` the right cap? It compounds directly with DPM pinned high.
- **A full perf-value change now touches 13 sites, up from 10** — any TUNE recommendation must enumerate all of them. Re-derived at this pin:

```
║ # ║ FILE      ║ LINE ║ WHAT IT CARRIES                                    ║
║───║───────────║──────║────────────────────────────────────────────────────║
║ 1 ║ script    ║ L587 ║ set -g CPUPOWER_GOVERNOR performance                ║
║ 2 ║ script    ║ L589 ║ set -g GPU_DPM_LEVEL high  + trailing comment       ║
║ 3 ║ script    ║ L591 ║ set -g EPP_PREFERENCE performance (+ _RY_EPP_LEVELS)║
║ 4 ║ script    ║ L882 ║ udev generator --description names "EPP performance"║
║ 5 ║ script    ║ L887 ║ "# AMD P-State EPP performance (maximum CPPC hint)" ║
║ 6 ║ script    ║ L889 ║ "# GPU performance level (gfx1151 clock-floor;      ║
║   ║           ║      ║ forced high)"                                      ║
║ 7 ║ README    ║ L118 ║ managed-files row: governor (`performance`)         ║
║ 8 ║ README    ║ L120 ║ managed-files row: GPU DPM level `high`             ║
║ 9 ║ README    ║ L227 ║ Service Keys row: CPUPOWER_GOVERNOR | performance   ║
║10 ║ README    ║ L231 ║ Service Keys row: GPU_DPM_LEVEL | high              ║
║11 ║ README    ║ L232 ║ Service Keys row: EPP_PREFERENCE | performance      ║
║12 ║ README    ║ L239 ║ CPU/GPU prose paragraph (EPP-restates-governor)     ║
║13 ║ CHANGELOG ║ L5+  ║ new entry inserted after the preamble               ║
```

  The growth from 10 to 13 is entirely README-side: the Service Keys table now carries all three values as its own rows (sites 9-11), where the 7.130.0 edition counted only the managed-files table and the prose paragraph. **The old brief's site list (script L870/L875/L877, README L134/L136/L235) is stale on every line number** — do not reuse it.
- Grep traps when quoting these values: `powersave` has 4 hits but only L587 is in scope (the others are `NM_WIFI_POWERSAVE` plumbing); `balance_performance` has exactly 1 hit (the surviving `_RY_EPP_LEVELS` member). Do not edit the phrase "clock-floor" — it stays accurate under `high` and is consistent across the generator and verify paths.
- Domain validation lives in **`_ir_validate_keys` (L695)**, checks at L707/708/709 — all three unmoved. Negative tests: rc 0 with high/performance/performance; rc **3** (EXIT_PREFLIGHT) for a bogus DPM level, bogus EPP, or a governor failing the regex. `_err_loud` **exits** rather than returns, so each negative case needs its own subprocess.
- Sources: docs.kernel.org amd-pstate + kernel-parameters, wiki.archlinux.org/CPU_frequency_scaling + AMDGPU, freedesktop.org ppd, kernel amdgpu `power_dpm_force_performance_level` docs.

## §3. Memory & VM

**SYSCTL_VALUES (11, `/etc/sysctl.d/95-ry-overrides.conf`, priority 95 after vendor 70-cachyos-settings.conf) — unchanged since 7.130.0:** kernel.nmi_watchdog=0 · net.core.default_qdisc=fq · net.core.netdev_budget=600 · net.core.netdev_budget_usecs=5000 · net.ipv4.tcp_congestion_control=bbr · net.ipv4.tcp_notsent_lowat=16384 · net.ipv4.tcp_slow_start_after_idle=0 · vm.compaction_proactiveness=0 · vm.max_map_count=2147483642 · vm.swappiness=150 · vm.watermark_boost_factor=0. zswap.enabled=0 (cmdline). THP/KSM/oomd left to CachyOS; vm.page-cluster + vm.vfs_cache_pressure dropped as vendor duplicates.

Note the emission form: values are stored `k=v` in the array but rendered `k = v` (sysctl.d form). Any parity check must normalise whitespace. The drop-in's annotation comment is **selective by design** (3 of 11 keys) at 64 characters against a ≤66 cap — there is no room to annotate more keys. Do not propose "fixing" the partial annotation.

- **`kernel.nmi_watchdog=0` — P0 #6.** Added at the 7.130.0 generation and unchanged since; it resolved the older assert/token asymmetry by backing the runtime assert with a value the profile actually sets rather than restoring the `nowatchdog` boot token. Confirm the mechanism holds: priority 95 loads **after** CachyOS's vendor `70-cachyos-settings.conf`, so this value wins regardless of what the distro package sets — that is correct *ordering*, not merely presence. Verify the runtime value is 0 on a clean boot with no `nowatchdog` token present. This remains the single most cheaply testable item in the brief (§15) and has still not been checked on hardware.
- **vm.watermark_boost_factor=0 — P0 #8:** confirm disabling watermark boosting removes post-fragmentation reclaim/kcompactd spikes (frametime consistency) on 128 GB unified; interaction with `compaction_proactiveness=0` (both suppress proactive compaction paths — coherent or redundant?); name the CachyOS vendor value it overrides (kernel default 15000).
- **Vendor-duplicate drop a no-op?** Confirm 70-cachyos-settings.conf still ships page-cluster=0 + vfs_cache_pressure=50; a differing vendor default makes the drop a silent change.
- **swappiness=150 + CachyOS zram + zswap=0 — P0 #18:** gratuitous on 128 GB or LLM-reclaim-helpful; no double compression (zswap off before zram); zram advisory-only (not managed, not asserted).
- vm.max_map_count (MAX_INT−5, SteamOS value) — Proton/anti-cheat sufficiency; compaction_proactiveness=0 for large unified allocs; oomd disabled on 128 GB.
- Sources: docs.kernel.org admin-guide/sysctl/vm + mm + kernel, wiki.archlinux.org/Zram + Sysctl.

## §4. Storage & filesystem

NVMe scheduler `none` via udev (99- sorts after vendor 60-ioschedulers.rules); `nvme_core.default_ps_max_latency_us=0` (§5); fstab ext4 `noatime,lazytime,commit=10`; fstrim.timer enabled; zswap off. Unchanged since 7.130.0.

- NVMe `none` vs mq-deadline/kyber on this dual-NVMe box; `nr_requests`/`read_ahead_kb` unset — propose ATTRs only with game-load/LLM-read evidence, else defaults optimal; confirm CachyOS still defaults kyber and 99- ordering wins.
- noatime+lazytime coexistence (lazytime residual value under noatime); **commit=10 durability vs every-boot forced fsck (§5)** — quantify the boot cost of `fsck.mode=force` and whether commit=10 + forced fsck is coherent or belt-and-braces. **Note the filesystem caveat:** the maintainer's open Stage-3 ledger records `fsck.mode=force` as largely **inert on a Btrfs root**. Establish which filesystem the root actually carries before costing this item — the fstab rewrite path is ext4-only (Appendix D), so an ext4 data volume and a Btrfs root can coexist here. State the assumption explicitly.
- fstrim.timer vs continuous discard; fstab rewrite invariants in Appendix D. `/etc/fstab` is NOT one of the 17 managed destinations and does not participate in the `.ry.orig` class — it has always had its own `.ry.bak` during rewrite (Appendix F1).
- Sources: docs.kernel.org block + ext4, wiki.archlinux.org/SSD + Ext4 + fsck + Btrfs.

## §5. Kernel cmdline (15 tokens — latency set)

Live-rendered from `KERNEL_PARAMS` (sorted, as emitted) — byte-identical to 7.130.0:

```
amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0
```

**The ASPM pair is this section's primary research load, and it now carries a live source conflict.**

- **⚠ SOURCE CONFLICT — `pcie_aspm.policy=performance` semantics. Resolve this before any ASPM finding.** Two statements are on file and they are not the same claim:
  - **This brief's own §5 at 7.130.0:** `performance` = *disable* ASPM + clock PM; `powersave` = enable all states; `default` = leave as BIOS configured. Sourced to `kernel-parameters.txt` and Helgaas's v6.9 documentation fix (commit `2e0239d47d75`), which established that `pcie_aspm=off` does **not** disable ASPM — `off` means leave the firmware configuration untouched.
  - **The profile README at 7.135.1:** `pcie_aspm.policy=performance` *biases every link away from* ASPM, addressing Bluetooth reconnect and NVMe latency; plain `pcie_aspm=off` only inherits the BIOS state; and it explicitly instructs the reader to confirm actual link state with `lspci -vv` (`LnkCtl: ASPM Disabled`) **rather than assuming it from the token**.
  The two agree completely on `pcie_aspm=off` and disagree on how absolute `policy=performance` is. The README's weaker phrasing is the safer of the two and is the one that comes with a verification instruction. **Settle it against `kernel-parameters.txt` and the aspm.c policy implementation, name the trusted side, and do not silently adopt either.** The old brief's flat "settled upstream — do not re-derive" framing is what allowed the discrepancy to sit unnoticed for a release cycle.
  **BEWARE a known stale source:** wireless.docs.kernel.org still carries the pre-6.9 text claiming "off — Disable ASPM". Cite `kernel-parameters.txt` or the Helgaas commit, never the wireless doc page.
- **`mt7925e.disable_aspm=1` — P0 #3.** The per-module option was dropped at 7.102.x on the theory that `pcie_aspm.policy=performance` carried the MT7925 mitigation globally; it was re-added at 7.129.0 and the profile ships **both**. The correct reading, and the one to verify: the global policy governs *link state* while the module option disables ASPM *at the endpoint* — complementary, not redundant. The README supports this reading and adds a concrete justification: coredumps are still reported on the Wi-Fi adapter without the endpoint option. Research: (a) does `policy=performance` genuinely affect L0s/L1 on every link on this board, or only on links the kernel is permitted to touch; (b) with both in place, is there any conflict or double-handling; (c) the NVMe latency claim under policy=performance; (d) whether `pcie_port_pm=off` is additionally needed or redundant. **Do not describe the module option as omitted** — that was true only between 7.102.x and 7.129.x. **This is host item A1** and the `lspci -vv` LnkCtl audit has still not been run on hardware after five releases; treat any claim about actual link state as UNCERTAIN until it is.
- **`clearcpuid=umip`:** UMIP off, kernel tainted since 5.19 — upstream explicitly states `clearcpuid` is not for production use. This is an **open maintainer trade-off**, not a defect: flag and quantify the taint consequence (support posture, any subsystem that refuses to load), do not auto-FIX. Confirm the `umip` string form is still accepted on current kernels; the README's rationale is that the string form is version-stable because CPUID bit numbers shift between kernels, and that the token can be dropped absent `umip_printk` stutter. Asserted generically (`_vrk_cmdline`), no UMIP-specific check.
- **`processor.max_cstate=1`:** costs idle power and boost residency under the 85 W ceiling, and **compounds with DPM pinned high** (§2). Open maintainer decision; the highest-leverage single token in this set.
- **`fsck.mode=force`:** largely inert on a Btrfs root — see §4 before costing it.
- **`amd_iommu=off`:** validator-paired to `BLACKLIST_AMDXDNA` (§11). ROCm unaffected on gfx1151; NPU is the named casualty. Weigh marginal latency vs DMA-isolation loss vs NPU loss.
- **`ipv6.disable=1`:** hard-coupled to the IPv4-only nftables ruleset (`_ir_validate_keys`). LAN impact, Steam/Proton netcode fallback behavior, dual-stack opt-out.
- **`btusb.enable_autosuspend=n`:** MT7925/BT reconnect fix; overlap with `usbcore.autosuspend=-1`. **`amd_pstate=active`** is the root of the whole CPU story — governor and EPP are downstream of it and meaningless without it. **`split_lock_detect=off` / `nvme_core ps_max_latency=0` / `zswap.enabled=0`:** validate each is current, non-deprecated, and correct for this silicon.
- No `preempt=` — KEEP-omitted (CachyOS boots full; `_vrk_cmdline` INFOs the model). Zero amdgpu/ttm params — hands-off (`_vrkm_amdgpu` no-ops without `amdgpu.*`).
- Input hygiene: tokens charset-gated `^[A-Za-z0-9._,=-]+$`; a new token outside it must also change the validator. The count tripwire asserts 15 — any TUNE that adds or removes a token updates both.
- Sources: docs.kernel.org kernel-parameters + PCIe/ASPM + amd-pstate + UMIP + IOMMU + ipv6-sysctl, git.kernel.org (commit 2e0239d47d75), wiki.archlinux.org/AMDGPU + IOMMU + fsck + Power_management.

## §6. Network & latency

Net sysctls §3; IPv6 off §5; nftables §11. NM: wifi.backend=wpa_supplicant, wifi.powersave=2 (off — MT7925/mt76 software PS causes latency spikes), logging WARN; dispatcher `LogLevelMax=notice` in its own managed file. resolved: **DNS=94.140.14.14 94.140.15.15 (AdGuard ad-block tier)**, MulticastDNS=no, LLMNR=no, DNSOverTLS=no, DNSSEC=no; generated header "AdGuard upstreams, plaintext, mDNS/LLMNR off". regdom US. Masked: NetworkManager-wait-online, avahi-daemon.service + .socket. Enabled: NetworkManager. All byte-identical to 7.130.0.

**Modprobe managed file (`60-ry-modules.conf`):** amdxdna blacklist only (default); `BLACKLIST_AMDXDNA=false` renders a comment-only file whose comment reads "MT7925 ASPM handled on the kernel command line". Validator `_grep_modprobe_entry` accepts comment-only.

- **✅ THE MODPROBE-LEFTOVER GAP IS CLOSED. Do not report it as open.** It stood for four release generations and was P0 #15 at the last pin. The implementation shipped at 7.132.0-7.133.0 (verify side) and was completed at 7.135.0 (check side + shared helper):

```
║ PIECE                  ║ WHAT IT DOES                                        ║
║────────────────────────║─────────────────────────────────────────────────────║
║ _ry_stale_ry_dropins   ║ find /etc/modprobe.d -maxdepth 1 -name '60-ry-*.conf'║
║                        ║ ! -name '60-ry-modules.conf' -printf '%f\n'; rc 0    ║
║ _vss_modprobe (verify) ║ _warn per stale file naming the supersession +       ║
║                        ║ pacman -Qo confirmation step; then _log             ║
║ _check_record_orphans  ║ _log MODPROBE_STALE_DROPIN: count= files=            ║
║   (--check, L2618)     ║ comma-joined; NEVER sets drift                      ║
║ README                 ║ managed-files row now states the sweep exists       ║
```

  **Design points to evaluate rather than re-raise:** (a) it is one shared helper with exactly two callers, not a verify-path copy — the 7.135.0 CHANGELOG calls this out explicitly; (b) `_check_record_orphans` is invoked from `_ry_do_check` **before** the phase loop (L2618) precisely so an early phase bail cannot skip the record; (c) it never sets DRIFT, so `--check` still exits 0 with stale drop-ins present. **The residual limitation is real and worth stating:** `_check_record_orphans` sits *after* the sudo/systemctl preflight bail at L2616-2617, so on a host where `sudo -n` is not cached the record is never written even though the sweep needs no privilege at all (`find` alone would do it). That is consistent with `--check` being sudo-gated by contract, but it is a detectable-without-privilege signal being withheld for a privilege reason. Assess whether the sweep should be hoisted above the preflight bail. **This is a design question, not a gap** — see P0 #21.
- **THE KEY DESIGN FINDING — a resolved-only DNS config is SILENTLY INEFFECTIVE under NetworkManager, and the profile handles it correctly.** Per-link DNS from DHCP via NM outranks the global `DNS=` line in resolved (systemd issue #33973). `Domains=~.` is **not** the fix — issue #33579 documents it querying both global and local servers, leaking queries and returning nondeterministic answers. The correct mechanism is NM's `[global-dns-domain-*]` block, and it ships in the generated NM drop-in (B6). The profile README now states the same reasoning independently. **P0 #14: verify this against current systemd/NM behavior** and confirm the drop-in's `servers=` line is the operative path. Any recommendation touching DNS must not regress to a resolved-only configuration.
- **DNS security delta — quantify only, the choice is CLOSED (§14).** DoT and DNSSEC are both off, so the resolver neither authenticates nor encrypts, and both host and router run plaintext to AdGuard by explicit maintainer decision. The rationale on file: AdGuard filters identically whether or not the transport is encrypted, so DoT buys only ISP query-name privacy; and router-side DoT in Strict mode fails closed, making one TLS endpoint a single point of failure for every device on the LAN. Uninterrupted connectivity for every device is the stated priority. The profile README states the same: `DNSOverTLS=yes` fails closed, so an unreachable endpoint would stop resolution outright. **Quantify the exposure in §11 ordering; do not re-recommend DoT or DNSSEC.**
- **MT7925 upstream status:** has the mt76 ASPM/coredump fix landed such that neither the ASPM policy override nor the per-module option is required? Cite commit + release. If landed, the profile is carrying two belt-and-braces mitigations for a fixed bug — a legitimate KEEP-with-note or TUNE-to-simplify, not a FIX. The README's claim that coredumps still occur without the endpoint option is the specific assertion to test.
- **netdev_budget=600/netdev_budget_usecs=5000 on dual 10 GbE — P0 #12:** confirm sizing for RTL8127 line rate or propose values with driver evidence; tcp_rmem/wmem/ring defaults sufficiency.
- bbr+fq currency (BBRv3 status in mainline/CachyOS); wifi.powersave=2 still correct for mt76; wpa_supplicant vs iwd parity (iwd opt-in intact; residual verify coverage = backend compare only — deliberate, Low/Low).
- **avahi masked (unit+socket):** confirm no host dependency (printer/`.local` discovery) and no D-Bus resurrection path; with resolved MulticastDNS=no, multicast discovery is fully closed. The README's stated rationale is collision with resolved as a second mDNS responder.
- regdom US: MT7925 TX-power/channel on current wireless-regdb; 6 GHz AFC status; non-US requires hand-edit.
- Same-basename replace caution (B5/B6): if CachyOS ships its own `99-cachyos-resolved.conf`/`99-cachyos-nm.conf`, deploy REPLACES them — confirm intended, not a vendor-update clash. **New mitigation at this pin:** both files are in the 13 destinations that now get a one-time `.ry.orig` preserve, so a pre-existing vendor file is captured once rather than destroyed (Appendix F1). That changes the severity of this item from silent-loss to recoverable-once, but only for hosts first deployed at ≥7.135.1. The dispatcher drop-in is a distinct path and does not participate in this collision class.
- Sources: docs.kernel.org networking, github.com/systemd/systemd issues 33973 + 33579, git.kernel.org mt76 + wireless-regdb + r8169, wiki.archlinux.org NetworkManager + Wireless + Sysctl, man.archlinux.org avahi-daemon + resolved.conf + NetworkManager.conf.

## §7. Boot chain & initramfs

loader.conf: default @saved, timeout 0, console-mode keep, editor no. sdboot-manage: DEFAULT_ENTRY manual, OVERWRITE/REMOVE_EXISTING/REMOVE_OBSOLETE yes, LINUX_FALLBACK_OPTIONS "quiet". mkinitcpio: MODULES=(amdgpu), HOOKS(11)= base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck, COMPRESSION zstd, COMPRESSION_OPTIONS=(-1 -T0), explicit BINARIES=()/FILES=(). Pre-deployed Phase 2 → one rebuild at `-Syu`. Unchanged since 7.130.0.

- HOOKS order (systemd/microcode/kms/sd-vconsole/block) current-correct; amdgpu early-KMS; fsck-hook handshake with `fsck.mode=force` (no boot prompt).
- **zstd -1 -T0:** quantify boot decompress vs default-3 (sub-100 ms class on NVMe) and image size vs ESP budget (`BOOT_SPACE_CRIT/WARN` 200/500 MB) with multiple kernels + fallback. TUNE to default-3 only if size threatens the budget; tokens are charset-gated + count-asserted — any TUNE updates both.
- Live drift caught: `_vsb_mkinitcpio` compares live `COMPRESSION=`/`COMPRESSION_OPTIONS` (multi-line join, last-wins warn) — confirm last-wins matches shell sourcing.
- timeout 0 + manual + REMOVE_EXISTING=yes wipes foreign BLS entries (EFI-resident loaders untouched); recovery path = live-USB → chroot; sdboot-manage currency vs kernel-install/UKI (UKI out of scope).
- **Fallback-entry exposure.** `LINUX_FALLBACK_OPTIONS="quiet"` strips all 15 params, so the fallback boots kernel-default IOMMU (AMD-Vi ON) + IPv6 ENABLED under an IPv4-only ruleset + **ASPM at firmware default with the MT7925 endpoint option absent**, while the modprobe amdxdna blacklist REMAINS active (asymmetry — it is a file, not a cmdline token). Restate the exposure against the 15-token main entry. Confirm the window is accepted or flag it.
- All four boot files are `_RY_BACKUP_TARGETS` and are explicitly EXCLUDED from the new `.ry.orig` path — they use `.ry.bak` plus post-write verify/restore, which is strictly stronger (Appendix F1). Do not recommend adding `.ry.orig` to the boot four.
- Sources: wiki.archlinux.org Mkinitcpio + systemd-boot, sdboot-manage upstream.

## §8. Packages

**PKGS_ADD (16):** nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa, mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, lm_sensors, rtkit, realtime-privileges, nftables, pacman-contrib (supplies pactree `PACTREE_TIMEOUT_S=60` + paccache -rk2/-ruk0).
**PKGS_DEL (9, `-Rns`, rdep-aware via pactree):** plymouth, cachyos-plymouth-bootanimation, cachyos-plymouth-theme, breeze-plymouth, plymouth-kcm, micro, cachyos-micro-settings, cachy-update, kdeconnect. **AUR:** none. **Vulkan (chwd, verify-only):** vulkan-radeon, lib32-vulkan-radeon.

**Unchanged since 7.130.0** — both sets are byte-identical. `ddcutil` and `git-delta` remain absent; `ufw` is not in PKGS_DEL and never was — its disposition lives on the mask side (§9).

- **NEW GATE — set contradictions now refuse to deploy.** `_ir_validate_sets` (L736, called from `_init_runtime` at L783) refuses on three intersections: `PKGS_ADD ∩ PKGS_DEL` (phase 4 would remove what phase 2 installed), `MASK ∩ EXPECTED_SERVICES` (phase 4 masks before it enables, so the enable would fail), and `MASK ∩ _RY_PKG_MANAGED_SERVICES` (a masked unit cannot be package-managed). All three shipped intersections are empty; each negative case exits 3. **Any §8/§9 recommendation that adds a package or unit must be checked against all three sets, because a contradictory suggestion is now a hard deploy refusal rather than a silent misbehavior.** Evaluate whether three is the complete set of contradictions worth gating.
- **The PKGS_ADD orphan is still undetectable — and that is now a stated position, not an omission (P0 #21).** A package dropped from `PKGS_ADD` stays installed forever, and nothing in the script or the verify surface can see it. The maintainer's reasoning is on file: detection would need a persisted manifest of what earlier versions installed, and a new state file would add its own drift surface. The README documents the limitation. **Assess whether that trade is right**, and if a lighter mechanism exists (e.g. deriving it from the pacman local database's explicit-install set plus install timestamps) that does not introduce new state. Do not report it as an unnoticed gap.
- **`ddcutil` absence is coupled to `POWERDEVIL_NO_DDCUTIL=1` (§1).** DDC/CI monitor control is deliberately off and the package is not installed on the profile's behalf; the script retains a conditional i2c-group hint that fires only when `pacman -Qq ddcutil` succeeds — i.e. for a package the profile no longer adds. Confirm nothing in the gaming metas pulls ddcutil back in as a dependency, which would make the env var the only thing suppressing the probe. Zero gaming impact.
- Confirm the gaming metas supply RADV/Proton/gamescope/MangoHud/GameMode; **GameMode omission — KEEP** (governor/EPP/DPM pinned profile-wide at maximum, so GameMode's principal lever is already applied unconditionally); confirm the meta's MangoHud does not clash with the shipped conf.
- With no ICD pin in ENV_VARS (§1), whether any meta can install a second Vulkan ICD is load-bearing — answer it here and cross-reference §1.
- `-D --asexplicit` post-`Syu` re-mark = orphan protection for PKGS_ADD members that pre-existed as dependencies (idempotent; failure warns).
- rtkit + realtime-privileges for PipeWire priority (rtkit-daemon socket-activated); lib32-mesa still needed beside lib32-vulkan-radeon; PKGS_DEL fallout tracked (`_RY_PKG_REMOVE_SKIPS`).
- Advisory: znver/x86-64-v4 (AVX-512) repo benefit over v3 for this build.
- Sources: wiki.cachyos.org, wiki.archlinux.org Gaming + PipeWire + RealtimeKit, archlinux.org/packages.

## §9. systemd units & time-sync

**Mask (11):** ananicy-cpp.service, power-profiles-daemon.service, NetworkManager-wait-online.service, avahi-daemon.service, avahi-daemon.socket, ufw.service, sleep.target, suspend.target, hibernate.target, hybrid-sleep.target, suspend-then-hibernate.target. **Enable (5):** fstrim.timer, NetworkManager.service, cpupower.service, nftables.service, bluetooth.service. **Untouched:** oomd (intentional), NetworkManager-dispatcher + rtkit-daemon (socket-activated), iwd, modemmanager. Unchanged since 7.130.0; MASK still declared at L612.
**NTP unconditional (warn-only):** `_ry_check_time_sync` scans chronyd/ntpd/openntpd; refuses to enable timesyncd if any is active; else enables timesyncd, re-checks after 2 s, runs `_ry_rtc_writeback` (`--systohc --utc`; RTCInLocalTZ defer branch) on sync. Escape = mask timesyncd (no opt-out env, by design).

**The five sleep/suspend targets mean there is no resume path on this host** — relevant to §2 (no GPU-reset re-assert needed) and to §11 (an always-on box has a different exposure profile than one that suspends). `LOGIND_IGNORE_KEYS` is a separate mechanism (inert keypresses), not sleep masking; do not conflate them.

- **✅ THE STALE-MASK GAP IS CLOSED (detection). Do not report it as open.** `_ry_orphan_masked_units` reads `systemctl list-unit-files --state=masked,masked-runtime --no-legend --plain`, parses the first whitespace-delimited field, and emits any unit not in `$MASK`. Two callers: `_vss_orphan_masks` (a `_verify_static_services` sub — the 48th verify sub, and the reason the count moved 59 → 60) and `_check_record_orphans` (JSONL key `MASK_ORPHAN`). Failure modes are handled conservatively: a non-zero `systemctl` rc returns nothing rather than guessing, and the parser is written to survive tab-vs-space column padding.
- **The severity choice is deliberate and is P0 #21.** `_vss_orphan_masks` reports at **`_info`, not `_warn`**, and its own message states why: *mask ownership is unattributable*. There is no `60-ry-`-style namespace for systemd units, so the script cannot tell a mask it created from a distro mask or a hand-made one; the message therefore tells the operator to unmask only what this profile masked under an earlier MASK and to leave the rest alone. Neither detector sets DRIFT, because a re-run cannot clear either and a permanently non-zero `--check` exit 10 would train the operator to ignore exit 10 entirely. **Evaluate that reasoning.** The counter-argument worth testing: an INFO in a long `--verify` run is easy to miss, and the JSONL record is only read deliberately.
- **ufw is masked, not removed — P0 #16, the security-relevant mechanism in this section.** The package is untouched and `ufw.service` is a MASK member. The ordering constraint is load-bearing and lives on the mask path:
  - `_csm_enable_nftables_first` is gated on `contains ufw.service $MASK` — it activates nftables and confirms a live default-deny ruleset *before* anything touches ufw.
  - `_csm_prepare_ufw_masking` returns non-zero on an unconfirmed ruleset, and `_configure_services_mask` then withholds `ufw.service` from the safe-mask set for that run.
  - Rationale in-script: `mask --now` stops ufw, and `ufw-init stop` flushes its rules — masking before the nftables default-deny is live would open an unfirewalled window.
  - Log keys to look for in a real run: `UFW_MASK_DEFERRED`, `UFW_RULE_FLUSH_OK|FAIL|SKIP`, `SECURITY_POSTURE`.
  Research: confirm the gate genuinely closes the window on a host where ufw is installed *and* active; confirm that a withheld mask leaves ufw's own rules intact (not half-flushed); and confirm `nftables.service` being a oneshot (unit state reads inactive) does not defeat the liveness check — the script judges by live policy-drop, not unit state, which is the correct approach but should be validated against current nftables packaging.
- **`ufw.service` is now itself a mask-orphan candidate.** If a future release drops `ufw.service` from MASK, `_ry_orphan_masked_units` will surface it — but as an unattributable INFO, in the same undifferentiated list as any distro mask. Confirm that is acceptable for the one masked unit with a security posture attached to it.
- Each mask safe on current CachyOS: ananicy-cpp + ppd (§2); avahi (§6); sleep targets = no suspend (always-on mini-PC).
- oneshot judged by live ruleset; `nft -c`-gated at deploy + `_post_nft`.
- fstrim.timer vs continuous discard; cpupower service vs CachyOS freq management; logind Handle*Key=ignore (8 keys incl. LongPress) — no lockout.
- **User-scope health check:** `_vrsv_user_units` asserts `plasma-powerdevil.service` is not failed and reports any failed `systemctl --user` units (Appendix C). Confirm this is the right scope — it is the only user-manager unit the profile inspects, and it exists because of the `POWERDEVIL_NO_DDCUTIL` pairing (§1).
- RTC write-back safety; no ownership conflict with timesyncd.
- Sources: man.archlinux.org systemd.unit + logind.conf + hwclock + timesyncd, wiki.archlinux.org Bluetooth + System_time + Uncomplicated_Firewall.

## §10. Version floors, firmware & known issues

**No floors are enforced in v7.135.1.** `KERNEL_MIN` + `_ir_validate_kernel_floor` were removed at 7.105.x; the Mesa soft-warning `vercmp` check is gone; and no advisory in-script kernel comment survives. The script gates on CPU model only. Everything below is *research guidance*, not a documented contract — do not describe any of it as enforced, and do not recommend restoring a gate (§14 Protected).

Research baselines for this brief (not script values):
- **RTL8127 r8169** support `f24f7b2f3af9` + suspend/shutdown-hang fix `ae1737e7339b` — cite the exact mainline releases. Consequence of error is doc-only.
- **gfx1151 GPU-hang fix is firmware, not kernel:** linux-firmware MES 0x86 (GC 11.5.1). Verify the shipping linux-firmware contains that revision or later; name the minimum release; confirm the kernel version is genuinely irrelevant to the fix (prior audit generations bounced between kernel-6.19 / post-0x83 / 0x86 labels — settle it with git.kernel.org/kernel-firmware evidence, ROCm #5724, Launchpad #2129150).
- **Floor-removal posture:** deploy runs on any kernel and any Mesa. Confirm this is acceptable given both RTL8127 legs are hardware-enablement rather than safety, and state plainly what breaks on a too-old kernel (NIC absent) versus too-old Mesa (RADV feature gaps) so the maintainer can decide whether the fully-ungated posture is still wanted.
- **MT7925:** §6 upstream-fix status; 6.17+ panic/deauth fixes assumed present on any current CachyOS kernel.
- **amdxdna:** the probe failure under `amd_iommu=off` is **`-ENODEV` (ret -19)** — the generated modprobe comment says so in-script. Confirm the errno is still current and watch for a kernel making the NPU IOMMU-optional (which would obsolete the blacklist).
- **Strix Halo ACP:** internal-mic ASoC/UCM still open upstream as of mid-2026; nothing to ship; upstream report is the only action.
- **fish floor:** the script targets fish 3.6+ and the F-1 defect fixed at 7.135.1 was a fish scoping semantic (`set -l` inside an `if` block is not visible after the block). Confirm that semantic is stable across the 3.6-4.x range the profile may encounter, since the fix depends on it being *consistently* true rather than version-dependent (Appendix F1).
- Recommend (advisory only) a current Mesa version for gfx1151 RADV and enumerate open gfx1151 RADV issues.
- Prefer kernel/firmware floors over DKMS for any landed fix.
- Sources: git.kernel.org linux-firmware + r8169 + mt76, gitlab.freedesktop.org/drm/amd + mesa, bugzilla.kernel.org, discuss.cachyos.org, docs.kernel.org accel/amdxdna, wiki.cachyos.org, fishshell.com/docs.

## §11. Security & safety deltas (quantify, ordered — no auto-FIX)

nftables IPv4-only default-deny-inbound (ufw **masked**; ipv6.disable=1): policy drop; ct invalid drop ("early drop of invalid connections"); ct established/related accept; `iif "lo"` accept; IPv4 ICMP {echo-request, destination-unreachable, time-exceeded, parameter-problem} accept (inbound ping ALLOWED); forward drop; output accept; no ICMPv6/NDP. `RY_REMOTE_PLAY_PORTS` (default false) appends TCP {47984, 47989, 48010, 27036, 27037} + UDP {47998-48010, 27031-27036} — measured at +187 B, 729 → 916 B. Rendered ruleset passes `nft -c -f` before commit; `_post_nft` re-validates before reload; restart failure downgrades to applies-at-boot (warned).

**Rule order is unchanged and the question it raised remains open, so it stays on the list:** the ruleset drops invalid *before* accepting loopback. Evaluate: (a) does dropping invalid before the loopback accept risk dropping legitimate loopback traffic that conntrack classifies invalid — the classic argument for `lo` first? (b) is early-invalid-drop a measurable win at this traffic volume, or purely stylistic? Most guidance places `lo accept` first specifically to avoid conntrack edge cases on loopback. **A legitimate FIX candidate if any loopback path can be classified invalid**; otherwise KEEP with a note. Cite nftables/netfilter guidance rather than folklore.

Ordered deltas:
1. **UMIP off** (`clearcpuid=umip`) — descriptor-table base leak, **kernel tainted since 5.19, and upstream states `clearcpuid` is not for production**. Headline open reduction; open maintainer trade-off (§5).
2. **AMD-Vi fully disabled** (`amd_iommu=off`) — no DMA isolation/remapping (USB4/TB, NVMe, NIC DMA unmediated). Named casualty: XDNA 2 NPU blacklisted. Opt-back-in is one validator pair (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`) restoring isolation and NPU together; coupling asymmetry intended (`amd_iommu=on` + blacklist-true valid; false-without-IOMMU refuses).
3. **Plaintext DNS with no validation** (DNSOverTLS=no, DNSSEC=no) to AdGuard 94.140.14.14/.15.15 — observable and spoofable on the wire, and no authentication of answers. **Quantify precisely, then stop: this is a CLOSED decision (§14, §6)** taken deliberately for LAN-wide connectivity reliability, on both the host and the router. State the exposure; do not recommend a change.
4. **IPv6 disabled + inbound IPv4 ping accepted** — net LAN delta = +ping −mDNS (avahi masked unit+socket + resolved MulticastDNS=no close multicast discovery entirely). `_vss_nft` hard-fails on missing echo-request (regression guard); `_vrsv_nft_assert_ping` warns live; do NOT flag ping-accept as a regression.
5. **split_lock_detect=off** — a misbehaving app can degrade the system.
6. **ufw masked rather than removed** — the package remains installed and could be unmasked and started by a user or a future script, at which point two firewall managers contend for the same netfilter tables. Quantify that contention risk against the benefit (reversibility, no package churn). Confirm the nftables-first gate (§9) is the only thing standing between a mis-sequenced run and an unfirewalled window.
7. **No sleep path** — all five sleep/suspend/hibernate targets masked (§9). An always-on box never gets the "locked on resume" checkpoint; note it in the posture summary, but it is a deliberate design for a headless-adjacent mini-PC.
8. **Remote-play ports** (default OFF) — validate the TCP/UDP sets against current Sunshine/Moonlight/Steam docs; default-OFF correct.
9. **Default-deny-inbound ships** — net positive; `flush ruleset` blast radius vs docker/libvirt/podman; no ICMP/new-conn rate limit (trusted-LAN assumption — state it).
10. **NEW — historical data-loss exposure, not a live one.** From 7.109.0 through 7.135.0 the first-adoption preserve for the 13 non-boot destinations never executed, so any pre-existing content at those paths was overwritten with no `.ry.orig` and no `.ry.bak`. The security-relevant members of that set are `/etc/nftables.conf` (a hand-written ruleset would have been replaced by the profile's, silently) and `/etc/NetworkManager/conf.d/99-cachyos-nm.conf`. This is **fixed at 7.135.1** and is void for a host first deployed at ≥7.135.1, but it belongs in the posture summary as a historical exposure with no forensic trace. Quantify what an operator can and cannot recover (Appendix F1, P0 #15).

## §12. HUD & Bluetooth

**MangoHud.conf (19 active + 1 commented, 0600) — unchanged, byte-verified this pass:** horizontal · legacy_layout=0 · position=top-left · toggle_hud=Shift_R+F12 · fps · frametime · frame_timing · gpu_stats · gpu_temp · gpu_core_clock · gpu_power · cpu_stats · `# cpu_temp intentionally disabled — enable if you want CPU temperature in the HUD` · cpu_mhz · cpu_power · vram · ram · font_size=20 · text_outline · background_alpha=0.4. Enabled via MANGOHUD=1. 383 B.
**bluetooth main.conf:** FastConnectable=true · AutoEnable=true · ReconnectAttempts=3. 147 B.
**Companion parity (v1.17.0 ↔ generator):** 19/19 active directives identical in set AND order; byte delta = comment lines only (repo identity header vs installer managed-file header; bare `# cpu_temp` vs installer's expanded comment) — functionally nil, comments are inert to MangoHud. Repo policy: installer is source of truth (repo CHANGELOG 1.14.0 realignment). **HUD-scoped floors per repo:** MangoHud ≥ 0.8.4 (Steam-Overlay Vulkan-layer fix), kernel ≥ 6.14, Mesa 24+ — HUD floors only, do not conflate with §10 (where the profile has no floors at all).

- **cpu_temp / cpu_power — P0 #19, and the record must be stated correctly.** MangoHud issue **#1794 (cpu_power reads 0 on Zen 5 while cpu_temp is active) is STILL OPEN upstream** — describe it as unresolved, not fixed. The profile README now cites the issue directly and states that leaving `cpu_temp` off is deliberate for exactly this reason, which is a new independent confirmation of the dormancy rationale. The actionable lever is **`cpu_custom_temp_sensor`**, added in MangoHud 0.8.3, which lets the HUD be pointed at the k10temp hwmon explicitly and fixes the asusec mispick that causes the wrong sensor to be selected on this board. Evaluate: setting `cpu_custom_temp_sensor` to the k10temp hwmon, and ensuring `energy_uj` is world-readable so `cpu_power` can populate. If #1794 remains open, enabling `cpu_temp` still costs `cpu_power` — so the dormancy is correct until upstream moves, and the custom-sensor option is the path that might allow both. Reconcile against repo README v1.17.0 (k10temp Tctl present but pickup unreliable) and repo CHANGELOG 1.15.0 ("CPU package sensor is not reported on the GTR9 Pro").
- **cpu_power live target:** confirm it populates from Zen 5 RAPL/hwmon under Wayland; blank/zero with `cpu_temp` inactive ⇒ FIX-to-investigate (distinct from the #1794 interaction).
- Byte-exact checks must use the full commented string; `grep -c '^# cpu_temp'` = 1, `grep -c '^cpu_temp'` = 0, `grep -c '^cpu_power'` = 1.
- Confirm all 19 directives valid on current MangoHud; gpu_temp/gpu_core_clock/vram/cpu_mhz populate from amdgpu under Wayland; overhead near-zero with gamescope. `vram` on this UMA part reports the BIOS carveout only — `ram` is the load-bearing figure (§14 special case); a low `vram` reading is not a finding.
- **HUD config is a `.ry.orig` beneficiary.** `~/.config/MangoHud/MangoHud.conf` is a USER_DESTINATIONS member and one of the 13 files that gained a working first-adoption preserve at 7.135.1. A user's own MangoHud.conf is now captured once as `.ry.orig` (0600 user file) instead of being destroyed. This is the exact scenario the fix was validated against. Any §12 recommendation that touches the HUD file should state the preserve behavior so a user is not surprised that only the *first* pre-existing version is kept.
- BlueZ keys current; ReconnectAttempts=3 + backoff sane; AutoEnable fixes adapter-off-at-boot; complements `btusb.enable_autosuspend=n`.
- Sources: github flightlessmango/MangoHud (#1794, #1825, 0.8.3 release notes for `cpu_custom_temp_sensor`), wiki.archlinux.org MangoHud + Bluetooth, mangohud-gtr9-pro v1.17.0 archive (README + CHANGELOG — lockstep policy, floors, directive history).

## §13. Candidate enhancements (absent knobs — gaming-first)

Knobs the profile does NOT set. Anchor every call to gfx1151 / Zen 5 / RDNA 3.5 / current Mesa + Proton-CachyOS. Reserve ADD-as-default for a clear, low-risk frametime/throughput win; never invent a flag; bias KEEP-omitted — the profile is intentionally lean.

**Do not propose re-adding a token this profile deliberately removed** — `nowatchdog`, `tsc=reliable`, `8250.nr_uarts=0`, `AMD_VULKAN_ICD`, `DXVK_LOG_PATH`, `VKD3D_CONFIG`, `ddcutil`, `git-delta`. Those belong in §5/§1/§8 as removal-validation questions, not here as candidates. The one exception is if research shows a removal caused a measurable regression, in which case it is a §5/§1 FIX with evidence, not a §13 ADD. **Note the standing precedent:** `mt7925e.disable_aspm=1` was removed at 7.102.x and re-added at 7.129.0 — removals are not permanent, but a re-add needs the same evidence bar as any FIX.

**Also do not propose a new preflight gate without checking `_ir_validate_sets`.** Three set contradictions are already refused at deploy (§8). A candidate that adds a package or unit which collides with an existing set is not a tuning suggestion — it is a deploy failure.

**13a. Kernel cmdline**
- `mitigations=off` — KEEP-omitted. Zen 5 unaffected by Inception/SRSO-class issues at hardware/microcode level; no measured gaming benefit. Re-open as ADD-opt-in only on a published gfx1151 Proton frametime delta > ~2%. IMPACT Low · RISK Med.
- `amdgpu.ppfeaturemask=0xffffffff` — KEEP-omitted. Undervolt/OC unimplemented on gfx1151 (overdrive/power-cap unsupported, ROCm #5750); CPU undervolt via ryzenadj is the real lever (out of scope). IMPACT Low · RISK Med.
- `preempt=full` — KEEP-omitted, redundant (CachyOS boots full; CONFIG_PREEMPT_DYNAMIC=y).
- `nvme_core.io_timeout` / `pcie_port_pm=off` — KEEP-omitted unless §5 ASPM research shows `pcie_aspm.policy=performance` leaves port PM active in a way that matters; else redundant beside ps_max_latency=0. **The §5 source conflict makes this less settled than it looked at 7.130.0** — if the policy only *biases* rather than disables, `pcie_port_pm=off` deserves a second look.

**13b. RADV / Mesa env**
- `RADV_PERFTEST` — KEEP-omitted (gpl default-on since 23.1; sam auto-on when all VRAM CPU-visible) / UNCERTAIN (nggc — no gfx1151 benchmark).
- `RADV_DEBUG` correctness toggles — KEEP-omitted unless a live gfx1151 rendering bug requires one.
- `MESA_VK_WSI_PRESENT_MODE` / `vblank_mode` / `mesa_glthread=true` — KEEP-omitted (per-game / GL-only).
- `VK_DRIVER_FILES` — evaluate in §1 as the successor to the long-removed `AMD_VULKAN_ICD`, not as a novel candidate. ADD-default only if a second ICD can realistically appear.

**13c. DXVK / VKD3D-Proton**
- dxvk.conf — KEEP-omitted (GPL default-on; numCompilerThreads auto). Legacy DXVK_ASYNC superseded (gplAsyncCache removed in DXVK 2.7) — never recommend the old async patch.
- **`VKD3D_CONFIG` — genuinely absent, so it sits here as a candidate rather than a live value.** Evaluate per §1 P0 #5: if `descriptor_heap` is not default-on upstream, restoring it is an ADD-opt-in with title-level evidence, not an ADD-default. Per-game flags stay per-game.
- Upscaler envs beyond §1 — KEEP-omitted (the FSR4 variable + per-title DXIL workaround is the shipped scope).

**13d. Firmware / platform (verify-only)**
- Resizable BAR / SAM — verify-only, auto-on (all VRAM CPU-visible; RADV auto-enables sam). Optional INFO via rocminfo / lspci BAR.
- BIOS UMA carveout vs GTT — KEEP-omitted for gaming (GTT ~62 GiB never bottlenecks a game; carveout is compute-oriented). The README's own note points at `/sys/module/ttm/parameters/pages_limit` and the ≤96 GiB carveout ceiling.
- BIOS power ceiling — verify-only: README prescribes flat SPL=fPPT=sPPT=85 W + STAPM Boost 0 + TjMax 90 (gains flatten past ~85 W). Installer-external; the only action is consistency — every §2/§13 power statement names its assumed budget. **This remains the load-bearing context for P0 #1**, which is now five releases old with no measurement.

**13e. Scheduler / memory**
- `read_ahead_kb` / `nr_requests` — KEEP-omitted, defaults optimal absent evidence (§4).
- `vm.max_map_count` — KEEP (sufficient; SteamOS value).
- CPU isolation (`isolcpus`, `nohz_full`, `rcu_nocbs`) — KEEP-omitted (hurts a 16C/32T gaming desktop).

## §14. Scope, protected items, special cases

**Scope:** recommendations only — do not emit a modified script. Out of scope: dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS, laptops, UKI, BIOS flashing (README link-out only). Per-game Proton tuning secondary to system-wide config.

**CLOSED MAINTAINER DECISIONS — these are not open questions. Flag a direct upstream contradiction as a note; never as a FIX.**
- **Plaintext DNS on both layers.** Host resolved and the ASUS RT-BE92U router both run plaintext to AdGuard's ad-block tier. Rationale on file: identical filtering either way, DoT buys only ISP query-name privacy, and router-side DoT in Strict mode fails closed — one TLS endpoint becomes a single point of failure for every LAN device. Priority is uninterrupted connectivity for all devices. Quantify in §11; do not re-recommend DoT or DNSSEC.
- **`GPU_DPM_LEVEL=high`, not `profile_peak`.** `high` forces the highest power state with gating still active; `profile_peak` is the technical maximum (adds mclk+pcie, disables gating) but kernel documentation scopes `profile_*` to measurement work, ArchWiki and amdgpu-clocks document auto|low|high|manual as the primary set, ROCm warns STABLE_PEAK is ASIC-specific and unverified on gfx1151, and Phoronix found forced `high` vs `auto` differs only in select cases. Do not re-offer `profile_peak`.
- **No cuts from the script.** `WINEDEBUG=-all`, BlueZ AutoEnable, NM `wifi.backend`, `cachyos-gaming-meta`, `MODULES=(amdgpu)`, `lib32-mesa` all stay. Do not offer removals of these.
- **NEW — the PKGS_ADD orphan is not to be "fixed" with a state file.** Detection needs a persisted manifest of what earlier versions installed; a new state file would add its own drift surface, so it was deliberately not built and is documented instead. A recommendation to add such a manifest must argue against that reasoning explicitly, not around it (§8, P0 #21).
- **NEW — neither orphan class sets DRIFT, by design.** A re-run cannot clear a stale drop-in or an orphaned mask, so returning exit 10 would make `--check` permanently non-zero. Verify reports WARN (drop-ins) / INFO (masks); `--check` records JSONL only. Do not recommend promoting either to DRIFT without addressing the "trains the operator to ignore exit 10" argument (§9, P0 #21).

**Protected (deliberately removed/disabled — do not recommend reinstating unless current upstream directly contradicts the rationale; then flag, not FIX):**
- `pcie_aspm=off` — superseded by `pcie_aspm.policy=performance`. The `off` semantics are settled (`off` does *not* disable ASPM); the *strength* of the policy form is the live conflict in §5. Do not blind-revert in either direction.
- `KERNEL_MIN` + `_ir_validate_kernel_floor` hard floor (removed 7.105.x) **and the Mesa soft-warning floor** — do not recommend re-enforcing either; the profile is deliberately ungated on versions.
- `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` (removed 7.98.x) · `RY_NO_NTP_REMEDIATION` (removed 7.96/97 — escape is masking timesyncd) · `clearcpuid=514` numeric form (renamed 7.94/95) · `archlinux-contrib` (removed 7.101.0) · `60-ry-blacklist-amdxdna.conf` standalone (merged 7.99.0) · `60-ry-mt7925e.conf` standalone drop-in (merged; the *cmdline token* returned at 7.129.0 — the standalone file did not, and both filenames are now actively swept for, §6).
- **Removed and still absent — do not re-propose:** `nowatchdog`, `tsc=reliable`, `8250.nr_uarts=0` (KERNEL_PARAMS); `AMD_VULKAN_ICD=RADV`, `DXVK_LOG_PATH=none`, `VKD3D_CONFIG=descriptor_heap` (ENV_VARS); `ddcutil`, `git-delta` (PKGS_ADD); `modemmanager.service` (MASK). Each is a §5/§1/§8/§9 validation question, not a candidate.
- **Removal reconciliation is asymmetric — audit any future removal against this rule.** Values inside a *generated file* self-heal on the next deploy, because every `_content_*` generator rewrites its file wholesale from the current array. Values in *external system state* do not. **The asymmetry is now three-tier, not two, and the tiers matter:**

```
║ TIER ║ CLASS                  ║ ON REMOVAL           ║ DETECTION AT 7.135.1        ║
║──────║────────────────────────║──────────────────────║─────────────────────────────║
║ 1    ║ value in a generated   ║ self-heals next      ║ n/a — cannot orphan          ║
║      ║ file (KERNEL_PARAMS,   ║ deploy; generator    ║                             ║
║      ║ ENV_VARS, SYSCTL…)     ║ rewrites wholesale   ║                             ║
║ 2    ║ masked unit dropped    ║ stays masked forever ║ DETECTED — _ry_orphan_      ║
║      ║ from MASK              ║                      ║ masked_units (INFO/JSONL)   ║
║ 2    ║ stale 60-ry-* drop-in  ║ stays on disk        ║ DETECTED — _ry_stale_ry_    ║
║      ║                        ║                      ║ dropins (WARN/JSONL)        ║
║ 3    ║ package dropped from   ║ stays installed      ║ NOT DETECTED — by design    ║
║      ║ PKGS_ADD               ║ forever              ║ (needs a persisted manifest)║
```

  **When recommending any removal, state which tier it falls in.** Tier 2 is now reportable but still not self-healing and still never sets DRIFT — the operator must act by hand. Tier 3 is invisible.
- **`ufw` as a package removal** — the profile masks `ufw.service` instead (7.118.0+). Do not recommend returning ufw to PKGS_DEL; the mask is the chosen mechanism and the nftables-first gate is built around it.
- `amdgpu.ppfeaturemask`, `--country` flag, TTM/GTT cap, RADV drirc, MangoHud repo-history removals — `fps_metrics` (added 1.10.0, dropped 1.13.0), `gpu_junction_temp` (hotspot mirrors the edge value), `throttling_status`(+`_graph`) (removed twice), `gpu_mem_clock` + `swap` (meaningless on a shared-memory APU) — do not re-propose without new evidence, `vm.page-cluster`/`vm.vfs_cache_pressure` (vendor-provided), ntsync autoload conf (assert-only), baloofilerc, `_kb_*` subs + `_ry_check_umip_disabled`, ICMPv6/NDP rules (do NOT re-add without restoring IPv6), the linux-firmware version advisory.

**Live config to evaluate KEEP-or-FIX-to-remove (not protected):** `PROTON_FSR4_UPGRADE` (name correctness plus the new "mainly pins a version" claim — §1), `POWERDEVIL_NO_DDCUTIL`, `vm.watermark_boost_factor=0`, `kernel.nmi_watchdog=0` (mechanism check, §3), the ASPM pair (`pcie_aspm.policy=performance` + `mt7925e.disable_aspm=1`, §5), the nftables rule ordering, MangoHud gpu_power/text_outline/toggle_hud/cpu_power, `ipv6.disable=1`, inbound-ping accept, `BLACKLIST_AMDXDNA=true` default (evaluate the NPU-off default, not the mechanism). `cpu_temp` stays a user opt-in. **Open maintainer trade-offs still awaiting decision, all latency-vs-power:** `clearcpuid=umip` (kernel taint), `processor.max_cstate=1` (idle power, compounding with DPM high), `fsck.mode=force` (largely inert on Btrfs).

**Special cases:**
- **IOMMU:** ships `amd_iommu=off`. Do NOT recommend `iommu=pt`/`amd_iommu=on` as default unless ROCm on gfx1151 provably requires it (it does not) OR a DMA-isolation requirement is established; opt-in is per-user and validator-enforced.
- **PCIe ASPM:** ships the policy form **plus** the MT7925 endpoint option. Do NOT flag either as redundant without `lspci -vv` LnkCtl evidence (§5). The host-side audit confirming per-link state has still not been run; treat any claim about actual link state as UNCERTAIN until it is. Do NOT cite wireless.docs.kernel.org for `pcie_aspm=off` semantics — that page is stale.
- **IPv6/nftables:** ships `ipv6.disable=1` + IPv4-only ruleset accepting inbound ping. Do NOT flag ping-accept (asserted regression guard); do NOT re-add ICMPv6/NDP without restoring IPv6. The rule *order* is fair game (§11).
- **Governor/EPP:** `performance` + `performance`. The profile has retracted the old "powersave is required to honor EPP" claim in writing and replaced it with "the governor pins EPP to max and rejects other values". **Verify the replacement claim (P0 #2); do not resurrect the retracted one**, and do not flag the governor as a defect on the strength of either.
- **GPU_DPM_LEVEL:** `high` is decided (above). Evaluate `high` vs `auto` on frametime evidence only; pre-v7.94/95 observations are void.
- **Sleep:** fully masked, no resume path. Do not propose sleep-hook re-assert workarounds for GPU/EPP state.
- **HUD `vram` on UMA:** reports only the BIOS carveout, never the 128 GB pool — do NOT flag a low `vram` reading as misconfiguration; `ram` carries the shared-pool load (§12, companion repo).
- **Version floors:** the profile has none. Do not treat "no floor" as an oversight to fix; treat it as a posture to evaluate (§10).
- **Backups:** 4 boot-critical destinations get `.ry.bak` + post-write verify/restore; the other 13 get a one-time `.ry.orig` on first adoption only. **These are different guarantees.** Do not describe the 13 as backed up on every write, and do not recommend `.ry.orig` for the boot four (it is weaker than what they have).

## §15. VERIFY block (post-reboot)

```fish
cat /proc/cmdline
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver          # amd-pstate-epp
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor        # performance
cat /sys/devices/system/cpu/cpu0/cpufreq/energy_performance_preference   # performance
cat /sys/devices/system/cpu/amd_pstate/status                    # active
cat /sys/devices/system/cpu/amd_pstate/dynamic_epp               # disabled (absent pre-6.16)
cat /sys/devices/system/cpu/amd_pstate/prefcore                  # enabled
cat /sys/devices/system/cpu/cpufreq/boost                        # 1
cat /proc/sys/kernel/nmi_watchdog                                # 0 — set by SYSCTL_VALUES, not a boot token (P0 #6)
systemd-analyze cat-config sysctl.d | rg -n 'nmi_watchdog'       # confirm 95-ry-overrides wins over vendor 70-cachyos-settings
cat /sys/block/nvme0n1/queue/scheduler                           # [none] (adjust node)
cat /sys/class/drm/card*/device/power_dpm_force_performance_level      # high
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l  # 0 — INFORMATIONAL ONLY
rg -o 'amd_iommu=\S+' /proc/cmdline                              # amd_iommu=off
rg -o 'ipv6\.disable=\S+' /proc/cmdline                          # ipv6.disable=1
rg -o 'clearcpuid=\S+' /proc/cmdline                             # clearcpuid=umip
rg -o 'pcie_aspm\S*' /proc/cmdline                               # pcie_aspm.policy=performance
rg -o 'mt7925e\S*' /proc/cmdline                                 # mt7925e.disable_aspm=1
rg -o 'processor.max_cstate=\S+' /proc/cmdline                   # 1
rg -o 'fsck\S+' /proc/cmdline                                    # fsck.mode=force fsck.repair=yes
rg -c 'nowatchdog|tsc=reliable|8250' /proc/cmdline               # 0 — confirms the three removals still hold
sudo lspci -vv | rg 'LnkCtl:.*ASPM'                              # §5 P0 #3: per-link ASPM — OPEN host audit A1, still unrun
cat /sys/module/mt7925e/parameters/disable_aspm 2>/dev/null      # Y/1 — endpoint option live (pairs with the policy)
ls -l /dev/ntsync                                                # present (assert-only)
lsmod | rg -c '^amdxdna'                                         # 0 (blacklisted; loaded = verify FAIL)
sudo dmesg | rg -i 'amdxdna'                                     # expect -ENODEV / -19 if it probes at all
sudo dmesg | rg -i 'AMD-Vi|DMAR'                                 # expect NO "AMD-Vi: Enabled"
ip -6 addr                                                       # expect no IPv6 addresses
cat /etc/modprobe.d/60-ry-modules.conf                           # header + amdxdna blacklist (default)
ls /etc/modprobe.d/60-ry-*.conf                                  # ONLY 60-ry-modules.conf — anything else is swept (§6, now detected)
pacman -Q linux-firmware                                         # §10 MES 0x86 currency check (no version gate exists)
pacman -Q mesa                                                   # §10 advisory only — no Mesa gate exists
pacman -Q pacman-contrib                                         # present
pacman -Qq ddcutil                                               # expect absent (not in PKGS_ADD; POWERDEVIL_NO_DDCUTIL=1 covers the probe)
vulkaninfo | rg -i 'driverName|deviceName'                       # RADV / Radeon 8060S — confirm RADV wins with no ICD pin
systemctl --user show-environment | rg 'PROTON_FSR4_UPGRADE|MANGOHUD|POWERDEVIL_NO_DDCUTIL'   # 1 / 1 / 1
systemctl --user show-environment | rg -c 'VKD3D_CONFIG'         # 0 — removed; nonzero = stale session
systemctl --user is-failed plasma-powerdevil.service             # expect "active"/"inactive", NOT "failed" (_vrsv_user_units)
systemctl --user --failed --plain --no-legend                    # expect empty
sysctl kernel.nmi_watchdog net.ipv4.tcp_congestion_control net.core.default_qdisc vm.max_map_count vm.compaction_proactiveness vm.swappiness vm.watermark_boost_factor
findmnt -no FSTYPE,OPTIONS /                                     # §4: name the root FS before costing fsck.mode=force
findmnt -no OPTIONS -t ext4                                      # noatime,lazytime,commit=10 on ext4 mounts
swapon --show; zramctl                                           # zram active (advisory; not managed)
iw reg get | rg -i country                                       # US
cat /etc/iw-regdomain                                            # COUNTRY=US
resolvectl status | rg -i 'DNS Servers|DNSSEC|DNSOverTLS'        # 94.140.14.14/.15.15 · DNSSEC=no · DoT=no — §11 item 3
resolvectl query example.com                                     # confirm the AdGuard pair answers, not a DHCP per-link server (P0 #14)
rg -n 'global-dns-domain' /etc/NetworkManager/conf.d/99-cachyos-nm.conf   # servers= line present — the mechanism that beats per-link DNS
sudo nft list chain inet filter input                            # policy drop; invalid-drop FIRST, then est/rel, then lo; IPv4 ICMP incl echo-request
sudo nft -c -f /etc/nftables.conf                                # syntax-valid
systemctl is-enabled ufw.service                                 # masked (NOT "not installed" — 7.118.0+ masks rather than removes)
systemctl is-active ufw.service                                  # inactive
systemctl is-enabled sleep.target suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target   # masked ×5 — no resume path
systemctl list-unit-files --state=masked,masked-runtime --no-legend --plain   # §9: compare against the 11 MASK members — extras are orphans
stat -c '%a %U:%G' /etc/NetworkManager/system-connections/*      # 0600 root:root
systemctl is-enabled bluetooth.service                           # enabled
systemctl is-enabled avahi-daemon.service avahi-daemon.socket    # masked masked
grep -c '^cpu_temp' ~/.config/MangoHud/MangoHud.conf             # 0
grep -c '^cpu_power' ~/.config/MangoHud/MangoHud.conf            # 1
grep -c '^# cpu_temp' ~/.config/MangoHud/MangoHud.conf           # 1 (byte-exact checks use the full string)
grep -c '^[a-z]' ~/.config/MangoHud/MangoHud.conf                # 19 active (companion v1.17.0 parity: set + order identical)
cat /sys/class/hwmon/hwmon*/name | rg -c '^k10temp$'             # ≥1 — P0 #19 sensor presence — INFORMATIONAL ONLY
sensors 2>/dev/null | rg -i -A1 '^k10temp'                       # Tctl populates? — §12 cpu_custom_temp_sensor target evidence
stat -c '%a' /sys/class/powercap/*/energy_uj                     # world-readable? — required for MangoHud cpu_power (§12)
```

**Backup-surface audit for P0 #15** (new; establishes what the 7.109.0-7.135.0 window cost on this host):

```fish
ls -l /etc/kernel/cmdline.ry.bak /etc/sdboot-manage.conf.ry.bak /boot/loader/loader.conf.ry.bak /etc/mkinitcpio.conf.ry.bak 2>/dev/null   # the 4 boot targets — expect present after any deploy
ls -l /etc/*.ry.orig /etc/**/*.ry.orig ~/.config/**/*.ry.orig 2>/dev/null   # first-adoption preserves — ABSENT for anything deployed before 7.135.1
rg -c 'PREEXISTING_PRESERVED|PREEXISTING_PRESERVE_FAIL' ~/ry-install/*.jsonl   # 0 on every pre-7.135.1 log = the dead-code window, confirmed
rg -c 'MODPROBE_STALE_DROPIN|MASK_ORPHAN' ~/ry-install/*.jsonl               # §6/§9 orphan records — new keys, only in ≥7.135.0 logs
```

**Idle-floor measurement for P0 #1** (the headline question, unrun for five releases — idle desktop, 60 s, before and after any perf-value change):

```fish
sudo turbostat --quiet --show PkgWatt,Busy%,Bzy_MHz,PkgTmp --interval 5 --num_iterations 12
cat /sys/class/drm/card*/device/hwmon/hwmon*/power1_average       # GPU package draw at idle
rg . /sys/devices/system/cpu/cpu*/cpuidle/state*/name             # confirm C-state depth available under max_cstate=1
```

**Hard `--verify` asserts (mismatch → exit 1/3):** every KERNEL_PARAMS token (15) + `rw` in /proc/cmdline (`_vrk_cmdline` generic loop); scaling_driver/governor/EPP/amd_pstate status/prefcore/boost/`dynamic_epp=disabled`; GPU `power_dpm_force_performance_level=$GPU_DPM_LEVEL` (comparison QUOTED); usbcore.autosuspend=-1, nvme_core ps_max_latency=0, zswap∈{N,0}, **nmi_watchdog=0**, NVMe `[none]`; managed modprobe blacklist entries NOT loaded (`_vrkm_blacklist_modprobe`); live mkinitcpio COMPRESSION/_OPTIONS match; regdom; nftables echo-request present (`_vss_nft` hard guard) + live warn (`_vrsv_nft_assert_ping`); NM system-connections 0600 root:root; PKGS_ADD 16 + Vulkan pkgs (`_vsp_required`); MASK 11 (`_verify_static_services`); SYSCTL_VALUES 11 (`_vss_sysctl` + `_vre_sysctl_runtime`); ENV_VARS 10 per-var (`_vre_envvars`, dynamic — no hardcoded names). Presence checks comment-proof (`_chk_grep` strips inline comments).
**Non-failing reports (new at this generation, exit 0):** unmanaged `60-ry-*.conf` drop-ins → `_vss_modprobe` WARN; masked units absent from `$MASK` → `_vss_orphan_masks` INFO. Neither sets DRIFT and neither changes the exit code — read them, do not grep for a non-zero status (§9, P0 #21).
**REMOVED asserts — do NOT verify:** `_vrkm_iommu`, `_vrk_clocksource`, `_vre_zram`, `_vre_tcp` (gone since 7.90.0); kernel-floor and Mesa-floor checks (both gone); no THP, KSM, `ttm.*`, drirc, `iommu=pt`, ICMPv6/NDP, baloo, or `_kb_*` assert exists. **No `VKD3D_CONFIG` assert exists** — the variable left ENV_VARS, and `_vre_envvars` iterates the array dynamically, so the verifier followed automatically with no edit.
---

# Appendix A — install-phase model (validate sequence, not prose)

```
1 Preflight     _install_preflight          — _ir_* gates (counts 21, keys incl BLACKLIST_AMDXDNA + charsets/metachar, SET CONTRADICTIONS, post-hooks, root UUID) + sysctl.d backup-target guard + leading-dash package guard. NO kernel-floor, NO mesa gate.
2 Packages      _install_packages           — mkinitcpio.conf pre-deployed -> pacman -Syu; PKGS_ADD (16) re-marked -D --asexplicit; chwd Vulkan
3 Configuration _install_system_files       — render+deploy 17 files (atomic tmp+rename); format-validate pre-write; nftables additionally nft -c; first-adoption .ry.orig preserve for the 13 non-boot destinations
4 Services      _install_configure_services — fstab -> resolved -> PKGS_DEL (-Rns) -> mask (nftables-first, ufw flush-then-mask; MASK 11) -> iwd handoff -> enable -> regdom -> NTP (chronyd/ntpd/openntpd guard) -> RTC write-back
5 Boot          _install_rebuild_boot       — taint-gate -> mkinitcpio -P -> sdboot-manage gen/update (gated on boot-critical writes)
6 Finalize      _install_finalize           — user daemon-reload -> paccache (-rk2, -ruk0) -> NetworkManager restart
```

Phase names are live-asserted from `_RY_PHASE_NAMES` (6) and mirrored 1:1 across the array, the progress reporter, the orchestrator, the section headers, and the README. `_PROG_STEPS` is DERIVED from `$_RY_PHASE_NAMES`, so phase order cannot drift between the reporter and the orchestrator.

- **Phase 1 gained a fourth validator at 7.135.0.** `_init_runtime` now calls `_ir_validate_counts` → `_ir_validate_keys` → `_ir_validate_sets` → `_ir_validate_post_hooks` (L781-784). `_ir_validate_sets` (L736) refuses deploy on `PKGS_ADD ∩ PKGS_DEL`, `MASK ∩ EXPECTED_SERVICES`, and `MASK ∩ _RY_PKG_MANAGED_SERVICES`; each raises `_err_loud` + `_pre_dispatch_exit $EXIT_PREFLIGHT`. All three shipped intersections are empty. Confirm the ordering is right — sets are checked after keys and before post-hooks, which means a domain-invalid scalar aborts before a set contradiction is ever reported.
- Firewall handoff lives in Phase 4: nftables is enabled and its default-deny ruleset confirmed live **before** ufw is flushed and `ufw.service` masked; on an unconfirmed ruleset the mask is withheld for the run (§9). Phase-5 regeneration fires only when a `_RY_BOOT_CRITICAL_DSTS` member changed. Flag any recommendation moving a cmdline/mkinitcpio change outside the Phase-5 gate.
- `_RY_BOOT_CRITICAL_DSTS` (4) = `_RY_BACKUP_TARGETS` (derived, count-asserted): /boot/loader/loader.conf, /etc/kernel/cmdline, /etc/sdboot-manage.conf, /etc/mkinitcpio.conf — all get `.ry.bak` + post-write verify/restore (plus fstab during rewrite). Preflight refuses a side-effecting generator in the backup set (sysctl.d guard). **The remaining 13 destinations get `.ry.orig` on first adoption only** — a different and weaker guarantee (Appendix F1).
- `_RY_POST_HOOKS` (17 entries, 16 distinct tags — `boot` is shared; dispatch FIRST-MATCH-WINS by list order, so ordering is load-bearing), live-dumped in declaration order: `/boot/*|loader`, `/etc/kernel/cmdline|cmdline`, `/etc/sdboot-manage.conf|boot`, `/etc/mkinitcpio.conf|boot`, `*/resolved.conf.d/*|resolved`, `*/logind.conf.d/*|logind`, `*/NetworkManager-dispatcher.service.d/*|nmdispatch`, `*/NetworkManager/conf.d/*|nm`, `/etc/iw-regdomain|regdom`, `/etc/bluetooth/main.conf|bluetooth`, `/etc/nftables.conf|nft`, `/etc/default/cpupower-service.conf|cpupower`, `*/sysctl.d/*|sysctl`, `/etc/udev/rules.d/*|udev`, `*/modprobe.d/*|modprobe`, `*/environment.d/*|envd`, `*/MangoHud/MangoHud.conf|mangohud`. `_ir_validate_post_hooks` refuses any tag lacking `_post_<tag>`. Handler names are `_post_<tag>` where tag is the part AFTER the `|`.
- Boot family shares `_post_boot_apply <target> <skip_mki>`: `_post_boot` -> skip_mki=false (full mkinitcpio -P); `_post_cmdline`/`_post_loader` -> true (sdboot regen only). Notify-only: `_post_logind`, `_post_modprobe` (reboot), `_post_envd`/`_post_mangohud` (session). `_post_envd` additionally attempts a `plasma-powerdevil.service` restart so `POWERDEVIL_NO_DDCUTIL` applies without a re-login, warning non-fatally when that fails — confirm the restart is the right remediation and that failure is genuinely non-fatal. `_post_nm` DEFERS the NM restart when Wi-Fi is the active route — confirm the deferred restart lands (Phase 6) and is surfaced.
- All destinations + `--install-file` values canonicalized via `realpath -m` at load (failure warns, falls back literal). Unmatched patterns log `POST_HOOK_NONE`; unchanged bytes log `POST_HOOK_SKIP_UNCHANGED`; `_post_udev` runs `udevadm verify` (systemd >=254) before reload + retrigger (block AND cpu).
- **`--check` orphan recording is NOT a phase.** `_check_record_orphans` runs from `_ry_do_check` at L2618, after the sudo/systemctl preflight bail (L2616-2617) and before the `_check_phase_files` / `_check_phase_cmdline` / `_check_phase_units` loop. Placement is deliberate: an earlier attempt put the records at the tail of the phase functions, where an early phase bail skipped them. JSONL order on a bailing run is CHECK START → MODPROBE_STALE_DROPIN → MASK_ORPHAN → CHECK_PREFLIGHT → CHECK END.

# Appendix B — exact rendered bodies (validate content, not paraphrase)

**Provenance:** every body below was produced by executing the v7.135.1 `_content_*` generator for that destination in a sandbox (non-root, truncated-execution harness cut at **L4834**) and capturing stdout verbatim. All 17 generators returned rc=0; all 17 outputs are byte-identical across three consecutive renders (determinism verified by per-file content hash, 17/17; sorted-manifest sha256 `18b4d210b6fbd519` — see the determinism gotcha in §Method). Both `BLACKLIST_AMDXDNA` states and both `RY_REMOTE_PLAY_PORTS` states were rendered — **19 bodies total**. Every generator emits a leading `#` header line; byte-exact/checksum comparisons include it.

**Every fence in this appendix is byte-identical to the 7.130.0 edition.** That was verified by diffing the previous edition's rendered fences against fresh 7.135.1 output: 18 of 20 fences matched exactly, and the 2 that did not are the size table below (not a body) and the 3-line remote-play fragment in B3 (published as a diff, not a whole file). **This 18/20 pattern is expected and is not a defect** — a whole-file match test will always report the remote-play variant absent.

**Measured sizes (as written files — `$fn > tmp; stat -c %s`, never `| string collect`, which strips trailing newlines and under-reports the total by 17 B):**

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

Content hashes are `$HOME`-independent; only destination PATHS differ per user. The udev rule at 639 B and the 5,093 B total remain the anchors for any perf-value change (a value substitution plus its comment edits moves both).

## B1. /etc/kernel/cmdline + /etc/sdboot-manage.conf + /etc/mkinitcpio.conf
```
rw root=UUID=<_ROOT_UUID> amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0
```

```
# sdboot-manage configuration — changes require: sudo sdboot-manage gen && sudo sdboot-manage update
LINUX_OPTIONS="amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0"
LINUX_FALLBACK_OPTIONS="quiet"
DEFAULT_ENTRY="manual"
REMOVE_EXISTING="yes"
OVERWRITE_EXISTING="yes"
REMOVE_OBSOLETE="yes"
```

```
# mkinitcpio configuration — changes require: sudo mkinitcpio -P && sudo sdboot-manage update
MODULES=(amdgpu)
BINARIES=()
FILES=()
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
COMPRESSION="zstd"
COMPRESSION_OPTIONS=(-1 -T0)
```

- **15 tokens** in both the cmdline and `LINUX_OPTIONS`, sorted identically. `mt7925e.disable_aspm=1` sits between `ipv6.disable=1` and `nvme_core...` (§5).
- The cmdline generator reads `$_ROOT_UUID` (SINGLE underscore), not `$_RY_ROOT_UUID`. Any reproduction harness must set it, or the generator returns `EXIT_GEN_NOUUID` (12) and `_ry_validate_configs` returns rc 3, which looks like a regression.
- `LINUX_FALLBACK_OPTIONS="quiet"` strips all 15 — the fallback exposure in §7.

## B2. /boot/loader/loader.conf
```
# systemd-boot loader configuration
default @saved
timeout 0
console-mode keep
editor no
```

## B3. /etc/nftables.conf (validate rule-by-rule)
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

With `RY_REMOTE_PLAY_PORTS=true` (NOT the default) the input chain gains three lines before the closing brace, taking the file from 729 B to 916 B (+187 B):

```
        # ry-install: remote-play inbound (RY_REMOTE_PLAY_PORTS=true)
        tcp dport { 47984, 47989, 48010, 27036, 27037 } accept
        udp dport { 47998-48010, 27031-27036 } accept
```

- Rule order is `invalid drop` -> `established,related accept` -> `iif "lo" accept`. The §11 question is whether any loopback path can be conntrack-classified invalid and therefore dropped before the `lo` accept is reached.
- `icmp type { echo-request, ... } accept` is a HARD-ASSERTED regression guard (`_vss_nft` fails on its absence). Inbound ping is intentional; never flag it.
- This file is one of the 13 that gained a working `.ry.orig` preserve at 7.135.1 — a hand-written ruleset at this path is now captured once instead of destroyed (§11 item 10, Appendix F1).

## B4. /etc/udev/rules.d/99-ry-perf.rules
```
# ry-install: udev performance rules (managed file, do not edit by hand)
# NVMe scheduler none (lowest tail latency; diverges from CachyOS kyber default)
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
# AMD P-State EPP performance (maximum CPPC hint)
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="performance"
# GPU performance level (gfx1151 clock-floor; forced high)
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="high"
```

- **639 B, unchanged.** The two perf comments are `# AMD P-State EPP performance (maximum CPPC hint)` (49 ch) and `# GPU performance level (gfx1151 clock-floor; forced high)` (58 ch). Both moved line number (L875/L877 → **L887/L889**) without changing a byte; the generator's own `--description` at **L882** also names "EPP performance" and is the fourth script-side perf site (§2).
- The GPU rule is `ACTION=="add"` only (not `add|change`) — add-only, no re-assert after a GPU reset. With all sleep targets masked (§9) there is no resume event, so enumeration is the only trigger that matters.
- EPP interpolates `$EPP_PREFERENCE` unquoted into the ATTR, which is why `_ir_validate_keys` L708 gates it against `_RY_EPP_LEVELS` before any write. **P0 #2 asks whether this write can land at all** under a `performance` governor — if the driver rejects it, the rule is issuing a doomed write and the failure would surface in the udev log, not in the file.

## B5. /etc/systemd/resolved.conf.d/99-cachyos-resolved.conf
```
# systemd-resolved: AdGuard upstreams, plaintext, mDNS/LLMNR off
[Resolve]
DNS=94.140.14.14 94.140.15.15
MulticastDNS=no
LLMNR=no
DNSOverTLS=no
DNSSEC=no
```

- `DNS=` carries the AdGuard ad-block pair. **This line alone is not sufficient under NetworkManager** — per-link DHCP DNS outranks it (systemd #33973). The operative mechanism is the NM drop-in in B6.
- `DNSSEC` and `DNSOverTLS` render from the `RESOLVED_DNSSEC` / `RESOLVED_DOT` globals under RENAMED keys, so a literal `grep DNSSEC=no` on the script returns nothing. The rendered body is ground truth.
- **154 B, byte-identical to 7.130.0.** A previous rebase misattributed a `DNSOverTLS` change to this file; there has been none. Verify any claimed drift here against the previous edition's rendered body, never against recollection.

## B6. NetworkManager 99-cachyos-nm.conf + dispatcher logging.conf
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

```
# LogLevelMax drops info-level dispatcher lines (journald-logged; StandardError=null ineffective)
[Service]
LogLevelMax=notice
```

- **`[global-dns-domain-*] servers=` is the load-bearing DNS line on this system** (P0 #14), not the resolved `DNS=`. Note the empty `[global-dns]` section header is required to open the block.
- `wifi.powersave=2` = OFF (mt76 software PS causes latency spikes). The value 2 means disabled in NM's enum — do not read it as a level.

## B7. /etc/iw-regdomain
```
# ry-install: wireless regulatory domain (managed file, do not edit by hand)
COUNTRY=US
```

## B8. /etc/bluetooth/main.conf + /etc/default/cpupower-service.conf
```
# ry-install: BlueZ daemon config (managed file, do not edit by hand)
[General]
FastConnectable=true

[Policy]
AutoEnable=true
ReconnectAttempts=3
```

```
# cpupower-service.conf — sourced by /usr/lib/systemd/scripts/cpupower (cpupower.service)
GOVERNOR='performance'
```

- `GOVERNOR='performance'` at 115 B. This file is the governor's only delivery path; `CPUPOWER_GOVERNOR` is renamed to `GOVERNOR=` on the way out, which is why grepping the generated file for the script's variable name returns nothing.

## B9. ~/.config/MangoHud/MangoHud.conf (19 active + 1 commented + file header)
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

- 19 active directives, 1 commented (`cpu_temp`), 1 header, 383 B. Byte-identical across 7.106 → 7.135.1 — the HUD generator has not changed in nine releases. Companion `mangohud-gtr9-pro` v1.17.0 parity holds: same set, same order, comment-only byte delta.

## B10. /etc/modprobe.d/60-ry-modules.conf
```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# blacklist amdxdna: XDNA NPU needs IOMMU, probes -ENODEV (ret -19) under amd_iommu=off
blacklist amdxdna
```

With `BLACKLIST_AMDXDNA=false` (the NPU opt-in path) the file renders comment-only — `_grep_modprobe_entry` accepts this:

```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# no directives: BLACKLIST_AMDXDNA=false (NPU path); MT7925 ASPM handled on the kernel command line
```

- The errno is `-ENODEV (ret -19)`; the false-state comment reads "MT7925 ASPM handled on the kernel command line", tracking the §5 token re-add. Both unchanged since 7.130.0.
- Neither the default nor the opt-in body references `60-ry-mt7925e.conf` or `60-ry-blacklist-amdxdna.conf` — and it no longer needs to. **Those filenames are now swept for at runtime** by `_ry_stale_ry_dropins`, which matches `60-ry-*.conf` excluding `60-ry-modules.conf` (§6). The four-generation gap this appendix carried is closed.

## B11. /etc/systemd/logind.conf.d/99-cachyos-logind.conf (LOGIND_IGNORE_KEYS 8)
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

- 8 keys, each rendered `Handle*Key=ignore`. This is one of only two service keys that reach their destination under their own name (the other is `COUNTRY=`); the other 18 are renamed on the way out. The profile README now documents the emitted form alongside the key.
- Inert keypresses only — this is NOT the sleep-masking mechanism (that is MASK, §9). Do not conflate them.

## B12. /etc/sysctl.d/95-ry-overrides.conf (SYSCTL_VALUES 11 - 3/6)
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

- 11 keys, 441 B. `kernel.nmi_watchdog = 0` leads the file (P0 #6).
- Stored `k=v` in the array, emitted `k = v`. Any parity check must normalise whitespace.
- The header annotation is SELECTIVE by design (it names the priority-95 ordering, not every key) at 64 ch against a <=66 cap. There is no room to annotate more; do not propose "fixing" it.

## B13. ~/.config/environment.d/10-environment.conf (ENV_VARS 10 - 1; 0600)
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

- 10 vars, 311 B, unchanged since 7.130.0. `PROTON_FSR4_UPGRADE=1` is the current FSR4 name (P0 #4); `VKD3D_CONFIG=descriptor_heap` remains absent (P0 #5).
- `_vre_envvars` iterates `$ENV_VARS` dynamically with no hardcoded names, so any future add/remove/rename propagates to the runtime verifier with **no verifier edit** — there is no stale assert to find for a removed variable.
# Appendix C — verify surface (assert ownership per recommendation)

A recommendation that changes a value MUST state which sub asserts it (hard-fail vs warn). **60 verify functions** (`^function _v`) live-enumerated from the v7.135.1 source: **12 `_verify_*` orchestrators + 48 subs** (was 12 + 47). The single added sub is `_vss_orphan_masks`. Total script functions moved 287 → 292; `sub:` parent markers 91 → 93. Cite the sub, not the orchestrator, when attributing an assert.

**The 5 functions added since the 7.130.0 pin, all audit-relevant:**

```
║ FUNCTION                 ║ ROLE                                              ║
║──────────────────────────║───────────────────────────────────────────────────║
║ _ir_validate_sets  L736  ║ preflight #3 of 4: refuses 3 set contradictions    ║
║ _ry_stale_ry_dropins     ║ shared sweep: unmanaged 60-ry-*.conf drop-ins      ║
║ _ry_orphan_masked_units  ║ shared diff: masked units absent from $MASK        ║
║ _vss_orphan_masks        ║ _verify_static_services sub — INFO (unattributable)║
║ _check_record_orphans    ║ _ry_do_check sub L2618 — JSONL only, never DRIFT   ║
```

- **Static boot** `_verify_static_boot` → `_vsb_loader` · `_vsb_sdboot` (LINUX_OPTIONS token set + keys) · `_vsb_cmdline` (/etc/kernel/cmdline token set — all 15 byte-asserted here) · `_vsb_mkinitcpio` (HOOKS/MODULES + live COMPRESSION/_OPTIONS via `_ry_mkinitcpio_array`, multi-line join, last-wins warn) · `_vsb_entries` (BLS entries + count; loader-entry paths realpath-canonicalized, WARNED textual-join downgrade when realpath absent). All hard-fail.
- **Static system** `_verify_static_system` → `_vss_logind` · `_vss_nmdispatch` · `_vss_nm` · `_vss_sysctl` (11 keys) · `_vss_regdom` · `_vss_bluetooth` · resolved inline `_chk_grep` loop · cpupower inline `_chk_grep GOVERNOR` · `_vss_udev` (all 3 rules; EPP from `$EPP_PREFERENCE`; GPU_DPM-aware) · **`_vss_modprobe`** (blacklist grep iff default-true, **plus the unmanaged `60-ry-*` sweep — WARN, non-failing**) · `_vss_nft` (hard-fail on missing echo-request).
- **Static user** `_verify_static_user` — ENV_VARS per-var `_chk_grep` + MangoHud `_chk_file` + `_chk_grep "fps"`. `_chk_grep` is comment-proof (awk strips inline comments, skips comment-only lines before `grep -wF`); "no non-comment lines" is a distinct FAIL; mid-read sudo lapse is a distinct warn.
- **Static packages** `_verify_static_packages` (pacman absent ⇒ warn-skip; `pacman -Qq` failure ⇒ warn-skip but `_vsp_pacman_conf` still runs) → `_vsp_required` (PKGS_ADD **16** + Vulkan, pacman-db-lock guard) · `_vsp_removed` · `_vsp_pacman_conf` (sudo-read fallback; grep rc>1 → warn-skip) · **`_verify_static_services`** (MASK **11**, plus the new `_vss_orphan_masks` INFO) · `_verify_static_syntax` · `_verify_static_checksum` → `_vsc_check_one` (embedded SHA256 == installed; graceful skip on EXIT_GEN_NOUUID).
- **Pre-deploy format validators** (`_ry_validate_configs` → `_rvc_dispatch`): `_grep_kv`, `_grep_kparam`, `_grep_sysctl_kv`, `_grep_modprobe_entry` (comment-only OK; else every non-comment line ∈ options/blacklist/install/alias/softdep/remove), `_grep_regdomain_entry`, `_grep_udev_entry`, `_grep_nft_entry`, `_grep_envd_entry`, `_grep_cpupower_entry`, `_grep_mangohud_entry`, `_grep_ini_header`; mkinitcpio case REQUIRES `MODULES=(`, `HOOKS=(`, `COMPRESSION="` lines; nftables additionally `nft -c -f` on the rendered tmpfile. **`_vmh_*` are mkinitcpio HOOK validators, not MangoHud** — `_ry_validate_mkinitcpio_hooks` dispatches to `_vmh_existence_only` or `_vmh_order_checks`.
- **Runtime kernel** `_verify_runtime_kparams`: `_vrk_cmdline` (every token + rw; preemption INFO from one cached `sudo -n dmesg`) · `_vrk_gpu_state` (QUOTED compare against `$GPU_DPM_LEVEL` = `high`) · `_vrk_cpu_state` (driver/governor/EPP/status/dynamic_epp/prefcore/boost — governor asserts `performance`, EPP asserts `performance`) · `_vrk_module_state` → `_vrkm_amdgpu` (hex-aware, no-op without amdgpu.*) · `_vrkm_blacklist` (module_blacklist= cmdline scan — currently no-op) · `_vrkm_blacklist_modprobe` (managed-content parse, `-`→`_` normalize, lsmod check; amdxdna LOADED ⇒ FAIL; lsmod absent ⇒ warn) · usbcore/nvme_core/zswap/**nmi_watchdog**/NVMe-none asserts.
  - `_vrkm_blacklist_modprobe` remains **generator-sourced** — it checks intended content, not on-disk extras. That was the mechanism behind the old §6 blindness; the fix did not change this function, it added a separate sweep alongside it (`_vss_modprobe`). Attribute the drop-in sweep to `_vss_modprobe`/`_ry_stale_ry_dropins`, never to `_vrkm_blacklist_modprobe`.
- **Runtime services** `_verify_runtime_services`: `_vrsv_sys_units` · `_vrsv_masked_inactive` (iterates `$MASK` — covers the avahi pair, `ufw.service`, and all five sleep targets) · `_vrsv_user_units` · `_vrsv_wifi` · plus `_vrsv_chk_active_enabled`, `_vrsv_nft_assert_ping` (warn), `_vrsv_chk_nftables` (oneshot judged by live policy drop), `_vrsv_chk_resolved`, `_vrsv_chk_cpupower_governor`, `_vrsv_wifi_nm_backend` (no iwd path; `_vrsv_wifi` skips when `_RY_PROFILE_USES_WIFI_BACKEND=false`; wlan via /sys/class/net/*/wireless; closes with firewall-posture INFO reporting `ufw=` and `nft_rules=`).
  - **`_vrsv_masked_inactive` still iterates `$MASK` alone** — it asserts that every *declared* mask is present and inactive. The *reverse* direction (masked units not declared) is the new `_vss_orphan_masks`, which lives on the **static** services path, not here. Do not conflate the two when attributing an assert.
  - **`_vrsv_user_units`** — skips cleanly with an INFO when no active user-bus; skips when `plasma-powerdevil.service` is not a known unit; hard-FAILs when the unit is failed (message points at `journalctl --user -u plasma-powerdevil -b` and `coredumpctl list org_kde_powerdevil`); additionally warns on any failed `systemctl --user` unit. This is the profile's only user-scope unit assertion and exists to catch the PowerDevil crash-loop class.
- **Runtime env** `_verify_runtime_env`: `_vre_envvars` (systemctl --user show-environment; quoted-value unwrap; **iterates `$ENV_VARS` dynamically with no hardcoded names**) · `_vre_sysctl_runtime` (/proc/sys, 11 keys) · `_vre_fstab` (ext4 noatime,lazytime,commit=10) · `_vre_ntsync` · `_vre_regdom` (iw reg get).
- **Runtime session** `_verify_runtime_session`: `_vrs_nm_perms` (0600 root:root) · `_vrs_installed_file_perms` (system 0644 / user 0600) · `_vrs_parent_dirs` → `_vpd_dir_perm_check` (0755/0700); `_vrs_vfat_skip` guards BOTH loops (vfat/undetermined $BOOT counted-skipped with INFO).
- **Aggregation** `_ry_verify_all`/`_verify_summary`: static first, runtime second; per-stage summaries summed; runtime preflight bail restores static totals; a static FAIL outranks the runtime bail code. Confirm no path zeroes static counters after a runtime bail.
- **Actionables at this pin** (two of the previous five are now closed):
  - (a) ~~leftover-file blindness~~ — **CLOSED**, `_vss_modprobe` sweep (§6).
  - (b) ~~stale-mask blindness~~ — **CLOSED (detect)**, `_vss_orphan_masks` (§9). What remains open is the *severity* question (P0 #21) and the fact that neither is self-healing.
  - (c) COMPRESSION multi-line/duplicate tolerance without false FAIL.
  - (d) comment-strip safety holds while no managed value contains `#` (boot-scalar metachar gate forbids it) — re-check on any new value.
  - (e) removed effect-asserts leave directive-level coverage intact; iwd narrowing deliberate (Low/Low).
  - (f) with no ICD variable in ENV_VARS, `_vre_envvars` asserts no ICD selection — the only Vulkan coverage left is `_vsp_required`'s package presence check (§1/§8).
  - (g) **NEW — no verify path asserts that `.ry.orig` exists**, and none can: absence is indistinguishable from "the destination had no pre-existing content", which is the common case. The preserve is best-effort and unverifiable by design (Appendix F1). State this when recommending reliance on it.

**Sandbox artifact, not a regression:** `_ry_validate_mkinitcpio_hooks` returns rc 1 and `_ry_validate_configs` returns rc 3 in a container because `/etc/mkinitcpio.conf` is absent. Always A/B a nonzero validator rc against the previous release before calling it a finding.

# Appendix D — fstab rewrite (`_install_fstab_opts`)

- Adds `noatime,lazytime,commit=10` to **ext4 field 4 only**; every other column and non-ext4 row byte-preserved; purely-numeric $4 rows pass through to the malformed guard — confirm they are then caught, not shipped.
- **The ext4-only scope is load-bearing for §4.** If the root filesystem is Btrfs, this rewrite never touches it, and `fsck.mode=force` is largely inert there too. Establish the actual root FS before costing either item (§15 has the one-line check).
- Verify-side conflict list exact: `defaults`, `relatime`, `atime`, `strictatime` (presence = rewrite-pending FAIL); existing `commit=` rewritten to 10; non-10 overrides tracked in `_RY_FSTAB_COMMIT_OVERRIDES` (surfaced). The profile README describes this as normalizing away redundant tokens.
- Gates: line-count parity + size floor + mandatory `findmnt --verify`; symlinked or whitespace-split /etc/fstab refused, not corrected.
- `/etc/fstab` is not a managed destination and never took part in the `.ry.orig` class; it has always had its own `.ry.bak` during rewrite. It is the one non-boot path with backup parity to the boot four.
- Confirm: idempotent; atomic (tmp+rename, `.ry.bak`); commit=10 vs every-boot fsck coherence (§4).

# Appendix E — preflight gates & exit codes

Init-time capability probes FIRST: `id` (hard-require, non-numeric `id -u` refuses) → `timeout --foreground --kill-after` → `find -maxdepth/-printf` → `mv -T` live-probe (two mktemp files, /tmp — vfat semantics untested by design) → `stat` → `date %z`; each rejects busybox/uutils (exit 3). TMPDIR erased (tmp pinned /tmp); umask set as the VARIABLE; `--check` silence pinned pre-argparse. Dependency gate: **37 hard-required commands + 20 warn-only optional tools** (both re-counted at this pin, unchanged) + `df --output` probe + systemd ≥250. Destinations canonicalized (`realpath -m`, literal fallback). `test -w /tmp` at **L161** bails rc3 before anything else — **relevant to any sandbox reproduction of this audit** (create a non-root user AND ensure /tmp is writable by it).

`_RY_ARGPARSE_SPEC` is **6** entries (single option-spec source shared by the root guard and the main argparse, declared with `set -g --` at L14): `--exclusive=verify,check,install-file`, `h/help`, `v/version`, `verify`, `check`, `install-file=`.

- `_ir_resolve_root_uuid` → EXIT_GEN_NOUUID 12; mode-scoped: `--install-file` FATAL only when target IS /etc/kernel/cmdline; else warn-continue; `--verify` warn-continues with generic root=UUID check.
- Hardware gate (CPU match; sole override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed unreadable; `--verify` warns). Deploy-mode hardware hard-gate precedes canon match; `SKIP_HW=1` on an unmanaged target → rc2, managed → generate+validate → sudo gate. Glued short flags early-intercept, first char wins (`-hv`/`-hV` → help, `-vh` → version).
- **Validator chain, now four deep (L781-784):** `_ir_validate_counts` (21 tripwires) → `_ir_validate_keys` (bool/yes-no/int enums; ISO-3166 COUNTRY with reserved-range rejection; GPU_DPM ∈ `_RY_DPM_LEVELS` at L707; EPP ∈ `_RY_EPP_LEVELS` at L708; governor regex `^[a-z][a-z0-9_-]*$` at L709; nftables↔ipv6.disable coupling; BLACKLIST_AMDXDNA=false↔IOMMU-on coupling; non-empty scalars; boot-scalar metachar gate; MKINITCPIO_COMPRESSION_OPTIONS charset `^-?[A-Za-z0-9]+$`; KERNEL_PARAMS charset `^[A-Za-z0-9._,=-]+$`) → **`_ir_validate_sets` (L736)** → `_ir_validate_post_hooks` → sysctl.d backup-target refusal → leading-dash package-name refusal. **No kernel-floor validator and no Mesa validator exist.**
- `_ir_validate_sets` in detail — three loops, each `_err_loud` + `_pre_dispatch_exit $EXIT_PREFLIGHT` on a hit:

```
║ INTERSECTION                       ║ WHY IT IS FATAL                       ║
║────────────────────────────────────║───────────────────────────────────────║
║ PKGS_ADD ∩ PKGS_DEL                ║ phase 2 installs, phase 4 -Rns removes║
║ EXPECTED_SERVICES ∩ MASK           ║ phase 4 masks before it enables       ║
║ _RY_PKG_MANAGED_SERVICES ∩ MASK    ║ a masked unit cannot be pkg-managed   ║
```

- Confirm: (a) counts/keys/sets run BEFORE any disk write; (b) bypass inventory is exactly ONE env; (c) `PACTREE_TIMEOUT_S=60`, `BOOT_SPACE_CRIT/WARN` 200/500 MB, `ROOT_AVAIL_CRIT/WARN` 2/5 GiB sane vs multiple kernels + fallback + zstd -1 image (§7).

Exit-code contract (audit for discipline — no bare `exit 1` collapsing 3/4/5/10). All 14 constants re-read live at this pin and identical to 7.130.0:

```
║ CODE ║ NAME              ║ RAISED WHEN                        ║
║──────║───────────────────║────────────────────────────────────║
║ 0    ║ EXIT_OK           ║ success                            ║
║ 1    ║ EXIT_FAIL         ║ generic failure                    ║
║ 2    ║ EXIT_USAGE        ║ CLI misuse                         ║
║ 3    ║ EXIT_PREFLIGHT    ║ preflight gate abort               ║
║ 4    ║ EXIT_BOOT_CRIT    ║ boot-critical rollback path        ║
║ 5    ║ EXIT_LOCK         ║ instance lock contention           ║
║ 10   ║ EXIT_DRIFT        ║ --check found drift                ║
║ 11-14║ EXIT_GEN_*        ║ internal gen sentinels (fn return) ║
║ 250  ║ EXIT_AS_MISUSE    ║ internal _as assert                ║
║ 251  ║ EXIT_RUN_TMPFAIL  ║ internal _run sentinel             ║
║ 255  ║ EXIT_RUN_MISUSE   ║ internal _run assert               ║
```

11–14 (GEN_NOFN, GEN_NOUUID, GEN_SYSCTL, GEN_ENVD) / 250 / 251 / 255 are annotated in-script as internal-only sentinels — never a process exit; checksum verify maps NOUUID to a graceful skip. Signals map to 128+N. Confirm none can reach a process exit. Timeout scalars: `_RY_RUN_TIMEOUT_DEFAULT=3600`, `_RY_LONGOP_HARD_CAP=7200`.

**Neither orphan class reaches EXIT_DRIFT (10), by design (§9, P0 #21).** `--check` returns 0 with a stale drop-in and an orphaned mask both present. Any recommendation that would make them drift must argue the exit-10 desensitization point explicitly.

Runtime contract re-smoked on the extracted 7.135.1 archive: root `--check` → rc 3, 0 B stdout, 0 B stderr, no JSONL (the root guard precedes LOG_DIR init); root `--verify` → rc 2, 92 B; `--version` → `v7.135.1` rc 0; `fish --no-execute` rc 0.

# Appendix F — robustness & correctness (safety invariants; FIX applies normally)

Audit whether the installer is safe to run at all on current fish (3.6 floor) / CachyOS. Flag any TOCTOU, fail-open, or partial-write window. Any GAP in F1/F2/F4 is release-blocking and outranks every tuning finding. **This appendix is F1–F6. A 7.122.0-era edition mislabelled it "G–L" despite no such appendices existing; do not cite appendices G through L — they have never existed.**

**F1. Atomic writes (`_awf_*`) and the first-adoption preserve — THE SECTION THAT CHANGED.**

Render-to-tmp (tee `$pipestatus` split-checked; generator fail → EXIT_GEN_*) → (nftables: `nft -c` tmpfile gate) → symlink-swap probe (rc 0/1/2; abort on swap) → chmod → `sudo -n true` re-assert → `mv -T` (tmp in dst parent, same-FS; capability probed on /tmp — vfat /boot rename semantics stand) → backup targets add `.ry.bak` pre + post-write re-read/restore (tri-state rc; `string collect --no-trim-newlines --allow-empty` preserves trailing newlines; cmdline UUID-dependence → NOUUID graceful skip). Non-backup files detect-only on mismatch (bounded by nft -c for the ruleset; a semantically-wrong ruleset still deploys — residual, state it). Confirm no probe pipes a fish builtin into an external reader (SIGPIPE capture-drop). **Determinism re-measured this pass: all 17 generators produced byte-identical output across three consecutive renders, including the four backup targets.**

**The first-adoption preserve was DEAD CODE from 7.109.0 through 7.135.0 and is live only from 7.135.1.** This is the most consequential robustness change in the rebase window and P0 #15 exists for it.

```
║ ASPECT      ║ DETAIL                                                        ║
║─────────────║───────────────────────────────────────────────────────────────║
║ Mechanism   ║ _RY_ORIG_SUFFIX = .ry.orig, declared L204 beside              ║
║             ║ _RY_BACKUP_TARGETS / _RY_BACKUP_SUFFIX                        ║
║ Where       ║ _ry_install_file L2092; preserve branch L2104-2115            ║
║ Scope       ║ every destination NOT in _RY_BACKUP_TARGETS — 13 of 17        ║
║ Trigger     ║ installed bytes readable (_read_rc 0) AND dst not a backup    ║
║             ║ target; copies dst -> dst.ry.orig with cp -p, ONCE            ║
║ Idempotency ║ skipped when .ry.orig already exists; a symlink at the        ║
║             ║ preserve path is removed first (never cp through a symlink)   ║
║ Logging     ║ PREEXISTING_PRESERVED (with sha256) / PREEXISTING_PRESERVE_   ║
║             ║ FAIL; a failed preserve WARNS and proceeds, non-fatal         ║
║ The defect  ║ _cur_bytes and _read_rc were `set -l` INSIDE the              ║
║             ║ `if test "$_gen_rc" -eq 0` block. fish `set -l` does not      ║
║             ║ leak past an `if`, so `set -q _read_rc` at the branch head    ║
║             ║ was ALWAYS FALSE and the branch never executed                ║
║ The fix     ║ L2098 hoists both to function scope before the `if`; the      ║
║             ║ inner assignment drops `-l`. +1 line / +102 B total           ║
```

**Audit consequences, in priority order:**
1. **Historical, unrecoverable:** any host first deployed between 7.109.0 and 7.135.0 had 13 destinations overwritten with no backup of any kind — no `.ry.orig`, no `.ry.bak`. The affected set includes `/etc/nftables.conf`, `/etc/NetworkManager/conf.d/99-cachyos-nm.conf`, `/etc/sysctl.d/95-ry-overrides.conf`, both resolved and logind drop-ins, `/etc/modprobe.d/60-ry-modules.conf`, and both user files. **There is no forensic trace** — the JSONL never emitted `PREEXISTING_PRESERVED` because the branch never ran, so absence of the key does not distinguish "nothing to preserve" from "preserve skipped". §15 carries the log query that confirms the window on a given host.
2. **Assess the primitive, not just the fix.** A one-time preserve is weaker than the boot four's `.ry.bak` + post-write re-read/restore. It captures only the *first* pre-existing version; every subsequent overwrite of user edits is silent, by design, because the profile treats managed files as owned. Evaluate whether that is the right contract for the two user-scope files in particular, where a user editing `~/.config/MangoHud/MangoHud.conf` between deploys is a realistic and reasonable act.
3. **It is unverifiable.** No verify path asserts `.ry.orig` exists and none can — absence is the normal case on a clean install (Appendix C item g). The mechanism is best-effort by construction.
4. **This was a regression of a previously-closed incident.** The 7.108.0 first deploy overwrote a hand-made drop-in unbacked; 7.109.0 was the release that was supposed to close that class. The fix shipped, the code was present and read correctly, and it never executed for 26 releases. **The audit lesson generalizes: presence of a guard in source is not evidence the guard runs.** Any F-section claim of the form "X is protected because the code does Y" should be backed by a behavioural probe, not a read.
5. Variant risk: the same fish scoping mistake (`set -l` inside a block, read after it) could exist elsewhere. Note for the researcher — the reliable way to find them is fish's own canonical re-indentation via `functions --no-details`, using INDENTATION as the block-depth signal. A `;`-splitting depth tracker over-reports massively on this script's packed style.

**F2. Instance lock (`_acquire_lock*`):** atomic `mkdir` (umask 0077, 0700) + pidfile via mktemp+`mv -Tf` (0600); in-script rationale: mkdir+pidfile, not flock — atomic on any fs, no fd inheritance into sudo children, stale-pid reclaim — confirm each leg holds on fish 3.6+. PID-recycle: /proc/PID/stat field-22 starttime + /proc/stat btime; divisor `getconf CLK_TCK` fallback USER_HZ=100 (correct by ABI); reclaim only if holder start > pidfile mtime + 2 s; unparseable ⇒ live, refuse; garbage pidfile settle 0.2 s → re-read → refuse; bounded 3 attempts, fail-closed. Re-read-before-rm guard; symlinked LOCK_DIR refused; `--preserve-root`; kill -0 EPERM ⇒ /proc liveness. Confirm the `^.*\) ` comm-strip survives `) ` in comm and field indexing after it. Lock lives at `$HOME/ry-install/.lock`; `--verify` and `--check` are lock-free.

**F3. Privilege handling (`_as`, `_run`):** `sudo -n` everywhere; one TTY-gated `sudo -v` prompt, non-TTY refuses (no mid-run hang); credential re-asserted before each critical write — confirm the once-prompt cannot recur. Tri-state rc 0/1/2 (drift vs lapse) in `_is_symlink`/`_installed_bytes`/`_ry_content_bytes` — audit every caller branches on 2 (incl. `_vsp_pacman_conf`). **The F-1 defect above was a `_installed_bytes` consumer**: `_ry_install_file` captured its tri-state rc into a block-scoped variable and then tested it outside the block, so the rc-2 (sudo lapse) path at L2102 logged correctly while the rc-0 path silently vanished. Re-audit every tri-state consumer for the same scoping shape. `_run` timeout: default 3600 s, >9-digit clamp 2147483647, invalid → default, 0 disables; long ops (pacman, mkinitcpio, sdboot-manage, paccache, updatedb, pkgfile; PATH-resolved, sudo value-flag skip list, `env`/`VAR=` prefixes skipped) FLOORED to 7200 s — confirm 7200 covers worst-case `-Syu` and a cap-kill is fatal-with-rollback. Capture hygiene: argv redacts /tmp/ry-* → [REDACTED]; overflow inlines bytes+sha256 + awk keyword scan of the elided middle (≤10 hits, ≤2000 chars; nothing on disk).

**F4. Boot-wipe gate & rollback:** `_irb_taint_gate` (taint OR failed mkinitcpio revert ⇒ skip rebuild, exit 4; `_taint` sets INSTALL_HAD_ERRORS + _RY_BOOT_TAINTED together — confirm every boot-critical write failure routes through it) → `mkinitcpio -P` fail ⇒ exit 4 → `$BOOT` resolved BEFORE the vfat gate; unresolved or non-vfat + REMOVE_EXISTING=yes ⇒ refuse wipe (exit 4) → `_preflight_boot_sanity` (vmlinuz + initramfs + entries or exit 4). Confirm: no path to REMOVE_EXISTING=yes with unverified $BOOT; the Phase-3 cmdline-write → Phase-5 rebuild window is covered by the param-stripped fallback (B1 asymmetry); EXIT_BOOT_CRIT terminal (no Finalize). mkinitcpio rollback: pre-Syu snapshot /run/ry-install (0700, mktemp, tagged) → `_mkinitcpio_revert` same-FS mktemp + byte-exact `cmp` + atomic mv; duplicate KEY= resolves last (matches `_ry_mkinitcpio_array`); tmpfs snapshot same-boot-only (acceptable). Flag if a pacman partial transaction can desync mkinitcpio.conf from installed modules without triggering revert. **The boot four were never affected by F-1** — they take the `.ry.bak` path, which is a separate branch and always ran.

**F5. Signal & exit teardown:** INT/TERM/HUP/QUIT/ABRT with explicit 128+N map (HUP:129 INT:130 QUIT:131 TERM:143 ABRT:134; unknown→130); idempotent; SIGPIPE marks output broken, JSONL continues; fish_exit prefers `_INTENDED_EXIT_CODE` → `_RY_INSTALL_LAST_EXIT` → `$status`. `--check` stderr-silence holds through the PRE-ARGPARSE window (root + --check-only ⇒ silent exit 3) — re-smoked at this pin, 0 B/0 B. umask set as the VARIABLE (autoload-race safe) — fish binds `umask` as a special variable to the process umask, so this is correct and not a builtin/variable confusion. Cleanup order: kill children (TERM to -P $fish_pid descendants; 0.5 s grace, 10 s under db.lck; then KILL; missing pgrep degrades flat 0.5 s) → mkinitcpio revert → tmpfile sweep (`_RY_TMPDIR_GLOBS` 6, PID-scoped `ry-*.$fish_pid.*`; live-dumped set is sudo-err, tee-err, run, argparse-err, fstab-tee-err, fstab-awk-err — confirm glob set == created set) → fs sweep → lock release LAST → erase globals. Log lifecycle: mv -T with cp -pT + rm recovery; both-fail keeps old path (warn); symlinked LOG_FILE removed, recreated 0600; log dir 0700, JSONL 0600; root-guard `@@LEFT@@`/`@@IF@@` display sentinels never leak unstripped. SIGKILL is the only cleanup bypass (stale-reclaim F2 recovers).

**F6. pacman transaction safety:** full `-Syu --needed` only; retry `-Syyu --needed`; second failure fatal; SYSTEM_UPGRADED via `pacman -Q | sha256sum` fingerprint (empty fails open to true). db.lck pre-checked, never removed; teardown reaper is $fish_pid-scoped (peer pacman untouchable) with 10 s grace under db.lck. PKGS_DEL `-Rns` rdep-aware (external dependants skip + `_RY_PKG_REMOVE_SKIPS`); paccache -rk2/-ruk0 separate; pactree-absent pre-Phase-2 warns only (`PACTREE_MISSING`). Confirm no `-S <pkg>` runs outside `-yu` context and db.lck is checked before any package op. **New adjacent gate:** `_ir_validate_sets` now refuses a `PKGS_ADD ∩ PKGS_DEL` collision at preflight, so the phase-2-installs/phase-4-removes cycle is no longer reachable by a bad edit (Appendix E).

**Output-channel invariant (audit target, all appendices) — RECONCILED, do not re-file as drift.** Every user-facing message funnels through `_msg_print` (**L1117**, was L1105), the single `>&2` authority honoring QUIET / `_RY_OUTPUT_BROKEN` / `_RY_NO_COLOR` / isatty(2). Two different counts of direct stderr writes are on file and **both are correct under their own scoping** — the discrepancy is a measurement-method artifact, not a regression:

```
║ MEASURE                                   ║ COUNT ║ METHOD                  ║
║───────────────────────────────────────────║───────║─────────────────────────║
║ raw `>&2` occurrences, whole file         ║  78   ║ textual, all scopes     ║
║ lines containing `>&2`                    ║  74   ║ textual                 ║
║ …of those, BEFORE the _msg_print def      ║  34   ║ pre-init preflight      ║
║ …at or AFTER the definition               ║  40   ║ internals + escapes     ║
║ inside a FUNCTION body                    ║  43   ║ fish `functions --no-   ║
║   (spread across 17 functions)            ║       ║ details` per name       ║
║ top-level (outside any function)          ║  35   ║ 78 − 43                 ║
```

The in-function 43 break down as `_acquire_lock` 6 + `_acquire_lock_fresh` 4 (the only pair that could use `_msg_print --force` equivalently), `_rdi_matrix_*` 12 (the matrix banner declares itself STDERR-ONLY), `_progress_*` 6 (raw terminal escapes), `_early_usage_exit` 3, `_ry_root_usage` 3, `_cleanup` 2, `_msg_print` 2 itself, `_run_emit_stream` 2, `_echo` 1, `_ry_sudo_cache_banner` 1. **`_msg_print` resolves to 10 hits = 1 definition + 9 wrapper call sites; that IS the funnel, not a shortfall.** The 35 top-level writes are pre-init preflight and correct by necessity — `_msg_print` is not yet defined at L78/L161. stdout carries only `--help` and `--version`. **Treat the invariant as "single authority for user-facing leveled output", not "sole writer to fd 2"** — the latter reading is what generates a false finding here every time. Flag any recommendation that would add a *leveled user-facing message* outside `_msg_print`.

**ROBUSTNESS verdict (required, separate):** per F1–F6 PASS / GAP / UNCERTAIN; GAPs in F1/F2/F4 surface first; correctness has no "deliberate trade-off" defense. **F1 must be graded against the fixed 7.135.1 behavior, with the 7.109.0-7.135.0 window recorded as historical exposure.**

---

## Method notes for reproducing this brief

Recorded so the next rebase does not re-pay these costs.

- **Harness:** cut the script just before the `# ── MAIN: ARGPARSE` banner. **Always locate the banner, never hardcode** — **L4834 at 7.135.1**, L4786 at 7.125.0–7.130.0, L4793 at 7.124.0, L4777 at 7.123.x. Then DELETE the L3 source guard (`if contains -- (status filename) - 'Standard input'; …; return 1; end`) and `source` the result as a non-root user. Without the deletion the guard fires on `source` and every count silently reads 0. Build as `sed -n '1,<cut-1>p' script | sed '3d' > harness.fish`.
- **Array counts via live fish eval, never text parsing.** `eval echo \$$name` COLLAPSES a fish array to one element and reports every count as 1 — use `eval "set vals \$$name"` then `count $vals`. Continuation-regex extractors truncate multi-line declarations, `set -g --` evades awk, and several service keys share one `set -g` line.
- **Function census by set difference, not by parsing.** `functions --all --names` before and after sourcing the harness; the difference is exactly the script's own functions (292 at this pin). `functions --names` alone HIDES underscore-prefixed names and will report only fish's own defaults. Hand-written depth trackers desync on this script's one-line `function …; …; end` definitions.
- **Generated bytes must be measured as WRITTEN FILES** (`$fn > tmp; stat -c %s`), never `(cmd | string collect)` — collect strips each trailing newline and a 17-file total reads 5,076 B instead of 5,093 B, a phantom 17 B deficit that looks like anchor drift.
- **Determinism must be compared per-file by CONTENT hash, or with a SORTED manifest.** A concatenated hash over the generator output directory is NOT comparable between passes when the glob order depends on harness filenames. Sorting the `sha256sum` manifest before hashing it makes it stable (`18b4d210b6fbd519` at this pin); comparing 17 per-file hashes is the safer form.
- **Harness variable gotcha:** the cmdline generator reads `$_ROOT_UUID` (single underscore), not `$_RY_ROOT_UUID`. Set it, or the generator returns 12 (GEN_NOUUID) and `_ry_validate_configs` returns rc 3, which looks like a regression.
- **The 18/20 Appendix-B match is the expected shape**, not a defect. The size table is not a body, and the remote-play variant is published as a 3-line diff fragment rather than a whole 916 B file, so a whole-file comparator reports it absent. Check the three added lines instead.
- **Verify every "before" column against the OLD BRIEF'S RENDERED BODY, not against memory.** The old script is not in the archive; only its brief is. A drift row asserting a change that never happened passes every count check and every byte check — only an old-vs-new body diff catches it. That is exactly how a phantom `DNSOverTLS` change entered a previous edition. The mechanical form: extract the previous edition's Appendix-B fences with a TOGGLE-based fence walker (a naive `re.findall` on ``` consumes alternating pairs and under-reports), substitute the UUID placeholder, and set-compare against fresh generator output.
- **Function-name parsing:** fish function names contain dots (`_content_HOME_.config_MangoHud_MangoHud.conf`), so a `[A-Za-z0-9_-]*` charset truncates them and fabricates duplicates. Match `\S+` after `function`.
- **Fence-aware heading checks are mandatory** on any document in this repo — a naive `^# ` scan false-reports h1 violations on the `#` comments inside the nft and config fences. This edition: 43 headings, 0 level skips, verified fence-aware.
- **Byte-vs-character length:** banner and line-length checks must count CHARACTERS. `awk length` under a C locale counts bytes and falsely reports the U+2500/U+2192 box-drawing content over any character cap.
- **Verify the upload before trusting it.** A fresh upload can be BEHIND what is recorded as shipped. sha256 the archive against recorded anchors first. At this pin: script `509c41bc` 4963 L / 293,399 B; README `98c3e111` 359 L / 23,511 B; CHANGELOG `77531af7` 163 L / 7,391 B; LICENSE `2e1e7c8a`; zip `9d83906c` 326,058 B, topdir `ry-install-v7_135_1`. Historical trap: two artifacts both call themselves 7.130.0 and differ only in CHANGELOG — for that version, hash the CHANGELOG, not the script.
- **Sandbox limits:** sudo-as-root is available in a container but the target host paths are not, so only sudo-fail and preflight paths are exercisable and a full install cannot complete by design. `_err_loud` EXITS rather than returns — run each negative test in its own subprocess. `ripgrep` 14.1.0 carries a false-negative bug; re-confirm every zero-hit assertion with `grep -P` or `python3`.

## Sources

docs.kernel.org (kernel-parameters, PCIe/ASPM, amd-pstate, sysctl/vm, sysctl/kernel, networking, block, ext4, UMIP, AMD-Vi, accel/amdxdna, /proc/stat, amdgpu power_dpm_force_performance_level) · git.kernel.org (linux-firmware, r8169, mt76, wireless-regdb, commit 2e0239d47d75 ASPM doc fix) · gitlab.freedesktop.org (mesa, drm/amd) · docs.mesa3d.org · github.com (HansKristian-Work/vkd3d-proton, CachyOS/proton-cachyos, flightlessmango/MangoHud #1794 + #1825 + 0.8.3 release notes, systemd/systemd #33973 + #33579, LizardByte/Sunshine, moonlight-stream) · invent.kde.org (powerdevil — POWERDEVIL_NO_DDCUTIL consumption) · wiki.archlinux.org (AMDGPU, IOMMU, fsck, Gaming, PipeWire, Zram, SSD, Ext4, Btrfs, Sysctl, NetworkManager, Wireless, nftables, Uncomplicated_Firewall, Security, Mkinitcpio, systemd-boot, Bluetooth, System_time, CPU_frequency_scaling, MangoHud, pacman) · wiki.cachyos.org · discuss.cachyos.org · bugzilla.kernel.org · man.archlinux.org (nft, avahi-daemon, systemd.unit, logind.conf, resolved.conf, NetworkManager.conf, hwclock, timesyncd, systemctl) · man7.org (mkdir/rename(2), proc(5), sysconf) · fishshell.com/docs (variable scope — `set -l` block visibility, F1) · amd.com ROCm · archlinux.org/packages · mangohud-gtr9-pro v1.17.0 companion archive (MangoHud.conf + README + CHANGELOG — HUD lockstep, floors, directive history).

**Do not cite** wireless.docs.kernel.org for `pcie_aspm` semantics — it carries stale pre-6.9 text (§5). Cite access dates + exact versions in the methodology block.
