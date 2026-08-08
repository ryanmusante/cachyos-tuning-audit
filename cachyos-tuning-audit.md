# CachyOS Tuning Audit — Beelink GTR9 Pro (gfx1151)

Pinned to `ry-install.fish` **7.158.0**. Deep-research brief: actionable items only,
**ordered by implementation safety** rather than by subsystem.

---

## 0. Provenance

Every number below was re-derived by live evaluation of the attached archive. Nothing was
carried forward from the 7.155.0 edition unaudited: every line anchor, byte count and
generated body was re-rendered, and the five bodies that moved were caught by diffing the
previous edition's embedded fences against fresh generator output. **One of those five
moved with a byte delta of zero** — see §1a.

```
║ ARTIFACT           ║ SHA256   ║ SIZE               ║
║────────────────────║──────────║────────────────────║
║ zip                ║ cc97b5a5 ║ 317,575 B          ║
║ ry-install.fish    ║ 13a467b8 ║ 4,915 L / 292,715 B║
║ README.md          ║ 061e3b79 ║   307 L /  18,711 B║
║ CHANGELOG.md       ║ 91634f1b ║   131 L /   4,392 B║
║ LICENSE            ║ 2e1e7c8a ║    21 L /   1,069 B║
```

Archive layout: 5 entries, `zip -0 -X` Stored, topdir `ry-install-v7_158_0`, script 0755,
docs 0644, 316,887 B uncompressed. All three content files self-report `7.158.0`.

**Disambiguate by zip, README or CHANGELOG hash — never by `--version`, and not by script
hash either.** All four 7.158.0 hashes above are unique to this release, but the general
rule stands and has fired repeatedly: 7.141.0 shipped twice with one script hash and
7.140.0 nine times with two, and 7.151.0 → 7.153.0 are version-string-only edits of one
another. A single script hash can cover several shipped artifacts; a zip, README or
CHANGELOG hash cannot.

Upstream reference points re-fixed at audit time (2026-08-07). **Nothing moved in the two
days since the 7.155.0 edition** — the version set is re-confirmed, not carried forward:

```
║ COMPONENT              ║ VERSION AT AUDIT ║ WAS (26-08-05) ║ SOURCE               ║
║────────────────────────║──────────────────║────────────────║──────────────────────║
║ Linux mainline         ║ 7.2.0-rc6        ║ unchanged      ║ torvalds/linux       ║
║                        ║                  ║                ║ Makefile             ║
║ linux-cachyos          ║ 7.1.6-1          ║ unchanged      ║ CachyOS/linux-cachyos║
║                        ║                  ║                ║ PKGBUILD             ║
║ Mesa main              ║ 26.3.0-devel     ║ unchanged      ║ mesa/mesa VERSION    ║
║ proton-cachyos         ║ 11.0-20260703    ║ unchanged      ║ CachyOS/proton-cachy-║
║                        ║                  ║                ║ os releases          ║
║ linux-firmware MES     ║ 2026-05-07 tag   ║ unchanged      ║ gc_11_5_1_mes_2.bin  ║
║                        ║                  ║                ║ log                  ║
║ MangoHud #1794         ║ OPEN             ║ unchanged      ║ issue page           ║
║ systemd #33579         ║ OPEN             ║ unchanged      ║ issue page           ║
```

Because the upstream set is static, **every §3 row keeps its previous VERIFIED date except
the three re-fetched above.** No §3 claim is invalidated by this rebase; what changed is the
artifact, not the upstream world.

---

## 1. Delta vs the 7.155.0 edition

### 1a. The perf surface still has not moved; five config bodies did, one of them at zero bytes

**The four perf scalars are byte-identical across twenty-eight releases, 7.130.0 →
7.158.0.** Governor `performance`, EPP `performance`, DPM `high` and driver
`amd-pstate-epp` have not been edited once in that window. A recommendation asserting that a
*perf* value moved in it is a stale-source error, not a finding.

**Two count-oracle values DID move, and that is new.** `ENV_VARS` 10 → 9 and
`SYSCTL_VALUES` 11 → 9. The previous edition's headline claim — "not one of these 21 values
has moved since 7.130.0" — is retired; it was already loose, because
`MKINITCPIO_COMPRESSION_OPTIONS` went 2 → 1 at the 7.140.0 `-T0` cut. Restated correctly:
**the four perf scalars have not moved since 7.130.0; the oracle has moved three times, at
7.140.0 and twice more after 7.155.0.**

**Five of the seventeen generated bodies changed content.** Determinism 3/3 renders, sorted
manifest sha `6bf7f8ea53c36c40` (harness-filename dependent — NOT comparable to the
`03b0863baa9fcde2` of the previous edition; per-file content hashes are the safe form).

```
║ FILE                          ║ 7.155.0 ║ 7.158.0 ║ CAUSE                        ║
║───────────────────────────────║─────────║─────────║──────────────────────────────║
║ /etc/kernel/cmdline           ║   352 B ║   351 B ║ fsck.mode=force -> auto at   ║
║                               ║         ║         ║ 7.158.0                      ║
║ /etc/sdboot-manage.conf       ║   544 B ║   543 B ║ same token, inside           ║
║                               ║         ║         ║ LINUX_OPTIONS                ║
║ /etc/mkinitcpio.conf          ║   276 B ║   276 B ║ COMPRESSION_OPTIONS -1 -> -3 ║
║                               ║         ║         ║ at 7.158.0. ZERO byte delta  ║
║ /etc/sysctl.d/95-ry-          ║   441 B ║   376 B ║ both net.core.netdev_budget  ║
║ overrides.conf                ║         ║         ║ keys removed at 7.157.0      ║
║ ~/.config/environment.d/      ║   306 B ║   282 B ║ PROTON_ENABLE_WAYLAND=1      ║
║ 10-environment.conf           ║         ║         ║ removed after 7.155.0        ║
║───────────────────────────────║─────────║─────────║──────────────────────────────║
║ 17-file total                 ║ 4,949 B ║ 4,858 B ║ -91 B, none of it perf       ║
```

**The mkinitcpio row is the most important line in this brief.** `-1` → `-3` is a real
configuration change to a boot-critical file. It moved **no count** (the array still has one
member) and **no byte anchor** (both tokens are two characters wide). A count check, a byte
check and a Σ check all pass across it. The only mechanism that catches it is a byte-exact
diff of the embedded fence against fresh generator output — which is why §7e embeds all
seventeen bodies and why the fence walker in §9 exists.

**Three consecutive rebases have each had a change a count check could not see** — 7.155.0's
`ENV_VARS`-held-at-10 FSR4 swap, and now a zero-delta body edit. Treat count parity as
evidence of nothing. The other twelve bodies are byte-identical, re-rendered and diffed.

### 1b. What did change

```
║ AREA          ║ 7.155.0    ║ 7.158.0    ║ NOTE                                 ║
║───────────────║────────────║────────────║──────────────────────────────────────║
║ script        ║ 4,915 L    ║ 4,915 L    ║ -47 B, 0 L; value edits only         ║
║ functions     ║ 294        ║ 294        ║ unchanged                            ║
║ verify fns    ║ 62 (12+50) ║ 62 (12+50) ║ identical prefix by prefix           ║
║ sub: markers  ║ 95         ║ 95         ║ all PARENT_OK                        ║
║ hard deps     ║ 37         ║ 37         ║ unchanged (L1535)                    ║
║ optional deps ║ 15         ║ 15         ║ unchanged (L1544)                    ║
║ banners       ║ 95         ║ 95         ║ blanks 95 == banners 95              ║
║ generators    ║ 17         ║ 17         ║ Sigma 4,949 -> 4,858 B               ║
║ oracle        ║ 21         ║ 21         ║ TWO values moved — see 1a            ║
║ harness cut   ║ L4786      ║ L4786      ║ FIRST rebase where it held           ║
║ README        ║ 309 L      ║ 307 L      ║ -5.6 % trim at 7.158.0               ║
║ CHANGELOG     ║ 143 L      ║ 131 L      ║ big trim + docs-entry purge          ║
```

**Every line anchor in this edition is identical to the previous one** — the script kept its
line count and lost 47 bytes to in-place value edits. **This does not relax the standing
rule:** anchors moved on four of the last five rebases, once backwards. Always locate a
symbol, never hardcode its line.

### 1c. Removals that must not be re-derived from a stale source

Nine features are gone. Each is a **retired question**, not an open gap:

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
║ RESOLVED_DNS_SERVERS (the     ║ 7.147.0  ║ The host pins NO upstream. Do NOT    ║
║ AdGuard pair) + the NM        ║          ║ report a missing DNS= line or a      ║
║ [global-dns-domain-*] block   ║          ║ lost NM override — both are by       ║
║                               ║          ║ design; the router is authoritative  ║
║ RESOLVED_DOT + RESOLVED_DNSSEC║ 7.148.0  ║ resolved drop-in is 4 lines. Do NOT  ║
║                               ║          ║ report DNSOverTLS=/DNSSEC= as        ║
║                               ║          ║ missing; both were redundant pins    ║
║ PROTON_FSR4_UPGRADE=1 and     ║ 7.154.0  ║ FSR4_WATERMARK=1 ships in their      ║
║ PROTON_FSR4_RDNA3_UPGRADE     ║          ║ place. The whole T1-3 RDNA3          ║
║                               ║          ║ workaround finding is CLOSED         ║
║ PROTON_ENABLE_WAYLAND=1       ║ after    ║ ENV_VARS 10 -> 9. T1-2, the previous ║
║                               ║ 7.155.0  ║ edition's strongest T1/T2 candidate, ║
║                               ║          ║ is CLOSED BY THE ARTIFACT. Do NOT    ║
║                               ║          ║ re-raise it as a global-scope defect ║
║ net.core.netdev_budget = 600  ║ 7.157.0  ║ SYSCTL_VALUES 11 -> 9. T2-1 CLOSED   ║
║ net.core.netdev_budget_usecs  ║          ║ on measurement (T0-5). An absent key ║
║ = 5000                        ║          ║ leaves the kernel default — do NOT   ║
║                               ║          ║ report 300/2000 as a regression      ║
```

**UNCERTAIN, and deliberately not guessed:** the exact release that dropped
`PROTON_ENABLE_WAYLAND` is not recoverable from the shipped artifact. The installer
CHANGELOG folds 7.139.0–7.157.0 into one range block, and the bullet carries no version.
The bound is `> 7.155.0` and `<= 7.157.0`, because the 7.155.0 edition rendered the variable
in its `10-environment.conf` fence. Do not state a release for it.

**The pattern to name: two of the last three rebases retired an open finding because the
artifact moved under it, not because new evidence arrived.** T1-3 at 7.154.0, T1-2 after
7.155.0. Re-render the archive before carrying any open item forward.

---

## 2. Action queue — ordered by implementation safety

The operative section. Tiers ascend by blast radius: T0 changes nothing, T5 must not be
touched. **Work top-down; do not act on a lower tier while a higher tier that gates it is
unrun.** Every recommendation must carry a tier.

**All eight T0 gates have now returned.** That is the structural change in this edition: for
twenty-five releases the brief carried eight unrun observations gating eleven lower-tier
items, and every one of them is now answered. Five gated items resolved into a shipped
change or a recorded maintainer KEEP; one turned out to be measuring an inert setting; one
returned an answer that closes a question permanently. **A gate result is history, not an
action** — T0 below is a results table, and the actions it unblocked live in their own tiers.

### T0 — RETURNED. Results, not actions.

```
║ ID   ║ OBSERVATION                  ║ RESULT                              ║ GATED  ║
║──────║──────────────────────────────║─────────────────────────────────────║────────║
║ T0-1 ║ turbostat idle-floor capture ║ 21.33 / 21.69 W package, 3.93 /     ║ T3-1   ║
║      ║ (two runs, idle desktop)     ║ 4.20 W core, at Busy% 0.13 / 0.24   ║        ║
║ T0-2 ║ lspci -vv LnkCtl ASPM per    ║ EVERY link reads "ASPM Disabled",   ║ T3-4   ║
║      ║ link                         ║ including links advertising L0s/L1  ║        ║
║      ║                              ║ and L1SubCap                        ║        ║
║ T0-3 ║ findmnt root FS type         ║ ext4 — NOT Btrfs                    ║ T3-3   ║
║ T0-4 ║ k10temp hwmon + energy_uj    ║ ONE labelled sensor, Tctl ->        ║ T1-1,  ║
║      ║ mode                         ║ temp1_input. No Tdie, no Tccd on    ║ T2-3   ║
║      ║                              ║ this part. energy_uj mode 400 =     ║        ║
║      ║                              ║ root-only                           ║        ║
║ T0-5 ║ softnet_stat squeezed column ║ 0 on all 32 CPUs, measured twice at ║ T2-1   ║
║      ║                              ║ ~2x traffic apart; dropped also 0   ║        ║
║ T0-6 ║ amd_pstate dynamic_epp state ║ disabled; EPP reads performance     ║ T5 EPP ║
║ T0-7 ║ .ry.orig / .ry.bak inventory ║ 4 .ry.bak (the boot-critical four)  ║ none — ║
║      ║                              ║ + 2 .ry.orig (resolved,             ║ foren- ║
║      ║                              ║ NetworkManager). No user-scope      ║ sic    ║
║      ║                              ║ .ry.orig. Exactly as designed       ║        ║
║ T0-8 ║ resolvectl per-link DNS      ║ resolv.conf mode: FOREIGN. Both     ║ T5 DNS ║
║      ║ readout                      ║ 10 GbE links are DOWN; wlan0 is the ║        ║
║      ║                              ║ default route                       ║        ║
```

**What each result closed:**

- **T0-1 → T3-1 is decidable and was decided.** The idle floor is ~21.5 W package against an
  85 W ceiling, with the CPU cores drawing under 4.2 W of it. The measurement did not show a
  runaway idle cost, and `processor.max_cstate=1` was retired to KEEP by maintainer decision
  (T5). The direction of the trade is still real; the magnitude is now on file rather than
  assumed. **Any future power argument must cite these two numbers, not re-open the
  question.**
- **T0-2 → T3-4 KEEP is now evidence-backed.** Every link reads `ASPM Disabled` *including
  links that advertise L0s/L1 and L1SubCap* — which is exactly what
  `pcie_aspm.policy=performance` is supposed to produce and is not what the firmware default
  would give. The token does real work on this board. `mt7925e.disable_aspm=1` is
  belt-and-braces at link level and stays: it suppresses `aspm_supported` at the endpoint,
  which the global policy does not.
- **T0-3 → T3-3 shipped.** The root filesystem is **ext4**, so `fsck.mode=force` was **not**
  inert — it forced a full check on every boot. The previous edition's "largely inert on a
  Btrfs root" framing was wrong for this host. Shipped as `fsck.mode=auto` at 7.158.0.
- **T0-4 → T1-1 has its sensor pair and T2-3 is declined.** `k10temp` exposes exactly one
  labelled sensor, `Tctl`, so the input is `temp1_input`; there is no Tdie and no Tccd on
  this part, and the question of *which* index is closed permanently. **Resolve hwmon by
  `name`, never by index** — the index is not stable across boots. `energy_uj` is mode
  **400**, root-only, so the `cpu_power` degradation cannot be fixed without a permission
  drop-in; that drop-in is DECLINED (T2-3).
- **T0-5 → T2-1 was measuring nothing.** Squeezed held at 0 on all 32 CPUs across two
  captures roughly 2x traffic apart, and dropped was 0 as well. Every authority makes
  `netdev_budget` tuning conditional on that column rising. Both keys were removed at
  7.157.0.
- **T0-6 → the T5 EPP no-op call is confirmed.** `dynamic_epp` reads `disabled`, matching the
  profile's assertion, and EPP reads `performance`. The udev EPP write remains a redundant
  but hotplug-safe no-op.
- **T0-7 → the backup design works as specified.** The absent user-scope `.ry.orig` is
  correct for a host whose user files had no pre-existing content, not a broken preserve.
- **T0-8 → the DNS posture is confirmed and one detail is new.** `/etc/resolv.conf` is in
  **foreign** mode — it is not resolved's stub — so the deployed resolved drop-in binds only
  resolved-routed queries. Low impact, no edit. Separately: **both 10 GbE links are down and
  wlan0 carries the default route**, which retires the "dual 10 GbE" premise that motivated
  the netdev tuning in the first place.

Advisory reads, ungated: **A1, the lspci ASPM audit, is satisfied by T0-2.** Still open:
**A2**, the installed proton-cachyos build, and the linux-firmware MES revision — both reads
with no gate and no dependent item.

### T1 — User/session scope. No root, no reboot, reversible by editing one file.

- **T1-1 · MangoHud CPU keys — the one open change item in this brief.** T0-4 supplied the
  pair (`k10temp,temp1_input`) but also closed the path: `cpu_custom_temp_sensor` requires
  `cpu_temp`, and MangoHud **#1794 is still OPEN** (re-confirmed 2026-08-07), so enabling
  `cpu_temp` on Zen 5 zeroes `cpu_power`. The resolved shape is therefore **not** to enable
  the custom sensor but to **ship the CPU keys commented**, recording the finding in the
  file where the next reader will meet it:
  `"cpu_stats"` becomes `"# cpu_stats intentionally disabled — enable for CPU load in the HUD"`,
  the existing `cpu_temp` comment stays, and a third line records the sensor pair and why it
  is not used. The HUD stops showing CPU load; that is the accepted cost.
  **Measured against the 7.155.0 base: script +204 B / +1 L, HUD body 383 → 575 B, no oracle
  move.** The MangoHud generator is byte-identical at 7.158.0, so the **body** figures
  re-base cleanly; the **script** figure does not — 292,715 B is the new base, not 292,762 B.
  **Must ship in lockstep with the standalone `mangohud-gtr9-pro` archive**, which carries the
  same file. IMPACT Low · RISK Low.
- **T1-2 · CLOSED BY THE ARTIFACT.** The previous edition called
  `PROTON_ENABLE_WAYLAND=1`-in-the-global-env-file "the strongest single T1/T2 candidate in
  the brief". The profile then removed it outright, dropping `ENV_VARS` 10 → 9. Upstream's
  framing (pass it *to your game*; aliased `PROTON_USE_WAYLAND=1`; Steam needs `-steamos3`
  for Steam Input under winewayland) is unchanged and correct — it simply no longer describes
  anything this profile sets. **Do not re-raise it.** Per-game opt-in is out of this brief's
  system scope. IMPACT n/a · RISK n/a.
- **T1-3 · CLOSED at 7.154.0 by the artifact, not withdrawn on evidence.** The FSR4 pin and
  the RDNA3 upgrade variable were removed and **`FSR4_WATERMARK=1`** shipped in their place.
  What survives is a **verification** variable, not an upgrade trigger. **Do not re-raise the
  `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` pairing as a profile gap** — there is no longer a
  global FSR4 upgrade flag for it to pair with. IMPACT n/a · RISK n/a.
- **T1-4 · Both user files have a working first-adoption preserve.** As of 7.135.1,
  `~/.config/environment.d/10-environment.conf` and `~/.config/MangoHud/MangoHud.conf` get a
  one-time `.ry.orig`. T0-7 found **no user-scope `.ry.orig` on this host**, which is the
  correct outcome for files that had no pre-existing content — absence is not evidence the
  preserve is broken. The *first* hand-edit is preserved, every subsequent one is silently
  overwritten by design. Both land 0600 by design (`_ry_install_file` L2048 sets 0644, then
  0600 when `use_sudo` is false). Do not raise the mode as drift.

### T2 — Managed config values. Root, no reboot, self-heals on the next deploy.

Everything here is Tier-1 under the removal-reconciliation model (§7f): the generator
rewrites the file wholesale, so a bad value cannot orphan.

- **T2-1 · CLOSED on measurement, keys removed at 7.157.0.** T0-5 returned squeezed 0 on all
  32 CPUs across two captures; the raised values were inert. Both `net.core.netdev_budget`
  keys are gone and `SYSCTL_VALUES` is 9. An absent key leaves the kernel default (300 /
  2000) — **that is not a regression and must not be reported as one.** The source conflict
  that made this hard is preserved in §3 for its own sake: the 600/4000 pairing is **Red
  Hat's**, not ESnet's, and ESnet recommends the defaults outright. Do not re-propose either
  key without a squeezed column that has actually moved, and see §4 item 3 for why the
  "dual 10 GbE" premise behind them did not describe the deployment.
- **T2-2 · nftables rule order — KEEP, settled.** The profile places `ct state invalid drop`
  before `iif "lo" accept`. That ordering is the one used by the Gentoo wiki's reference
  workstation ruleset and by the widely redistributed "early drop of invalid packets"
  template; the nftables.org quick reference and the ArchWiki dispatcher example put loopback
  first. **There is no upstream rule requiring loopback-first**, and no documented case of
  legitimate loopback traffic being classified `invalid` on a host that is not doing NAT or
  policy routing — this host does neither. KEEP with a note. If a FIX is nonetheless argued,
  it is a pure order swap, `nft -c` gated, re-validated by `_post_nft`. IMPACT Low · RISK Low.
- **T2-3 · `energy_uj` permission drop-in — DECLINED, do not re-offer.** T0-4 returned mode
  **400**: the powercap node is root-only, so MangoHud's `cpu_power` cannot read it as the
  user. The fix would be a permission drop-in, i.e. an **18th managed file** — a scope
  addition that moves three oracle counts (`SYSTEM_DESTINATIONS` 15→16, `_RY_POST_HOOKS`
  17→18 if it needs a hook, managed files 17→18), each a hard deploy gate. It is declined for
  a second and stronger reason: relaxing RAPL permissions re-opens the **PLATYPUS** side
  channel (§3). Leaving `cpu_power` degraded is the correct trade. **A recommendation to add
  the drop-in must address PLATYPUS explicitly, not around it.**
- **T2-4 · `amd_pstate=active` restates the CachyOS compiled default.** linux-cachyos sets
  `CONFIG_X86_AMD_PSTATE_DEFAULT_MODE=3`, and `drivers/cpufreq/Kconfig.x86` defines 3 as
  **Active (EPP)**. The cmdline token therefore changes nothing on the kernel this host
  actually runs. **KEEP** — the profile does not own the kernel package, the mode is a
  build-time choice that a CachyOS rebuild could flip, and the token costs one cmdline slot.
  It is belt-and-braces, not load-bearing. Contrast T3-4, where the equivalent config option
  is explicitly **not** set and T0-2 has now measured the token doing real work.
- **T2-5 · `VKD3D_CONFIG=descriptor_heap` removal — validated, closed.** Mesa main
  (26.3.0-devel) documents `RADV_DEBUG=noheap` as the switch that *disables*
  `VK_EXT_descriptor_heap`, and `heap` no longer appears in the `RADV_EXPERIMENTAL` list.
  Default-on is confirmed; the removal is correct on any Mesa this profile will meet.

### T3 — Kernel command line. Root + reboot. Reversible, but costs a boot cycle.

The cmdline is charset-gated (`^[A-Za-z0-9._,=-]+$`) and count-asserted at 15 — any add or
removal updates both, `_vsb_entry_options` (L2184) asserts every non-fallback BLS entry
carries every token, and `_vsb_sdboot_dropins` (L2116) warns when a drop-in could override
`LINUX_OPTIONS` behind all of it. **`KERNEL_PARAMS` stays at 15** — `fsck.mode` changed value,
not membership.

- **T3-1 · `processor.max_cstate=1` — measured, then retired to KEEP by maintainer
  decision.** T0-1 returned 21.33 / 21.69 W package and 3.93 / 4.20 W core at Busy% 0.13 /
  0.24. It remains the highest-leverage single token in the set on paper — it blocks deep CPU
  idle and compounds with DPM pinned `high` — and the owner has accepted that cost against
  idle-exit latency. **Do not re-propose removal on power grounds without a new measurement
  that beats these numbers.** The 85 W PPT caps peak regardless, so the metric remains idle,
  not the load ceiling.
- **T3-2 · `clearcpuid=umip` — retired to KEEP by maintainer decision, despite the taint.**
  Re-confirmed against mainline 7.2.0-rc6: `clearcpuid` has **zero occurrences** in
  `Documentation/admin-guide/kernel-parameters.txt` while the code still parses it —
  undocumented-but-live, with no deprecation warning and no doc contract. It also sets
  `TAINT_CPU_OUT_OF_SPEC`. On the evidence it is the strongest removal candidate in the
  cmdline set; the owner has weighed descriptor-table-base leak plus support posture against
  `umip_printk` stutter and kept it. **Record the trade, do not re-litigate it.** The string
  form is deliberate — CPUID bit numbers shift between kernels, the name does not; never
  propose the numeric `clearcpuid=514`.
- **T3-3 · SHIPPED at 7.158.0 — `fsck.mode=force` → `auto`.** T0-3 established the root
  filesystem is **ext4**, so the previous edition's "largely inert on a Btrfs root" reading
  was wrong: `force` ran a full check on **every** boot. `fsck.repair=yes` stays. Neither key
  is a kernel parameter — both are systemd-side, parsed by `systemd-fsck`, so cite systemd
  documentation, not `kernel-parameters.txt`. Byte effect: cmdline 352 → 351, sdboot 544 →
  543. The fstab rewrite path being ext4-only never established the root type, and that
  inference trap stands for the next reader.
- **T3-4 · The ASPM pair — CONFIRMED load-bearing by T0-2.** Every PCIe link on this board
  reads `LnkCtl: ASPM Disabled`, including links advertising L0s/L1 and L1SubCap. Three
  supporting facts: (a) `drivers/pci/pcie/Kconfig` `PCIEASPM_PERFORMANCE` disables ASPM L0s
  and L1 even where the BIOS enabled them; (b) linux-cachyos ships `CONFIG_PCIEASPM_DEFAULT=y`
  with `PCIEASPM_PERFORMANCE` **explicitly unset**, so the built-in policy is `default` (BIOS)
  and the cmdline token is what switches it; (c) `mt7925e.disable_aspm` is a live writable
  module param (perm 0644) in mainline mt76 that calls `mt76_pci_disable_aspm(pdev)` at probe
  and suppresses `aspm_supported`. The pair is **complementary, not redundant** — global
  policy governs link state, the module option disables at the endpoint. **Do not describe
  the module option as omitted** (true only between 7.102.x and 7.129.x) and **do not simplify
  the pair away.** `pcie_aspm=off` is documented as "don't touch ASPM configuration at all"
  and does NOT disable it — never propose it as an equivalent. mt76 now force-disables ASPM on
  MT7927 hardware regardless of the param; MT7925 is not covered by that quirk.

### T4 — Boot chain, firewall handoff, and detector severity. Reboot + recovery exposure.

- **T4-1 · Fallback-entry exposure — still open.** `LINUX_FALLBACK_OPTIONS="quiet"` strips all
  15 params, so the fallback boots with IOMMU **on**, IPv6 **enabled** under an IPv4-only
  ruleset, and ASPM at firmware default with the MT7925 endpoint option absent — while the
  modprobe amdxdna blacklist *remains* active, because it is a file rather than a cmdline
  token. That asymmetry is the sharp edge: the fallback gets IOMMU back but keeps the NPU
  blacklisted, which is the one combination the validator refuses to *deploy*. T0-2 sharpens
  it — the fallback is the only boot path on this host where ASPM is not disabled. Confirm the
  window is accepted or flag it. Verify will never surface it — see §8 for why.
- **T4-2 · SHIPPED at 7.158.0 — `COMPRESSION_OPTIONS=(-1)` → `(-3)`.** Smaller initramfs,
  more ESP headroom; `-3` only ever shrinks the image relative to `-1`, so the only cost is
  build time, and reverting is free. **The `df -h /boot` headroom gate was never run** — it is
  a live-host check and the change was applied on the maintainer's explicit order without it.
  Record that gap rather than presenting the change as fully gated. Byte effect: **zero** —
  `mkinitcpio.conf` stays at 276 B because both tokens are two characters wide (§1a).
  `BOOT_SPACE_CRIT/WARN` (L612) are 200/500 **MiB**, not MB; the gate is
  `_check_avail /boot 1048576 MiB`.
- **T4-3 · `timeout 0` + `DEFAULT_ENTRY manual` + `REMOVE_EXISTING=yes`** wipes foreign BLS
  entries (EFI-resident loaders untouched). Recovery path is live-USB → chroot. With
  `timeout 0` and no saved EFI variable, a fresh ESP falls back to sd-boot's own sort order
  until the menu is used once. UKI is out of scope. **Adjacency:** `_vsb_sdboot_dropins`
  warns on any `*.conf` under `/usr/lib/sdboot-manage.conf.d` or `/etc/sdboot-manage.conf.d` —
  a packaged drop-in from a future CachyOS update would silently outrank the managed
  `LINUX_OPTIONS`, and the WARN is the only signal.
- **T4-4 · ufw masked, not removed — confirm the nftables-first gate closes the window.**
  `_csm_enable_nftables_first` is gated on `contains ufw.service $MASK` and confirms a live
  default-deny ruleset before anything touches ufw; `_csm_prepare_ufw_masking` returns
  non-zero on an unconfirmed ruleset and `_configure_services_mask` then withholds
  `ufw.service` from the safe-mask set for that run. Rationale: `mask --now` stops ufw and
  `ufw-init stop` flushes its rules, so masking before default-deny is live would open an
  unfirewalled window. Validate on a host where ufw is installed *and* active; confirm a
  withheld mask leaves ufw's own rules intact rather than half-flushed; confirm
  `nftables.service` being a oneshot (unit state reads inactive after a clean load) does not
  defeat the liveness check — the script judges by live policy-drop, which is correct but
  should be validated against current nftables packaging. Log keys: `UFW_MASK_DEFERRED`,
  `UFW_RULE_FLUSH_OK|FAIL|SKIP`, `SECURITY_POSTURE`.
- **T4-5 · Orphan-detector severity — a design question, not a gap.** Three detectors ship and
  none sets DRIFT: `_vss_modprobe` (L2271) WARNs on unmanaged `60-ry-*.conf` drop-ins,
  `_vss_orphan_masks` (L2414) INFOs on masked units absent from `$MASK`, and
  `_vsb_sdboot_dropins` (L2116) WARNs on sdboot-manage drop-ins. `--check` records
  `MODPROBE_STALE_DROPIN` / `MASK_ORPHAN` / `SDBOOT_DROPIN_PRESENT` to JSONL only. The
  reasoning on file: a re-run cannot clear any of them, so exit 10 would go permanently
  non-zero and train the operator to ignore exit 10 entirely. Mask ownership is genuinely
  **unattributable** — there is no `60-ry-`-style namespace for systemd units — which is why it
  is `_info`, not `_warn`. **Evaluate the trade.** The counter-argument worth testing: an INFO
  in a long `--verify` run is easy to miss and JSONL is only read deliberately. Any
  recommendation to promote one to DRIFT must address the desensitization argument explicitly
  and must say which of the three it applies to — they are not equally attributable.
- **T4-6 · `_check_record_orphans` (L2589) sits after the sudo/systemctl preflight bail.** A
  stale drop-in is detectable with `find` alone — no privilege — yet the record is withheld
  when `sudo -n` is not cached. Consistent with `--check` being sudo-gated by contract, but it
  is a privilege-free signal withheld for a privilege reason. Assess whether the sweep should
  be hoisted above the bail. Its *placement inside the mode* is already correct and
  deliberate: it runs before the phase loop, because at the tail of `_check_phase_*` it was
  skipped whenever preflight bailed. Do not re-propose that move.
- **T4-7 · LOW, open — the nftables ruleset is IPv6-unsafe if `ipv6.disable=1` is ever
  removed.** `table inet` with `policy drop` covers IPv6, but the ruleset carries **no
  `icmpv6` accept rule**, so NDP / RA / MLD would be silently dropped and IPv6 would fail with
  no diagnostic. Nothing enforces the coupling in either direction. Inert while IPv6 is off.
  Fix is an `icmpv6` accept line (inert today) or a preflight assert that refuses the pair.
- **T4-8 · INFO, maintenance-only.** The `_RY_POST_HOOKS` boot entry keys on the glob
  `/boot/*`, so any future managed file under `/boot` would silently route to the `loader`
  hook rather than getting its own. Inert today — `loader.conf` is the only `/boot`
  destination. Record it before adding one.

### T5 — Closed and protected. Do not recommend changing these.

Flag a direct upstream contradiction as a **note**; never as a FIX.

- **DNS is not the host's problem any more — three layers, and the profile owns none of
  them.** Since 7.147.0–7.148.0 the host pins **nothing**: no `DNS=`, no `DNSOverTLS=`, no
  `DNSSEC=`, and no NetworkManager `[global-dns]` block. It takes per-link DHCP DNS from the
  router in plaintext, by design. The router forwards to AdGuard's ad-block tier
  (94.140.14.14 / 94.140.15.15) **over DoT** since 2026-07-31, and AdGuard validates DNSSEC on
  its own resolvers. **Do not propose host-side DoT.** The router's DNS Privacy Protocol is
  WAN-side only; it does not serve DoT to LAN clients, so `DNSOverTLS=yes` on the host would
  TLS-handshake a plaintext-only resolver and, failing closed, take DNS down. T0-8's foreign-mode
  result narrows this further: the drop-in is not the authority for every lookup on this
  host. Quantify the residual LAN-segment exposure in §5 and stop.
- **`GPU_DPM_LEVEL=high`, not `profile_peak`.** `high` forces the highest power state with
  clock and power gating still active. `profile_peak` adds mclk/pcie forcing and disables
  gating, but kernel documentation scopes `profile_*` to measurement work, ArchWiki and
  amdgpu-clocks document `auto|low|high|manual` as the primary set, ROCm warns STABLE_PEAK is
  ASIC-specific and unverified on gfx1151, and Phoronix found forced `high` vs `auto` differs
  only in select cases. Evaluate `high` vs `auto` on frametime evidence only. The
  `profile_peak` variant measures udev **647 B** (+8) — recorded so it is never re-measured.
- **The udev EPP write is a redundant no-op and that is fine.** Under
  `CPUPOWER_GOVERNOR=performance` the driver returns `-EBUSY` for any `epp > 0` and only
  offers "performance" in `energy_performance_available_preferences`; writing "performance"
  maps to epp 0 and IS accepted. T0-6 confirmed `dynamic_epp` reads `disabled`, so nothing
  currently blocks the write. Do not file the redundancy as a defect — it is a hotplug-safe
  assertion of a state the governor already imposes, and it is the only mechanism that
  survives a CPU hotplug event.
- **`processor.max_cstate=1` and `clearcpuid=umip` are protected by maintainer decision**, not
  by absence of evidence. Both were measured or documented against, both were kept. See T3-1
  and T3-2. A recommendation to remove either must argue against the recorded decision.
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
  `nowatchdog`, `tsc=reliable`, `8250.nr_uarts=0` (cmdline); `AMD_VULKAN_ICD`, `DXVK_LOG_PATH`,
  `VKD3D_CONFIG`, `PROTON_ENABLE_WAYLAND` (env); `ddcutil`, `git-delta` (packages);
  `modemmanager.service` (mask); both `net.core.netdev_budget` keys (sysctl);
  `RY_REMOTE_PLAY_PORTS` and its port sets (7.137.0); the preemption advisory (7.139.0 r2);
  the redundant `-T0` (7.140.0); `pcie_aspm=off`; `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK`;
  `RY_NO_NTP_REMEDIATION`; `clearcpuid=514` numeric form; `archlinux-contrib`; the standalone
  `60-ry-blacklist-amdxdna.conf` and `60-ry-mt7925e.conf` drop-ins (both filenames are now
  actively swept for). **Standing precedent:** `mt7925e.disable_aspm=1` was removed at
  7.102.x and re-added at 7.129.0 — removals are not permanent, but a re-add needs the same
  evidence bar as a FIX.
- **Do not re-add** ICMPv6/NDP rules without restoring IPv6 (T4-7 is the tracked form of that
  question); do not flag inbound-ping accept (`_vss_nft` hard-fails on its absence,
  `_vrsv_nft_assert_ping` warns live); do not propose sleep-hook re-assert workarounds (all
  five sleep targets are masked, so there is no resume path and the udev `ACTION=="add"` rule
  is the only event that matters); do not flag a low MangoHud `vram` reading on UMA (it
  reports the BIOS carveout only — `ram` carries the shared pool); do not propose annotating
  more sysctl keys (the drop-in carries ONE comment, its header — see the §7e fence; per-key
  annotation was never shipped).

---

## 3. Settled — do not re-research

Confirmed from primary sources. Re-verify only if a citation is challenged. The **VERIFIED**
column is the date the claim was last checked against the live source. Three rows were
re-fetched on **2026-08-07** for this rebase (the version set and both issue states); every
other row is carried forward with its previous date, because the upstream world did not move
between 2026-08-05 and 2026-08-07.

```
║ CLAIM                          ║ VERDICT / SOURCE                    ║ VERIFIED ║
║────────────────────────────────║─────────────────────────────────────║──────────║
║ Does the performance governor  ║ YES. amd-pstate.c mainline:         ║ 26-08-05 ║
║ reject a non-max EPP write?    ║ epp > 0 && policy ==                ║          ║
║                                ║ CPUFREQ_POLICY_PERFORMANCE returns  ║          ║
║                                ║ -EBUSY. Writing "performance" maps  ║          ║
║                                ║ to epp 0 and IS accepted, so the    ║          ║
║                                ║ udev rule lands as a redundant      ║          ║
║                                ║ no-op. Rejection is pr_debug —      ║          ║
║                                ║ never in default dmesg              ║          ║
║ Available-preferences readout  ║ Under the performance policy the    ║ 26-08-05 ║
║ under performance policy       ║ sysfs file emits ONLY "performance" ║          ║
║ dynamic_epp availability       ║ CLOSED. Ships since 7.1 and         ║ 26-08-05 ║
║                                ║ linux-cachyos builds 7.1.6, so it   ║          ║
║                                ║ IS present on this host. Kernel     ║          ║
║                                ║ default FALSE (static bool). When   ║          ║
║                                ║ enabled it blocks ALL manual EPP    ║          ║
║                                ║ writes with -EBUSY. T0-6 read       ║          ║
║                                ║ "disabled" live on this host        ║          ║
║ amd_dynamic_epp= boot param    ║ DOCUMENTED in mainline kernel-      ║ 26-08-05 ║
║                                ║ parameters.txt: disable | enable.   ║          ║
║                                ║ Profile does not set it and should  ║          ║
║                                ║ not. If a future default flips, the ║          ║
║                                ║ udev EPP write breaks               ║          ║
║ amd_pstate default mode on     ║ Kconfig.x86 X86_AMD_PSTATE_DEFAULT_ ║ 26-08-05 ║
║ CachyOS                        ║ MODE: 3 = Active (EPP). CachyOS     ║          ║
║                                ║ ships =3, so amd_pstate=active      ║          ║
║                                ║ RESTATES the compiled default       ║          ║
║ pcie_aspm.policy=performance   ║ DISABLES. drivers/pci/pcie/Kconfig  ║ 26-08-05 ║
║ semantics                      ║ PCIEASPM_PERFORMANCE disables ASPM  ║          ║
║                                ║ L0s and L1 even where the BIOS      ║          ║
║                                ║ enabled them. Cite the Kconfig —    ║          ║
║                                ║ the "since 4.2" figure on           ║          ║
║                                ║ kernelconfig.io is a database-floor ║          ║
║                                ║ artifact                            ║          ║
║ Is the ASPM cmdline token      ║ YES, and now measured. CachyOS sets ║ 26-08-05 ║
║ load-bearing on CachyOS?       ║ PCIEASPM_DEFAULT=y and              ║ + T0-2   ║
║                                ║ PCIEASPM_PERFORMANCE is NOT set.    ║          ║
║                                ║ T0-2: every link on this board      ║          ║
║                                ║ reads LnkCtl ASPM Disabled          ║          ║
║ pcie_aspm=off semantics        ║ "Don't touch ASPM configuration at  ║ 26-08-05 ║
║                                ║ all. Leave any configuration done   ║          ║
║                                ║ by firmware unchanged." Does NOT    ║          ║
║                                ║ disable ASPM                        ║          ║
║ clearcpuid documentation state ║ ZERO occurrences in mainline        ║ 26-08-05 ║
║                                ║ kernel-parameters.txt while the     ║          ║
║                                ║ code still parses it. Also sets     ║          ║
║                                ║ TAINT_CPU_OUT_OF_SPEC               ║          ║
║ mt7925e.disable_aspm exists    ║ YES. mt76/mt7925/pci.c module_param ║ 26-08-05 ║
║                                ║ _named, perm 0644, calls            ║          ║
║                                ║ mt76_pci_disable_aspm() at probe    ║          ║
║                                ║ and suppresses aspm_supported.      ║          ║
║                                ║ MT7927 is force-disabled by a       ║          ║
║                                ║ separate quirk; MT7925 is not       ║          ║
║ RTL8127 in r8169               ║ Present in mainline r8169_main.c;   ║ 26-07-27 ║
║                                ║ first landed v6.16, absent at v6.15 ║          ║
║ PROTON_FSR4_UPGRADE currency   ║ MOOT — REMOVED at 7.154.0 with      ║ 26-08-05 ║
║                                ║ PROTON_FSR4_RDNA3_UPGRADE, because  ║          ║
║                                ║ proton-cachyos 11.0-20260702+       ║          ║
║                                ║ copies amdxcffx64.dll itself.       ║          ║
║                                ║ FSR4_WATERMARK=1 ships instead and  ║          ║
║                                ║ is a VERIFICATION variable. Do not  ║          ║
║                                ║ re-raise the RDNA3 workaround       ║          ║
║ PROTON_ENABLE_WAYLAND scope    ║ MOOT — REMOVED from ENV_VARS after  ║ 26-07-27 ║
║                                ║ 7.155.0. Upstream framing stands    ║          ║
║                                ║ (per-game; aliased PROTON_USE_      ║          ║
║                                ║ WAYLAND=1; Steam needs -steamos3    ║          ║
║                                ║ for Steam Input under winewayland)  ║          ║
║                                ║ but describes nothing this profile  ║          ║
║                                ║ sets                                ║          ║
║ ntsync currency                ║ CONFIG_NTSYNC=m in linux-cachyos    ║ 26-08-05 ║
║                                ║ (module, not builtin). ntsync is    ║          ║
║                                ║ the default; PROTON_NO_NTSYNC=1 is  ║          ║
║                                ║ the opt-out. Profile neither sets   ║          ║
║                                ║ nor checks it, correctly            ║          ║
║ RADV descriptor heap           ║ DEFAULT-ON. Mesa 26.3.0-devel docs: ║ 26-08-05 ║
║                                ║ RADV_DEBUG=noheap disables VK_EXT_  ║          ║
║                                ║ descriptor_heap; heap is gone from  ║          ║
║                                ║ the RADV_EXPERIMENTAL list          ║          ║
║ MESA_SHADER_CACHE_MAX_SIZE     ║ Documented; number + K/M/G suffix.  ║ 26-07-27 ║
║ accepts 16G                    ║ Default 1 GB if unset               ║          ║
║ MangoHud cpu_custom_temp_      ║ CURRENT. Form is <hwmon>,<input>.   ║ 26-07-27 ║
║ sensor                         ║ T0-4 supplies the pair for this     ║ + T0-4   ║
║                                ║ host: k10temp,temp1_input           ║          ║
║ MangoHud #1794                 ║ STILL OPEN — cpu_power reads 0 when ║ 26-08-07 ║
║                                ║ cpu_temp is active on Zen 5. This   ║          ║
║                                ║ is why T1-1 comments the CPU keys   ║          ║
║                                ║ instead of enabling the sensor      ║          ║
║ netdev_budget guidance         ║ Kernel defaults 300 / 2000. The     ║ 26-07-27 ║
║ attribution                    ║ 600/4000 pair is RED HAT's (RHEL    ║          ║
║                                ║ 8/9/10), NOT ESnet's. ESnet's 100 G ║          ║
║                                ║ page recommends DEFAULTS and warns  ║          ║
║                                ║ changes can cut throughput. Both    ║          ║
║                                ║ gate on softnet_stat column 3,      ║          ║
║                                ║ which T0-5 measured at 0. Keys      ║          ║
║                                ║ removed 7.157.0                     ║          ║
║ nftables invalid-before-       ║ NO UPSTREAM RULE either way.        ║ 26-07-27 ║
║ loopback ordering              ║ Gentoo's reference ruleset uses the ║          ║
║                                ║ profile's order; nftables.org and   ║          ║
║                                ║ ArchWiki put loopback first. KEEP   ║          ║
║ NM vs resolved DNS precedence  ║ systemd #33973 closed-completed —   ║ 26-08-07 ║
║                                ║ per-link DHCP DNS outranks global   ║          ║
║                                ║ DNS=. Domains=~. is NOT the fix     ║          ║
║                                ║ (#33579, re-confirmed OPEN). NM     ║          ║
║                                ║ [global-dns-domain-*] was the       ║          ║
║                                ║ correct mechanism but was REMOVED   ║          ║
║                                ║ at 7.147.0 — the host now pins      ║          ║
║                                ║ nothing, so the precedence problem  ║          ║
║                                ║ has no surface left on this profile ║          ║
║ resolv.conf mode on this host  ║ FOREIGN (T0-8) — /etc/resolv.conf   ║ T0-8     ║
║                                ║ is not resolved's stub, so the      ║          ║
║                                ║ managed drop-in binds only          ║          ║
║                                ║ resolved-routed queries. Low        ║          ║
║                                ║ impact, no edit                     ║          ║
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
║ energy_uj readability          ║ MODE 400 on this host (T0-4) —      ║ T0-4     ║
║                                ║ root-only. Post-PLATYPUS kernels    ║          ║
║                                ║ restrict RAPL deliberately;         ║          ║
║                                ║ re-opening it re-opens the side     ║          ║
║                                ║ channel. T2-3 DECLINED on that      ║          ║
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

## 4. Corrections this rebase makes to the 7.155.0 edition

Binding. Do not re-raise anything this list withdraws. The 7.155.0 edition's own seven
corrections (the T5 DNS framing, T1-3's closure, the 11-site perf count, sixteen service
keys, the MiB/MB fix, the empirical verify count, the sysctl-annotation retraction) all still
stand and are not restated here.

1. **T0 is no longer unrun, and that invalidates every "gated on T0-n, do not decide"
   sentence in the previous edition.** All eight returned. Five gated items resolved into a
   shipped change or a recorded maintainer KEEP, one was measuring an inert setting, one is
   now permanently closed. A recommendation that still describes T0 as unrun is reading a
   pre-7.156.0 copy.
2. **T3-3's premise was wrong: the root filesystem is ext4, not Btrfs.** "Largely inert on a
   Btrfs root" was the framing; on this host `fsck.mode=force` forced a full check on every
   boot. Shipped as `auto` at 7.158.0. **The inference trap that produced the error is worth
   keeping:** the fstab rewrite path is ext4-only, which established nothing about the root,
   and a verify log showing ext4 checks passing does not prove the root is ext4 either. Only
   `findmnt` on `/` settles it.
3. **T2-1 was measuring nothing, and the premise behind it does not hold on this host.**
   Squeezed read 0 on all 32 CPUs across two captures ~2x traffic apart, dropped also 0. Both
   keys were removed at 7.157.0. Separately, T0-8 found **both 10 GbE links down with wlan0
   as the default route** — the "dual 10 GbE" justification for the tuning was not describing
   the deployed configuration.
4. **"Not one of these 21 oracle values has moved since 7.130.0" is retired.**
   `MKINITCPIO_COMPRESSION_OPTIONS` moved 2 → 1 at 7.140.0, and `ENV_VARS` 10 → 9 and
   `SYSCTL_VALUES` 11 → 9 both moved after 7.155.0. What is true and worth carrying: **the
   four perf scalars have not moved since 7.130.0, twenty-eight releases.**
5. **A generated body changed with a byte delta of zero** — `mkinitcpio.conf`
   `COMPRESSION_OPTIONS` `-1` → `-3` (§1a). First instance on record of a change invisible to
   counts, byte anchors and the Σ total at once.
6. **T1-2 was closed by the artifact, for the second time in three rebases.** The previous
   edition's strongest T1/T2 candidate — moving `PROTON_ENABLE_WAYLAND=1` out of the global
   env file — was resolved by the profile removing the variable. Its upstream reading was
   correct when written and now describes nothing the profile sets.
7. **`energy_uj` is mode 400, so T2-3 is not a scope question but a security one.** The
   previous edition framed the drop-in as "an 18th managed file, weigh it against leaving
   `cpu_power` degraded". The stronger objection is PLATYPUS: relaxing RAPL permissions
   re-opens a documented side channel. DECLINED, not merely deferred.
8. **`k10temp` exposes exactly one labelled sensor.** T0-4 closes the "do not guess the
   index" caution with an answer: `Tctl` → `temp1_input`, no Tdie and no Tccd on this part.
   The residual rule is different and permanent: **resolve hwmon by `name`, never by index** —
   the index is not stable across boots.
9. **Line anchors did not move this rebase — the first time that has happened.** The script
   held 4,915 lines and lost 47 bytes to in-place value edits. Every perf global, validator,
   generator and README perf row is where the 7.155.0 edition put it. **This is a fact about
   7.158.0, not a new rule.** Anchors moved on four of the previous five rebases, once
   backwards; always locate the symbol.
10. **The 278 expected `--verify` OK count is stale and must not be cited.** It was empirical
    from a clean-host run, and the verify surface has since lost two sysctl keys and one
    environment variable, each of which is an asserted item. **Re-baseline on the next clean
    run; treat the figure as UNKNOWN until then.**

---

## 5. Security posture — quantify only, ordered by exposure

No auto-FIX in this section.

1. **UMIP off** (`clearcpuid=umip`) — descriptor-table base leak, kernel tainted
   (`TAINT_CPU_OUT_OF_SPEC`), and undocumented upstream while still parsed. Headline open
   reduction on the evidence; **retired to KEEP by maintainer decision** (T3-2). State the
   exposure, do not re-propose the removal.
2. **AMD-Vi fully disabled** (`amd_iommu=off`) — no DMA isolation or remapping; USB4/TB, NVMe
   and NIC DMA unmediated. Named casualty: the XDNA 2 NPU, blacklisted. Opting back in is one
   validator-enforced pair (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`), restoring
   isolation and the NPU together. The coupling asymmetry is intentional and was re-probed for
   this edition: `amd_iommu=on` with blacklist true is valid; blacklist false without the
   IOMMU refuses to deploy with rc 3. **The fallback entry inverts this pairing** (IOMMU on,
   blacklist still active) — see T4-1.
3. **DNS: plaintext on the LAN leg only, encrypted and validated beyond the router.** The
   host↔router hop is plaintext DHCP-supplied DNS with no host-side validation; the
   router↔AdGuard hop is DoT and AdGuard validates DNSSEC. Residual exposure is scoped to the
   LAN segment and to trusting the router's answer — the AD bit is a bit any in-path party
   could set, and nothing on the host checks it. Accepted on file; state the exposure and stop
   (T5, which owns the host-side-DoT prohibition and the foreign-mode qualifier).
4. **IPv6 disabled + inbound IPv4 ping accepted** — net LAN delta is `+ping −mDNS`. Avahi is
   masked (unit *and* socket) and resolved has `MulticastDNS=no`, so multicast discovery is
   fully closed. Ping-accept is an asserted regression guard, not a defect. Latent coupling:
   `table inet` + `policy drop` covers IPv6, but the ruleset carries **no `icmpv6` accept
   rule**, so if `ipv6.disable=1` were ever removed, NDP / RA / MLD would be silently dropped
   and IPv6 would fail with no diagnostic. Nothing enforces the coupling. Inert while IPv6 is
   off. Tracked as T4-7. LOW.
5. **`split_lock_detect=off`** — a misbehaving application can degrade the whole system.
6. **ufw masked rather than removed** — the package stays installed and could be unmasked and
   started, at which point two firewall managers contend for the same netfilter tables.
   Quantify that against the benefit (reversibility, no package churn); the nftables-first
   gate (T4-4) is the only thing standing between a mis-sequenced run and an unfirewalled
   window.
7. **RAPL stays restricted.** `energy_uj` is mode 400 and the permission drop-in is declined
   (T2-3). This is a posture *win* recorded in the security list so that a future "make
   `cpu_power` work" request is costed correctly.
8. **sdboot-manage drop-in override path** — a packaged drop-in can replace `LINUX_OPTIONS`
   entirely, and until 7.140.0 nothing looked. Now WARN-only (T4-3 / T4-5).
9. **No sleep path** — all five sleep/suspend/hibernate targets masked. An always-on box never
   gets the "locked on resume" checkpoint. Deliberate for a headless-adjacent mini-PC.
10. **Historical and now confirmed void: the `.ry.orig` dead-code window.** From 7.109.0
    through 7.135.0 the first-adoption preserve never executed, so 13 of 17 destinations were
    overwritten with **no backup of any kind**. Security-relevant members of that set:
    `/etc/nftables.conf` and `/etc/NetworkManager/conf.d/99-cachyos-nm.conf`. Fixed at
    7.135.1. **This host was reinstalled fresh on 2026-07-19 at a version well past the fix,
    and T0-7 returned the designed inventory** (4 `.ry.bak` + 2 `.ry.orig`, no user-scope
    `.ry.orig`), so the window is void here. **The generalizable lesson stands and is the
    reason T0-7 existed: presence of a guard in source is not evidence the guard runs.** Any
    claim of the form "X is protected because the code does Y" needs a behavioural probe.

**Default-deny-inbound ships and is net positive.** Residual notes: `flush ruleset` blast
radius against docker/libvirt/podman; no ICMP or new-connection rate limit (trusted-LAN
assumption — state it).

---

## 6. Verify block

Grouped to match the tiers. All commands are reads unless marked. **Every command here is
self-resolving — no token needs manual substitution.** A fence that requires the operator to
fill in a device node or a path gets pasted verbatim and fails; that has happened twice.

**T0 re-reads — the gates have returned, so these are now regression checks with expected
values:**

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
grep -l '^k10temp$' /sys/class/hwmon/hwmon*/name               # expect exactly one path
sensors 2>/dev/null | grep -A2 '^k10temp'                      # expect Tctl only, no Tdie/Tccd
stat -c '%a %n' /sys/class/powercap/*/energy_uj                # expect 400 — root-only (T2-3)

# T0-5  softnet_stat squeezed column. Expect 0 on all 32 CPUs
awk '{print strtonum("0x" $3)}' /proc/net/softnet_stat | sort -n | uniq -c

# T0-6  EPP mechanics
cat /sys/devices/system/cpu/amd_pstate/dynamic_epp             # expect: disabled
grep -c 'amd_dynamic_epp' /proc/cmdline                        # expect 0 — profile must not set it

# T0-7  backup inventory. Expect 4 .ry.bak + up to 2 .ry.orig, no user-scope .ry.orig
find /etc /boot -maxdepth 3 -name '*.ry.bak' -o -maxdepth 3 -name '*.ry.orig' 2>/dev/null | sort
find "$HOME/.config" -name '*.ry.orig' 2>/dev/null             # expect empty on a fresh adoption
find "$HOME/ry-install/logs" -name '*.jsonl' -exec grep -l 'PREEXISTING_PRESERVED' {} + 2>/dev/null

# T0-8  DNS and routing reality
resolvectl status                                              # per-link DHCP DNS; NO global server
head -n 20 /etc/resolv.conf                                    # foreign mode — not resolved's stub
ip -brief link show                                            # both 10 GbE links DOWN on this host
ip route show default                                          # default route is wlan0
```

The `find` forms above replace the previous edition's `**` globs, which silently missed the
`logs/<date>/` level — see §7g for the log path and why a zero-hit glob reads identically to
"the key was never emitted".

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
cat /sys/block/nvme*n*/queue/scheduler                                   # [none] on every NVMe
```

**Cmdline tokens and their removals:**

```fish
grep -o 'fsck\.mode=[a-z]*' /proc/cmdline             # auto since 7.158.0 — NOT force
grep -o 'amd_iommu=[^ ]*\|ipv6\.disable=[^ ]*\|clearcpuid=[^ ]*' /proc/cmdline
grep -o 'pcie_aspm[^ ]*\|mt7925e[^ ]*' /proc/cmdline
grep -o 'processor\.max_cstate=[^ ]*\|amd_pstate=[^ ]*' /proc/cmdline
grep -c 'nowatchdog\|tsc=reliable\|8250' /proc/cmdline    # 0 — the three removals still hold
grep -c 'preempt=' /proc/cmdline                          # 0 — never pinned
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l   # 0 — INFORMATIONAL
sudo dmesg | grep -i 'AMD-Vi\|DMAR'                       # expect NO "AMD-Vi: Enabled"
```

**Boot chain (includes the 7.158.0 compression change):**

```fish
ls /usr/lib/sdboot-manage.conf.d /etc/sdboot-manage.conf.d 2>/dev/null   # expect: absent or empty
grep -rc 'LINUX_OPTIONS' /etc/sdboot-manage.conf.d 2>/dev/null           # any hit outranks the managed conf
grep -h '^options' /boot/loader/entries/*.conf | grep -v fallback        # 15 tokens per entry
grep -h '^options' /boot/loader/entries/*fallback*.conf                  # "quiet" only — T4-1 window
grep '^COMPRESSION' /etc/mkinitcpio.conf                                 # zstd and (-3)
df -h /boot                                                              # the gate T4-2 never ran
ls -l --block-size=M /boot/initramfs-*.img                               # image size after -3
bootctl status | grep -i 'default\|timeout\|entry'
```

**Memory, network, DNS:**

```fish
sysctl kernel.nmi_watchdog net.ipv4.tcp_congestion_control net.core.default_qdisc \
       net.ipv4.tcp_notsent_lowat net.ipv4.tcp_slow_start_after_idle \
       vm.max_map_count vm.compaction_proactiveness vm.swappiness vm.watermark_boost_factor
sysctl net.core.netdev_budget net.core.netdev_budget_usecs   # 300 / 2000 kernel defaults, unmanaged
grep -c netdev /etc/sysctl.d/95-ry-overrides.conf            # 0 — both keys removed at 7.157.0
grep -c '^[a-z]' /etc/sysctl.d/95-ry-overrides.conf          # 9 keys
systemd-analyze cat-config sysctl.d | grep -n 'nmi_watchdog' # 95-ry wins over vendor 70
resolvectl status | grep -i 'DNS Servers\|DNSSEC\|DNSOverTLS'  # per-link only; no global pin
resolvectl query example.com                                 # answered via the router
grep -c 'global-dns' /etc/NetworkManager/conf.d/99-cachyos-nm.conf   # 0 — block removed 7.147.0
swapon --show; zramctl                                       # zram advisory, not managed
iw reg get | grep -i country                                 # US
```

**Firewall and units:**

```fish
sudo nft list chain inet filter input   # policy drop; invalid-drop FIRST, then est/rel, then lo
sudo nft -c -f /etc/nftables.conf
grep -c icmpv6 /etc/nftables.conf       # 0 — inert while ipv6.disable=1; see T4-7
systemctl is-enabled ufw.service        # masked — NOT "not installed"
systemctl is-enabled sleep.target suspend.target hibernate.target \
                     hybrid-sleep.target suspend-then-hibernate.target   # masked x5
systemctl list-unit-files --state=masked,masked-runtime --no-legend --plain  # compare vs MASK 11
systemctl is-enabled avahi-daemon.service avahi-daemon.socket bluetooth.service
systemctl is-enabled systemd-resolved.service          # enabled or static
systemctl --user is-failed plasma-powerdevil.service   # NOT "failed"
systemctl --user --failed --plain --no-legend          # empty
stat -c '%a %U:%G %n' /etc/NetworkManager/system-connections/*   # 0600 root:root
```

**Userspace / HUD:**

```fish
lsmod | grep -c '^amdxdna'                                # 0 — loaded = verify FAIL
ls /etc/modprobe.d/60-ry-*.conf                           # ONLY 60-ry-modules.conf
ls -l /dev/ntsync                                         # present (assert-only)
vulkaninfo | grep -i 'driverName\|deviceName'             # RADV / Radeon 8060S, no ICD pin
systemctl --user show-environment | grep 'FSR4_WATERMARK\|MANGOHUD\|POWERDEVIL_NO_DDCUTIL'
systemctl --user show-environment | grep -c 'VKD3D_CONFIG\|PROTON_FSR4_UPGRADE\|PROTON_ENABLE_WAYLAND'
grep -c '^[A-Z]' ~/.config/environment.d/10-environment.conf   # 9 variables
grep -c '^cpu_power' ~/.config/MangoHud/MangoHud.conf     # 1
grep -c '^cpu_temp' ~/.config/MangoHud/MangoHud.conf      # 0 — deliberate (#1794)
grep -c '^[a-z]' ~/.config/MangoHud/MangoHud.conf         # 19 active directives, 18 after T1-1
```

The `PROTON_ENABLE_WAYLAND` check above expects **0**. A non-zero result on a host that has
deployed 7.156.0 or later means a stale login session still carries the variable — the
environment file no longer sets it, but `systemd --user` does not retract what it already
imported. Log out rather than filing it as drift.

**`grep -c` exits 1 with no output on zero matches.** Any zero-hit assertion above must be
read as "exit 1 or `0`", not as a failure. `rg` is equivalent for these but sandbox rg 14.1.0
carries a false-negative bug — re-confirm zero-hit results with `grep` or `python3`.

---

## 7. Reference data

All values live-evaluated from the 7.158.0 script.

### 7a. Count oracle — 21 tripwires, asserted by `_ir_validate_counts` (L664)

```
║ KERNEL_PARAMS            15 ║ PKGS_ADD                 16 ║ _RY_BOOT_CRITICAL_DSTS  4 ║
║ MKINITCPIO_HOOKS         11 ║ PKGS_DEL                  9 ║ _RY_PHASE_NAMES         6 ║
║ MKINITCPIO_MODULES        1 ║ MASK                     11 ║ _RY_BACKUP_TARGETS      4 ║
║ MKINITCPIO_COMPRESSION_   1 ║ EXPECTED_VULKAN_PKGS      2 ║ _RY_TMPDIR_GLOBS        6 ║
║   OPTIONS  (2 pre-7.140)    ║ EXPECTED_SERVICES         5 ║ SYSTEM_DESTINATIONS    15 ║
║ LOGIND_IGNORE_KEYS        8 ║ _RY_PKG_MANAGED_SERVICES  1 ║ USER_DESTINATIONS       2 ║
║ ENV_VARS       9 (was 10)   ║ _RY_POST_HOOKS           17 ║ _RY_ARGPARSE_SPEC       6 ║
║ SYSCTL_VALUES  9 (was 11)   ║                             ║                           ║
```

**Two values moved this rebase**, the first oracle movement since the 7.140.0 `-T0` cut:
`ENV_VARS` 10 → 9 (`PROTON_ENABLE_WAYLAND` removed after 7.155.0) and `SYSCTL_VALUES` 11 → 9
(both `net.core.netdev_budget` keys removed at 7.157.0). `_RY_EPP_LEVELS` (5) and
`_RY_DPM_LEVELS` (9) are deliberately **not** in the oracle — both are value-bearing, not
count invariants. `RESOLVED_DNS_SERVERS` was never in the oracle either and was **removed
outright at 7.147.0**; a brief or script that still expects it is pre-7.147.0. Managed files =
17 (15 system + 2 user), recomputed at load; a mismatch refuses with exit 3.
**Count these by live fish eval, never by text parsing.**

**The oracle is a weak detector of content change** and this rebase proves it twice over
(§1a). Byte anchors (§7d) catch some of it; only the embedded bodies (§7e) catch all of it.

### 7b. Perf scalars and their maxima

```
║ GLOBAL                  ║ L   ║ VALUE          ║ CEILING                          ║
║─────────────────────────║─────║────────────────║──────────────────────────────────║
║ CPUPOWER_GOVERNOR       ║ 585 ║ performance    ║ AT MAX (regex ^[a-z][a-z0-9_-]*$)║
║ GPU_DPM_LEVEL           ║ 587 ║ high           ║ CLOSED at high (T5), not max     ║
║ EPP_PREFERENCE          ║ 589 ║ performance    ║ AT MAX (_RY_EPP_LEVELS, 5)       ║
║ EXPECTED_SCALING_DRIVER ║ 590 ║ amd-pstate-epp ║ NO MAX — verify-only, tunes      ║
║                         ║     ║                ║ NOTHING; follows a cmdline       ║
║                         ║     ║                ║ amd_pstate= change, never leads  ║
║ BLACKLIST_AMDXDNA       ║ 591 ║ true           ║ boolean; false REQUIRES          ║
║                         ║     ║                ║ amd_iommu=on iommu=pt            ║
```

**No line moved from the previous edition.** `_RY_EPP_LEVELS` (L589, same line as
`EPP_PREFERENCE`): `default performance balance_performance balance_power power`.
`_RY_DPM_LEVELS` (L588, 9 members): `auto low high manual profile_standard profile_min_sclk
profile_min_mclk profile_peak perf_determinism`. Changing `EXPECTED_SCALING_DRIVER` makes the
verifier assert a driver the kernel never loaded, producing a false `_chk_eq` failure on
every `--verify`. Domain validation lives in `_ir_validate_keys` (L692). Re-probed for this
edition, each in its own subprocess: shipped values rc 0; a bogus DPM level, a bogus EPP, a
governor failing the regex, and `BLACKLIST_AMDXDNA=false` under `amd_iommu=off` all exit
**3** (`EXIT_PREFLIGHT`). `_err_loud` **exits** rather than returns.

**A full perf-value change touches 11 sites. Enumerate all of them in any TUNE.**

```
║ #  ║ FILE      ║ LINE ║ CARRIES                                              ║
║────║───────────║──────║──────────────────────────────────────────────────────║
║ 1  ║ script    ║  585 ║ set -g CPUPOWER_GOVERNOR performance                 ║
║ 2  ║ script    ║  587 ║ set -g GPU_DPM_LEVEL high + trailing comment         ║
║ 3  ║ script    ║  589 ║ set -g EPP_PREFERENCE performance (+ _RY_EPP_LEVELS) ║
║ 4  ║ script    ║  865 ║ "# AMD P-State EPP performance (maximum CPPC hint)"  ║
║ 5  ║ script    ║  867 ║ "# GPU performance level (gfx1151 clock-floor;       ║
║    ║           ║      ║ forced high)"                                        ║
║ 6  ║ README    ║   96 ║ managed-files row: governor (`performance`)          ║
║ 7  ║ README    ║   98 ║ managed-files row: NVMe none, P-State EPP, DPM `high`║
║ 8  ║ README    ║  201 ║ Service Keys row: CPUPOWER_GOVERNOR | performance    ║
║ 9  ║ README    ║  205 ║ Service Keys row: GPU_DPM_LEVEL | high               ║
║ 10 ║ README    ║  206 ║ Service Keys row: EPP_PREFERENCE | performance       ║
║ 11 ║ CHANGELOG ║   7+ ║ new entry inserted after the preamble                ║
```

**Two NON-sites, recorded so they are not re-added and not silently lost:**

```
║ WHAT                        ║ WHERE      ║ WHY IT IS NOT AN EDIT SITE          ║
║─────────────────────────────║────────────║─────────────────────────────────────║
║ udev generator function     ║ script 860 ║ --description reads "Generate       ║
║ header                      ║            ║ content for combined udev perf      ║
║                             ║            ║ rules" — no perf value in it. Keep  ║
║                             ║            ║ as a LOCATE anchor only             ║
║ README CPU/GPU prose        ║ gone       ║ restructured into Gaming Stack and  ║
║ paragraph                   ║            ║ Kernel Parameter Notes bullets at   ║
║                             ║            ║ 7.151.0; neither restates governor, ║
║                             ║            ║ EPP or DPM. Do not re-add — Tuning  ║
║                             ║            ║ Notes must not restate values       ║
```

**Two sites follow automatically and need no edit:** script L866 and L868 interpolate
`$EPP_PREFERENCE` and `$GPU_DPM_LEVEL` into the udev rules, and `_vss_udev` (L2258) greps for
the interpolated values, so both track any change. Verify by re-rendering the udev body, not
by reading the lines.

**Grep traps:** `powersave` has 4 hits but only the governor global is in scope — the rest are
`NM_WIFI_POWERSAVE` plumbing. `balance_performance` has exactly 1 hit, the surviving
`_RY_EPP_LEVELS` member. Do not edit the phrase "clock-floor" — it is accurate under `high`
and consistent across the generator and verify paths. `grep -ci 'audit\|spec'` reports false
hits from `_RY_ARGPARSE_SPEC` and from "inspect"/"inspection"; real audit refs are **0** in
all shipped files.

**Measured deltas from the last perf change** (7.128.0, powersave/auto/balance_performance →
max): udev 657→639 (−18), cpupower 113→115 (+2); the split was value substitution −6 and
comment edits −10. A spec revision predicting −6 by omitting the comment edits was wrong.
**The 17-file total is now 4,858 B** — a re-apply must predict against that figure, not 4,949
or 5,093, and the udev body itself is unchanged at 639 B. **On a re-apply, use verbatim
strings — never paraphrase edit wording from memory; the acceptance test is reproducing the
predicted SHAs, not a passing functional battery.**

### 7c. Configured values

**KERNEL_PARAMS (15, sorted as emitted, L574):** `amd_iommu=off` `amd_pstate=active`
`btusb.enable_autosuspend=n` `clearcpuid=umip` **`fsck.mode=auto`** `fsck.repair=yes`
`ipv6.disable=1` `mt7925e.disable_aspm=1` `nvme_core.default_ps_max_latency_us=0`
`pcie_aspm.policy=performance` `processor.max_cstate=1` `quiet` `split_lock_detect=off`
`usbcore.autosuspend=-1` `zswap.enabled=0`. **`fsck.mode` changed value at 7.158.0
(`force` → `auto`); membership and the count of 15 did not change.**

Documentation status in mainline `kernel-parameters.txt` — a token being absent is not a
defect, but it changes which source to cite:

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

**ENV_VARS (9, L594, `~/.config/environment.d/10-environment.conf`, 0600):**
`DXVK_LOG_LEVEL=none` `FSR4_WATERMARK=1` `MANGOHUD=1` `MESA_SHADER_CACHE_MAX_SIZE=16G`
`POWERDEVIL_NO_DDCUTIL=1` `PROTON_LOCAL_SHADER_CACHE=1` `VKD3D_DEBUG=none`
`VKD3D_SHADER_DEBUG=none` `WINEDEBUG=-all`. **`PROTON_ENABLE_WAYLAND=1` was removed after
7.155.0**, taking the array 10 → 9 and the body 306 → 282 B; the exact release is not
recoverable from the shipped artifact (§1c). No drirc, no ttm/amdgpu module params, no ICD
pin. `_vre_envvars` (L3005) iterates the array dynamically, so the verifier follows any
`ENV_VARS` edit with no verifier change — which is why removing a variable needed no verify
edit and left no trace in the verify surface.

**SYSCTL_VALUES (9, L596, `/etc/sysctl.d/95-ry-overrides.conf`, priority 95):**
`kernel.nmi_watchdog=0` `net.core.default_qdisc=fq` `net.ipv4.tcp_congestion_control=bbr`
`net.ipv4.tcp_notsent_lowat=16384` `net.ipv4.tcp_slow_start_after_idle=0`
`vm.compaction_proactiveness=0` `vm.max_map_count=2147483642` `vm.swappiness=150`
`vm.watermark_boost_factor=0`. Stored `k=v`, emitted `k = v` — any parity check must
normalise whitespace. **Both `net.core.netdev_budget` keys were removed at 7.157.0** after
T0-5 measured squeezed at 0; an absent key leaves the kernel default (300 / 2000). Do not
re-propose them. **Standing correction, do not let it return:** the generator (L848) emits a
99-character header comment and then exactly one `key = value` line per entry. It does not
annotate individual keys and never did. **Known non-defect:** on a non-initial network
namespace the `net.core.*` and `net.ipv4.*` keys fail with ENOENT because those tables are
init_net only — a container signature, not a host fault.

**MASK (11, L609):** `ananicy-cpp.service` `power-profiles-daemon.service`
`NetworkManager-wait-online.service` `avahi-daemon.service` `avahi-daemon.socket`
`ufw.service` `sleep.target` `suspend.target` `hibernate.target` `hybrid-sleep.target`
`suspend-then-hibernate.target`.

**EXPECTED_SERVICES (5):** `fstrim.timer` `NetworkManager.service` `cpupower.service`
`nftables.service` `bluetooth.service`. **`_RY_PKG_MANAGED_SERVICES` (1):**
`NetworkManager.service` — it is in both sets; strip it from `EXPECTED_SERVICES` first when
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
modconf kms keyboard sd-vconsole block filesystems fsck — **byte-identical to Arch mkinitcpio
41's shipped default**; COMPRESSION zstd, **`COMPRESSION_OPTIONS=(-3)`** since 7.158.0
(L577); explicit `BINARIES=()` / `FILES=()`. `_vmh_order_checks` enforces **eleven** HOOKS
constraints; a systemd-after-sd-vconsole swap returns **2**, not 1, because that permutation
breaks two ordered pairs.

**Sixteen service keys** are documented in the profile README — a direct line-by-line count of
the script's SERVICE KEYS block matches the README's 16-row table exactly, in the same order.
Only two reach their destination under their own name (`COUNTRY`; `LOGIND_IGNORE_KEYS` → 8
`Handle*Key=ignore` lines). The rest are **renamed on the way out** —
`BT_AUTO_ENABLE`→`AutoEnable`, `EPP_PREFERENCE`→udev
`ATTR{cpufreq/energy_performance_preference}`,
`GPU_DPM_LEVEL`→`ATTR{device/power_dpm_force_performance_level}`,
`CPUPOWER_GOVERNOR`→`GOVERNOR=`, `RESOLVED_MDNS`→`MulticastDNS=`, `RESOLVED_LLMNR`→`LLMNR=`
and so on — which is why grepping a generated file for the script's variable name returns
nothing. `EXPECTED_SCALING_DRIVER` writes nothing at all. **The sdboot generator emits
`REMOVE_EXISTING=` before `OVERWRITE_EXISTING=` while the README table follows declaration
order; that is deliberate — never "fix" the table to emission order.**

### 7d. Generated-file byte anchors

```
║ FILE                                             ║ 7.155 ║ 7.158 ║ DELTA ║
║──────────────────────────────────────────────────║───────║───────║───────║
║ /boot/loader/loader.conf                         ║    89 ║    89 ║       ║
║ /etc/kernel/cmdline                              ║   352 ║   351 ║    -1 ║
║ /etc/sdboot-manage.conf                          ║   544 ║   543 ║    -1 ║
║ /etc/mkinitcpio.conf                             ║   276 ║   276 ║    0* ║
║ /etc/systemd/resolved.conf.d/99-cachyos-resolved ║    90 ║    90 ║       ║
║ /etc/systemd/logind.conf.d/99-cachyos-logind     ║   292 ║   292 ║       ║
║ NetworkManager-dispatcher.service.d/logging.conf ║   127 ║   127 ║       ║
║ /etc/NetworkManager/conf.d/99-cachyos-nm.conf    ║   148 ║   148 ║       ║
║ /etc/iw-regdomain                                ║    88 ║    88 ║       ║
║ /etc/bluetooth/main.conf                         ║   147 ║   147 ║       ║
║ /etc/nftables.conf                               ║   729 ║   729 ║       ║
║ /etc/default/cpupower-service.conf               ║   115 ║   115 ║       ║
║ /etc/sysctl.d/95-ry-overrides.conf               ║   441 ║   376 ║   -65 ║
║ /etc/udev/rules.d/99-ry-perf.rules               ║   639 ║   639 ║       ║
║ /etc/modprobe.d/60-ry-modules.conf               ║   183 ║   183 ║       ║
║ ~/.config/environment.d/10-environment.conf      ║   306 ║   282 ║   -24 ║
║ ~/.config/MangoHud/MangoHud.conf                 ║   383 ║   383 ║       ║
║──────────────────────────────────────────────────║───────║───────║───────║
║ TOTAL (17 managed files)                         ║ 4,949 ║ 4,858 ║   -91 ║
```

`0*` marks `/etc/mkinitcpio.conf`: content changed, byte count did not. **A byte-anchor
table cannot represent that row and this is the reason §7e exists.**

The udev rule at **639 B** and the **4,858 B** total are the anchors for any perf-value
change — a value substitution plus its comment edits moves both. None of the five deltas
above is perf: the udev, cpupower and (post-removal) sysctl bodies that carry every tuned
value are otherwise byte-identical to the previous edition. **Measure as written files**
(`$fn > tmp; stat -c %s`), never `string collect` — see §9 for the phantom deficit it
produces.

### 7e. Generated bodies — all seventeen, byte-exact

Rendered from the 7.158.0 generators under the §9 harness, 3/3 deterministic
(sorted-manifest sha `6bf7f8ea53c36c40`). All seventeen were re-rendered for this rebase and
diffed against the previous edition's fences; **five came back different and are marked
CHANGED below**, one of them at a byte delta of zero. Reproduce before diffing; do not diff
against a prior edition's fences without re-rendering — that diff is what caught all five.

**`/boot/loader/loader.conf`** (89 B):

```
# systemd-boot loader configuration
default @saved
timeout 0
console-mode keep
editor no
```

**`/etc/kernel/cmdline`** (351 B) — **CHANGED, was 352 B. fsck.mode=force -> auto at 7.158.0.
The UUID below is a 36-char stub — a real render is the same length**:

```
rw root=UUID=12345678-1234-1234-1234-123456789abc amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=auto fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0
```

**`/etc/sdboot-manage.conf`** (543 B) — **CHANGED, was 544 B. Same fsck.mode token, inside
LINUX_OPTIONS**:

```
# sdboot-manage configuration — changes require: sudo sdboot-manage gen && sudo sdboot-manage update
LINUX_OPTIONS="amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=auto fsck.repair=yes ipv6.disable=1 mt7925e.disable_aspm=1 nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off usbcore.autosuspend=-1 zswap.enabled=0"
LINUX_FALLBACK_OPTIONS="quiet"
DEFAULT_ENTRY="manual"
REMOVE_EXISTING="yes"
OVERWRITE_EXISTING="yes"
REMOVE_OBSOLETE="yes"
```

**`/etc/mkinitcpio.conf`** (276 B) — **CHANGED at ZERO byte delta. COMPRESSION_OPTIONS (-1) ->
(-3) at 7.158.0: same array length, same token width, same 276 B. No count check and no byte
check can see this — only this fence**:

```
# mkinitcpio configuration — changes require: sudo mkinitcpio -P && sudo sdboot-manage update
MODULES=(amdgpu)
BINARIES=()
FILES=()
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
COMPRESSION="zstd"
COMPRESSION_OPTIONS=(-3)
```

**`/etc/systemd/resolved.conf.d/99-cachyos-resolved.conf`** (90 B):

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

**`/etc/NetworkManager/conf.d/99-cachyos-nm.conf`** (148 B):

```
# NetworkManager configuration — wpa_supplicant backend
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

**`/etc/nftables.conf`** (729 B):

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

**`/etc/default/cpupower-service.conf`** (115 B):

```
# cpupower-service.conf — sourced by /usr/lib/systemd/scripts/cpupower (cpupower.service)
GOVERNOR='performance'
```

**`/etc/sysctl.d/95-ry-overrides.conf`** (376 B) — **CHANGED, was 441 B. Both
net.core.netdev_budget keys removed at 7.157.0 after T0-5 measured squeezed at 0. Nine keys,
one header comment, no per-key annotation**:

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

**`/etc/udev/rules.d/99-ry-perf.rules`** (639 B):

```
# ry-install: udev performance rules (managed file, do not edit by hand)
# NVMe scheduler none (lowest tail latency; diverges from CachyOS kyber default)
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
# AMD P-State EPP performance (maximum CPPC hint)
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="performance"
# GPU performance level (gfx1151 clock-floor; forced high)
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="high"
```

**`/etc/modprobe.d/60-ry-modules.conf`** (183 B):

```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# blacklist amdxdna: XDNA NPU needs IOMMU, probes -ENODEV (ret -19) under amd_iommu=off
blacklist amdxdna
```

**`~/.config/environment.d/10-environment.conf`** (282 B, mode 0600) — **CHANGED, was 306 B.
PROTON_ENABLE_WAYLAND=1 removed after 7.155.0, taking ENV_VARS 10 -> 9**:

```
# Environment for systemd --user services and graphical sessions (Plasma, Flatpak, D-Bus apps)
DXVK_LOG_LEVEL=none
FSR4_WATERMARK=1
MANGOHUD=1
MESA_SHADER_CACHE_MAX_SIZE=16G
POWERDEVIL_NO_DDCUTIL=1
PROTON_LOCAL_SHADER_CACHE=1
VKD3D_DEBUG=none
VKD3D_SHADER_DEBUG=none
WINEDEBUG=-all
```

**`~/.config/MangoHud/MangoHud.conf`** (383 B, mode 0600, 19 active directives):

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
║      ║ drop-in                      ║                    ║ SDBOOT_DROPIN_PRESENT║
║ 3    ║ package dropped from         ║ stays installed    ║ NOT DETECTED —       ║
║      ║ PKGS_ADD                     ║ forever            ║ by design (T5)       ║
║ 3    ║ env var dropped from         ║ stays in a LIVE    ║ NOT DETECTED — the   ║
║      ║ ENV_VARS                     ║ user session until ║ file self-heals; the ║
║      ║                              ║ next login         ║ session does not     ║
```

Tier 2 is reportable but never self-healing and never sets DRIFT — the operator must act by
hand. Tier 3 is invisible. **The second Tier-3 row is new and this rebase produced it:**
removing `PROTON_ENABLE_WAYLAND` rewrote `10-environment.conf` correctly, but
`systemd --user` does not retract a variable it has already imported, so a running session
keeps it until logout. Nothing detects that and nothing should — but a "why is it still set"
report is a session-lifetime question, not drift.

### 7g. Gates, exits, backups

- **Preflight validator chain, four deep, called from `_init_runtime` (L740):**
  `_ir_validate_counts` (L664, 21 tripwires) → `_ir_validate_keys` (L692, domains and
  charsets, incl. the nftables↔`ipv6.disable` coupling and the
  `BLACKLIST_AMDXDNA=false`↔IOMMU-on coupling) → `_ir_validate_sets` (L727) →
  `_ir_validate_post_hooks` (L4524). All four re-probed at 7.158.0: rc 0 on shipped values.
  **There is no `_ry_validate_keys`, `_ry_validate_counts` or `_ry_validate_post_hooks`** —
  those names return rc 127, which looks like a signal and is only an unknown command. Real
  sibling validators: `_ry_validate_mkinitcpio_hooks`, `_ry_validate_mkinitcpio_modules`,
  `_rvc_dispatch`, `_ry_validate_configs`.
- **`_ir_validate_sets` refuses three intersections**, each `_err_loud` + exit 3:
  `PKGS_ADD ∩ PKGS_DEL` (phase 2 installs what phase 4 removes), `EXPECTED_SERVICES ∩ MASK`
  (phase 4 masks before it enables), `_RY_PKG_MANAGED_SERVICES ∩ MASK` (a masked unit cannot
  be package-managed). All three shipped intersections are empty; the third is normally
  shadowed by the second, which is a forward guard rather than a defect. A fourth
  (`EXPECTED_VULKAN_PKGS ∩ PKGS_DEL`) is deliberately uncovered and currently empty. **Any
  recommendation that adds a package or unit must be checked against all three — a
  contradiction is a hard deploy refusal, not a silent misbehaviour.**
- **Phases (6, `_RY_PHASE_NAMES`):** Preflight → Packages → Configuration →
  Services (fstab → resolved → pkg-remove → mask → enable → regdom) → Boot → Finalize,
  mirrored 1:1 across the array, `_progress`, the orchestrator, the log headers and the
  README. `_PROG_STEPS` is derived from the array, so phase order cannot drift.
- **Hardware gate:** CPU model match on `EXPECTED_CPU_MATCH` = `Ryzen AI Max` (L613); sole
  override `RY_INSTALL_SKIP_HARDWARE_CHECK=1` (`SKIP_HW=1` does nothing); fail-closed on an
  unreadable model; `--verify` warns, deploy and `--check` exit 3.
- **Runtime env inputs are exactly three:** `RY_RUN_TIMEOUT`, `RY_INSTALL_SKIP_HARDWARE_CHECK`,
  `NO_COLOR` (plus multiplexer detection). Every profile toggle is an embedded scalar set
  unconditionally with `set -g`, so an exported env var of the same name is clobbered —
  opting in means editing the script. `NO_COLOR` needs a **non-empty** value to fire;
  `TERM=dumb` disables colour independently. `RY_RUN_TIMEOUT=0` disables the timeout; package
  and boot operations floor at 7200 s (`_RY_LONGOP_HARD_CAP`).
- **Dependencies:** 37 hard-required commands (missing any → rc 1) at L1535 + **15**
  warn-only optional tools at L1544, plus a `df --output` probe and systemd ≥ 250. Optional
  list: bootctl journalctl modinfo pgrep zcat tput lsmod modprobe pkill nmcli ping realpath ip
  lspci kill. Profile-installed tools (`nft`, `pactree`, `paccache`, `sysctl`, `udevadm`,
  `bluetoothctl`, `timedatectl`, `ufw`, `iw`) are call-site `command -q` guarded and
  deliberately undeclared — do not "fix" the asymmetry. **INFO, open (consistency only):
  `readlink` is the only invoked external with neither a dep-list entry nor a `command -q`
  guard.** It is coreutils and effectively unreachable; record it, do not escalate it.
- **Exit model, 14 constants:** 0 OK · 1 FAIL · 2 USAGE · 3 PREFLIGHT · 4 BOOT_CRIT ·
  5 LOCK · 10 DRIFT; internal sentinels 11 GEN_NOFN, 12 GEN_NOUUID, 13 GEN_SYSCTL,
  14 GEN_ENVD, 250 AS_MISUSE, 251 RUN_TMPFAIL, 255 RUN_MISUSE — function returns only, never
  a process exit. Signals map to 128+N. **No orphan class reaches EXIT_DRIFT.** Root
  `--check` → rc 3, silent, no JSONL (the root guard precedes LOG_DIR init); all other root
  modes → rc 2, loud, 92 B. Lock dir `$HOME/ry-install/.lock` with a `pid` file inside;
  `--verify` and `--check` are lock-free. The systemd 250 floor is a hard refusal at L1541 and
  the fish 3.6 floor at L156 — both preflight, both loud.
- **The expected live `--verify` OK count is UNKNOWN at 7.158.0.** The previous edition's
  **278** was empirical from a clean-host run and is now stale: the verify surface lost two
  asserted sysctl keys and one asserted environment variable. Re-baseline on the next clean
  run. It was never a literal in the script and must never be cited as an invariant.
- **JSONL logs land under `$HOME/ry-install/logs/<YYYY-MM-DD>/<mode>-<timestamp>-<pid>.jsonl`**,
  not in the profile root. A glob that misses the `logs/<date>/` level returns zero hits
  silently, which reads as "the key was never emitted".
- **Argument parsing (`_RY_ARGPARSE_SPEC`, 6 entries):**
  `--exclusive=verify,check,install-file` `h/help` `v/version` `verify` `check`
  `install-file=`. No positional arguments; `--` ends option parsing and anything after it
  exits 2. Glued short flags early-intercept with first char winning (`-hv` → help, `-vh` →
  version); `-h`/`-v` are honored before the root guard. `--install` is **not** a flag —
  install is the default no-arg mode, and fish's wgetopt abbreviation resolves `--install` to
  `--install-file` while rejecting `--ver` as ambiguous. That is documented fish default
  behaviour; `-S` would break the 3.6 floor.
- **Backups are two different guarantees.** The 4 boot-critical destinations
  (`/boot/loader/loader.conf`, `/etc/kernel/cmdline`, `/etc/sdboot-manage.conf`,
  `/etc/mkinitcpio.conf` — `_RY_BACKUP_TARGETS` and `_RY_BOOT_CRITICAL_DSTS` are the same
  four) get `.ry.bak` **plus post-write re-read and restore**. The other 13 get a one-time
  `.ry.orig` on **first adoption only** — captured once, then every subsequent overwrite is
  silent by design. `/etc/fstab` is not a managed destination and has always had its own
  `.ry.bak` during rewrite. T0-7 confirmed the shape live: 4 `.ry.bak`, 2 `.ry.orig`, no
  user-scope `.ry.orig`. **Do not describe the 13 as backed up on every write, and do not
  recommend `.ry.orig` for the boot four — it is weaker than what they have.** No verify path
  asserts `.ry.orig` exists and none can: absence is indistinguishable from "the destination
  had no pre-existing content", which is the common case and is what this host shows.

---

## 8. Verify-surface ownership

**62 verify functions = 12 `_verify_*` + 50 subs** (`_vsb` 7 · `_vss` 10 · `_vsp` 3 ·
`_vsc` 1 · `_vrk` 4 · `_vrkm` 3 · `_vrsv` 10 · `_vre` 5 · `_vrs` 4 · `_vpd` 1 · `_vmh` 2).
The census is **identical prefix-by-prefix to 7.155.0 and 7.141.0** — no verify function was
added, removed or renamed across three rebases. **Every line number below is also unchanged
from the previous edition**, because the script held its line count. A recommendation that
changes a value must state which **sub** asserts it, and whether that sub hard-fails or warns.

```
║ ORCHESTRATOR             ║ SUBS THAT MATTER FOR TUNING FINDINGS             ║
║──────────────────────────║──────────────────────────────────────────────────║
║ _verify_static_boot      ║ _vsb_loader (2079) · _vsb_sdboot (2084) ·        ║
║ (2228)                   ║ _vsb_sdboot_dropins (2116; WARN) ·               ║
║                          ║ _vsb_cmdline (2130, all 15 tokens + root= + rw) ·║
║                          ║ _vsb_mkinitcpio (2157; live COMPRESSION/_OPTIONS,║
║                          ║ multi-line join, last-wins) · _vsb_entries (2201)║
║                          ║ -> _vsb_entry_options (2184)                     ║
║ _verify_static_system    ║ _vss_logind (2231) · _vss_nmdispatch (2237) ·    ║
║ (2280)                   ║ _vss_nm (2238) · _vss_sysctl (2244, NOW 9 keys) ·║
║                          ║ _vss_regdom (2250) · _vss_bluetooth (2251) ·     ║
║                          ║ _vss_udev (2258; 3 rules, EPP + DPM aware) ·     ║
║                          ║ _vss_modprobe (2271; blacklist + stale-dropin    ║
║                          ║ sweep, WARN) · _vss_nft (2266; HARD-FAILS on a   ║
║                          ║ missing echo-request accept)                     ║
║ _verify_static_user      ║ inline: ENV_VARS (NOW 9) + MangoHud directives   ║
║ (2303)                   ║                                                  ║
║ _verify_static_packages  ║ _vsp_required (2315; PKGS_ADD 16 + Vulkan 2) ·   ║
║ (2380)                   ║ _vsp_removed (2338) · _vsp_pacman_conf (2348)    ║
║ _verify_static_services  ║ MASK 11 inline + _vss_orphan_masks (2414; INFO,  ║
║ (2390)                   ║ admin-scope /etc + /run only)                    ║
║ _verify_static_syntax    ║ live mkinitcpio HOOKS presence, multi-line       ║
║ (2421)                   ║ tolerated                                        ║
║ _verify_static_checksum  ║ _vsc_check_one (2435; expected vs installed      ║
║ (2464)                   ║ SHA256 per destination)                          ║
║ _verify_runtime_kparams  ║ _vrk_cmdline (2638; every token + rw) ·          ║
║ (2806)                   ║ _vrk_gpu_state (2657; QUOTED compare vs          ║
║                          ║ $GPU_DPM_LEVEL) · _vrk_cpu_state (2679; cpu0     ║
║                          ║ detail + FULL cpufreq-policy uniformity sweep) · ║
║                          ║ _vrk_module_state (2781) -> _vrkm_amdgpu (2724), ║
║                          ║ _vrkm_blacklist (2744), _vrkm_blacklist_modprobe ║
║                          ║ (2760; amdxdna LOADED = FAIL)                    ║
║ _verify_runtime_services ║ _vrsv_sys_units (2889) -> _vrsv_chk_active_      ║
║ (3002)                   ║ enabled (2809), _vrsv_chk_nftables (2832; judged ║
║                          ║ by live policy-drop, not unit state),            ║
║                          ║ _vrsv_chk_resolved (2862; enabled|static),       ║
║                          ║ _vrsv_chk_cpupower_governor (2874) ·             ║
║                          ║ _vrsv_nft_assert_ping (2824; WARN) ·             ║
║                          ║ _vrsv_wifi (2927) -> _vrsv_wifi_nm_backend (2906)║
║                          ║ · _vrsv_masked_inactive (2967; iterates $MASK) · ║
║                          ║ _vrsv_user_units (2985)                          ║
║ _verify_runtime_env      ║ _vre_envvars (3005; dynamic over $ENV_VARS) ·    ║
║ (3111)                   ║ _vre_sysctl_runtime (3023; NOW 9, absent-vs-     ║
║                          ║ unreadable split, WARNs on an absent knob) ·     ║
║                          ║ _vre_fstab (3041; every ext4 entry) ·            ║
║                          ║ _vre_ntsync (3072) · _vre_regdom (3094)          ║
║ _verify_runtime_session  ║ _vrs_nm_perms (3114) · _vrs_installed_file_perms ║
║ (3221)                   ║ (3140) -> _vrs_vfat_skip (3129) ·                ║
║                          ║ _vrs_parent_dirs (3187) -> _vpd_dir_perm_check   ║
║                          ║ (3168)                                           ║
║ _verify_summary (1168)   ║ pass/fail/warn tally — not an assertion path     ║
```

**Three assertion counts fell without a single verify-function edit.** `_vss_sysctl` and
`_vre_sysctl_runtime` went 11 keys → 9 and `_verify_static_user` / `_vre_envvars` went 10
variables → 9, purely because every one of them iterates the profile array rather than
carrying a literal list. That is the design working — and it is also why the empirical
`--verify` OK count moved and why no diff of the verify functions would have shown it.

**`_vre_fstab` is the reason the root-filesystem question was answerable at all, and also the
reason it went unanswered for so long.** It enumerates every ext4 entry with a per-entry
`_fail`, which produces a clean log full of ext4 checks on a host whose root might be
anything. `findmnt /` is the only settling read; T0-3 ran it and the answer was ext4.

**`_vss_nm` asserts no DNS content.** Its managed file lost the `[global-dns]` and
`[global-dns-domain-*]` blocks at 7.147.0, so what it checks now is the wifi backend,
powersave and log level only. Nothing in the verify surface asserts which resolver the host
actually uses — that is T0-8, and it is a read, not an assert.

**Attribution traps:**

- `_vrkm_blacklist_modprobe` is **generator-sourced** — it checks intended content, not
  on-disk extras. Attribute the drop-in sweep to `_vss_modprobe` / `_ry_stale_ry_dropins`
  (L2559), never to it.
- `_vrsv_masked_inactive` asserts every *declared* mask is present and inactive. The reverse
  direction (masked units not declared) is `_vss_orphan_masks`, which lives on the **static**
  services path. Do not conflate the two.
- `_vsb_entry_options` is a sub of **`_vsb_entries`**, not of `_verify_static_boot`. It skips
  `*-fallback.conf` by design, which is why T4-1 is invisible to verify.
- `_vss_nft` hard-fails on a missing inbound echo-request accept; `_vrsv_nft_assert_ping` only
  warns. A recommendation to drop inbound ping must address the hard-fail.
- `_ry_orphan_masked_units` (L2563) filters to **admin scope only** (`/etc`, `/run`) — vendor
  `/usr/lib` masks and `Alias=` cascades are deliberately dropped. `Alias=` matters here:
  masking `avahi-daemon.service` makes its upstream alias read masked too.
- **Removed asserts — do NOT verify:** `_vrkm_iommu`, `_vrk_clocksource`, `_vre_zram`,
  `_vre_tcp` (all gone since 7.90.0); kernel-floor and Mesa-floor checks; the preemption
  advisory (7.139.0 r2). No THP, KSM, `ttm.*`, drirc, `iommu=pt`, ICMPv6/NDP or baloo assert
  exists. No `VKD3D_CONFIG`, `PROTON_ENABLE_WAYLAND` or `netdev_budget` assert exists — all
  three left their arrays and the dynamic verifiers followed automatically.
- **Sandbox artifact, not a regression:** `_ry_validate_mkinitcpio_hooks` returns rc 1 and
  `_ry_validate_configs` returns rc 3 in a container because `/etc/mkinitcpio.conf` is absent;
  `_vmh_order_checks` must be called directly to test HOOKS ordering. Always A/B a nonzero
  validator rc against the previous release.

**Output-channel invariant — reconciled; do not re-file as drift.** Every leveled user-facing
message funnels through `_msg_print` (**L1087**), honouring QUIET / `_RY_OUTPUT_BROKEN` /
`_RY_NO_COLOR` / isatty(2). Raw `>&2` counts of ~78 (whole-file) and ~43 (inside function
bodies) are **both correct under their own scoping**; the remainder are top-level pre-init
preflight writes made before `_msg_print` is defined. The invariant means "single authority
for leveled user-facing output", not "sole writer to fd 2" — the latter reading generates a
false finding every time. stdout carries only `--help` and `--version`.

---

## 9. Reproduction method and its traps

Recorded so the next rebase does not re-pay these costs.

- **Harness.** Cut the script just before the `# ── MAIN: ARGPARSE` banner — **L4786 at both
  7.155.0 and 7.158.0**, L4861 at 7.141.0, L4845 at 7.139.0, L4834 at 7.135.1. **This is the
  first rebase where the anchor did not move; that is a fact about 7.158.0, not a licence to
  hardcode it.** Always locate the banner. Then delete the L3 source guard and `source` the
  result as a non-root user with a **writable `$HOME`**:
  `sed -n '1,4785p' ry-install.fish | sed '3d' > harness.fish`. Without the L3 deletion the
  guard fires on `source` and every count silently reads 0. Without a writable `$HOME` the
  log-directory init aborts the source part-way and counts read 0 the same way — a different
  cause with an identical symptom. Pre-create `$HOME/.config/fish` and
  `$HOME/.local/share/fish`, and make the worktree **and every parent directory** `a+rX`
  (`/home/claude` is 0700, and a source failure there looks like an empty function list).
- **Shadow `exit` before sourcing** (`function exit; end`, then `functions -e exit` afterwards)
  so the fallen-through top level runs to completion; oracle counts, scalars and all 17
  generators then probe in ONE shell. TRAP: that flow calls `_ry_erase_handlers`, which
  `functions -e`'s `_cleanup*` and `_progress_on_winch`, so the live function table
  UNDER-COUNTS handler families — take the census from `^function ` at column 0 in source
  (**294** at 7.141.0, 7.155.0 and 7.158.0), not from a before/after `functions --all --names`
  diff. A second count trap: `grep -oP '^function \K[A-Za-z0-9_]+'` reports **293**, because
  it truncates `_content_*` names at their first dot. Use `[^ ;]+`.
- **`test -w /tmp` bails rc 3 before anything else.** Any sandbox reproduction needs a non-root
  user *and* a writable `/tmp` (`chmod 1777 /tmp`), with probe scripts world-readable.
- **Array counts by live fish eval, never text parsing.** `eval echo \$$name` collapses a fish
  array and reports every count as 1 — use `eval "set vals \$$name"` then `count $vals`.
  Continuation regexes truncate multi-line declarations (`SYSCTL_VALUES` at L596 is one),
  `set -g --` evades awk, and several service keys share one `set -g` line, which breaks any
  line-anchored `^set -g NAME` scalar extraction — use a non-anchored `finditer`, de-duplicate
  preserving order, and strip trailing `;`.
- **Filter generators to `_content__*` + `_content_HOME*`** via
  `string match -qr '^_content_(_|HOME)'`. A bare `_content_*` glob also catches the
  `_content_fn_for` dispatcher (L921) and returns 18. Fish function names contain dots
  (`_content_HOME_.config_MangoHud_MangoHud.conf`), so a `[A-Za-z0-9_-]*` charset truncates
  them and fabricates duplicates — match `\S+` after `function`.
- **Set `_ROOT_UUID`** (single underscore, not `_RY_ROOT_UUID`) with any 36-char valid-shape
  UUID, or the cmdline generator returns 12 / `GEN_NOUUID` and `_ry_validate_configs` returns
  rc 3, which reads as a regression. With the stub, the render reproduces the host byte count
  exactly.
- **Generated bytes must be measured as WRITTEN FILES** (`$fn > tmp; stat -c %s`), never
  `(cmd | string collect)` — collect strips each trailing newline and the 17-file total reads
  4,841 B instead of 4,858 B, a phantom deficit that looks like anchor drift. `string length`
  counts characters and under-reports for the same reason.
- **Determinism must be compared per-file by content hash, or with a SORTED manifest.** A
  concatenated hash over the output directory is not comparable between passes because the
  glob order depends on harness filenames. The manifest sha changes between editions even when
  every body is identical — `03b0863baa9fcde2` at 7.155.0, `6bf7f8ea53c36c40` here. That is
  expected, not drift.
- **Verify every "before" column against the OLD EDITION'S RENDERED BODY, not memory.** The old
  script is not in the archive; only its brief is. A drift row asserting a change that never
  happened passes every count check and every byte check — only an old-vs-new body diff catches
  it. Extract the previous edition's fences with a **toggle-based** fence walker; a naive
  `re.findall` on triple-backticks consumes alternating pairs and under-reports. **The walker
  earned its keep for the third consecutive rebase**, and this time it caught something nothing
  else could: `mkinitcpio.conf` changed content at **zero byte delta and zero count delta**.
  Two extraction gotchas worth reusing: the walker inherits the *last seen* bold label, so
  fences following the final body are mis-attributed to it (harmless — take the first fence per
  label, and do not read the extras as an 18th body); and the cmdline fence will always
  "differ" unless the stub UUID matches the one embedded in §7e — compare everything after the
  `root=UUID=` token.
- **Fence-aware heading checks are mandatory.** A naive `^# ` scan false-reports h1 violations
  on the `#` comments inside the nft, udev, sysctl and config fences.
- **Byte-vs-character length.** Banner and line-length checks must count CHARACTERS; `awk
  length` under a C locale counts bytes and falsely reports the U+2500/U+2192 box-drawing
  content over any character cap. In box tables `—` is ONE character; validate width uniformity
  per contiguous `^║` block, and when a row must grow, rebuild the block with recomputed column
  widths rather than padding by hand.
- **CRLF and banner false positives.** A byte-level `b'\r' in raw` test reports dozens of hits
  that all sit inside string literals; test `line.endswith(b'\r')` instead — 0 lines actually
  end with CR. And a "contains U+2500" banner-width check flags a one-line function whose box
  characters are in runtime `_echo` output, not a comment banner; split the classes.
- **Verify the upload before trusting it.** A fresh upload can be behind what is recorded as
  shipped. Hash the archive against §0 first. Collision precedent is routine: 7.141.0 shipped
  twice with an identical script hash, 7.140.0 shipped nine times with only two distinct script
  hashes. Read the `VERSION` line plus a structural count, then confirm with the
  zip/README/CHANGELOG hash. **All 8-hex anchors are `sha256sum | cut -c1-8`, not CRC-32.**
- **Sandbox limits.** Target host paths do not exist, so only sudo-fail and preflight paths are
  exercisable and a full install cannot complete by design. `_err_loud` **exits** rather than
  returns — run each negative test in its own subprocess. Useful shims: a 3-line `sudo` wrapper
  (strip `-n`/`--`, exec) makes `_as true` paths runnable unprivileged;
  `functions FN | string replace -a /sys/... /tmp/fx/... | source` re-binds hardcoded sysfs
  paths for fixture tests. fish `count` counts ARGUMENTS, not stdin — `… | count` is always 0;
  use `count (cmd)`. `find -type d` returns 0 hits for cpufreq because
  `/sys/devices/system/cpu/cpuN/cpufreq` is a symlink — use `-xtype d` or a glob + `test -d`.
- **Network from the sandbox.** git.kernel.org cgit `/plain/` (with `?h=vX.Y` tag pinning),
  raw.githubusercontent.com, gitlab.freedesktop.org `/-/raw/`, and github.com HTML pages for
  release tags and issue state (`grep -o 'octicon-issue-[a-z]*'`). **api.github.com
  rate-limits on the shared egress IP** — parse HTML instead.

---

## 10. Scope and output contract

**In scope:** recommendations only — do not emit a modified script. Hardware-anchored to
gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE / 85 W BIOS ceiling.
**Note from T0-8: both 10 GbE links are down on the deployed host and wlan0 carries the
default route** — a networking recommendation that assumes the wired links are in use is
describing the hardware, not the deployment.

**Out of scope:** dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS,
laptops, UKI, BIOS flashing. Per-game Proton tuning is secondary to system-wide config; T1-2
and T1-3 have both moved there permanently.

**Rules:**

1. Respect deliberate trade-offs — flag and quantify, do not auto-FIX. Reserve FIX for
   incorrect, superseded, deprecated, or harmful values.
2. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is
   non-trivial.
3. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN. One
   UNCERTAIN is already on the record and must not be resolved by guessing: the exact release
   that dropped `PROTON_ENABLE_WAYLAND` (§1c).
4. Flag every source conflict and name the trusted side. The netdev-budget conflict (Red Hat
   vs ESnet) is preserved in §3 even though the keys are gone, because it is the reason the
   measurement gate existed. The live conflict that remains is nftables rule order (Gentoo vs
   nftables.org/ArchWiki). Do not resolve it silently.
5. Give exact versions (kernel / Mesa / linux-firmware / proton-cachyos / package) and exact
   before→after, mapped to the in-script global.
6. **Do not carry values forward from any pre-7.158.0 edition.** A recommendation that
   re-derives a removed token, or that asserts a *perf* value changed in the 7.130.0 →
   7.158.0 window, is a stale-source error rather than a finding. The converse binds equally:
   DNS, the FSR4 variables, `PROTON_ENABLE_WAYLAND`, both netdev keys, `fsck.mode` and
   `COMPRESSION_OPTIONS` **did** change in this window.
7. **A gate that has returned is not an action.** All eight T0 items are answered; citing one
   as "unrun" or gating a recommendation on it is a stale-source error.
8. A question closed by a code change is closed. A question closed by upstream evidence (§3)
   is closed. A question closed by a recorded maintainer decision (T3-1, T3-2, T2-3) is closed
   — argue against the decision explicitly or leave it alone.
9. A correction in §4 is binding. Do not re-raise a finding this edition withdrew.

**Required output:**

- **Findings matrix** (box-drawn, code-fenced, grouped by tier): ITEM · CURRENT · CALL
  (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE.
- **Before→after** for each TUNE/FIX, naming the in-script global and — for any perf value —
  all 11 sites from §7b.
- **Tier placement** for every recommendation, and the removal tier (§7f) for any removal.
- **Oracle delta** for anything that changes an array length, plus the affected byte anchors
  from §7d. **If a change moves no count and no byte anchor, say so explicitly and cite the
  §7e fence** — the `mkinitcpio.conf` precedent means silence there is now a defect.
- **Security delta** (§5, ordered).
- **Verdict** per tier plus overall (PASS / PASS-WITH-FIXES / FAIL).
- **Methodology:** source list with access dates and versions; unknowns marked UNCERTAIN.

---

## Sources

**Primary, re-accessed 2026-08-07:** git.kernel.org torvalds/linux `Makefile` (7.2.0-rc6) ·
github.com CachyOS/linux-cachyos PKGBUILD (7.1.6-1) · gitlab.freedesktop.org mesa/mesa
`VERSION` (26.3.0-devel) · github.com CachyOS/proton-cachyos releases (11.0-20260703) ·
github.com flightlessmango/MangoHud issue #1794 (open) · github.com systemd/systemd issue
#33579 (open).

**Primary, accessed 2026-08-05 and unchanged since:**
`Documentation/admin-guide/kernel-parameters.txt`, `drivers/pci/pcie/Kconfig`,
`drivers/cpufreq/amd-pstate.c`, `drivers/cpufreq/Kconfig.x86`,
`drivers/net/wireless/mediatek/mt76/mt7925/pci.c`,
`drivers/net/ethernet/realtek/r8169_main.c` · git.kernel.org linux-firmware
`amdgpu/gc_11_5_1_mes_2.bin` history · CachyOS/linux-cachyos config
(`X86_AMD_PSTATE_DEFAULT_MODE=3`, `PCIEASPM_DEFAULT=y`, `PCIEASPM_PERFORMANCE` not set,
`NTSYNC=m`) · mesa `docs/envvars.rst` · Proton-EM `em-10` docs (`FSR4.md`,
`EM-ADDITIONS.md`) · docs.redhat.com RHEL 8/9/10 network performance tuning ·
fasterdata.es.net 100 G "Other Tuning" · wiki.gentoo.org Nftables/Examples · wiki.nftables.org
quick reference · wiki.archlinux.org Nftables.

**Live host measurements (T0-1 … T0-8), captured on the deployed GTR9 Pro** and recorded in
§2. These are observations of one machine, not upstream claims; re-measure rather than cite
them for a different host.

**Standing reference:** docs.kernel.org (PCIe/ASPM, amd-pstate, sysctl/vm, sysctl/kernel,
networking, block, ext4, UMIP, AMD-Vi, accel/amdxdna, amdgpu
`power_dpm_force_performance_level`, powercap/RAPL) · docs.mesa3d.org · wiki.archlinux.org
(AMDGPU, IOMMU, fsck, Gaming, Zram, SSD, Ext4, Sysctl, NetworkManager, Wireless,
Uncomplicated_Firewall, Mkinitcpio, systemd-boot, Bluetooth, CPU_frequency_scaling, MangoHud,
pacman) · wiki.cachyos.org · discuss.cachyos.org · man.archlinux.org (nft, avahi-daemon,
systemd.unit, logind.conf, resolved.conf, NetworkManager.conf, systemd-fsck) · invent.kde.org
powerdevil · fishshell.com/docs · strixhalo.wiki (Power Modes, C-States) · amd.com ROCm ·
the standalone `mangohud-gtr9-pro` companion archive.

**Do not cite** wireless.docs.kernel.org for `pcie_aspm` semantics (pre-6.9 text);
kernelconfig.io for Kconfig introduction versions; any source attributing the 600/4000 netdev
pairing to ESnet. Cite access dates and exact versions in the methodology block.
