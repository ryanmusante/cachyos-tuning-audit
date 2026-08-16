# CachyOS Tuning Audit — Beelink GTR9 Pro (gfx1151)

Pinned to `ry-install.fish` **7.163.0**, edition **r2**. Actionable items only, ordered by
implementation safety. **§3 is the standing hunt for gaming and performance headroom** — the
queue being empty of defects is not evidence the box is at its ceiling. r2 re-verified the
same archive, expanded §3 to twenty candidates with a shortlist, and cut prose again.

---

## 0. Provenance

Every value below was re-derived by live evaluation of the attached archive. All seventeen
generator bodies were re-rendered and diffed against the 7.162.2 fences: **two came back
different.** **r2 re-hashed the upload and re-rendered all seventeen a second time on
2026-08-16: archive hash and every body byte-identical to r1, Σ 5,393 B, so no value in this
brief moved.** A re-verification that finds nothing is still the only thing that licenses
carrying the numbers forward.

```
║ ARTIFACT             ║ SHA256   ║ SIZE                 ║
║──────────────────────║──────────║──────────────────────║
║ ry-install-main.zip  ║ c890a83b ║ 86,813 B (Deflated)  ║
║ ry-install.fish      ║ 1f2a1bae ║ 4,919 L / 293,423 B  ║
║ README.md            ║ 688d795f ║ 304 L /  19,103 B    ║
║ CHANGELOG.md         ║ 2ac8288a ║ 143 L /   5,440 B    ║
║ LICENSE              ║ 2e1e7c8a ║ 21 L /   1,069 B     ║
```

Audited from the GitHub `main` archive — git-archive form, topdir `ry-install-main`, 5
entries, no mode bits, 319,035 B uncompressed. All four members self-report `7.163.0`. **No
release zip was supplied this rebase, so no release anchors are recorded**; do not
reconstruct them. Repacking from `main` must re-apply 0755 to the script.

**Disambiguate by zip, README or CHANGELOG hash — never by `--version`, and never by script
hash alone.** 7.162.0, 7.162.1 and 7.162.2 shipped inside one week, two of them differing by
two bytes of script; 7.141.0 shipped twice with one script hash and 7.140.0 nine times with
two.

Upstream reference points. **Rows dated 26-08-16 were re-fetched for this rebase**; the rest
carry their previous verification date. Mesa's `VERSION` returned HTTP 504 twice — recorded
as source_unreachable and carried forward, not re-derived.

```
║ COMPONENT           ║ AT AUDIT        ║ VERIFIED ║ SOURCE                     ║
║─────────────────────║─────────────────║──────────║────────────────────────────║
║ Linux mainline      ║ 7.2.0-rc7       ║ 26-08-16 ║ torvalds/linux Makefile    ║
║ linux-cachyos       ║ 7.1.8-1         ║ 26-08-16 ║ PKGBUILD _major/_minor     ║
║ Mesa main           ║ 26.3.0-devel    ║ 26-08-15 ║ mesa VERSION (504 today)   ║
║ proton-cachyos      ║ 11.0-20260703   ║ 26-08-16 ║ releases/latest redirect   ║
║ linux-firmware MES  ║ 2026-05-07 tag  ║ 26-08-15 ║ gc_11_5_1_mes_2.bin log    ║
║ MangoHud #1794      ║ OPEN            ║ 26-08-16 ║ issue page                 ║
║ systemd #33579      ║ OPEN            ║ 26-08-16 ║ issue page                 ║
```

---

## 1. Delta vs the 7.162.2 edition

### 1a. What moved

**The four perf scalars are byte-identical across thirty-three releases, 7.130.0 →
7.163.0.** Governor `performance`, EPP `performance`, DPM `high`, driver `amd-pstate-epp`.
A recommendation asserting that a *perf* value moved in that window is a stale-source error.

```
║ AREA          ║ 7.162.2     ║ 7.163.0     ║ NOTE                                               ║
║───────────────║─────────────║─────────────║────────────────────────────────────────────────────║
║ script        ║ 4,919 L     ║ 4,919 L     ║ +64 B, 0 L — both edits are inside packed printf   ║
║               ║             ║             ║ lines                                              ║
║ oracle        ║ 21          ║ 21          ║ ENV_VARS 9 -> 10; every other value held           ║
║ generators    ║ 17          ║ 17          ║ Sigma 5,338 -> 5,393 B; 15 bodies byte-identical   ║
║ harness cut   ║ L4790       ║ L4790       ║ HELD — first time in six rebases                   ║
║ README        ║ 305 L       ║ 304 L       ║ GSK_RENDERER row + NM cell reworded                ║
║ CHANGELOG     ║ 156 L       ║ 143 L       ║ 7.163.0 block; history folded to one range         ║
║ unchanged     ║ —           ║ —           ║ functions 294 · verify fns 62 (12+50, identical    ║
║               ║             ║             ║ prefix by prefix for five rebases) · sub: 95 ·     ║
║               ║             ║             ║ banners 95 · deps 37 + 16                          ║
```

**Two generated bodies changed, both at non-zero byte delta:**

```
║ FILE                         ║ 7.162.2   ║ 7.163.0   ║ CAUSE                                ║
║──────────────────────────────║───────────║───────────║──────────────────────────────────────║
║ ~/.config/environment.d/     ║ 282 B     ║ 299 B     ║ GSK_RENDERER=ngl added; ENV_VARS 9   ║
║ 10-environment.conf          ║           ║           ║ -> 10                                ║
║ /etc/NetworkManager/conf.d/  ║ 148 B     ║ 186 B     ║ new [main] section with              ║
║ 99-cachyos-nm.conf           ║           ║           ║ autoconnect-retries-default=0        ║
║──────────────────────────────║───────────║───────────║──────────────────────────────────────║
║ 17-file total                ║ 5,338 B   ║ 5,393 B   ║ +55 B, none of it perf               ║
```

Both changes are gaming-relevant and neither is a perf scalar. `GSK_RENDERER=ngl` is the
system-wide install of the GTK4-Vulkan workaround for the RADV/GTK4 abort seen on gfx1151;
`autoconnect-retries-default=0` makes NetworkManager retry a failed Wi-Fi activation
forever instead of giving up after four tries, which is the mitigation for the recurring
~24 h `wlan0` reassociation drop — a mitigation, not a fix (§3, G-8).

**The count tripwire was synced this time.** `_ir_validate_counts` carries `ENV_VARS:10`,
and all four validators return rc 0 unshadowed with stderr visible on the extracted archive.
The standing check that came out of the 7.162.0 escape was exercised on its first
count-moving release and passed — record it as a pass, not as a non-event.

**One new gap, filed as T2-6:** `autoconnect-retries-default=0` is hardcoded in the NM
generator and `_vss_nm` greps only for `wifi.backend`, `wifi.powersave` and `level` — the
new value is deployed but unasserted. Contrast `GSK_RENDERER`, which needed no verifier edit
at all because `_vre_envvars` and the static user check both iterate `$ENV_VARS`. Dynamic
iteration follows an array; a hardcoded literal does not.

### 1b. Retired questions — do not re-derive from a stale source

Each of these is a **closed question**, not an open item. A brief or recommendation that
raises one is reading a stale artifact.

```
║ REMOVED / CLOSED               ║ AT           ║ CONSEQUENCE                                    ║
║────────────────────────────────║──────────────║────────────────────────────────────────────────║
║ RY_REMOTE_PLAY_PORTS + port    ║ 7.137.0      ║ nftables body is single-form. No port set to   ║
║ sets                           ║              ║ open. The 5353/udp mDNS finding is retired     ║
║ Preemption-model advisory      ║ 7.139.0 r2   ║ the profile never pinned preempt=. Do NOT      ║
║                                ║              ║ report a missing preempt check                 ║
║ Redundant -T0 compression flag ║ 7.140.0      ║ do NOT report it as a lost threading option    ║
║ RESOLVED_DNS_SERVERS, NM       ║ 7.147.0      ║ the host pins NO upstream. A missing DNS= line ║
║ [global-dns-domain-*]          ║              ║ is by design — the router is authoritative     ║
║ RESOLVED_DOT, RESOLVED_DNSSEC  ║ 7.148.0      ║ resolved drop-in is 4 lines. Both were         ║
║                                ║              ║ redundant pins; do NOT report them missing     ║
║ PROTON_FSR4_UPGRADE,           ║ 7.154.0      ║ FSR4_WATERMARK=1 ships instead and is a        ║
║ PROTON_FSR4_RDNA3_UPGRADE      ║              ║ VERIFICATION variable. T1-3 closed             ║
║ PROTON_ENABLE_WAYLAND=1        ║ after        ║ T1-2 closed BY THE ARTIFACT. Per-game use      ║
║                                ║ 7.155.0      ║ survives (§3c); global scope does not          ║
║ both net.core.netdev_budget    ║ 7.157.0      ║ T2-1 closed on measurement. An absent key      ║
║ keys                           ║              ║ leaves the kernel default — not a regression   ║
║ fsck.mode=force -> auto        ║ 7.158.0      ║ T3-3 shipped; the root is ext4, so force ran a ║
║                                ║              ║ full check every boot                          ║
║ COMPRESSION_OPTIONS -1 -> -3   ║ 7.158.0      ║ T4-2 shipped at zero byte delta                ║
║ missing icmpv6 accept (a gap)  ║ 7.159.0      ║ RFC 4890 host-minimum accept SHIPPED. T4-7     ║
║                                ║              ║ closed. Do NOT call the ruleset IPv6-unsafe    ║
║ MangoHud cpu_stats active      ║ <= 7.160.0   ║ shipped commented with an inert-sensor note.   ║
║                                ║              ║ T1-1 closed                                    ║
║ clearcpuid=umip                ║ 7.160.0      ║ T3-2 closed, boot untainted. Re-add trigger is ║
║                                ║              ║ a NAMED title regressing                       ║
║ amd_iommu=off,                 ║ 7.162.0      ║ replaced by amd_iommu=on iommu=pt + false. Do  ║
║ BLACKLIST_AMDXDNA=true         ║              ║ NOT report a missing amdxdna blacklist         ║
║ --check stale-drop-in sweep    ║ <= 7.162.1   ║ T4-6 closed — the privilege-free sweep now     ║
║ below the sudo bail            ║              ║ runs first                                     ║
```

**Three releases are not recoverable from the shipped artifact and must never be guessed:**
the removal of `PROTON_ENABLE_WAYLAND` (`> 7.155.0`, `<= 7.157.0`), the commenting of the
MangoHud CPU keys (`> 7.158.0`, `<= 7.160.0`), and the `--check` sweep hoist (`> 7.158.0`,
`<= 7.162.1`). The installer CHANGELOG folds its history into ranges and none of those
bullets carries a version.

**Six of the queue's own items have been closed by the artifact across five rebases.**
Re-render the archive before carrying any open item forward.

---

## 2. Action queue — ordered by implementation safety

Tiers ascend by blast radius: T0 changes nothing, T5 must not be touched. Every
recommendation carries a tier. §3 candidates inherit these tiers.

### T0 — RETURNED. Results, not actions.

All eight gates returned at the 7.158.0 edition and the results stand; §7 carries their
regression forms. **A gate result is history, not an action.**

```
║ ID    ║ OBSERVATION                 ║ RESULT                                               ║
║───────║─────────────────────────────║──────────────────────────────────────────────────────║
║ T0-1  ║ turbostat idle floor        ║ 21.33 / 21.69 W package, 3.93 / 4.20 W core at       ║
║       ║                             ║ Busy% 0.13 / 0.24                                    ║
║ T0-2  ║ lspci LnkCtl ASPM per link  ║ EVERY link reads "ASPM Disabled", including links    ║
║       ║                             ║ advertising L0s/L1 and L1SubCap                      ║
║ T0-3  ║ root FS type                ║ ext4 — NOT Btrfs                                     ║
║ T0-4  ║ k10temp hwmon + energy_uj   ║ ONE labelled sensor, Tctl -> temp1_input. No Tdie,   ║
║       ║                             ║ no Tccd. energy_uj mode 400 = root-only              ║
║ T0-5  ║ softnet_stat squeezed       ║ 0 on all 32 CPUs, twice at ~2x traffic apart;        ║
║       ║                             ║ dropped also 0                                       ║
║ T0-6  ║ amd_pstate dynamic_epp      ║ disabled; EPP reads performance                      ║
║ T0-7  ║ .ry.bak / .ry.orig          ║ 4 .ry.bak (the boot-critical four) + 2 .ry.orig.     ║
║       ║ inventory                   ║ No user-scope .ry.orig — correct for files with      ║
║       ║                             ║ no pre-existing content                              ║
║ T0-8  ║ resolvectl + routing        ║ resolv.conf mode FOREIGN. Both 10 GbE links DOWN;    ║
║       ║                             ║ wlan0 is the default route                           ║
```

Three results still bind and are cited rather than re-argued: **T0-1** is the only power
figure any `max_cstate` argument may use; **T0-2** is what makes the ASPM pair
evidence-backed rather than assumed; **T0-8** retires the "dual 10 GbE" premise for
everything except §3's G-8, which is about putting the links back into service.

Advisory reads: **A1** (lspci ASPM audit) satisfied by T0-2. **The MES read returned** — the
unit reports MES 0x91 / KIQ 0x75, past the 0x86 gfx1151 hang fix (host capture 2026-08-14).
**A2** — the installed proton-cachyos build — is the one open read, and §3 G-14 is why it
matters.

### T1 — User/session scope. No root, no reboot, reversible by editing one file.

**T1-1, T1-2 and T1-3 are closed by the artifact** and are defined once in the §1b roster
along with T2-1, T3-2, T3-3, T4-2, T4-6 and T4-7. They are not restated as items here.

- **T1-4 · Both user files have a working first-adoption preserve.** Since 7.135.1,
  `10-environment.conf` and `MangoHud.conf` get a one-time `.ry.orig`. The *first* hand-edit
  is preserved, every later one is silently overwritten by design. Both land 0600
  (`_ry_install_file` sets 0644, then 0600 when `use_sudo` is false) — **not mode drift.**
  Absence of a user-scope `.ry.orig` (T0-7) is the correct outcome for files that had no
  pre-existing content, not a broken preserve.
- **Lockstep remainder, still open:** the standalone `mangohud-gtr9-pro` archive is
  **DIVERGENT** — its conf still carries `cpu_stats` active while the installer generator
  ships it commented. Reconcile is owned by that repo, toward the installer. §3 G-9 will
  touch the same body; sequence them.

### T2 — Managed config values. Root, no reboot, self-heals on the next deploy.

Everything here is Tier-1 under §8f: the generator rewrites the file wholesale, so a bad
value cannot orphan.

- **T2-2 · nftables rule order — KEEP, settled.** The profile places `ct state invalid drop`
  before `iif "lo" accept`. Gentoo's reference workstation ruleset uses that order;
  nftables.org and the ArchWiki put loopback first. **There is no upstream rule either way**,
  and no documented case of legitimate loopback traffic classified `invalid` on a host doing
  neither NAT nor policy routing. If a FIX is argued it is a pure order swap, `nft -c` gated,
  re-validated by `_post_nft`. IMPACT Low · RISK Low.
- **T2-3 · `energy_uj` permission drop-in — DECLINED, do not re-offer.** T0-4 returned mode
  400. The fix would be an **18th managed file**, moving three oracle counts, and it re-opens
  the **PLATYPUS** RAPL side channel. It is also pointless: MangoHud reads `cpu_power` from
  the APU `gpu_metrics` metric on this box, never from RAPL (§4). **A recommendation to add
  the drop-in must address PLATYPUS explicitly.**
- **T2-4 · `amd_pstate=active` restates the CachyOS compiled default** (`X86_AMD_PSTATE_
  DEFAULT_MODE=3` = Active/EPP), so the token changes nothing on this kernel. **KEEP** — the
  profile does not own the kernel package and a rebuild could flip the default.
  Belt-and-braces, not load-bearing; contrast `pcie_aspm.policy=performance` and `iommu=pt`,
  where the config option is unset and the token is what switches the behavior.
- **T2-6 · NEW · `autoconnect-retries-default=0` is deployed but unasserted.** The NM
  generator hardcodes it inside a packed `printf`; `_vss_nm` asserts `wifi.backend`,
  `wifi.powersave` and `level` only. A hand-edit or a competing drop-in that removes it
  passes `--verify` clean, and the failure mode is silent: Wi-Fi stops coming back after a
  rekey drop and nothing reports drift. The profile's own design rule is that every managed
  value is asserted. **Remedy: one `_chk_grep` line in `_vss_nm` for
  `autoconnect-retries-default=0`.** Byte cost is verifier-side only — no generated body
  moves, no oracle count moves, and the `--verify` OK count rises by 1 on top of the +2 the
  `ENV_VARS` addition already produced (266 -> 268, measured). IMPACT Low · RISK Low. **The
  queue has two FIX-shaped items this edition — this and T3-5** — after four rebases with
  none.

### T3 — Kernel command line. Root + reboot. Reversible, but costs a boot cycle.

The cmdline is charset-gated (`^[A-Za-z0-9._,=-]+$`) and count-asserted at 15. It is emitted
**twice** (`/etc/kernel/cmdline` and `LINUX_OPTIONS` in `/etc/sdboot-manage.conf`) and
verified **three times** (both static blocks plus runtime `/proc/cmdline`), so a one-token
change is never +/-1 anywhere. `_vsb_entry_options` asserts every non-fallback BLS entry
carries every token; `_vsb_sdboot_dropins` WARNs when a packaged drop-in could override
`LINUX_OPTIONS` behind all of it.

- **T3-1 · `processor.max_cstate=1` — measured (T0-1), then retired to KEEP by maintainer
  decision.** It blocks deep CPU idle and compounds with DPM pinned `high`; the owner
  accepted that cost against idle-exit latency. **Do not re-propose removal on power grounds
  without a measurement that beats T0-1.** The 85 W PPT caps peak regardless, so the metric
  is idle, not the load ceiling.
- **T3-4 · The ASPM pair — CONFIRMED load-bearing by T0-2**, which measured every link
  `ASPM Disabled`. The pair is **complementary, not redundant**: the global policy governs
  link state, the module option disables at the MT7925 endpoint. **`pcie_aspm=off` means
  "don't touch ASPM at all" and does NOT disable it — never propose it as an equivalent.**
  Mechanism and citations in §4 (three rows); mt76's MT7927 force-disable quirk does not
  cover MT7925.
- **T3-5 · `amd_iommu=on` is an inert no-op and the host now proves it — REOPENED as a FIX
  candidate.** `parse_amd_iommu_options()` matches fullflush / force_enable / off /
  force_isolation / pgtbl_v1 / pgtbl_v2 / irtcachedis / nohugepages / v2_pgsizes_only — no
  `on`. The 2026-08-16 host capture carries the proof: dmesg logs **`AMD-Vi: Unknown option
  - 'on'`** at 0.05 s, while `Default domain type: Passthrough` confirms `iommu=pt` did take
  effect. IOMMU enablement comes from the built-in default plus IVRS and never needed a
  token, so **`iommu=pt` is the load-bearing half and `amd_iommu=on` does nothing at all.**
  **`--verify` cannot see this and never will:** all three KERNEL_PARAMS surfaces are
  string-presence checks against the emitted files and `/proc/cmdline` — presence, never
  efficacy. Removal cascade, costed: `KERNEL_PARAMS` 15 -> 14, the `_ir_validate_counts`
  literal, the README KP table row **and its prose token count**, and Σ **-26 B** (13 B x two
  emissions). What is lost: the documented `off` <-> `on` reverse-switch symmetry that Tuning
  Notes describes. IMPACT Low · RISK Low — the return is a clean boot log and one fewer
  unparsed token, not performance.

### T4 — Boot chain, firewall handoff, detector severity. Reboot + recovery exposure.

- **T4-1 · Fallback-entry exposure — open.** `LINUX_FALLBACK_OPTIONS="quiet"` strips all 15
  params. The fallback differs on: IOMMU **translated DMA-lazy** rather than passthrough (more
  isolation, more overhead — the *inverse* of the tuned entry); IPv6 enabled, behind a ruleset
  that now handles it; ASPM at firmware default with the endpoint option absent — **the only
  boot path here where ASPM is not disabled**; no C-state cap; `zswap` back on, stacking a
  second compression layer in front of zram; default fsck keys. Confirm the window is accepted
  or flag it. **Verify will never surface it** — `_vsb_entry_options` skips `*-fallback.conf`
  by design (§9).
- **T4-3 · `timeout 0` + `DEFAULT_ENTRY manual` + `REMOVE_EXISTING=yes`** wipes foreign BLS
  entries (EFI-resident loaders untouched). Recovery is live-USB -> chroot. With `timeout 0`
  and no saved EFI variable, a fresh ESP falls back to sd-boot's own sort order until the
  menu is used once. UKI is out of scope. **Adjacency:** a packaged drop-in under
  `/usr/lib/sdboot-manage.conf.d` or `/etc/sdboot-manage.conf.d` would silently outrank the
  managed `LINUX_OPTIONS`, and `_vsb_sdboot_dropins`' WARN is the only signal.
- **T4-4 · ufw masked, not removed — confirm the nftables-first gate closes the window.**
  `_csm_enable_nftables_first` confirms a live default-deny ruleset before anything touches
  ufw; on an unconfirmed ruleset `_configure_services_mask` withholds `ufw.service` from that
  run's safe-mask set. Rationale: `mask --now` plus `ufw-init stop` flushes ufw's rules, so
  masking first would open an unfirewalled window. **Three things to validate:** a host where
  ufw is installed *and* active; that a withheld mask leaves ufw's rules intact rather than
  half-flushed; and that `nftables.service` being a oneshot (unit reads inactive after a
  clean load) does not defeat the liveness check — the script judges by live policy-drop,
  which is correct but unre-validated against current packaging. Keys: `UFW_MASK_DEFERRED`,
  `UFW_RULE_FLUSH_OK|FAIL|SKIP`, `SECURITY_POSTURE`.
- **T4-5 · Orphan-detector severity — a design question, not a gap.** Three detectors ship
  and none sets DRIFT: `_vss_modprobe` WARNs on unmanaged `60-ry-*.conf`, `_vss_orphan_masks`
  INFOs on masked units absent from `$MASK`, `_vsb_sdboot_dropins` WARNs on sdboot-manage
  drop-ins. `--check` records `MODPROBE_STALE_DROPIN` / `MASK_ORPHAN` /
  `SDBOOT_DROPIN_PRESENT` to JSONL only. Reasoning on file: a re-run cannot clear any of
  them, so exit 10 would go permanently non-zero and train the operator to ignore it; mask
  ownership is genuinely **unattributable**, which is why it is `_info`. **Counter-argument
  to test: an INFO in a long run is easy to miss and JSONL is only read deliberately.** Any
  promotion to DRIFT must address desensitization *and* name which of the three it applies
  to — they are not equally attributable.
- **T4-8 · INFO, maintenance-only.** The `_RY_POST_HOOKS` boot entry keys on the glob
  `/boot/*`, so a future managed file under `/boot` would silently route to the `loader`
  hook. Inert today — `loader.conf` is the only `/boot` destination. Record it before adding
  one.

### T5 — Closed and protected. Do not recommend changing these.

Flag a direct upstream contradiction as a **note**; never as a FIX.

- **DNS — three layers, and the profile owns none of them.** The host pins **nothing**: no
  `DNS=`, no `DNSOverTLS=`, no `DNSSEC=`, no NetworkManager `[global-dns]` — it takes per-link
  DHCP DNS from the router in plaintext, by design, and the router forwards to AdGuard over
  **DoT** with DNSSEC validated there. **Do not propose host-side DoT:** the router's DNS
  Privacy Protocol is WAN-side only, so `DNSOverTLS=yes` would TLS-handshake a plaintext-only
  resolver and, failing closed, take DNS down. In foreign mode (T0-8) the drop-in binds only
  resolved-routed queries anyway. Quantify the residual in §6 and stop.
- **`GPU_DPM_LEVEL=high`, not `profile_peak`.** `high` forces the highest power state with
  clock and power gating still active; the `profile_*` modes disable gating and are scoped by
  the kernel docs to profiling work. Evaluate `high` vs `auto` on frametime evidence only
  (§3d), and see §3b G-5 for why `manual` is not a free upgrade. The `profile_peak` variant
  measures udev **647 B** (+8) — recorded so it is never re-measured.
- **The udev EPP write is a redundant no-op and that is fine** (mechanism in §4). It is a
  hotplug-safe assertion of a state the governor already imposes — the only mechanism that
  survives a CPU hotplug event.
- **`processor.max_cstate=1` is protected by maintainer decision**, not by absence of
  evidence (T3-1). `clearcpuid=umip` shows the other outcome: the maintainer reversed his own
  KEEP at 7.160.0, and the recorded trade is what made the reversal free to re-derive.
- **No cuts from the script.** `WINEDEBUG=-all`, BlueZ `AutoEnable`, NM `wifi.backend`,
  `cachyos-gaming-meta`, `MODULES=(amdgpu)`, `lib32-mesa` all stay.
- **The PKGS_ADD orphan is not to be "fixed" with a state file.** A package dropped from
  `PKGS_ADD` stays installed forever and nothing can see it. Detection needs a persisted
  manifest, which adds its own drift surface. Documented rather than built.
- **No version gates, by design.** No kernel floor, no Mesa floor, no advisory comment.
  Deploy / `--check` / `--verify` run on any kernel and any Mesa. Treat it as a posture to
  evaluate, not an oversight.
- **Removed and still absent — each is a validation question, never a candidate:**
  `nowatchdog`, `tsc=reliable`, `8250.nr_uarts=0`, `clearcpuid=umip`, `amd_iommu=off`
  (cmdline); `AMD_VULKAN_ICD`, `DXVK_LOG_PATH`, `VKD3D_CONFIG`, `PROTON_ENABLE_WAYLAND`
  (env); `ddcutil`, `git-delta` (packages); `modemmanager.service` (mask); both
  `net.core.netdev_budget` keys (sysctl); `RY_REMOTE_PLAY_PORTS`; the preemption advisory;
  `-T0`; `pcie_aspm=off`; `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK`; `RY_NO_NTP_REMEDIATION`;
  `clearcpuid=514`; `archlinux-contrib`; the standalone `60-ry-blacklist-amdxdna.conf` and
  `60-ry-mt7925e.conf` drop-ins (both filenames are actively swept for). **Precedent:**
  `mt7925e.disable_aspm=1` was removed at 7.102.x and re-added at 7.129.0 — removals are not
  permanent, but a re-add needs the same evidence bar as a FIX.
- **Do not** remove the ICMPv6 base accept as dead code under `ipv6.disable=1` (it exists for
  the fallback entry); do not flag the inbound-ping accept (`_vss_nft` hard-fails on its
  absence); do not propose sleep-hook re-assert workarounds (all five sleep targets are
  masked, so there is no resume path and the udev `ACTION=="add"` rule is the only event that
  matters); do not flag a low MangoHud `vram` reading on UMA (it reports the BIOS carveout
  only — `ram` carries the shared pool); do not propose per-key sysctl annotation (the
  generator emits one header comment and one `key = value` line per entry, and never did
  otherwise).

---

## 3. Gaming and performance — the standing hunt

The action queue closes items faster than it opens them, and that is a property of the
artifact, not evidence that the box is at its ceiling. This section is the **open candidate
space**: what could still move frametimes on this hardware, what each lever costs, and what
evidence would settle it.

**Nothing here is a recommendation until its gate returns.** Every row carries a gate, a
tier from §2 and a source. Rows without a documented mechanism are marked UNCERTAIN and stay
that way — an invented flag is worse than an empty section.

### 3a. Already at maximum — no headroom on these axes

Do not re-propose anything in this table as an improvement.

```
║ AXIS                ║ STATE          ║ WHY THERE IS NOTHING LEFT                               ║
║─────────────────────║────────────────║─────────────────────────────────────────────────────────║
║ CPU governor        ║ performance    ║ validated against ^[a-z][a-z0-9_-]*$; nothing           ║
║                     ║                ║ ranks above it                                          ║
║ CPU EPP             ║ performance    ║ maps to epp 0; any epp > 0 write returns -EBUSY         ║
║                     ║                ║ under this governor                                     ║
║ GPU DPM             ║ high           ║ only profile_peak is above it, and that is a            ║
║                     ║                ║ T5 decision, not an untried option                      ║
║ Idle / link latency ║ pinned         ║ max_cstate=1, ASPM disabled globally and at the         ║
║                     ║                ║ MT7925 endpoint, NVMe PS latency 0, USB and BT          ║
║                     ║                ║ autosuspend off                                         ║
║ Memory map ceiling  ║ 2147483642     ║ vm.max_map_count is at the kernel maximum               ║
║ Shader cache        ║ 16 G + local   ║ MESA_SHADER_CACHE_MAX_SIZE=16G and                      ║
║                     ║                ║ PROTON_LOCAL_SHADER_CACHE=1 both set                    ║
║ Governor authority  ║ exclusive      ║ power-profiles-daemon and ananicy-cpp masked, so        ║
║                     ║                ║ nothing competes for the policy                         ║
║ Vulkan stack        ║ RADV, unpinned ║ vulkan-radeon + lib32 via chwd, no ICD pin;             ║
║                     ║                ║ VK_EXT_descriptor_heap is default-on                    ║
║ Wine sync           ║ ntsync         ║ CONFIG_NTSYNC=m and proton-cachyos defaults to          ║
║                     ║                ║ it; the profile verifies rather than sets it            ║
║ Storage queue       ║ none           ║ NVMe scheduler none outranks the CachyOS kyber          ║
║                     ║                ║ vendor rule                                             ║
║ Log noise           ║ silenced       ║ WINEDEBUG, DXVK_LOG_LEVEL, VKD3D_DEBUG and              ║
║                     ║                ║ VKD3D_SHADER_DEBUG all off                              ║
```

### 3b. Open candidates, ranked by expected frametime effect

```
║ ID    ║ LEVER                    ║ TIER   ║ GATE — what would settle it                   ║
║───────║──────────────────────────║────────║───────────────────────────────────────────────║
║ G-1   ║ 85 W PPT ceiling (BIOS)  ║ BIOS   ║ PkgWatt pinned at 85 W under a 10-min game    ║
║       ║                          ║        ║ load => the box is power-bound and every      ║
║       ║                          ║        ║ software lever below returns little           ║
║ G-2   ║ GTT / VRAM budget        ║ T3 /   ║ a title that stutters at high texture         ║
║       ║                          ║ BIOS   ║ settings while MangoHud vram+gtt shows the    ║
║       ║                          ║        ║ carve exhausted                               ║
║ G-3   ║ vm.page-cluster=0 (+     ║ T2     ║ swap-in activity during a session at all;     ║
║       ║ watermark_scale_factor)  ║        ║ if zram is never touched the change is inert  ║
║       ║                          ║        ║ and should not ship                           ║
║ G-4   ║ mitigations=off          ║ T3     ║ a CPU-bound title measured both ways, plus    ║
║       ║                          ║        ║ an explicit §6 acceptance of the exposure     ║
║ G-5   ║ pp_power_profile_mode    ║ T5     ║ requires DPM manual — blocked until T5        ║
║       ║ 3D_FULL_SCREEN           ║ gated  ║ reopens; recorded so it is not re-derived     ║
║ G-6   ║ RADV_PERFTEST=nggc       ║ T1     ║ per-title A/B; the flag is non-default, so    ║
║       ║                          ║        ║ the burden is on the uplift                   ║
║ G-7   ║ MESA_VK_WSI_PRESENT_MODE ║ T1     ║ a title whose chosen present mode fights      ║
║       ║                          ║        ║ VRR; frametime graph, not average FPS         ║
║ G-8   ║ bring a 10 GbE link up   ║ host   ║ none — this is a cable, and it removes an     ║
║       ║                          ║        ║ entire latency class plus the wlan0 drop      ║
║ G-9   ║ MangoHud frametime       ║ T2     ║ none — it is the instrument every other       ║
║       ║ logging                  ║        ║ gate in this section needs                    ║
║ G-10  ║ MANGOHUD_DLSYM=1 for     ║ T1     ║ any OpenGL title in the library showing no    ║
║       ║ OpenGL titles            ║        ║ HUD                                           ║
║ G-11  ║ transparent_hugepage=    ║ T3     ║ measured both ways on a stutter-prone         ║
║       ║ always                   ║        ║ title; lean KEEP at the CachyOS default       ║
║ G-12  ║ shader-cache shape       ║ T1     ║ none — this is a trap to avoid, not a         ║
║       ║                          ║        ║ change to make                                ║
║ G-13  ║ PipeWire quantum         ║ T2 +   ║ audible crackle or measured audio latency     ║
║       ║                          ║ scope  ║ under load                                    ║
║ G-14  ║ proton-cachyos build     ║ host   ║ installed build vs 11.0-20260703              ║
║       ║ currency (A2)            ║        ║                                               ║
║ G-15  ║ gamescope session        ║ per-   ║ any title that misses the frame budget at     ║
║       ║                          ║ game   ║ native resolution                             ║
║ G-16  ║ CCD-aware core pinning   ║ per-   ║ a title whose 1% lows improve when confined   ║
║       ║                          ║ game   ║ to one CCD                                    ║
║ G-17  ║ amdgpu.ppfeaturemask     ║ UNCER- ║ do not propose a value — the safe set is      ║
║       ║                          ║ TAIN   ║ undocumented and APU OverDrive support on     ║
║       ║                          ║        ║ gfx1151 is not established                    ║
║ G-18  ║ cap the frame rate below ║ per-   ║ PkgWatt at 85 W with frametimes               ║
║       ║ the power wall           ║ game   ║ oscillating; a cap that holds the clock       ║
║       ║                          ║        ║ steady is the fix, not more headroom          ║
║ G-19  ║ per-app drirc instead of ║ T1 /   ║ any per-title Mesa workaround that is         ║
║       ║ global env vars          ║ scope  ║ currently a candidate for ENV_VARS —          ║
║       ║                          ║        ║ drirc is the correct home                     ║
║ G-20  ║ amdgpu.sg_display=0      ║ T3     ║ observed display flicker or corruption        ║
║       ║                          ║        ║ only; do not ship it prophylactically         ║
```

**G-1 · The 85 W ceiling is the frame of every other row.** The part is rated to 140 W and
the BIOS holds a flat 85 W. T0-1 measured the idle floor at ~21.5 W package; the headroom
between that and 85 W is what all sixteen remaining candidates compete for. Owned by the
BIOS reference repo, not by this profile — but any candidate below that raises sustained
draw is spending the *same* budget, so measure PkgWatt before and after every one of them.
Sustained clocks on a mini-PC are also thermally bound: the fan profile and the box's
placement are real, free, non-software variables and belong in the same measurement.

**G-2 · The GTT / VRAM budget is the most gaming-specific lever on this box.** 128 GB
unified, and the last host capture measured a **32 GiB VRAM carveout with 47 GiB GTT**. Two
real knobs exist besides the BIOS UMA setting: `ttm.pages_limit` (module param, `ulong`,
0644, `drivers/gpu/drm/ttm/ttm_tt.c`; when unset at init it takes the page count the driver
passes, so it is a *ceiling* the driver already chose) and `amdgpu.gttsize` (MB, `-1` =
auto, `drivers/gpu/drm/amd/amdgpu/amdgpu_drv.c`). Neither appears in
`kernel-parameters.txt` — cite the module sources. Titles that size their asset budget on
reported VRAM behave differently across the carve, and MangoHud's `vram` readout shows the
carveout only, so read `gtt` alongside it or the picture is wrong (T5 prohibition).
**Uncertainty to respect:** raising GTT does not raise bandwidth. On a unified part every
byte still crosses the same LPDDR5X, so a bigger budget removes eviction stutter and buys
nothing on a title that already fits.

**G-3 · Two documented zram sysctls the profile does not carry.** The ArchWiki Zram page's
recipe is `vm.swappiness`, `vm.watermark_boost_factor=0`, `vm.watermark_scale_factor=125`
and `vm.page-cluster=0`. The profile ships the boost factor at 0 and swappiness at 150 — two
of four — while `page-cluster` and `watermark_scale_factor` are absent. `page-cluster=0`
disables swap-in readahead, which on a compressed in-memory device is pure decompression
overhead; it is the anti-stutter half of the recipe. Cost: `SYSCTL_VALUES` 9 -> 10 or 11 and
one byte-anchor move on `95-ry-overrides.conf`; the file self-heals (§8f Tier 1). **Do not
ship it blind** — if the session never swaps, the key does nothing and the profile gains a
key it cannot justify.

**G-4 · `mitigations=off` is the largest remaining CPU-side lever and a genuine security
regression.** Documented in `kernel-parameters.txt` as a curated aggregation of the
arch-specific mitigation switches. Syscall-heavy and I/O-heavy paths — which is what a Wine
process tree is — pay the most for mitigations, so this is the one cmdline token that could
plausibly move 1% lows on this box. It is also the only candidate here that widens the
attack surface rather than the power envelope. **Quantify only; §6 owns the exposure and it
needs an explicit acceptance, not a benchmark alone.** Note the interaction with
`split_lock_detect=off`, already shipped: the profile has precedent for trading a protection
for latency, so the argument is about degree.

**G-5 · `pp_power_profile_mode` does not compose with the DPM decision.** The amdgpu docs
are explicit: the profile mode is adjustable when `power_dpm_force_performance_level` is
`manual`, and `manual` also hands sclk/mclk level selection to userspace via `pp_dpm_sclk` /
`pp_dpm_mclk` / `pp_dpm_pcie`. The shipped `high` gives a clock floor with gating still
active; `manual` gives neither unless every level is pinned by hand, and the profiling modes
above it disable gating outright. **So this is not "DPM high plus a gaming profile" — it is a
different power model.** Blocked behind a T5 reopen; recorded here so the next reader does
not re-derive it from the sysfs docs and mistake it for free.

**G-6 · `RADV_PERFTEST=nggc`.** Mesa documents `nggc` as NGG culling for GFX11+; gfx1151 is
GFX11.5, so it applies. It is a `PERFTEST` flag, meaning non-default by driver decision, so
a candidate must clear a real uplift bar rather than a "does not regress" bar. Two adjacent
entries are worth knowing and *not* setting: `nosam` disables the optimizations that engage
when all VRAM is CPU-visible, which on a unified part is exactly the wrong direction, and
`dmashaders` targets non-resizable-BAR systems, which this is not. Promotion to `ENV_VARS`
is Tier-1 (10 -> 11) and free to revert; per-game first.

**G-7 · `MESA_VK_WSI_PRESENT_MODE`.** Documented override of the client-requested present
mode: `fifo`, `relaxed`, `mailbox`, `immediate`. This is a frame-pacing tool, not a
throughput tool — judge it on a frametime graph, never on average FPS, and judge it together
with VRR rather than against it. Per-game.

**G-8 · The box games over Wi-Fi with both 10 GbE links down (T0-8).** For online play this
outranks every software row above it: a wired link removes an entire latency and jitter
class, and it sidesteps the recurring ~24 h `wlan0` reassociation drop that 7.163.0's
`autoconnect-retries-default=0` **mitigates rather than fixes** — infinite retries mean the
link comes back, not that it never left. A drop mid-match is still a drop. Cost: a cable.
The profile's BBR + `fq` sysctls and the removed `netdev_budget` keys are all downstream of
this choice; do not re-tune the stack around a link that is not in use.

**G-9 · The profile's HUD cannot produce evidence, and half this brief demands evidence.**
The shipped `MangoHud.conf` is deliberately readout-only: 18 active directives, CPU keys
commented. Every "evaluate on frametime evidence only" gate — DPM `high` vs `auto`, G-4,
G-6, G-11, G-15 — needs logging: an output folder, a log duration and the log keybind, or
the equivalent per-user override that leaves the managed body alone. **Sequencing matters:**
the `mangohud-gtr9-pro` companion repo is already divergent from the installer generator
(T1), so a logging block must land in the installer first and be mirrored, never the other
way round. This is the highest-leverage row in the section precisely because it unblocks the
others.

**G-10 · `MANGOHUD=1` covers the Vulkan layer only.** The OpenGL path is a `dlsym` hook, so
older OpenGL titles in the library run with no HUD at all and silently produce no
measurement. Session-scope, reversible, and it costs one `ENV_VARS` slot if promoted.

**G-11 · THP policy.** `transparent_hugepage=` is documented with `always|madvise|never`.
`always` is the classic Wine/Proton lever and the classic stutter source: khugepaged
compaction is exactly the work `vm.compaction_proactiveness=0` was set to suppress, so the
two levers argue with each other. Lean KEEP at the distribution default; if it is ever
measured, measure 1% lows, not averages.

**G-12 · A shader-cache trap to avoid, not a change to make.** Mesa documents that
`MESA_DISK_CACHE_SINGLE_FILE=1` selects the Fossilize DB implementation and that **this
implementation does not support size limits via `MESA_SHADER_CACHE_MAX_SIZE`** — so any
future "use Steam's precompiled Fossilize caches" suggestion would silently void the
profile's 16 G pin. Record it; do not act on it.

**G-13 · Audio latency has the packages but no configuration.** `rtkit` and
`realtime-privileges` are both in `PKGS_ADD`, so the privilege half is done, but nothing
configures the PipeWire quantum. A drop-in would be an **18th managed file** — three oracle
counts move and the scope-addition argument from T2-3 applies in full. Per-user first; only
promote if it measures.

**G-14 · Proton build currency is the cheapest recurring uplift on the list.** proton-cachyos
carries the FSR4 upgrade path and the Wayland and ntsync work that the profile has already
stopped setting variables for; the head is 11.0-20260703. This is advisory read A2 and it
should return every rebase.

**G-15 · gamescope buys more on an iGPU-class part than any sysctl in this brief.** A nested
compositor with a fixed render resolution, FSR/NIS upscaling and a frame limiter converts
the 85 W ceiling from a hard wall into a dial. Nothing to install — it is in the CachyOS
gaming stack already in `PKGS_ADD`. Per-game launch option, zero profile surface, fully
reversible: the ideal shape for a candidate on this box.

**G-16 · Core placement.** 16C/32T across two CCDs; a few engines lose 1% lows to cross-CCD
traffic. `taskset` in the launch options is the whole mechanism. Per-game, no profile
surface, and it is a measurement before it is a recommendation.

**G-17 · `amdgpu.ppfeaturemask` — UNCERTAIN, and staying that way.** It is a real module
param (hexint, 0444, `amdgpu_drv.c`), which is exactly why it is dangerous: the safe value
set is undocumented, the meaningful bits differ per ASIC generation, and OverDrive support
on an APU of this class is not established. **Do not propose a value.** Listed so that a
future pass records the same verdict instead of re-discovering the parameter and guessing.

**G-18 · A frame cap is the cheapest way to spend an 85 W budget well.** Under a hard power
ceiling an uncapped renderer oscillates: it sprints, hits the wall, throttles, recovers — and
the frametime graph shows it even when average FPS looks fine. Capping below the wall keeps
clocks steady and converts wasted headroom into consistency. Three real mechanisms, in order
of scope: DXVK's `dxgi.maxFrameRate` / `d3d9.maxFrameRate` in a `dxvk.conf` (documented in
upstream's shipped `dxvk.conf`), MangoHud's `fps_limit`, and gamescope's own limiter (G-15).
**Check the flag spelling against the installed binary's `--help`** rather than copying one
from a brief. Per-game, zero profile surface, and it composes with G-1's measurement instead
of competing with it.

**G-19 · The profile ships no drirc, and that is the right place for per-title fixes.** Mesa
reads per-application driver overrides from a drirc file; the profile deliberately sets none
(§8c). That matters for this section's discipline: when a title needs a RADV workaround, the
correct home is a per-app drirc entry, **not** a new global `ENV_VARS` member that applies to
every process on the box. G-6 and G-7 are both written per-game first for exactly this
reason. Promoting any of them to `ENV_VARS` is a scope decision that needs its own argument,
not a convenience.

**G-20 · `amdgpu.sg_display=0` is a stability lever, not a performance one.** Real module
param (`amdgpu_drv.c`, int, 0444, `-1 = auto (default), 0 = disable`). Scatter-gather display
lets the display engine scan out of system memory, which on a unified APU is the normal path;
disabling it has been the standard remedy for display flicker and corruption on APUs of this
family. **Gate it on an observed symptom.** Shipping it prophylactically costs a
`KERNEL_PARAMS` slot and forces the display engine into carveout memory for no measured
return — the opposite trade from G-2.

### 3c. Per-game and session practice — out of profile scope, recorded once

The profile is system-wide by design and §11 keeps per-game tuning secondary. These are the
practices that belong in Steam launch options rather than in any managed file:

- **`PROTON_ENABLE_WAYLAND=1` per-game.** Removed from `ENV_VARS` after 7.155.0 and closed
  as a global-scope item (T1-2), but upstream's framing was always per-game — aliased as
  `PROTON_USE_WAYLAND=1`, and Steam needs `-steamos3` for Steam Input under winewayland.
  Native Wayland skips XWayland's translation layer; it is a per-title decision because
  input and HDR behavior vary.
- **gamescope wrapper** — fixed render resolution plus upscale plus frame limit, applied per
  title (see G-15 and G-18). **Gotcha worth recording: `MANGOHUD=1` does not reach a game
  running inside gamescope** — the nested compositor needs its own HUD flag, so a
  gamescope-wrapped title silently produces no measurement and no overlay. That interacts
  directly with G-9 and G-10: check gamescope's `--help` on the installed build for the flag,
  and confirm the HUD actually appears before trusting any A/B run done under it.
- **CCD pinning** — `taskset` in front of `%command%` (see G-16).
- **FSR4 on gfx1151 is a measurement, not a win by default.** proton-cachyos carries the
  upgrade path and `FSR4_WATERMARK=1` is the on-screen proof it engaged — that is the
  variable's entire purpose, and it is why it survived when the upgrade variables were
  removed at 7.154.0. RDNA 3.5 is not the architecture FSR4 was built for, so an upgrade
  that costs frametime is not an upgrade. Check the watermark, then check the frame graph.
- **True fullscreen** for direct scanout, and VRR enabled at the display level — both are
  compositor-side and cost nothing.

### 3d. Measurement method — how to earn the evidence

Every gate above resolves to the same three-step protocol. Run it once per candidate, never
two candidates at a time.

1. **Establish the power frame first (G-1).** If PkgWatt sits at 85 W for the whole run, the
   box is power-bound and a clock-side candidate cannot help; re-rank accordingly.
2. **A/B one variable, same save, same scene, same duration.** Compare 1% and 0.1% lows and
   the frametime graph — never average FPS, which hides exactly the stutter these levers
   target.
3. **Record the result against the candidate ID**, including a null result. A measured null
   is what retires a row from §3b permanently; an unmeasured hunch keeps it alive forever.

The commands are in §7 under "Gaming measurement".

### 3e. Shortlist — what to do first, by effort against return

Ordered by return per unit of effort rather than by raw effect. The first four cost nothing
and touch no managed file.
```
║ #   ║ DO                                     ║ WHY IT IS HERE                                    ║
║─────║────────────────────────────────────────║───────────────────────────────────────────────────║
║ 1   ║ G-8 — plug in a 10 GbE link            ║ a cable removes a whole latency class and the     ║
║     ║                                        ║ wlan0                                             ║
║     ║                                        ║ drop; no profile surface at all                   ║
║ 2   ║ G-14 — confirm the proton-cachyos      ║ one read, and the build carries more uplift than  ║
║     ║ build                                  ║ any                                               ║
║     ║                                        ║ sysctl in this brief                              ║
║ 3   ║ G-15 + G-18 — gamescope with a frame   ║ converts the 85 W wall from a ceiling into a      ║
║     ║ cap, on one title                      ║ dial;                                             ║
║     ║                                        ║ per-game and fully reversible                     ║
║ 4   ║ G-1 — capture PkgWatt under load       ║ decides whether anything below it is worth        ║
║     ║                                        ║ running                                           ║
║ 5   ║ G-9 — give the HUD a logging path      ║ the only item that unblocks every remaining gate; ║
║     ║                                        ║ must land installer-first (T1 lockstep)           ║
║ 6   ║ G-2 — read vram AND gtt during a       ║ the most gaming-specific lever on this box, and   ║
║     ║ stutter                                ║ the                                               ║
║     ║                                        ║ reading itself is free                            ║
║ 7   ║ G-3 — check pswpin/pswpout first       ║ a two-key sysctl change, inert unless the session ║
║     ║                                        ║ actually swaps                                    ║
║ 8   ║ G-6 / G-7 — per-game Mesa A/B          ║ cheap to test, non-default by driver decision, so ║
║     ║                                        ║ the burden is on the uplift                       ║
║ 9   ║ G-4 — measure, then cost it in §6      ║ largest CPU-side lever left and the only          ║
║     ║                                        ║ candidate                                         ║
║     ║                                        ║ that widens the attack surface                    ║
```

### 3f. Protected in this space — do not re-propose

- **sched_ext.** `scx-tools`, `scx-scheds` and `scx_loader.service` were **rejected by the
  maintainer by name** (2026-08-15) — out of the profile in full: no `PKGS_ADD` member, no
  `EXPECTED_SERVICES` member, no count movement, no gate. This is a decision, not a gap. Do
  not re-propose a sched_ext scheduler unless it is reopened by name.
- **`profile_peak`** — T5, decided, and G-5 records why the sysfs docs make it look freer
  than it is.
- **`preempt=` pinning** — the advisory was removed at 7.139.0 r2; the profile has never
  pinned it and a "missing preempt check" is not a finding.
- **Both `net.core.netdev_budget` keys** — measured inert by T0-5 and removed at 7.157.0.
- **`pcie_aspm=off`** — does not disable ASPM (T3-4); never an equivalent for the policy
  token.
- **Host-side DoT and any resolver pin** — T5.
- **Per-key sysctl annotation** and **MangoHud `vram` on UMA** — both are recurring false
  findings (T5).

---

## 4. Settled — do not re-research

Confirmed from primary sources. Re-verify only if a citation is challenged. **VERIFIED** is
the date the claim was last checked against the live source.

```
║ CLAIM                          ║ VERDICT / SOURCE                                 ║ VERIFIED ║
║────────────────────────────────║──────────────────────────────────────────────────║──────────║
║ performance governor vs a      ║ REJECTS. epp > 0 && policy == CPUFREQ_POLICY_    ║ 26-08-15 ║
║ non-max EPP write              ║ PERFORMANCE returns -EBUSY, and the rejection is ║          ║
║                                ║ pr_debug — never in default dmesg. Writing       ║          ║
║                                ║ "performance" maps to epp 0 and IS accepted, so  ║          ║
║                                ║ the udev rule is a redundant no-op               ║          ║
║ available-preferences readout  ║ under the performance policy the sysfs file      ║ 26-08-15 ║
║                                ║ emits ONLY "performance"                         ║          ║
║ dynamic_epp availability       ║ CLOSED. Present here (ships since 7.1); kernel   ║ 26-08-16 ║
║                                ║ default false. When enabled it blocks ALL manual ║          ║
║                                ║ EPP writes with -EBUSY. T0-6 read "disabled"     ║          ║
║                                ║ live                                             ║          ║
║ amd_dynamic_epp= boot param    ║ DOCUMENTED (disable | enable). The profile does  ║ 26-08-16 ║
║                                ║ not set it and should not; if a future default   ║          ║
║                                ║ flips, the udev EPP write breaks                 ║          ║
║ amd_pstate default mode on     ║ X86_AMD_PSTATE_DEFAULT_MODE=3 = Active (EPP), so ║ 26-08-15 ║
║ CachyOS                        ║ amd_pstate=active RESTATES the compiled default  ║          ║
║ pcie_aspm.policy=performance   ║ DISABLES. pcie/Kconfig: PCIEASPM_PERFORMANCE     ║ 26-08-05 ║
║ semantics                      ║ disables L0s and L1 even where the BIOS enabled  ║          ║
║                                ║ them. Cite the Kconfig, not kernelconfig.io      ║          ║
║ is the ASPM token load-bearing ║ YES, and measured. PCIEASPM_DEFAULT=y with       ║ 26-08-15 ║
║ on CachyOS?                    ║ PCIEASPM_PERFORMANCE NOT set; T0-2 read LnkCtl   ║          ║
║                                ║ ASPM Disabled on every link                      ║          ║
║ pcie_aspm=off semantics        ║ "Don't touch ASPM configuration at all." Does    ║ 26-08-15 ║
║                                ║ NOT disable ASPM                                 ║          ║
║ clearcpuid documentation state ║ ZERO occurrences in kernel-parameters.txt while  ║ 26-08-15 ║
║                                ║ the code still parses it; also sets              ║          ║
║                                ║ TAINT_CPU_OUT_OF_ SPEC. Dropped from the profile ║          ║
║                                ║ at 7.160.0                                       ║          ║
║ amd_iommu=on parser status     ║ NOT a parsed value. parse_amd_iommu_options()    ║ 26-08-15 ║
║                                ║ accepts fullflush / force_enable / off /         ║          ║
║                                ║ force_isolation / pgtbl_v1 / pgtbl_v2 /          ║          ║
║                                ║ irtcachedis / nohugepages / v2_pgsizes_only;     ║          ║
║                                ║ anything else logs "AMD-Vi: Unknown option".     ║          ║
║                                ║ IOMMU init is the hardware default — T3-5        ║          ║
║ iommu=pt semantics on CachyOS  ║ LOAD-BEARING. pt = passthrough default domain,   ║ 26-08-15 ║
║                                ║ equal to iommu.passthrough=1. CachyOS ships      ║          ║
║                                ║ IOMMU_DEFAULT_ DMA_LAZY=y with                   ║          ║
║                                ║ IOMMU_DEFAULT_PASSTHROUGH unset.                 ║          ║
║                                ║ DRM_ACCEL_AMDXDNA=m — the NPU driver exists to   ║          ║
║                                ║ load                                             ║          ║
║ mt7925e.disable_aspm exists    ║ YES. mt76/mt7925/pci.c, perm 0644, calls         ║ 26-08-05 ║
║                                ║ mt76_pci_ disable_aspm() at probe and suppresses ║          ║
║                                ║ aspm_supported. MT7927 is force-disabled by a    ║          ║
║                                ║ separate quirk; MT7925 is not                    ║          ║
║ mitigations= is documented     ║ YES. kernel-parameters.txt: a curated, arch-     ║ 26-08-16 ║
║                                ║ independent aggregation of the arch-specific     ║          ║
║                                ║ mitigation options. Basis for G-4                ║          ║
║ transparent_hugepage= is       ║ YES. Format [always|madvise|never]. Basis for    ║ 26-08-16 ║
║ documented                     ║ G-11                                             ║          ║
║ ttm.pages_limit /              ║ YES, both module params, neither in kernel-      ║ 26-08-16 ║
║ amdgpu.gttsize exist           ║ parameters.txt. pages_limit: ttm_tt.c, ulong,    ║          ║
║                                ║ 0644, defaults to the page count the driver      ║          ║
║                                ║ passes at init. gttsize: amdgpu_drv.c, MB, -1 =  ║          ║
║                                ║ auto. Basis for G-2                              ║          ║
║ amdgpu.ppfeaturemask exists    ║ YES — amdgpu_drv.c, hexint, 0444, described only ║ 26-08-16 ║
║                                ║ as "all power features enabled (default)". No    ║          ║
║                                ║ documented safe value set. G-17 stays UNCERTAIN  ║          ║
║ pp_power_profile_mode          ║ Requires                                         ║ 26-08-16 ║
║ precondition                   ║ power_dpm_force_performance_level=manual, which  ║          ║
║                                ║ also hands sclk/mclk/pcie level selection to     ║          ║
║                                ║ userspace; the profiling modes disable clock and ║          ║
║                                ║ power gating. Basis for G-5                      ║          ║
║ RADV_PERFTEST flag set         ║ nggc = NGG culling for GFX11+ (gfx1151 is        ║ 26-08-16 ║
║                                ║ GFX11.5). Adjacent and NOT to be set here: nosam ║          ║
║                                ║ disables the all-VRAM-CPU-visible optimizations, ║          ║
║                                ║ dmashaders targets non-resizable-BAR systems     ║          ║
║ MESA_VK_WSI_PRESENT_MODE       ║ DOCUMENTED override of the client present mode:  ║ 26-08-16 ║
║                                ║ fifo, relaxed, mailbox, immediate. Basis for G-7 ║          ║
║ MESA_SHADER_CACHE_MAX_SIZE     ║ number + optional K/M/G, gigabytes assumed, 1 GB ║ 26-08-16 ║
║                                ║ if unset. MESA_DISK_CACHE_SINGLE_FILE=1 switches ║          ║
║                                ║ to the Fossilize DB, which does NOT honour the   ║          ║
║                                ║ size limit — G-12                                ║          ║
║ RADV descriptor heap           ║ DEFAULT-ON. RADV_DEBUG=noheap disables VK_EXT_   ║ 26-08-05 ║
║                                ║ descriptor_heap; heap is gone from               ║          ║
║                                ║ RADV_EXPERIMENTAL                                ║          ║
║ vm zram sysctl recipe          ║ ArchWiki Zram: swappiness,                       ║ 26-08-16 ║
║                                ║ watermark_boost_factor=0,                        ║          ║
║                                ║ watermark_scale_factor=125, page-cluster=0. The  ║          ║
║                                ║ profile carries two of the four — G-3            ║          ║
║ ntsync currency                ║ CONFIG_NTSYNC=m in linux-cachyos. ntsync is the  ║ 26-08-15 ║
║                                ║ default; PROTON_NO_NTSYNC=1 is the opt-out. The  ║          ║
║                                ║ profile neither sets nor forces it, correctly    ║          ║
║ PROTON_FSR4_UPGRADE currency   ║ MOOT — removed at 7.154.0 with the RDNA3         ║ 26-08-16 ║
║                                ║ variable because proton-cachyos 11.0-20260702+   ║          ║
║                                ║ copies amdxcffx64.dll itself. FSR4_WATERMARK=1   ║          ║
║                                ║ ships instead and is a VERIFICATION variable     ║          ║
║ PROTON_ENABLE_WAYLAND scope    ║ MOOT at system scope — removed after 7.155.0.    ║ 26-07-27 ║
║                                ║ Upstream framing stands per-game (aliased        ║          ║
║                                ║ PROTON_USE_ WAYLAND=1; Steam needs -steamos3 for ║          ║
║                                ║ Steam Input under winewayland) — §3c             ║          ║
║ MangoHud                       ║ CURRENT as an option, INERT on this box.         ║ 26-08-15 ║
║ cpu_custom_temp_sensor         ║ UpdateCpuTemp() short-circuits to the APU        ║          ║
║                                ║ gpu_metrics value on any APU before a hwmon file ║          ║
║                                ║ is read                                          ║          ║
║ MangoHud cpu_power source      ║ APU metric, not RAPL. InitCpuPowerData() order:  ║ 26-08-15 ║
║                                ║ hwmon -> APU gpu_metrics -> powercap RAPL, and   ║          ║
║                                ║ Zen 5 k10temp exposes no power inputs, so        ║          ║
║                                ║ energy_uj is never consulted and T2-3 costs the  ║          ║
║                                ║ HUD nothing                                      ║          ║
║ MangoHud #1794                 ║ STILL OPEN — cpu_power reads 0 when cpu_temp is  ║ 26-08-16 ║
║                                ║ active, reported on a RAPL-sourced Zen 5         ║          ║
║                                ║ desktop. Moot here: both CPU keys ship commented ║          ║
║ netdev_budget attribution      ║ Kernel defaults 300 / 2000. The 600/4000 pair is ║ 26-07-27 ║
║                                ║ RED HAT's, NOT ESnet's — ESnet recommends the    ║          ║
║                                ║ defaults. Both gate on softnet column 3,         ║          ║
║                                ║ measured 0 by T0-5                               ║          ║
║ nftables                       ║ NO UPSTREAM RULE either way. Gentoo uses the     ║ 26-07-27 ║
║ invalid-before-loopback        ║ profile's order; nftables.org and ArchWiki put   ║          ║
║                                ║ loopback first. KEEP                             ║          ║
║ NM vs resolved DNS precedence  ║ systemd #33973 closed-completed — per-link DHCP  ║ 26-08-16 ║
║                                ║ DNS outranks global DNS=. Domains=~. is NOT the  ║          ║
║                                ║ fix (#33579, OPEN). The host pins nothing, so    ║          ║
║                                ║ the problem has no surface here                  ║          ║
║ resolv.conf mode on this host  ║ FOREIGN (T0-8) — not resolved's stub, so the     ║ T0-8     ║
║                                ║ managed drop-in binds only resolved-routed       ║          ║
║                                ║ queries. Low impact, no edit                     ║          ║
║ MES firmware timeline (gfx1151 ║ 2025-11-19 update -> 2025-12-01 REVERT ->        ║ 26-08-15 ║
║ hang)                          ║ 2026-02-25 re-land (=0x86) -> 2026-05-07 update, ║          ║
║                                ║ still the head. First tag shipping 0x86 =        ║          ║
║                                ║ 20260309. UNIT VERIFIED 2026-08-14: MES 0x91 /   ║          ║
║                                ║ KIQ 0x75. Given the revert precedent, advise     ║          ║
║                                ║ "newest tag", never a minimum                    ║          ║
║ POWERDEVIL_NO_DDCUTIL=1        ║ EXACT variable PowerDevil reads. Not a silent    ║ 26-07-25 ║
║                                ║ no-op                                            ║          ║
║ nmi_watchdog vendor            ║ CachyOS 70-cachyos-settings.conf already sets    ║ 26-07-25 ║
║ interaction                    ║ nmi_watchdog=0 and swappiness 100 and ships a    ║          ║
║                                ║ zram-generator config, so swappiness=150 is      ║          ║
║                                ║ coherent. Priority 95 loads after 70 — correct   ║          ║
║                                ║ ORDERING. KEEP the redundancy or the value       ║          ║
║                                ║ depends on a package ry does not own             ║          ║
║ energy_uj readability          ║ MODE 400 on this host (T0-4) — root-only.        ║ T0-4     ║
║                                ║ Post-PLATYPUS kernels restrict RAPL              ║          ║
║                                ║ deliberately; re-opening it re-opens the side    ║          ║
║                                ║ channel                                          ║          ║
║ RTL8127 in r8169               ║ present in mainline r8169_main.c; first landed   ║ 26-07-27 ║
║                                ║ v6.16                                            ║          ║
║ ROCm #6165                     ║ CLOSED (an earlier pass said open)               ║ 26-07-25 ║
║ fish set -l block scoping      ║ documented and stable — erased when the block    ║ 26-07-25 ║
║                                ║ ends; dates to fish 3.3, well below the 3.6      ║          ║
║                                ║ floor                                            ║          ║
```

**Known-stale sources — do not cite:** `wireless.docs.kernel.org` for `pcie_aspm` semantics
(pre-6.9 text; Helgaas's v6.9 fix, commit 2e0239d47d75, is the correction); kernelconfig.io
for Kconfig introduction versions; any source attributing the 600/4000 netdev pairing to
ESnet.

---

## 5. Corrections this rebase makes

Binding. Do not re-raise anything this list withdraws. The previous edition's corrections
stand and are not restated.

1. **The pin is 7.163.0, and the delta is two generated bodies, not five.** `ENV_VARS` 9 ->
   10 (`GSK_RENDERER=ngl`) and a new `[main]` section in the NM drop-in
   (`autoconnect-retries-default=0`). Sigma 5,338 -> 5,393 B. Any claim that `ENV_VARS` is 9
   is now a stale-source error, as is any byte anchor of 282 B for `10-environment.conf` or
   148 B for `99-cachyos-nm.conf`.
2. **The 7.162.0 escape's standing check has now been exercised and passed.** A count moved
   and `_ir_validate_counts` moved with it. Record the pass — the check is cheap precisely
   because it is usually silent.
3. **The harness cut did not move.** L4790 at both 7.162.2 and 7.163.0, after moving on five
   of the previous six rebases. This is a fact about this release only; keep locating the
   banner.
4. **A new class of gap is on the record: a managed value with no assertion (T2-6).** The
   previous editions' framing — "the dynamic verifiers follow any array edit automatically"
   — is true and was used to explain why removing a variable needed no verify edit. Its
   converse was never stated: a value hardcoded inside a generator body has no array to
   follow, so it gets no verifier unless one is written by hand. Both 7.163.0 changes are in
   this brief because they are the two halves of that contrast.
5. **The audit's own scope was too narrow.** Four rebases running, the queue's answer to
   "what should change" has been "nothing, the artifact already closed it". That is a
   correct audit result and a useless research result. §3 is the correction: the brief now
   carries a standing candidate surface for gaming and performance, with gates, and §11
   requires it to be returned every rebase.
6. **`FSR4_WATERMARK=1` is not a performance variable and never was.** It is on-screen
   verification that the FSR4 path engaged — which is why it survived the 7.154.0 removal of
   the two upgrade variables. Do not report it as a tuning value, and do not report its
   presence as evidence that FSR4 is beneficial on gfx1151 (§3c).
7. **Line numbers are gone from this edition.** They moved +2 to +4 on five of the last six
   rebases and cost re-derivation every time while never once being load-bearing. §9 now
   names owners, not addresses. **Always locate a symbol.**

---

## 6. Security posture — quantify only, ordered by exposure

No auto-FIX in this section.

1. **IOMMU on in passthrough mode** (`amd_iommu=on iommu=pt`) — the IOMMU initializes,
   interrupt remapping is live, VFIO/SR-IOV are possible and the XDNA NPU loads. What `pt`
   trades: the **default domain is identity**, so DMA from kernel-owned devices (NVMe, NICs,
   USB4/TB ingress) is not translated or isolated. That is the point of `pt` — near-zero
   DMA-mapping overhead — and it is the residual to quantify. Devices handed to VFIO get
   translated domains regardless. The fallback entry boots the compiled default instead, so
   **the fallback is the *more* hardened DMA posture** (T4-1).
2. **DNS: plaintext on the LAN leg only, encrypted and validated beyond the router.**
   Host<->router is plaintext DHCP-supplied DNS with no host-side validation; router<->AdGuard
   is DoT with DNSSEC validated there. Residual: the LAN segment, and trusting the router's
   answer — the AD bit is a bit any in-path party could set and nothing on the host checks
   it. Accepted on file (T5).
3. **`mitigations=off` is NOT shipped, and G-4 proposes evaluating it.** Recorded here so
   the exposure is costed in this section rather than inside a benchmark: it would disable
   the curated set of CPU-vulnerability mitigations system-wide, on a machine that also runs
   a browser and a Wine process tree fed by third-party game content. **A G-4 recommendation
   is incomplete without an explicit acceptance in this list.** Precedent, not permission:
   `split_lock_detect=off` already trades a protection for latency.
4. **IPv6 disabled + inbound IPv4 ping accepted** — net LAN delta is `+ping -mDNS`. Avahi is
   masked (unit *and* socket) and resolved has `MulticastDNS=no`, so multicast discovery is
   fully closed. Ping-accept is an asserted regression guard. Since 7.159.0 the ruleset
   carries the ICMPv6 base accept, inert under `ipv6.disable=1` and live on the fallback, so
   removing the token no longer silently breaks NDP; the residue is that dual-stack service
   rules would need adding by hand, which preflight WARNs about rather than refusing.
5. **`split_lock_detect=off`** — a misbehaving application can degrade the whole system.
6. **ufw masked rather than removed** — the package stays installed and could be unmasked and
   started, at which point two firewall managers contend for the same netfilter tables. The
   nftables-first gate (T4-4) is the only thing between a mis-sequenced run and an
   unfirewalled window.
7. **sdboot-manage drop-in override path** — a packaged drop-in can replace `LINUX_OPTIONS`
   entirely. WARN-only (T4-3 / T4-5).
8. **No sleep path** — all five sleep/suspend/hibernate targets masked, so an always-on box
   never gets the "locked on resume" checkpoint. Deliberate.
9. **RAPL stays restricted.** `energy_uj` is mode 400 and the permission drop-in is declined
   (T2-3) — a posture *win*, and doubly moot now that MangoHud sources `cpu_power` from the
   APU metric.
10. **UMIP restored** — `clearcpuid=umip` removed at 7.160.0. The descriptor-table base leak
    is closed and the kernel is untainted.
11. **The generalizable lesson from the `.ry.orig` dead-code window** (7.109.0 - 7.135.0, 13
    of 17 destinations overwritten with no backup; fixed at 7.135.1, void on this host since
    the 2026-07-19 reinstall): **presence of a guard in source is not evidence the guard
    runs.** Any claim of the form "X is protected because the code does Y" needs a behavioral
    probe. T0-7 is that probe.

**Default-deny-inbound ships and is net positive.** Residual notes: `flush ruleset` blast
radius against docker/libvirt/podman; no ICMP or new-connection rate limit (trusted-LAN
assumption — state it).

---

## 7. Verify block

All commands are reads unless marked. **Every command is self-resolving — no token needs
manual substitution.** A fence that requires the operator to fill in a device node or a path
gets pasted verbatim and fails; that has happened twice.

**T0 regression forms — the gates have returned, so these carry expected values:**

```fish
# T0-1  idle floor. Expect ~21.3-21.7 W package, ~3.9-4.2 W core at Busy% < 0.3
sudo turbostat --quiet --show PkgWatt,CorWatt,Busy%,Bzy_MHz,PkgTmp --interval 5 --num_iterations 12
cat /sys/class/drm/card*/device/hwmon/hwmon*/power1_average    # GPU package draw at idle
cat /sys/devices/system/cpu/cpu0/cpuidle/state*/name           # C-state depth under max_cstate=1

# T0-2  every PCIe link must read "ASPM Disabled"
sudo lspci -vv 2>/dev/null | grep -o 'ASPM [A-Za-z0-9 ]*' | sort | uniq -c
cat /sys/module/mt7925e/parameters/disable_aspm                # Y or 1 — endpoint option live

# T0-3  root FS type. Expect ext4 — this is what retired fsck.mode=force
findmnt -no FSTYPE,OPTIONS /
findmnt -no TARGET,OPTIONS -t ext4                             # noatime,lazytime,commit=10

# T0-4  sensors and RAPL permissions
for d in /sys/class/hwmon/hwmon*; echo $d (cat $d/name); end    # resolve by NAME, never index
sensors 2>/dev/null | grep -A2 '^k10temp'                      # expect Tctl only, no Tdie/Tccd
stat -c '%a %n' /sys/class/powercap/*/energy_uj                # expect 400 — root-only (T2-3)

# T0-5  softnet_stat squeezed column. Expect 0 on all 32 CPUs
awk '{print strtonum("0x" $3)}' /proc/net/softnet_stat | sort -n | uniq -c

# T0-6  EPP mechanics
cat /sys/devices/system/cpu/amd_pstate/dynamic_epp             # expect: disabled
grep -c 'amd_dynamic_epp' /proc/cmdline                        # expect 0 — must not be set

# T0-7  backup inventory. Expect 4 .ry.bak + up to 2 .ry.orig, no user-scope .ry.orig
find /etc /boot -maxdepth 3 -name '*.ry.bak' -o -maxdepth 3 -name '*.ry.orig' 2>/dev/null | sort
find "$HOME/.config" -name '*.ry.orig' 2>/dev/null             # expect empty on a fresh adoption

# T0-8  DNS and routing reality
resolvectl status                                              # per-link DHCP DNS; NO global server
head -n 20 /etc/resolv.conf                                    # foreign mode — not resolved's stub
ip -brief link show                                            # both 10 GbE links DOWN (G-8)
ip route show default                                          # default route is wlan0
```

**CPU / GPU state:**

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
cat /sys/block/nvme*n*/queue/scheduler                                   # [none] on every NVMe
```

**Gaming measurement (§3d). Frame the run with power first, then A/B one variable:**

```fish
# G-1  power frame. 85 W sustained means the box is power-bound
sudo turbostat --quiet --show PkgWatt,CorWatt,GFXWatt,Busy%,Bzy_MHz,PkgTmp --interval 10 --num_iterations 60

# G-2  memory budget while a title runs. Read gtt WITH vram — vram is the carveout only
cat /sys/class/drm/card*/device/mem_info_vram_total /sys/class/drm/card*/device/mem_info_vram_used
cat /sys/class/drm/card*/device/mem_info_gtt_total  /sys/class/drm/card*/device/mem_info_gtt_used
cat /sys/module/ttm/parameters/pages_limit                     # ceiling in pages, 4 KiB each
cat /sys/module/amdgpu/parameters/gttsize                      # -1 = auto

# G-3  is zram ever touched? A never-swapping session makes page-cluster inert
swapon --show; zramctl
grep -E '^(pswpin|pswpout)' /proc/vmstat                       # non-zero => the G-3 gate is open
sysctl vm.page-cluster vm.watermark_scale_factor vm.watermark_boost_factor vm.swappiness

# G-4  mitigation state, per vulnerability
grep -r . /sys/devices/system/cpu/vulnerabilities/ 2>/dev/null

# G-5  what the DPM decision forecloses. profile_peak/manual are T5-gated
cat /sys/class/drm/card*/device/pp_power_profile_mode          # current mode table
cat /sys/class/drm/card*/device/pp_dpm_sclk                    # levels; * marks active

# G-6/G-7  are the Mesa levers set? Expect NOT SET — they are candidates, not shipped
systemctl --user show-environment | grep -c 'RADV_PERFTEST\|MESA_VK_WSI_PRESENT_MODE'

# G-9  can the HUD produce evidence at all? Expect 0 today — that is the finding
grep -c 'output_folder\|log_duration\|toggle_logging' ~/.config/MangoHud/MangoHud.conf

# G-14  installed Proton builds (A2). Compare against 11.0-20260703
pacman -Q 2>/dev/null | grep -i proton
ls -1 ~/.steam/root/compatibilitytools.d 2>/dev/null
```

**Cmdline tokens and their removals:**

```fish
grep -o 'fsck\.mode=[a-z]*' /proc/cmdline             # auto since 7.158.0 — NOT force
grep -o 'amd_iommu=[^ ]*\|iommu=pt\|ipv6\.disable=[^ ]*' /proc/cmdline   # on + pt + 1
grep -c 'clearcpuid' /proc/cmdline                    # 0 — removed at 7.160.0
grep -o 'pcie_aspm[^ ]*\|mt7925e[^ ]*' /proc/cmdline
grep -o 'processor\.max_cstate=[^ ]*\|amd_pstate=[^ ]*' /proc/cmdline
grep -c 'nowatchdog\|tsc=reliable\|8250\|preempt=\|mitigations=' /proc/cmdline   # 0 — none shipped
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l   # NON-zero — IOMMU on
sudo dmesg | grep 'Default domain type'               # Passthrough (set via kernel command line)
sudo dmesg | grep -i 'AMD-Vi' | head -n 5             # an "Unknown option" line is T3-5
```

**Boot chain:**

```fish
ls /usr/lib/sdboot-manage.conf.d /etc/sdboot-manage.conf.d 2>/dev/null   # expect: absent or empty
grep -rc 'LINUX_OPTIONS' /etc/sdboot-manage.conf.d 2>/dev/null           # any hit outranks the conf
grep -h '^options' /boot/loader/entries/*.conf | grep -v fallback        # 15 tokens per entry
grep -h '^options' /boot/loader/entries/*fallback*.conf                  # "quiet" only — T4-1
grep '^COMPRESSION' /etc/mkinitcpio.conf                                 # zstd and (-3)
df -h /boot                                                              # the gate T4-2 never ran
bootctl status | grep -i 'default\|timeout\|entry'
```

**Memory, network, DNS:**

```fish
sysctl kernel.nmi_watchdog net.ipv4.tcp_congestion_control net.core.default_qdisc \
       net.ipv4.tcp_notsent_lowat net.ipv4.tcp_slow_start_after_idle \
       vm.max_map_count vm.compaction_proactiveness vm.swappiness vm.watermark_boost_factor
sysctl net.core.netdev_budget net.core.netdev_budget_usecs   # 300 / 2000 defaults, unmanaged
grep -c '^[a-z]' /etc/sysctl.d/95-ry-overrides.conf          # 9 keys
systemd-analyze cat-config sysctl.d | grep -n 'nmi_watchdog' # 95-ry wins over vendor 70
resolvectl status | grep -i 'DNS Servers\|DNSSEC\|DNSOverTLS'  # per-link only; no global pin
grep -c 'autoconnect-retries-default=0' /etc/NetworkManager/conf.d/99-cachyos-nm.conf  # 1 — T2-6
grep -c 'global-dns' /etc/NetworkManager/conf.d/99-cachyos-nm.conf   # 0 — removed 7.147.0
journalctl -b -u NetworkManager | grep -c 'link timed out'   # the wlan0 drop G-8 rides on
iw reg get | grep -i country                                 # US
```

**Firewall and units:**

```fish
sudo nft list chain inet filter input   # policy drop; invalid-drop FIRST, then est/rel, then lo
sudo nft -c -f /etc/nftables.conf
grep -c '^ *icmpv6' /etc/nftables.conf  # 1 — the base accept, shipped 7.159.0
systemctl is-enabled ufw.service        # masked — NOT "not installed"
systemctl is-enabled sleep.target suspend.target hibernate.target \
                     hybrid-sleep.target suspend-then-hibernate.target   # masked x5
systemctl list-unit-files --state=masked,masked-runtime --no-legend --plain  # compare vs MASK 11
systemctl is-enabled avahi-daemon.service avahi-daemon.socket bluetooth.service
systemctl --user --failed --plain --no-legend          # empty
stat -c '%a %U:%G %n' /etc/NetworkManager/system-connections/*   # 0600 root:root
```

**Userspace / HUD:**

```fish
lsmod | grep -c '^amdxdna'                                # >= 1 — the NPU driver loads now
grep -c '^blacklist' /etc/modprobe.d/60-ry-modules.conf   # 0 — comment-only, BLACKLIST false
ls /etc/modprobe.d/60-ry-*.conf                           # ONLY 60-ry-modules.conf
ls -l /dev/ntsync                                         # present (assert-only)
vulkaninfo | grep -i 'driverName\|deviceName'             # RADV / Radeon 8060S, no ICD pin
systemctl --user show-environment | grep -c '^[A-Z]'      # includes the 10 ENV_VARS
grep -c '^[A-Z]' ~/.config/environment.d/10-environment.conf   # 10 variables since 7.163.0
grep -c '^GSK_RENDERER=ngl' ~/.config/environment.d/10-environment.conf   # 1 — the GTK4 workaround
grep -c '^cpu_stats\|^cpu_temp' ~/.config/MangoHud/MangoHud.conf   # 0 — both commented (T1-1)
grep -c '^[a-z]' ~/.config/MangoHud/MangoHud.conf         # 18 active directives
```

**`grep -c` exits 1 with no output on zero matches.** Read every zero-hit assertion above as
"exit 1 or `0`", not as a failure. `rg` is equivalent for these, but sandbox rg 14.1.0 carries
a false-negative bug — re-confirm zero-hit results with `grep` or `python3`.

**A newly added `ENV_VARS` member WARNs, not fails, until the next login.** `systemd --user`
does not import a variable retroactively, so `GSK_RENDERER` reads NOT SET in any session that
predates the 7.163.0 deploy. That is the addition-side twin of the §8f Tier-3 removal row —
a session-lifetime question, never drift.

---

## 8. Reference data

All values live-evaluated from the 7.163.0 script. **No line numbers appear in this edition**
— they moved +2 to +4 on five of the last six rebases and were never load-bearing. Locate a
symbol.

### 8a. Count oracle — 21 tripwires, asserted by `_ir_validate_counts`

```
║ KERNEL_PARAMS            15 ║ PKGS_ADD                 16  ║ _RY_BOOT_CRITICAL_DSTS  4 ║
║   (14 at 7.160.0-7.161.0)   ║ PKGS_DEL                  9  ║ _RY_PHASE_NAMES         6 ║
║ MKINITCPIO_HOOKS         11 ║ MASK                     11  ║ _RY_BACKUP_TARGETS      4 ║
║ MKINITCPIO_MODULES        1 ║ EXPECTED_VULKAN_PKGS      2  ║ _RY_TMPDIR_GLOBS        6 ║
║ MKINITCPIO_COMPRESSION_   1 ║ EXPECTED_SERVICES         5  ║ SYSTEM_DESTINATIONS    15 ║
║   OPTIONS  (2 pre-7.140)    ║ _RY_PKG_MANAGED_SERVICES  1  ║ USER_DESTINATIONS       2 ║
║ ENV_VARS                 10 ║ _RY_POST_HOOKS           17  ║ _RY_ARGPARSE_SPEC       6 ║
║   (9 before 7.163.0)        ║ SYSCTL_VALUES              9 ║                           ║
```

**`ENV_VARS` moved 9 -> 10 at 7.163.0 and the tripwire literal moved with it** — the standing
check that came out of the 7.162.0 escape passed on its first real exercise. `KERNEL_PARAMS`
round-tripped 15 -> 14 -> 15 across 7.160.0 - 7.162.0, which is why **count parity is
evidence of nothing.** `_RY_EPP_LEVELS` (5) and `_RY_DPM_LEVELS` (9) are deliberately not in
the oracle — both are value-bearing, not count invariants. Managed files = 17 (15 system + 2
user), recomputed at load; a mismatch refuses with exit 3. **Count these by live fish eval,
never by text parsing.**

**Standing check, earned the hard way:** every array-count change diffs the
`_ir_validate_counts` literals explicitly, and every certification runs that function
**unshadowed with stderr visible**. 7.162.0 shipped a `KERNEL_PARAMS:14` literal after the
count moved to 15 and every run refused rc 3 until 7.162.1; the certifying battery missed it
because its harness sourced with `2>/dev/null`, which swallows `_err_loud`.

**The oracle is a weak detector of content change.** It has now missed a zero-delta body
edit, a count-invisible value swap, and a round-trip. Only the embedded bodies (§8e) catch
every class.

### 8b. Perf scalars, maxima, and the eleven change sites

```
║ GLOBAL                  ║ VALUE          ║ CEILING                                        ║
║─────────────────────────║────────────────║────────────────────────────────────────────────║
║ CPUPOWER_GOVERNOR       ║ performance    ║ AT MAX (regex ^[a-z][a-z0-9_-]*$)              ║
║ GPU_DPM_LEVEL           ║ high           ║ CLOSED at high (T5) — not the same as at max   ║
║ EPP_PREFERENCE          ║ performance    ║ AT MAX (_RY_EPP_LEVELS, 5 members)             ║
║ EXPECTED_SCALING_DRIVER ║ amd-pstate-epp ║ NO MAX — verify-only, tunes NOTHING; follows a ║
║                         ║                ║ cmdline amd_pstate= change, never leads one    ║
║ BLACKLIST_AMDXDNA       ║ false          ║ boolean; false + amd_iommu=off is a preflight  ║
║                         ║                ║ refusal (rc 3)                                 ║
```

`_RY_EPP_LEVELS`: `default performance balance_performance balance_power power`.
`_RY_DPM_LEVELS` (9): `auto low high manual profile_standard profile_min_sclk
profile_min_mclk profile_peak perf_determinism`. Domain validation lives in
`_ir_validate_keys`. Re-probed at 7.163.0, each in its own subprocess: shipped values rc 0;
a bogus DPM level, a bogus EPP, a governor failing the regex, and `BLACKLIST_AMDXDNA=false`
under `amd_iommu=off` all exit **3**; the dual-stack case (`ipv6.disable=1` removed) returns
**0** with a WARN. **`_err_loud` exits rather than returns.**

**A full perf-value change touches 11 sites. Enumerate all of them in any TUNE.**

```
║ #   ║ FILE       ║ CARRIES                                                              ║
║─────║────────────║──────────────────────────────────────────────────────────────────────║
║ 1   ║ script     ║ set -g CPUPOWER_GOVERNOR performance                                 ║
║ 2   ║ script     ║ set -g GPU_DPM_LEVEL high + its trailing comment                     ║
║ 3   ║ script     ║ set -g EPP_PREFERENCE performance (line also holds _RY_EPP_LEVELS)   ║
║ 4   ║ script     ║ udev generator comment "# AMD P-State EPP performance (maximum       ║
║     ║            ║ CPPC hint)"                                                          ║
║ 5   ║ script     ║ udev generator comment "# GPU performance level (gfx1151             ║
║     ║            ║ clock-floor; forced high)"                                           ║
║ 6   ║ README     ║ managed-files row: governor (`performance`)                          ║
║ 7   ║ README     ║ managed-files row: NVMe none, P-State EPP, DPM `high`                ║
║ 8   ║ README     ║ Service Keys row: CPUPOWER_GOVERNOR | performance                    ║
║ 9   ║ README     ║ Service Keys row: GPU_DPM_LEVEL | high                               ║
║ 10  ║ README     ║ Service Keys row: EPP_PREFERENCE | performance                       ║
║ 11  ║ CHANGELOG  ║ new entry inserted after the preamble                                ║
```

**Two NON-sites**, recorded so they are neither re-added nor silently lost: the udev
generator's `--description` carries no perf value and is a LOCATE anchor only; the README's
old CPU/GPU prose paragraph was restructured into Gaming Stack and Kernel Parameter Notes
bullets at 7.151.0 and **must not be re-added** — Tuning Notes does not restate values.
**Two sites follow automatically:** the udev generator interpolates `$EPP_PREFERENCE` and
`$GPU_DPM_LEVEL`, and `_vss_udev` greps for the interpolated values, so both track any
change. Verify by re-rendering the udev body, not by reading lines.

**Grep traps:** `powersave` has 4 hits and only the governor global is in scope — the rest is
`NM_WIFI_POWERSAVE` plumbing. `balance_performance` has exactly 1 hit, the surviving
`_RY_EPP_LEVELS` member. Do not edit the phrase "clock-floor" — it is accurate under `high`
and consistent across the generator and verify paths. `grep -ci 'audit\|spec'` reports false
hits from `_RY_ARGPARSE_SPEC` and "inspect"; real audit refs are **0** in all shipped files.

**Measured deltas from the last perf change** (7.128.0): udev 657 -> 639 B (-18), cpupower
113 -> 115 B (+2) — the split was value substitution -6 and comment edits -10. A revision
predicting -6 by omitting the comment edits was wrong. **The 17-file total is now 5,393 B**;
predict against that figure and against an unchanged 639 B udev body. **On a re-apply use
verbatim strings — never paraphrase edit wording from memory. The acceptance test is
reproducing the predicted SHAs, not a passing functional battery.**

### 8c. Configured values

**KERNEL_PARAMS (15, sorted as emitted):** `amd_iommu=on` `amd_pstate=active`
`btusb.enable_autosuspend=n` `fsck.mode=auto` `fsck.repair=yes` `iommu=pt` `ipv6.disable=1`
`mt7925e.disable_aspm=1` `nvme_core.default_ps_max_latency_us=0`
`pcie_aspm.policy=performance` `processor.max_cstate=1` `quiet` `split_lock_detect=off`
`usbcore.autosuspend=-1` `zswap.enabled=0`. Emitted cmdline = these 15 plus `rw` and
`root=UUID=`. **`amd_iommu=on` is the inert token T3-5 proposes removing.**

```
║ TOKEN                               ║ IN kernel-parameters.txt  ║ CITE INSTEAD         ║
║─────────────────────────────────────║───────────────────────────║──────────────────────║
║ amd_pstate=                         ║ YES                       ║ —                    ║
║ amd_iommu=                          ║ YES; `on` invalid (T3-5)  ║ amd/init.c parser    ║
║ iommu= (the pt half)                ║ YES; pt = passthrough     ║ —                    ║
║ processor.max_cstate=               ║ YES                       ║ —                    ║
║ split_lock_detect=                  ║ YES                       ║ —                    ║
║ usbcore.autosuspend=                ║ YES                       ║ —                    ║
║ pcie_aspm= (policy= is a modparam)  ║ pcie_aspm= only           ║ pcie/Kconfig         ║
║ mitigations= (G-4 candidate)        ║ YES                       ║ —                    ║
║ transparent_hugepage= (G-11)        ║ YES                       ║ —                    ║
║ fsck.mode= / fsck.repair=           ║ NO — systemd-side         ║ systemd-fsck(8)      ║
║ ipv6.disable=                       ║ NO                        ║ networking docs      ║
║ zswap.enabled=                      ║ NO                        ║ admin-guide/mm       ║
║ nvme_core.default_ps_max_latency_us ║ NO — module param         ║ drivers/nvme         ║
║ btusb.enable_autosuspend=           ║ NO — module param         ║ drivers/bluetooth    ║
║ mt7925e.disable_aspm=               ║ NO — module param         ║ mt76/mt7925/pci.c    ║
║ ttm.pages_limit (G-2)               ║ NO — module param         ║ ttm/ttm_tt.c         ║
║ amdgpu.gttsize (G-2)                ║ NO — module param         ║ amdgpu/amdgpu_drv.c  ║
║ amdgpu.ppfeaturemask (G-17)         ║ NO — module param         ║ amdgpu/amdgpu_drv.c  ║
```

**ENV_VARS (10, `~/.config/environment.d/10-environment.conf`, 0600):**
`DXVK_LOG_LEVEL=none` `FSR4_WATERMARK=1` **`GSK_RENDERER=ngl`** `MANGOHUD=1`
`MESA_SHADER_CACHE_MAX_SIZE=16G` `POWERDEVIL_NO_DDCUTIL=1` `PROTON_LOCAL_SHADER_CACHE=1`
`VKD3D_DEBUG=none` `VKD3D_SHADER_DEBUG=none` `WINEDEBUG=-all`. **`GSK_RENDERER=ngl` is new at
7.163.0** — it moves GTK4 off its Vulkan renderer, which aborts on gfx1151, and it is the
system-wide install of a workaround previously applied per-application. `PROTON_ENABLE_
WAYLAND=1` left after 7.155.0. No drirc, no ttm/amdgpu module params, no ICD pin.
`_vre_envvars` and the static user check both iterate the array, so a member added or removed
here needs **no verifier edit** — the contrast that makes T2-6 a finding.

**SYSCTL_VALUES (9, `/etc/sysctl.d/95-ry-overrides.conf`, priority 95):**
`kernel.nmi_watchdog=0` `net.core.default_qdisc=fq` `net.ipv4.tcp_congestion_control=bbr`
`net.ipv4.tcp_notsent_lowat=16384` `net.ipv4.tcp_slow_start_after_idle=0`
`vm.compaction_proactiveness=0` `vm.max_map_count=2147483642` `vm.swappiness=150`
`vm.watermark_boost_factor=0`. Stored `k=v`, emitted `k = v` — normalize whitespace in any
parity check. Both `netdev_budget` keys were removed at 7.157.0; an absent key leaves the
kernel default. **The generator emits ONE header comment and then exactly one `key = value`
line per entry — it does not annotate individual keys and never did.** G-3 proposes the two
missing members of the ArchWiki zram recipe. **Known non-defect:** in a non-initial network
namespace the `net.*` keys fail ENOENT because those tables are init_net only — a container
signature, not a host fault.

**MASK (11):** `ananicy-cpp.service` `power-profiles-daemon.service`
`NetworkManager-wait-online.service` `avahi-daemon.service` `avahi-daemon.socket`
`ufw.service` `sleep.target` `suspend.target` `hibernate.target` `hybrid-sleep.target`
`suspend-then-hibernate.target`. **EXPECTED_SERVICES (5):** `fstrim.timer`
`NetworkManager.service` `cpupower.service` `nftables.service` `bluetooth.service`.
**`_RY_PKG_MANAGED_SERVICES` (1):** `NetworkManager.service` — in both sets; strip it before
isolating the intersection gates.

**PKGS_ADD (16):** nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa,
mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, lm_sensors, rtkit,
realtime-privileges, nftables, pacman-contrib. **PKGS_DEL (9, `-Rns`, rdep-aware via
pactree):** plymouth, cachyos-plymouth-bootanimation, cachyos-plymouth-theme,
breeze-plymouth, plymouth-kcm, micro, cachyos-micro-settings, cachy-update, kdeconnect.
**AUR: none** — 13 resolve in Arch extra/multilib/core and 3 in `[cachyos]`. Vulkan via chwd,
verify-only: vulkan-radeon + lib32-vulkan-radeon. `cachyos-gaming-meta` is what puts
gamescope and the rest of the gaming stack on the box (G-15), so nothing in §3c needs
installing.

**LOGIND_IGNORE_KEYS (8):** `HandlePowerKey` `HandlePowerKeyLongPress` `HandleSuspendKey`
`HandleSuspendKeyLongPress` `HandleHibernateKey` `HandleHibernateKeyLongPress`
`HandleRebootKey` `HandleRebootKeyLongPress`, emitted as `<key>=ignore`. The 4 uncovered
`Handle*` keys are the 3 lid-switch ones plus `HandleSecureAttentionKey` — correctly out of
scope for a mini-PC.

**MKINITCPIO:** `MODULES=(amdgpu)` (early KMS); HOOKS (11) base systemd autodetect microcode
modconf kms keyboard sd-vconsole block filesystems fsck — **byte-identical to Arch mkinitcpio
41.1's shipped default**; COMPRESSION zstd with `COMPRESSION_OPTIONS=(-3)`; explicit
`BINARIES=()` / `FILES=()`. `_vmh_order_checks` enforces **eleven** HOOKS constraints; a
systemd-after-sd-vconsole swap returns **2**, not 1, because that permutation breaks two
ordered pairs.

**Sixteen service keys** are documented in the README, matching the script's SERVICE KEYS
block row for row. Only two reach their destination under their own name (`COUNTRY`;
`LOGIND_IGNORE_KEYS` -> 8 `Handle*Key=ignore` lines); the rest are **renamed on the way out**
(`BT_AUTO_ENABLE`->`AutoEnable`, `EPP_PREFERENCE`->udev `ATTR{cpufreq/energy_performance_
preference}`, `CPUPOWER_GOVERNOR`->`GOVERNOR=`, `RESOLVED_MDNS`->`MulticastDNS=`), **so
grepping a generated file for a script variable name returns nothing.**
`EXPECTED_SCALING_DRIVER` writes nothing at all, and `BLACKLIST_AMDXDNA=false` emits nothing.
**The block declares 18 globals; `_RY_DPM_LEVELS` and `_RY_EPP_LEVELS` are internal validation
arrays with no README representation — strip them before any parity comparison.** **The
sdboot generator emits `REMOVE_EXISTING=` before `OVERWRITE_EXISTING=` while the README table
follows declaration order; never "fix" the table to emission order.**

### 8d. Generated-file byte anchors

```
║ FILE                                             ║ 7.162.2 ║ 7.163.0 ║ DELTA ║
║──────────────────────────────────────────────────║─────────║─────────║───────║
║ /boot/loader/loader.conf                         ║ 89      ║ 89      ║       ║
║ /etc/kernel/cmdline                              ║ 343     ║ 343     ║       ║
║ /etc/sdboot-manage.conf                          ║ 535     ║ 535     ║       ║
║ /etc/mkinitcpio.conf                             ║ 276     ║ 276     ║       ║
║ /etc/systemd/resolved.conf.d/99-cachyos-resolved ║ 90      ║ 90      ║       ║
║ /etc/systemd/logind.conf.d/99-cachyos-logind     ║ 292     ║ 292     ║       ║
║ NetworkManager-dispatcher.service.d/logging.conf ║ 127     ║ 127     ║       ║
║ /etc/NetworkManager/conf.d/99-cachyos-nm.conf    ║ 148     ║ 186     ║ +38   ║
║ /etc/iw-regdomain                                ║ 88      ║ 88      ║       ║
║ /etc/bluetooth/main.conf                         ║ 147     ║ 147     ║       ║
║ /etc/nftables.conf                               ║ 1,059   ║ 1,059   ║       ║
║ /etc/default/cpupower-service.conf               ║ 115     ║ 115     ║       ║
║ /etc/sysctl.d/95-ry-overrides.conf               ║ 376     ║ 376     ║       ║
║ /etc/udev/rules.d/99-ry-perf.rules               ║ 639     ║ 639     ║       ║
║ /etc/modprobe.d/60-ry-modules.conf               ║ 177     ║ 177     ║       ║
║ ~/.config/environment.d/10-environment.conf      ║ 282     ║ 299     ║ +17   ║
║ ~/.config/MangoHud/MangoHud.conf                 ║ 555     ║ 555     ║       ║
║──────────────────────────────────────────────────║─────────║─────────║───────║
║ TOTAL (17 managed files)                         ║ 5,338   ║ 5,393   ║ +55   ║
```

The udev rule at **639 B** and the **5,393 B** total are the anchors for any perf-value
change. Neither delta above is perf. **A byte-anchor table cannot represent every change** —
`/etc/mkinitcpio.conf` once changed content at a zero byte delta, which is why §8e exists.
**Measure as written files** (`$fn > tmp; stat -c %s`), never `string collect`.

### 8e. Generated bodies — all seventeen, byte-exact

Rendered from the 7.163.0 generators under the §10 harness, **3/3 deterministic**; sorted
per-file manifest sha `d9b5b3f3e4bea768` (method in §10 — comparable only across rebases
using the same method). All seventeen were re-rendered and diffed against the 7.162.2 fences;
**two came back different and are marked CHANGED.** Reproduce before diffing; do not diff
against a prior edition's fences without re-rendering — that diff is what caught both.

**`/boot/loader/loader.conf`** (89 B):

```
# systemd-boot loader configuration
default @saved
timeout 0
console-mode keep
editor no
```

**`/etc/kernel/cmdline`** (343 B) — **the UUID below is a 36-char stub; a real render is the
same length. Compare everything after the `root=UUID=` token**:

```
rw root=UUID=12345678-1234-1234-1234-123456789abc amd_iommu=on amd_pstate=active btusb.enable_autosuspend=n fsck.mode=auto fsck.repair=yes iommu=pt ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0
```

**`/etc/sdboot-manage.conf`** (535 B) — the same 15 tokens again, inside `LINUX_OPTIONS`:

```
# sdboot-manage configuration — changes require: sudo sdboot-manage gen && sudo sdboot-manage update
LINUX_OPTIONS="amd_iommu=on amd_pstate=active btusb.enable_autosuspend=n fsck.mode=auto fsck.repair=yes iommu=pt ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0"
LINUX_FALLBACK_OPTIONS="quiet"
DEFAULT_ENTRY="manual"
REMOVE_EXISTING="yes"
OVERWRITE_EXISTING="yes"
REMOVE_OBSOLETE="yes"
```

**`/etc/mkinitcpio.conf`** (276 B):

```
# mkinitcpio configuration — changes require: sudo mkinitcpio -P && sudo sdboot-manage update
MODULES=(amdgpu)
BINARIES=()
FILES=()
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
COMPRESSION="zstd"
COMPRESSION_OPTIONS=(-3)
```

**`/etc/systemd/resolved.conf.d/99-cachyos-resolved.conf`** (90 B) — four lines; no `DNS=`,
no `DNSOverTLS=`, no `DNSSEC=` by design (T5):

```
# systemd-resolved: link DNS from DHCP, mDNS/LLMNR off
[Resolve]
MulticastDNS=no
LLMNR=no
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

**`/etc/NetworkManager/conf.d/99-cachyos-nm.conf`** (186 B) — **CHANGED, was 148 B. New
`[main]` section carrying `autoconnect-retries-default=0`; unasserted by `_vss_nm` (T2-6)**:

```
# NetworkManager configuration — wpa_supplicant backend
[main]
autoconnect-retries-default=0

[device]
wifi.backend=wpa_supplicant

[connection]
wifi.powersave=2

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

**`/etc/nftables.conf`** (1,059 B) — the ICMPv6 base accept is the RFC 4890 host minimum,
inert under `ipv6.disable=1` and live on the fallback entry. **Do not remove it as dead
code**:

```
#!/usr/bin/nft -f
# ry-install: default-deny-inbound (ufw masked). ICMPv6 is live on the fallback entry. Add inbound ports below.
flush ruleset
table inet filter {
    chain input {
        type filter hook input priority filter; policy drop;
        ct state invalid drop # early drop of invalid connections
        ct state established,related accept
        iif "lo" accept
        # IPv4 ICMP: inbound ping (echo-request) + error/PMTUD types (replies match ct established)
        icmp type { echo-request, destination-unreachable, time-exceeded, parameter-problem } accept
        # ICMPv6: NDP, MLD, and error/PMTUD types (RFC 4890 host minimum); live on the fallback entry
        icmpv6 type { echo-request, destination-unreachable, packet-too-big, time-exceeded, parameter-problem, nd-router-solicit, nd-router-advert, nd-neighbor-solicit, nd-neighbor-advert, mld-listener-query } accept
    }
    chain forward { type filter hook forward priority filter; policy drop; }
    chain output { type filter hook output priority filter; policy accept; }
}
```

**`/etc/default/cpupower-service.conf`** (115 B):

```
# cpupower-service.conf — sourced by /usr/lib/systemd/scripts/cpupower (cpupower.service)
GOVERNOR='performance'
```

**`/etc/sysctl.d/95-ry-overrides.conf`** (376 B) — one header comment, then one `key = value`
line per entry. **No per-key annotation exists and none ever did**:

```
# ry-install sysctl tunables (priority 95 — loaded after CachyOS vendor 70-cachyos-settings.conf)
kernel.nmi_watchdog = 0
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
net.ipv4.tcp_notsent_lowat = 16384
net.ipv4.tcp_slow_start_after_idle = 0
vm.compaction_proactiveness = 0
vm.max_map_count = 2147483642
vm.swappiness = 150
vm.watermark_boost_factor = 0
```

**`/etc/udev/rules.d/99-ry-perf.rules`** (639 B) — the perf anchor:

```
# ry-install: udev performance rules (managed file, do not edit by hand)
# NVMe scheduler none (lowest tail latency; diverges from CachyOS kyber default)
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
# AMD P-State EPP performance (maximum CPPC hint)
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="performance"
# GPU performance level (gfx1151 clock-floor; forced high)
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="high"
```

**`/etc/modprobe.d/60-ry-modules.conf`** (177 B) — comment-only under
`BLACKLIST_AMDXDNA=false`; `_vrkm_blacklist_modprobe` is generator-sourced, finds zero
entries and returns early, so **an amdxdna load is expected and unasserted**:

```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# no directives: BLACKLIST_AMDXDNA=false (NPU path); MT7925 ASPM handled on the kernel command line
```

**`~/.config/environment.d/10-environment.conf`** (299 B) — **CHANGED, was 282 B.
`GSK_RENDERER=ngl` added; ENV_VARS 9 -> 10**:

```
# Environment for systemd --user services and graphical sessions (Plasma, Flatpak, D-Bus apps)
DXVK_LOG_LEVEL=none
FSR4_WATERMARK=1
GSK_RENDERER=ngl
MANGOHUD=1
MESA_SHADER_CACHE_MAX_SIZE=16G
POWERDEVIL_NO_DDCUTIL=1
PROTON_LOCAL_SHADER_CACHE=1
VKD3D_DEBUG=none
VKD3D_SHADER_DEBUG=none
WINEDEBUG=-all
```

**`~/.config/MangoHud/MangoHud.conf`** (555 B, 22 L = header + 18 active + 3 comments) —
readout-only by design; **it carries no logging directive, which is finding G-9**:

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
# cpu_stats intentionally disabled — enable for CPU load in the HUD
# cpu_temp intentionally disabled — enable if you want CPU temperature in the HUD
# cpu_custom_temp_sensor is inert here — MangoHud reads apu_cpu_temp from gpu_metrics before any hwmon lookup
cpu_mhz
cpu_power
vram
ram
font_size=20
text_outline
background_alpha=0.4
```

### 8f. Removal and addition reconciliation — state the tier for any change you recommend

```
║ TIER ║ CLASS                          ║ ON CHANGE                ║ DETECTION                 ║
║──────║────────────────────────────────║──────────────────────────║───────────────────────────║
║ 1    ║ value in a generated file      ║ self-heals on the next   ║ n/a — cannot orphan       ║
║      ║ (KERNEL_PARAMS, ENV_VARS,      ║ deploy; the generator    ║                           ║
║      ║ SYSCTL_VALUES, ...)            ║ rewrites wholesale       ║                           ║
║ 2    ║ masked unit dropped from MASK  ║ stays masked forever     ║ INFO + MASK_ORPHAN        ║
║ 2    ║ stale 60-ry-*.conf drop-in     ║ stays on disk            ║ WARN + MODPROBE_STALE_    ║
║      ║                                ║                          ║ DROPIN                    ║
║ 2    ║ external sdboot-manage drop-in ║ n/a — never ours         ║ WARN + SDBOOT_DROPIN_     ║
║      ║                                ║                          ║ PRESENT                   ║
║ 3    ║ package dropped from PKGS_ADD  ║ stays installed forever  ║ NOT DETECTED — by         ║
║      ║                                ║                          ║ design (T5)               ║
║ 3    ║ env var dropped from ENV_VARS  ║ stays in a LIVE session  ║ NOT DETECTED — the file   ║
║      ║                                ║ until next login         ║ self-heals, the session   ║
║      ║                                ║                          ║ does not                  ║
║ 3    ║ env var ADDED to ENV_VARS      ║ absent from a LIVE       ║ WARN "NOT SET in          ║
║      ║                                ║ session until next login ║ current session" — not    ║
║      ║                                ║                          ║ drift                     ║
║ —    ║ value hardcoded in a generator ║ self-heals like Tier 1   ║ UNASSERTED unless a       ║
║      ║ body with no array behind it   ║                          ║ _chk_grep is written by   ║
║      ║                                ║                          ║ hand — T2-6               ║
```

Tier 2 is reportable but never self-healing and never sets DRIFT — the operator must act by
hand. Tier 3 is invisible in both directions, and **7.163.0 supplied the addition-side twin**:
`systemd --user` does not import a new variable retroactively, so `GSK_RENDERER` reads NOT SET
in any session predating the deploy. Nothing detects that and nothing should. The last row is
new this edition and is the shape of T2-6.

### 8g. Gates, exits, backups

- **Preflight validator chain, four deep, called from `_init_runtime`:** `_ir_validate_counts`
  (21 tripwires) -> `_ir_validate_keys` (domains and charsets; the nftables<->`ipv6.disable`
  coupling is a WARN, `BLACKLIST_AMDXDNA=false`<->`amd_iommu=off` is a refusal) ->
  `_ir_validate_sets` -> `_ir_validate_post_hooks`. All four re-probed at 7.163.0 on the
  extracted archive, unshadowed, stderr visible: **rc 0, silent.** **There is no
  `_ry_validate_keys`, `_ry_validate_counts` or `_ry_validate_post_hooks`** — those names
  return rc 127, which looks like a signal and is only an unknown command. Real siblings:
  `_ry_validate_mkinitcpio_hooks`, `_ry_validate_mkinitcpio_modules`, `_rvc_dispatch`,
  `_ry_validate_configs`.
- **`_ir_validate_sets` refuses three intersections**, each `_err_loud` + exit 3:
  `PKGS_ADD ∩ PKGS_DEL`, `EXPECTED_SERVICES ∩ MASK`, `_RY_PKG_MANAGED_SERVICES ∩ MASK`. All
  three shipped intersections are empty; the third is normally shadowed by the second — a
  forward guard, not a defect. A fourth (`EXPECTED_VULKAN_PKGS ∩ PKGS_DEL`) is deliberately
  uncovered. **Any recommendation that adds a package or unit must be checked against all
  three — a contradiction is a hard deploy refusal, not a silent misbehavior.**
- **Phases (6):** Preflight -> Packages -> Configuration -> Services (fstab -> resolved ->
  pkg-remove -> mask -> enable -> regdom) -> Boot -> Finalize, mirrored 1:1 across the array,
  `_progress`, the orchestrator, the log headers and the README. `_PROG_STEPS` derives from
  the array, so phase order cannot drift.
- **Hardware gate:** CPU model match on `EXPECTED_CPU_MATCH` = `Ryzen AI Max`; sole override
  `RY_INSTALL_SKIP_HARDWARE_CHECK=1` (`SKIP_HW=1` does nothing); fail-closed on an unreadable
  model; `--verify` warns, deploy and `--check` exit 3.
- **Runtime env inputs are exactly three:** `RY_RUN_TIMEOUT`, `RY_INSTALL_SKIP_HARDWARE_CHECK`,
  `NO_COLOR`. **Every profile toggle is an embedded scalar set unconditionally with `set -g`,
  so an exported env var of the same name is clobbered — opting in means editing the script.**
  `NO_COLOR` needs a non-empty value; `RY_RUN_TIMEOUT=0` disables the timeout, and package and
  boot operations floor at 7200 s.
- **Dependencies:** 37 hard-required commands (missing any -> rc 1) + **16** warn-only
  optional tools, plus a `df --output` probe and systemd >= 250. Profile-installed tools
  (`nft`, `pactree`, `paccache`, `sysctl`, `udevadm`, `bluetoothctl`, `timedatectl`, `ufw`,
  `iw`) are call-site `command -q` guarded and deliberately undeclared — **do not "fix" the
  asymmetry.**
- **Exit model, 14 constants:** 0 OK · 1 FAIL · 2 USAGE · 3 PREFLIGHT · 4 BOOT_CRIT · 5 LOCK ·
  10 DRIFT; sentinels 11 GEN_NOFN, 12 GEN_NOUUID, 13 GEN_SYSCTL, 14 GEN_ENVD, 250 AS_MISUSE,
  251 RUN_TMPFAIL, 255 RUN_MISUSE — **function returns only, never process exits.** Signals
  map to 128+N. **No orphan class reaches EXIT_DRIFT.** Root `--check` -> rc 3 silent, no
  JSONL; other root modes -> rc 2 loud. Lock dir `$HOME/ry-install/.lock`; `--verify` and
  `--check` are lock-free.
- **The expected live `--verify` OK count is 268 at 7.163.0** — **189 static + 79 runtime**,
  from the clean post-deploy host run of 2026-08-16 (0 fail / 0 warn / 0 gen_fail, exit 0).
  The +2 over 7.162.2's 266 is exactly the new `ENV_VARS` member, counted once statically and
  once at runtime. **Empirical, never a script literal** — the 7.155.0 edition's 278 died when
  the verify surface shrank. Re-baseline after any verify-surface change; T2-6 would make it
  269.
- **JSONL logs land under `$HOME/ry-install/logs/<YYYY-MM-DD>/<mode>-<timestamp>-<pid>.jsonl`**,
  not in the profile root. A glob that misses the `logs/<date>/` level returns zero hits
  silently, which reads as "the key was never emitted".
- **Argument parsing (`_RY_ARGPARSE_SPEC`, 6 entries):**
  `--exclusive=verify,check,install-file` `h/help` `v/version` `verify` `check`
  `install-file=`. No positionals; `--` ends parsing and anything after exits 2. Glued short
  flags early-intercept, first char winning (`-hv` -> help); `-h`/`-v` are honored before the
  root guard. **`--install` is not a flag** — install is the default no-arg mode, and fish's
  wgetopt abbreviation resolves `--install` to `--install-file` while rejecting `--ver` as
  ambiguous. Documented fish behavior; `-S` would break the 3.6 floor.
- **Backups are two different guarantees.** The 4 boot-critical destinations get `.ry.bak`
  **plus post-write re-read and restore**; the other 13 get a one-time `.ry.orig` on **first
  adoption only**, after which every overwrite is silent by design. `/etc/fstab` is not a
  managed destination and has always had its own `.ry.bak` during rewrite. **Do not describe
  the 13 as backed up on every write, and do not recommend `.ry.orig` for the boot four — it
  is weaker than what they have.** No verify path asserts `.ry.orig` exists and none can:
  absence is indistinguishable from "no pre-existing content", the common case here.

---

## 9. Verify-surface ownership

**62 verify functions = 12 `_verify_*` + 50 subs** (`_vsb` 7 · `_vss` 10 · `_vsp` 3 · `_vsc` 1
· `_vrk` 4 · `_vrkm` 3 · `_vrsv` 10 · `_vre` 5 · `_vrs` 4 · `_vpd` 1 · `_vmh` 2). The census
is **identical prefix-by-prefix across five rebases** — no verify function has been added,
removed or renamed — even as asserts keep landing *inside* existing subs. A recommendation
that changes a value must state which **sub** asserts it and whether that sub hard-fails or
warns.

```
║ ORCHESTRATOR             ║ SUBS THAT MATTER FOR TUNING FINDINGS                           ║
║──────────────────────────║────────────────────────────────────────────────────────────────║
║ _verify_static_boot      ║ _vsb_loader · _vsb_sdboot · _vsb_sdboot_dropins (WARN) ·       ║
║                          ║ _vsb_cmdline (all 15 tokens + root= + rw) · _vsb_mkinitcpio    ║
║                          ║ (live COMPRESSION/_OPTIONS, multi-line join, last-wins) ·      ║
║                          ║ _vsb_entries -> _vsb_entry_options                             ║
║ _verify_static_system    ║ _vss_logind · _vss_nmdispatch · _vss_nm (backend, powersave,   ║
║                          ║ log level ONLY — see T2-6) · _vss_sysctl (9 keys) ·            ║
║                          ║ _vss_regdom · _vss_bluetooth · _vss_udev (3 rules, EPP + DPM   ║
║                          ║ aware) · _vss_modprobe (blacklist + stale-dropin sweep, WARN)  ║
║                          ║ · _vss_nft (HARD-FAILS on a missing echo-request or            ║
║                          ║ icmpv6-type accept)                                            ║
║ _verify_static_user      ║ inline: ENV_VARS (10, iterated) + MangoHud directives          ║
║ _verify_static_packages  ║ _vsp_required (PKGS_ADD 16 + Vulkan 2) · _vsp_removed ·        ║
║                          ║ _vsp_pacman_conf                                               ║
║ _verify_static_services  ║ MASK 11 inline + _vss_orphan_masks (INFO, admin-scope /etc     ║
║                          ║ and /run only)                                                 ║
║ _verify_static_syntax    ║ live mkinitcpio HOOKS presence, multi-line tolerated           ║
║ _verify_static_checksum  ║ _vsc_check_one (expected vs installed SHA256 per destination)  ║
║ _verify_runtime_kparams  ║ _vrk_cmdline (every token + rw) · _vrk_gpu_state (QUOTED       ║
║                          ║ compare vs $GPU_DPM_LEVEL) · _vrk_cpu_state (cpu0 detail +     ║
║                          ║ full cpufreq-policy uniformity sweep) · _vrk_module_state ->   ║
║                          ║ _vrkm_amdgpu, _vrkm_blacklist, _vrkm_blacklist_modprobe        ║
║                          ║ (content-sourced — a comment-only body yields zero entries     ║
║                          ║ and an early return)                                           ║
║ _verify_runtime_services ║ _vrsv_sys_units -> _vrsv_chk_active_enabled, _vrsv_chk_        ║
║                          ║ nftables (judged by live policy-drop, not unit state),         ║
║                          ║ _vrsv_chk_resolved, _vrsv_chk_cpupower_governor ·              ║
║                          ║ _vrsv_nft_assert_ping (WARN) · _vrsv_wifi ->                   ║
║                          ║ _vrsv_wifi_nm_backend · _vrsv_masked_inactive (iterates        ║
║                          ║ $MASK) · _vrsv_user_units                                      ║
║ _verify_runtime_env      ║ _vre_envvars (dynamic over $ENV_VARS — followed the 7.163.0    ║
║                          ║ addition with no edit) · _vre_sysctl_runtime (9 keys,          ║
║                          ║ absent-vs-unreadable split, WARNs on an absent knob) ·         ║
║                          ║ _vre_fstab (every ext4 entry; logs the root-FS type) ·         ║
║                          ║ _vre_ntsync · _vre_regdom                                      ║
║ _verify_runtime_session  ║ _vrs_nm_perms · _vrs_installed_file_perms -> _vrs_vfat_skip ·  ║
║                          ║ _vrs_parent_dirs -> _vpd_dir_perm_check                        ║
║ _verify_summary          ║ pass/fail/warn tally — not an assertion path                   ║
```

**The assertion surface moves with zero census movement.** The sysctl and env verifiers
iterate the profile arrays, so asserts appear and disappear as arrays change — which is why
the empirical OK count moves (266 -> 268 this release) and why no function-list diff would
ever show it.

**Attribution traps:**

- **`_vss_nm` asserts three values and the generator emits four.** `autoconnect-retries-
  default=0` has no `_chk_grep` — T2-6, and the general shape is the last row of §8f.
- **Nothing in the verify surface asserts efficacy, only presence.** All three KERNEL_PARAMS
  surfaces are string matches against the emitted files and `/proc/cmdline`, so a token the
  kernel *rejects* passes every check — exactly how `amd_iommu=on` verified clean while dmesg
  logged `Unknown option` (T3-5). Efficacy is a dmesg or sysfs read, and it belongs in §7.
- `_vrkm_blacklist_modprobe` is **generator-sourced** — it checks intended content, not
  on-disk extras, and with the comment-only body it returns early. Attribute the drop-in
  sweep to `_vss_modprobe` / `_ry_stale_ry_dropins`, never to it.
- `_vrsv_masked_inactive` asserts every *declared* mask is present and inactive; the reverse
  direction is `_vss_orphan_masks`, on the **static** path. Do not conflate them, and note
  `_ry_orphan_masked_units` filters to **admin scope only** (`/etc`, `/run`) — vendor masks
  and `Alias=` cascades are deliberately dropped.
- `_vsb_entry_options` is a sub of **`_vsb_entries`**, not of `_verify_static_boot`. It skips
  `*-fallback.conf` by design — which is why T4-1 is invisible to verify.
- `_vss_nft` hard-fails on a missing inbound echo-request accept; `_vrsv_nft_assert_ping` only
  warns. A recommendation to drop inbound ping must address the hard-fail.
- **Removed asserts — do NOT verify:** `_vrkm_iommu`, `_vrk_clocksource`, `_vre_zram`,
  `_vre_tcp` (gone since 7.90.0); kernel-floor and Mesa-floor checks; the preemption advisory.
  No THP, KSM, `ttm.*`, drirc or baloo assert exists — which is what makes G-2 and G-11
  unverifiable today as well as unshipped.
- **Sandbox artifact, not a regression:** `_ry_validate_mkinitcpio_hooks` returns rc 1 and
  `_ry_validate_configs` rc 3 in a container because `/etc/mkinitcpio.conf` is absent; call
  `_vmh_order_checks` directly to test HOOKS ordering. Always A/B a nonzero validator rc
  against the previous release.

**Output-channel invariant — reconciled; do not re-file as drift.** Every leveled user-facing
message funnels through `_msg_print`, honoring QUIET / `_RY_OUTPUT_BROKEN` / `_RY_NO_COLOR` /
isatty(2). Raw `>&2` counts of ~78 whole-file and ~43 inside function bodies are **both
correct under their own scoping**; the remainder are top-level pre-init preflight writes made
before `_msg_print` is defined. The invariant means "single authority for leveled
user-facing output", not "sole writer to fd 2" — the latter reading generates a false finding
every time. stdout carries only `--help` and `--version`.

---

## 10. Reproduction method and its traps

Recorded so the next rebase does not re-pay these costs.

- **Harness.** Cut the script just before the `# ── MAIN: ARGPARSE` banner — **L4790 at both
  7.162.2 and 7.163.0**, L4786 at 7.155.0/7.158.0, L4861 at 7.141.0. **Always locate the
  banner.** Then delete the L3 source guard and `source` the result as a non-root user with a
  **writable `$HOME`**: `sed -n '1,4789p' ry-install.fish | sed '3d' > harness.fish`. Without
  the L3 deletion the guard fires and every count silently reads 0; without a writable `$HOME`
  the log-directory init aborts the source part-way and counts read 0 the same way —
  different cause, identical symptom. Pre-create `$HOME/.config/fish` and
  `$HOME/.local/share/fish`, and make the worktree **and every parent directory** `a+rX`.
- **Shadow `exit` before sourcing** (`function exit; end`, then `functions -e exit`) so the
  fallen-through top level completes; oracle counts, scalars and all 17 generators then probe
  in ONE shell. TRAP: that flow calls `_ry_erase_handlers`, so the live function table
  UNDER-COUNTS handler families — census from `^function ` at column 0 in source (**294**),
  never from a `functions --all --names` diff. Second trap: `grep -oP '^function
  \K[A-Za-z0-9_]+'` reports 293, truncating `_content_*` names at the first dot. Use
  `[^ ;]+`.
- **Source with stderr visible, and run `_ir_validate_counts` unshadowed in every cert.**
  Sourcing with `2>/dev/null` swallows `_err_loud` refusals — exactly how the 7.162.0 tripwire
  escape shipped certified. Independently, WARN/INFO text is invisible in the harness even
  with stderr open: `QUIET` is pinned `true` pre-argparse and flipped only in MAIN, which the
  cut removes. **Assert warn branches by rc and by JSONL, never by watching for text.**
- **`test -w /tmp` bails rc 3 before anything else.** A sandbox reproduction needs a non-root
  user *and* a writable `/tmp` (`chmod 1777 /tmp`), with probe scripts world-readable.
- **Array counts by live fish eval, never text parsing.** `eval echo \$$name` collapses a
  fish array and reports every count as 1 — use `eval "set vals \$$name"` then `count $vals`.
  Continuation regexes truncate multi-line declarations, `set -g --` evades awk, and several
  service keys share one `set -g` line, which breaks any line-anchored `^set -g NAME` scalar
  extraction — use a non-anchored `finditer`, de-duplicate preserving order, strip trailing
  `;`.
- **Filter generators with `string match -qr '^_content_(_|HOME)'` in a loop.** A bare
  `_content_*` glob also catches the `_content_fn_for` dispatcher and returns 18, and
  `string match -r` with a capture group returns the groups as extra elements — filter, do not
  map. Fish function names contain dots, so a `[A-Za-z0-9_-]*` charset truncates them and
  fabricates duplicates.
- **Set `_ROOT_UUID`** (single underscore, not `_RY_ROOT_UUID`) with any 36-char valid-shape
  UUID, or the cmdline generator returns 12 / `GEN_NOUUID` and `_ry_validate_configs` returns
  rc 3, which reads as a regression. With the stub, the render reproduces the host byte count
  exactly.
- **Generated bytes must be measured as WRITTEN FILES** (`$fn > tmp; stat -c %s`), never
  `(cmd | string collect)` — collect strips each trailing newline and the 17-file total reads
  17 B short, a phantom deficit that looks like anchor drift. `string length` counts
  characters and under-reports for the same reason.
- **Determinism by SORTED per-file manifest:** `sha256sum` over the name-sorted per-generator
  `sha256sum` list = **`d9b5b3f3e4bea768`** at 7.163.0, comparable across future rebases using
  the same method. Older whole-directory shas were harness-filename dependent and are
  method-incompatible; a sha change across methods is expected, not drift.
- **Verify every "before" column against the OLD EDITION'S RENDERED BODY, not memory.** The
  old script is not in the archive; only its brief is. A drift row asserting a change that
  never happened passes every count check and every byte check — only an old-vs-new body diff
  catches it. Extract the previous edition's fences with a **toggle-based** fence walker; a
  naive `re.findall` on triple-backticks consumes alternating pairs and under-reports. Two
  reusable gotchas: the walker inherits the *last seen* bold label, so fences after the final
  body are mis-attributed (take the first fence per label, and do not read the extras as an
  18th body); and the cmdline fence always "differs" unless the stub UUID matches §8e —
  compare everything after `root=UUID=`.
- **Fence-aware heading checks are mandatory.** A naive `^# ` scan false-reports H1 violations
  on the `#` comments inside the nft, udev, sysctl and config fences.
- **Byte-vs-character length.** Banner and line-length checks must count CHARACTERS; `awk
  length` under a C locale counts bytes and falsely flags box-drawing content. In box tables
  `—` is ONE character; validate width uniformity per contiguous `^║` block, and when a row
  must grow, rebuild the block with recomputed column widths rather than padding by hand.
- **CRLF and banner false positives.** A byte-level `b'\r' in raw` test reports dozens of hits
  that all sit inside string literals; test `line.endswith(b'\r')` instead. A "contains
  U+2500" banner check flags functions whose box characters are in runtime `_echo` output —
  split the comment and code classes first.
- **Verify the upload before trusting it.** A fresh upload can be *ahead* of the pin — this
  one was, arriving as 7.163.0 against a 7.162.2 pin. Hash the archive against §0, read
  `VERSION` plus a structural count, then confirm with the zip/README/CHANGELOG hash.
  **All 8-hex anchors are `sha256sum | cut -c1-8`, not CRC-32.**
- **Sandbox limits.** Target host paths do not exist, so only sudo-fail and preflight paths
  are exercisable and a full install cannot complete by design. `_err_loud` **exits** rather
  than returns — run each negative test in its own subprocess. fish `count` counts ARGUMENTS,
  not stdin. `find -type d` returns 0 hits for cpufreq because the path is a symlink — use
  `-xtype d` or a glob + `test -d`.
- **Network from the sandbox.** git.kernel.org cgit `/plain/` (with `?h=vX.Y` tag pinning),
  raw.githubusercontent.com, docs.mesa3d.org, docs.kernel.org, wiki.archlinux.org, and
  github.com HTML pages for issue state (`grep -o 'octicon-issue-[a-z]*'`) and for the latest
  release (follow `releases/latest` and read the redirect target). **api.github.com
  rate-limits on the shared egress IP.** gitlab.freedesktop.org returned HTTP 504 twice this
  rebase — one retry, then record source_unreachable and carry the previous value forward.

---

## 11. Scope and output contract

**In scope:** recommendations only — do not emit a modified script. Hardware-anchored to
gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE / 85 W BIOS ceiling.
**Both 10 GbE links are down and wlan0 carries the default route** (T0-8) — a networking
recommendation that assumes the wired links are in use is describing the hardware, not the
deployment. That gap is itself candidate G-8.

**Out of scope:** dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS,
laptops, UKI, BIOS flashing. Per-game Proton tuning is secondary to system-wide config and
lives in §3c; BIOS *settings* (the 85 W ceiling, the UMA carve) belong to the BIOS reference
companion, and §3 names them only because they frame every other candidate.

**Rules:**

1. Respect deliberate trade-offs — flag and quantify, do not auto-FIX. Reserve FIX for
   incorrect, superseded, deprecated, or harmful values.
2. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is
   non-trivial.
3. Never invent params, flags, keys, options or URLs. Cite a source or mark UNCERTAIN. Four
   UNCERTAINs are on the record and must not be resolved by guessing: the three unrecoverable
   release bounds (§1b) and G-17's safe value set.
4. Flag every source conflict and name the trusted side. The live conflict is nftables rule
   order (Gentoo vs nftables.org/ArchWiki). The netdev-budget conflict (Red Hat vs ESnet) is
   preserved in §4 even though the keys are gone, because it is why the measurement gate
   existed.
5. Give exact versions (kernel / Mesa / linux-firmware / proton-cachyos / package) and exact
   before -> after, mapped to the in-script global.
6. **Do not carry values forward from any pre-7.163.0 edition.** `ENV_VARS` is 10, Σ is
   5,393 B, the NM body is 186 B and the environment body is 299 B. A recommendation
   asserting that a *perf* value changed in the 7.130.0 -> 7.163.0 window is a stale-source
   error; the converse binds equally for everything §1b lists as changed.
7. **A gate that has returned is not an action.** All eight T0 items are answered.
8. A question closed by a code change is closed. A question closed by upstream evidence (§4)
   is closed. A question closed by a recorded maintainer decision (T3-1, T2-3, DPM `high`,
   sched_ext) is closed — argue against the decision explicitly or leave it alone.
9. A correction in §5 is binding.
10. **§3 must be returned every rebase, not just when something breaks.** Each G-item comes
    back with its gate state — open, measured, or retired — and a measured null retires a row
    permanently. Adding a row requires a documented mechanism and a source; "I have seen this
    recommended" is not one. An empty §3b is a reportable finding about the audit, not a
    clean bill of health for the machine.

**Required output:**

- **Findings matrix** (box-drawn, code-fenced, grouped by tier): ITEM · CURRENT · CALL
  (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE.
- **Gaming uplift matrix** for §3: ID · LEVER · GATE STATE · MEASURED EFFECT (or `not_
  determinable`) · TIER · NEXT STEP. Frametime and 1% lows, never average FPS.
- **Before -> after** for each TUNE/FIX, naming the in-script global and — for any perf value
  — all 11 sites from §8b.
- **Tier placement** for every recommendation, and the reconciliation tier (§8f) for any
  removal *or* addition.
- **Oracle delta** for anything that changes an array length, plus the affected byte anchors
  from §8d. **If a change moves no count and no byte anchor, say so explicitly and cite the
  §8e fence** — the `mkinitcpio.conf` precedent means silence there is a defect.
- **Verify-surface statement:** which sub asserts the changed value, or an explicit "nothing
  does" (T2-6's shape), and whether the assertion tests presence or efficacy.
- **Security delta** (§6, ordered).
- **Verdict** per tier plus overall (PASS / PASS-WITH-FIXES / FAIL).
- **Methodology:** source list with access dates and versions; unknowns marked UNCERTAIN.

---

## Sources

**Primary, accessed 2026-08-16:** git.kernel.org torvalds/linux `Makefile` (7.2.0-rc7),
`Documentation/admin-guide/kernel-parameters.txt` (`mitigations=`, `transparent_hugepage=`),
`drivers/gpu/drm/ttm/ttm_tt.c` (`pages_limit`), `drivers/gpu/drm/amd/amdgpu/amdgpu_drv.c`
(`gttsize`, `ppfeaturemask`) · docs.mesa3d.org `envvars` (`RADV_PERFTEST`, `RADV_DEBUG`,
`MESA_VK_WSI_PRESENT_MODE`, `MESA_SHADER_CACHE_MAX_SIZE`, `MESA_DISK_CACHE_SINGLE_FILE`) ·
docs.kernel.org `gpu/amdgpu/thermal` (`power_dpm_force_performance_level`,
`pp_power_profile_mode`) · wiki.archlinux.org Zram (the four-key swap recipe) ·
raw.githubusercontent.com CachyOS/linux-cachyos PKGBUILD (7.1.8-1) · github.com
CachyOS/proton-cachyos `releases/latest` (cachyos-11.0-20260703-slr) · MangoHud #1794 and
systemd #33579 issue pages (both OPEN).

**Primary, accessed 2026-08-15 and carried forward:** `drivers/cpufreq/amd-pstate.c` and
`Kconfig.x86` · `drivers/iommu/amd/init.c` · `arch/x86/kernel/pci-dma.c` ·
`drivers/iommu/iommu.c` · linux-cachyos config (`X86_AMD_PSTATE_DEFAULT_MODE=3`,
`PCIEASPM_DEFAULT=y`, `PCIEASPM_PERFORMANCE` unset, `NTSYNC=m`, `DRM_ACCEL_AMDXDNA=m`,
`IOMMU_DEFAULT_DMA_LAZY=y`, `ZSWAP_DEFAULT_ON=y`) · MangoHud master `src/cpu.cpp` ·
linux-firmware `amdgpu/gc_11_5_1_mes_2.bin` history · mesa `VERSION` (26.3.0-devel —
**HTTP 504 on both attempts this rebase; carried, not re-derived**).

**Primary, accessed 2026-08-05 and unchanged since:** `drivers/pci/pcie/Kconfig` ·
`drivers/net/wireless/mediatek/mt76/mt7925/pci.c` ·
`drivers/net/ethernet/realtek/r8169_main.c` · Proton-EM `em-10` docs (`FSR4.md`,
`EM-ADDITIONS.md`) · docs.redhat.com RHEL 8/9/10 network performance tuning ·
fasterdata.es.net 100 G "Other Tuning" · wiki.gentoo.org Nftables/Examples ·
wiki.nftables.org quick reference · wiki.archlinux.org Nftables.

**Live host measurements**, captured on the deployed GTR9 Pro: T0-1 … T0-8; the 2026-08-14
capture (MES 0x91 / KIQ 0x75, untainted boot, amdxdna loading, VRAM 32 GiB / GTT 47 GiB); the
2026-08-16 post-deploy run (7.163.0, `--verify` 268 OK = 189 static + 79 runtime, 0 fail /
0 warn, exit 0) and its dmesg evidence for T3-5. These are observations of one machine, not
upstream claims — re-measure rather than cite them for a different host.

**Standing reference:** docs.kernel.org (PCIe/ASPM, amd-pstate, sysctl/vm, sysctl/kernel,
networking, block, ext4, UMIP, AMD-Vi, accel/amdxdna, amdgpu
`power_dpm_force_performance_level`, powercap/RAPL) · docs.mesa3d.org · wiki.archlinux.org
(AMDGPU, IOMMU, fsck, Gaming, Zram, SSD, Ext4, Sysctl, NetworkManager, Wireless,
Uncomplicated_Firewall, Mkinitcpio, systemd-boot, Bluetooth, CPU_frequency_scaling, MangoHud,
Gamescope, pacman) · wiki.cachyos.org · discuss.cachyos.org · man.archlinux.org · invent.kde.org
powerdevil · fishshell.com/docs · strixhalo.wiki (Power Modes, C-States) · amd.com ROCm · the
standalone `mangohud-gtr9-pro` and BIOS reference companion archives.

**Do not cite** wireless.docs.kernel.org for `pcie_aspm` semantics (pre-6.9 text);
kernelconfig.io for Kconfig introduction versions; any source attributing the 600/4000 netdev
pairing to ESnet.
