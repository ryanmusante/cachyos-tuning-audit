# cachyos-tuning-audit — ry-install Tuning (CachyOS · Beelink GTR9 Pro)

**Target:** `ry-install.fish` **v7.105.9** (attached, 2026-07-14; 4923 lines, 288 functions, `fish --no-execute` clean; script delta vs 7.105.8 = version string only, 2 lines).
**Source of truth:** script > README > CHANGELOG. Every value below is re-derived from the v7.105.9 script; where any other source disagrees, the script wins and the disagreement must be flagged.
**Platform:** Beelink GTR9 Pro · Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤96 GB as VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP (README BIOS ceiling 85 W) · CachyOS · systemd-boot.
**Companion source:** `mangohud-gtr9-pro` **v1.17.0** (attached, 2026-07-04; MIT; MangoHud.conf + README + CHANGELOG) — standalone publication of the embedded HUD config; installer is source of truth (repo CHANGELOG 1.14.0). Cross-audited in §12 / B9; HUD-scoped floors in §12.

Findings closed in-script at ≤7.101.0 are excluded; do not re-open them. The Protected list (§14) is the do-not-recommend inventory.

---

## Mission

Deep-research brief: evaluate every configured value against current upstream for this exact silicon and return a prioritized, evidence-backed report targeting **gaming improvements, performance enhancements, and overall system speed gain**. The profile deliberately trades PCI passthrough, the XDNA NPU, power saving, PCIe ASPM, IPv6, and inbound-ping filtering for latency, throughput, and a simpler IPv4-only ruleset — confirm each trade is current and correct, surface anything superseded or harmful, and quantify safety deltas without second-guessing intentional design.

## Rules

1. Item-by-item, hardware-anchored to gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE.
2. Respect deliberate trade-offs: flag and quantify, do not auto-FIX. Reserve FIX for incorrect, superseded, deprecated, or harmful values.
3. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is non-trivial.
4. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN.
5. Flag every source conflict and name the trusted side.
6. Give exact versions (kernel / Mesa / linux-firmware / vkd3d-proton / pkg) and exact before→after, mapped to the in-script global.

## Output

- **Findings matrix** (box-drawn, code-fenced, grouped by section): ITEM · CURRENT · CALL (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE.
- **Candidate-enhancement matrix** (§13, separate): ITEM · PRESENT?(no) · CALL (ADD-default/ADD-opt-in/KEEP-omitted) · IMPACT · RISK · EVIDENCE.
- **Before→after** for each TUNE/FIX/ADD: exact current string, exact replacement, in-script global.
- **VERIFY block** (post-reboot commands, §15).
- **Security delta vs CachyOS defaults** (§11, ordered).
- **Verdict:** one per section (OPTIMAL/TUNE/FIX) plus overall (PASS/PASS-WITH-FIXES/FAIL).
- **ROBUSTNESS verdict** (§G–§L, separate from tuning).
- **Methodology:** source list with access dates and versions; unknowns marked UNCERTAIN.

## Hard data (all counts asserted by `_ir_validate_counts`, 21 tripwires)

KERNEL_PARAMS 17 · MKINITCPIO_HOOKS 11 · MKINITCPIO_MODULES 1 · LOGIND_IGNORE_KEYS 8 · **ENV_VARS 12** · **SYSCTL_VALUES 10** · PKGS_ADD 18 · PKGS_DEL 9 · MASK 12 · EXPECTED_VULKAN_PKGS 2 · EXPECTED_SERVICES 5 · _RY_PKG_MANAGED_SERVICES 1 · _RY_POST_HOOKS 17 · _RY_ARGPARSE_SPEC 7 · _RY_BOOT_CRITICAL_DSTS 4 · _RY_PHASE_NAMES 6 · _RY_BACKUP_TARGETS 4 · _RY_TMPDIR_GLOBS 6 · SYSTEM_DESTINATIONS 15 · USER_DESTINATIONS 2 · MKINITCPIO_COMPRESSION_OPTIONS 2.

Managed files = 17 (15 system + 2 user; `_RY_MANAGED_FILE_COUNT`), recomputed at load; mismatch refuses (exit 3).

**Gates (v7.105.9):**
- **Kernel floor is ADVISORY ONLY** — comment at script line 15: `6.18.4 — RTL8127 r8169 + suspend-hang fix ae1737e7339b; gfx1151 fix is firmware (linux-firmware MES 0x86), not kernel`. The former hard-fail validator `_ir_validate_kernel_floor` and `KERNEL_MIN` are REMOVED (validators 4→3); deploy/`--check`/`--verify` run on any kernel. Do not describe any kernel gate as enforced.
- **CPU gate** `Ryzen AI Max` — sole skip env `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed on unreadable model; `--verify` warns, deploy/`--check` exit 3.
- **Mesa < 26.0** — soft warn only; `vercmp` output validated `^-?\d+$` before compare.
- Preflight validators, in `_init_runtime` order: `_ir_validate_counts` → `_ir_validate_keys` → `_ir_validate_post_hooks` (3 total).
- The only runtime env inputs are `RY_RUN_TIMEOUT`, `RY_INSTALL_SKIP_HARDWARE_CHECK`, `NO_COLOR`. Every profile toggle (`BLACKLIST_AMDXDNA`, `NM_WIFI_BACKEND`, `COUNTRY`, `GPU_DPM_LEVEL`, `EPP_PREFERENCE`, `RY_REMOTE_PLAY_PORTS`) is an embedded scalar set unconditionally (`set -g`) — an exported env var of the same name is clobbered; opting in means editing the script.

---

## P0. Research priority queue (search in this order)

```
║ #  ║ QUESTION (search anchors)                                            ║ SECTION ║
║────║──────────────────────────────────────────────────────────────────────║─────────║
║ 1  ║ pcie_aspm=off vs pcie_aspm.policy=performance semantics — does       ║ §5      ║
║    ║ "off" leave BIOS-enabled links in L0s/L1 (kernel never touches       ║         ║
║    ║ ASPM)? MT7925 + NVMe mitigation still effective? lspci LnkCtl        ║         ║
║ 2  ║ VKD3D_CONFIG=descriptor_heap — current vkd3d-proton effect on        ║ §1      ║
║    ║ RADV/RDNA3.5; default-enabled yet? global-force regressions          ║         ║
║ 3  ║ vm.watermark_boost_factor=0 — reclaim/kcompactd spike removal on     ║ §3      ║
║    ║ 128 GB unified; interaction with compaction_proactiveness=0          ║         ║
║ 4  ║ PROTON_FSR4_RDNA3_UPGRADE=1 — Proton-CachyOS consumption, min        ║ §1      ║
║    ║ version, DXIL_SPIRV_CONFIG=wmma_rdna3_workaround companion           ║         ║
║ 5  ║ EPP balance_performance→performance / GPU_DPM auto→high under the    ║ §2      ║
║    ║ 85 W ceiling — gfx1151 frametime/1%-low evidence only                ║         ║
║ 6  ║ linux-firmware MES 0x86 (GC 11.5.1) — shipping revision contains     ║ §10     ║
║    ║ the gfx1151 hang fix? min linux-firmware release; kernel-irrelevant  ║         ║
║ 7  ║ ntsync vs fsync on 16C/32T — CONFIG_NTSYNC=y, /dev/ntsync,           ║ §1      ║
║    ║ PROTON_NO_NTSYNC opt-out currency                                    ║         ║
║ 8  ║ RTL8127 r8169 commits f24f7b2f3af9 + ae1737e7339b — exact mainline   ║ §10     ║
║    ║ landing releases (advisory-floor legs)                               ║         ║
║ 9  ║ netdev_budget 600/5000 + BBR/fq sizing for dual 10 GbE (RTL8127)     ║ §6      ║
║ 10 ║ MT7925 mt76 upstream ASPM/coredump fix status — per-module option    ║ §6      ║
║    ║ dropped 7.102.x; is global ASPM-off still required?                  ║         ║
║ 11 ║ Modprobe-leftover gap now UNGUARDED — pre-7.99 stale drop-ins have   ║ §6/B10  ║
║    ║ 0 in-script refs AND the README migration note is gone               ║         ║
║ 12 ║ zstd -1 -T0 initramfs vs default-3 — boot decompress + ESP budget    ║ §7      ║
║ 13 ║ Every-boot fsck.mode=force + commit=10 — boot cost vs durability     ║ §4/§5   ║
║ 14 ║ swappiness=150 + CachyOS zram + zswap.enabled=0 coherence            ║ §3      ║
║ 15 ║ cpu_temp dormant — reconcile #1794 (README: cpu_power→0 on Zen 5     ║ §12     ║
║    ║ when active) vs repo v1.17.0 k10temp-pickup claim; fixed upstream?   ║         ║
║ 16 ║ amdxdna -EINVAL (ret -22) under amd_iommu=off — errno current; any   ║ §10     ║
║    ║ kernel making the NPU IOMMU-optional?                                ║         ║
║ 17 ║ Candidate knobs (§13) — mitigations, ppfeaturemask, RADV_PERFTEST,   ║ §13     ║
║    ║ read_ahead_kb: KEEP-omitted re-checks                                ║         ║
║ 18 ║ Security-delta quantification (UMIP, AMD-Vi off, IPv6 off,           ║ §11     ║
║    ║ split_lock, plaintext DNS) — quantify, no auto-FIX                   ║         ║
║ 19 ║ HUD floors current? MangoHud ≥0.8.4 (Steam-Overlay fix) · kernel     ║ §12     ║
║    ║ ≥6.14 · Mesa 24 — companion mangohud-gtr9-pro v1.17.0; HUD-scoped    ║         ║
║    ║ floors only, do not conflate with §10 profile floors                 ║         ║
```

---

## §1. GPU · Vulkan · Proton runtime

**ENV_VARS (12, `~/.config/environment.d/10-environment.conf`):** AMD_VULKAN_ICD=RADV · DXVK_LOG_LEVEL=none · DXVK_LOG_PATH=none · MANGOHUD=1 · MESA_SHADER_CACHE_MAX_SIZE=16G · PROTON_ENABLE_WAYLAND=1 · PROTON_FSR4_RDNA3_UPGRADE=1 · PROTON_LOCAL_SHADER_CACHE=1 · **VKD3D_CONFIG=descriptor_heap** · VKD3D_DEBUG=none · VKD3D_SHADER_DEBUG=none · WINEDEBUG=-all. No drirc (uma:1 native); no ttm/amdgpu module params; Vulkan stack via chwd (vulkan-radeon + lib32-vulkan-radeon).

- **VKD3D_CONFIG=descriptor_heap (NEW 7.102.x, shipped globally):** confirm the option exists in current vkd3d-proton and its effect on RADV/RDNA 3.5 (mutable-descriptor path); determine whether current vkd3d-proton enables it by default (env then redundant → FIX-to-remove) or per-GPU; enumerate known per-title regressions from forcing it globally; check whether additional VKD3D_CONFIG flags are gaming-relevant defaults now. Unverifiable ⇒ UNCERTAIN, not FIX.
- **PROTON_FSR4_RDNA3_UPGRADE=1:** confirm current Proton-CachyOS consumes it for FSR3.1→FSR4 on RDNA 3.5, the minimum Proton version, and the `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` companion status; no-op on non-FSR titles. Unverified ⇒ FIX-to-remove; verified ⇒ KEEP + cite.
- **ntsync (assert-only, no autoload conf):** `_vre_ntsync`/`_ntsync_state` (builtin|loaded|loaded_nodev|missing). Confirm: ntsync vs fsync currency; CachyOS `CONFIG_NTSYNC=y` (node without autoload); `loaded_nodev` still a real failure; frametime benefit on 16C/32T; `PROTON_NO_NTSYNC=1` opt-out current.
- **RADV heap:** confirm uma:1 on current Mesa (drirc removed by design).
- **GTT:** kernel auto-sizes (~62 GiB); >62 GiB single allocations route to BIOS UMA carveout (≤96 GB), not deprecated `amdgpu.gttsize`; verify `/sys/module/ttm/parameters/pages_limit`. `amd_iommu=off` does not change the ceiling.
- PROTON_ENABLE_WAYLAND maturity/fallback on current Proton; MESA_SHADER_CACHE_MAX_SIZE=16G sizing; MANGOHUD=1 overhead with gamescope/GameMode; AMD_VULKAN_ICD vs VK_DRIVER_FILES currency.
- **XDNA NPU:** blacklisted by default (§10/§11) — zero gaming impact; LLM/NPU work needs the validator-paired opt-in.
- Sources: github.com/HansKristian-Work/vkd3d-proton (env docs), docs.mesa3d.org (RADV, APU heap), gitlab.freedesktop.org/mesa + drm/amd, github CachyOS/proton-cachyos, amd.com ROCm, docs.kernel.org accel/amdxdna.

## §2. CPU performance & power

amd_pstate=active · governor **powersave** (`CPUPOWER_GOVERNOR`, cpupower-service.conf) · EPP **balance_performance** via udev (`EPP_PREFERENCE`, enum `_RY_EPP_LEVELS`) · `EXPECTED_SCALING_DRIVER=amd-pstate-epp` · `dynamic_epp=disabled` asserted · GPU_DPM_LEVEL=auto (enum `_RY_DPM_LEVELS`; add-only udev rule, `ENV{DEVTYPE}=="drm_minor"`) · prefcore + boost=1 asserted. Masked: power-profiles-daemon, ananicy-cpp, modemmanager.

- **governor=powersave + EPP=balance_performance — special case, do not flag the governor.** Under amd_pstate=active, `powersave` honors EPP while `performance` pins max and ignores it; this triple is the documented EPP-honoring max-perf config on Zen 5. `balance_performance`→`performance` stays UNCERTAIN without gfx1151/Zen-5 frametime data; any change is the `EPP_PREFERENCE` global only.
- **GPU_DPM_LEVEL=auto:** re-evaluate `auto` vs `high` for frametime/1%-lows under the shared package budget. Two inertia facts: the rule is add-only (no re-assert after GPU reset) and only began firing at the v7.94/95 matcher fix — pre-fix "high made no difference" observations are void. **Name the assumed budget in every power call (85 W README ceiling vs 140 W stock).**
- Confirm EPP live-applies (`_post_udev`: `udevadm verify` ≥254, reload + retrigger cpu/block) and the GPU rule matches at enumeration; `dynamic_epp` node ≥6.16; prefcore/boost on Strix Halo; masks (ppd, ananicy-cpp, modemmanager) safe on current CachyOS.
- `processor.max_cstate=1` (§5): idle power/thermal vs wake-latency/jitter; boost-headroom interaction; is `1` the right cap?
- Sources: docs.kernel.org amd-pstate + kernel-parameters, wiki.archlinux.org/CPU_frequency_scaling + AMDGPU, freedesktop.org ppd.

## §3. Memory & VM

**SYSCTL_VALUES (10, `/etc/sysctl.d/95-ry-overrides.conf`, priority 95 after vendor 70-cachyos-settings.conf):** net.core.default_qdisc=fq · net.core.netdev_budget=600 · net.core.netdev_budget_usecs=5000 · net.ipv4.tcp_congestion_control=bbr · net.ipv4.tcp_notsent_lowat=16384 · net.ipv4.tcp_slow_start_after_idle=0 · vm.compaction_proactiveness=0 · vm.max_map_count=2147483642 · vm.swappiness=150 · **vm.watermark_boost_factor=0**. zswap.enabled=0 (cmdline). THP/KSM/oomd left to CachyOS; vm.page-cluster + vm.vfs_cache_pressure dropped as vendor duplicates.

- **vm.watermark_boost_factor=0 (NEW 7.102.x):** confirm disabling watermark boosting removes post-fragmentation reclaim/kcompactd spikes (frametime consistency) on 128 GB unified; interaction with `compaction_proactiveness=0` (both suppress proactive compaction paths — coherent or redundant?); name the CachyOS vendor value it overrides (kernel default 15000).
- **Vendor-duplicate drop a no-op?** Confirm 70-cachyos-settings.conf still ships page-cluster=0 + vfs_cache_pressure=50; a differing vendor default makes the drop a silent change.
- **swappiness=150 + CachyOS zram + zswap=0:** gratuitous on 128 GB or LLM-reclaim-helpful; no double compression (zswap off before zram); zram advisory-only (not managed, not asserted).
- vm.max_map_count (MAX_INT−5, SteamOS value) — Proton/anti-cheat sufficiency; compaction_proactiveness=0 for large unified allocs; oomd disabled on 128 GB.
- Sources: docs.kernel.org admin-guide/sysctl/vm + mm, wiki.archlinux.org/Zram + Sysctl.

## §4. Storage & filesystem

NVMe scheduler `none` via udev (99- sorts after vendor 60-ioschedulers.rules); `nvme_core.default_ps_max_latency_us=0` (§5); fstab ext4 `noatime,lazytime,commit=10`; fstrim.timer enabled; zswap off.

- NVMe `none` vs mq-deadline/kyber on this dual-NVMe box; `nr_requests`/`read_ahead_kb` unset — propose ATTRs only with game-load/LLM-read evidence, else defaults optimal; confirm CachyOS still defaults kyber and 99- ordering wins.
- noatime+lazytime coexistence (lazytime residual value under noatime); **commit=10 durability vs every-boot forced fsck (§5)** — quantify the boot cost of fsck.mode=force on ext4 NVMe and whether commit=10 + forced fsck is coherent or belt-and-braces; fstrim.timer vs continuous discard.
- fstab rewrite invariants: Appendix D.
- Sources: docs.kernel.org block + ext4, wiki.archlinux.org/SSD + Ext4 + fsck.

## §5. Kernel cmdline (17 tokens — latency set)

```
8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm=off processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0
```

- **`pcie_aspm=off` (CHANGED 7.102.x from `pcie_aspm.policy=performance`) — top cmdline item.** Semantics differ: `off` disables the kernel ASPM driver entirely and inherits whatever link states firmware programmed; `policy=performance` actively selects the performance policy. Research: (a) does `off` leave any BIOS-enabled link in L0s/L1 on this board (audit every `lspci -vv` LnkCtl); (b) is the MT7925 mitigation (coredump/BT-reconnect/assoc) still effective via global off now that the per-module `mt7925e disable_aspm=1` option is dropped; (c) NVMe latency claim under off vs policy=performance; (d) whether `pcie_port_pm=off` is additionally needed or redundant. README rationale (byte-exact, 7.105.9): "(MT7925 coredump / BT-reconnect / assoc fix + NVMe latency); drop to restore ASPM defaults."
- **`clearcpuid=umip`:** UMIP off, kernel tainted; string form version-stable. Confirm `umip` name accepted on current kernels; README drop-condition: no `umip_printk` stutter. Asserted generically (`_vrk_cmdline`), no UMIP-specific check.
- **`amd_iommu=off`:** validator-paired to `BLACKLIST_AMDXDNA` (§11). ROCm unaffected on gfx1151; NPU is the named casualty. Weigh marginal latency vs DMA-isolation loss vs NPU loss.
- **`ipv6.disable=1`:** hard-coupled to the IPv4-only nftables ruleset (`_ir_validate_keys`). LAN impact, Steam/Proton netcode fallback behavior, README dual-stack opt-out.
- **processor.max_cstate=1:** §2. **btusb.enable_autosuspend=n:** MT7925/BT reconnect fix; overlap with usbcore.autosuspend=-1. **fsck.mode=force + fsck.repair=yes:** §4 boot-cost + auto-repair safety + hook handshake. **amd_pstate=active / split_lock_detect=off / tsc=reliable / nowatchdog / 8250.nr_uarts=0 / usbcore.autosuspend=-1 / nvme_core ps_max_latency=0 / zswap.enabled=0:** validate each is current, non-deprecated, and correct for this silicon.
- No `preempt=` — KEEP-omitted (CachyOS boots full; `_vrk_cmdline` INFOs the model). Zero amdgpu/ttm params — hands-off (`_vrkm_amdgpu` no-ops without `amdgpu.*`).
- Input hygiene: tokens charset-gated `^[A-Za-z0-9._,=-]+$`; a new token outside it must also change the validator.
- Sources: docs.kernel.org kernel-parameters + PCIe/ASPM + amd-pstate + UMIP + IOMMU + ipv6-sysctl, wiki.archlinux.org/AMDGPU + IOMMU + fsck + Power_management.

## §6. Network & latency

Net sysctls §3; IPv6 off §5; nftables §11. NM: wifi.backend=wpa_supplicant, wifi.powersave=2 (off — MT7925/mt76 software PS causes latency spikes), logging WARN, dispatcher LogLevelMax=notice. resolved: MulticastDNS=no, LLMNR=no, DNSOverTLS=no, DNSSEC=allow-downgrade (plaintext; diverges from CachyOS DoH). regdom US. Masked: NetworkManager-wait-online, modemmanager, avahi-daemon.service + .socket. Enabled: NetworkManager.
**Modprobe managed file (`60-ry-modules.conf`, CHANGED 7.102.x):** amdxdna blacklist only (default); `BLACKLIST_AMDXDNA=false` renders a comment-only file (validator `_grep_modprobe_entry` accepts comment-only). The `options mt7925e disable_aspm=1` line is REMOVED — coverage moved to `pcie_aspm=off`.

- **⚠ OPEN GAP — modprobe-leftover migration now UNGUARDED (Low/Low → re-rate).** Superseded `60-ry-mt7925e.conf` / `60-ry-blacklist-amdxdna.conf` have zero in-script references, `_vrkm_blacklist_modprobe` is generator-sourced (checks intended content, not on-disk extras), and the v7.105.9 README (unchanged from 7.105.8 here) no longer carries the one-time `sudo rm` migration note — guard count is now zero. A stale `60-ry-blacklist-amdxdna.conf` keeps the NPU blacklisted after opt-in, invisible to every verify path. A stale `60-ry-mt7925e.conf` is benign-redundant under `pcie_aspm=off` (INFO). ADD-check candidate; verify-side stale-file scan of `/etc/modprobe.d/60-ry-*` against SYSTEM_DESTINATIONS.
- **MT7925 upstream status:** has the mt76 ASPM/coredump fix landed such that neither global ASPM-off nor a per-module option is required? Cite commit + release; if landed, the `pcie_aspm=off` rationale loses its Wi-Fi leg and rests on NVMe latency alone.
- **netdev_budget=600/netdev_budget_usecs=5000 on dual 10 GbE:** confirm sizing for RTL8127 line rate or propose values with driver evidence; tcp_rmem/wmem/ring defaults sufficiency.
- bbr+fq currency (BBRv3 status in mainline/CachyOS); wifi.powersave=2 still correct for mt76; wpa_supplicant vs iwd parity (iwd opt-in intact; residual verify coverage = backend compare only — deliberate, Low/Low).
- **avahi masked (unit+socket):** confirm no host dependency (printer/`.local` discovery) and no D-Bus resurrection path; with resolved MulticastDNS=no, multicast discovery is fully closed.
- regdom US: MT7925 TX-power/channel on current wireless-regdb; 6 GHz AFC status; non-US requires hand-edit.
- Same-basename replace caution (B5/B6): if CachyOS ships its own `99-cachyos-resolved.conf`/`99-cachyos-nm.conf`, deploy REPLACES them — confirm intended, not a vendor-update clash.
- Sources: docs.kernel.org networking, git.kernel.org mt76 + wireless-regdb + r8169, wiki.archlinux.org NetworkManager + Wireless + Sysctl, man.archlinux.org avahi-daemon.

## §7. Boot chain & initramfs

loader.conf: default @saved, timeout 0, console-mode keep, editor no. sdboot-manage: DEFAULT_ENTRY manual, OVERWRITE/REMOVE_EXISTING/REMOVE_OBSOLETE yes, LINUX_FALLBACK_OPTIONS "quiet". mkinitcpio: MODULES=(amdgpu), HOOKS(11)= base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck, COMPRESSION zstd, COMPRESSION_OPTIONS=(-1 -T0), explicit BINARIES=()/FILES=(). Pre-deployed Phase 2 → one rebuild at `-Syu`.

- HOOKS order (systemd/microcode/kms/sd-vconsole/block) current-correct; amdgpu early-KMS; fsck-hook handshake with `fsck.mode=force` (no boot prompt).
- **zstd -1 -T0:** quantify boot decompress vs default-3 (sub-100 ms class on NVMe) and image size vs ESP budget (`BOOT_SPACE_CRIT/WARN` 200/500 MB) with multiple kernels + fallback. TUNE to default-3 only if size threatens the budget; tokens are charset-gated + count-asserted — any TUNE updates both.
- Live drift caught: `_vsb_mkinitcpio` compares live `COMPRESSION=`/`COMPRESSION_OPTIONS` (multi-line join, last-wins warn) — confirm last-wins matches shell sourcing.
- timeout 0 + manual + REMOVE_EXISTING=yes wipes foreign BLS entries (EFI-resident loaders untouched); recovery path = live-USB → chroot; sdboot-manage currency vs kernel-install/UKI (UKI out of scope).
- **Fallback-entry exposure:** `LINUX_FALLBACK_OPTIONS="quiet"` strips all params — fallback boots kernel-default IOMMU (AMD-Vi ON) + IPv6 ENABLED under the IPv4-only ruleset; the modprobe amdxdna blacklist REMAINS active (asymmetry). Confirm the exposure window is accepted or flag it.
- Sources: wiki.archlinux.org Mkinitcpio + systemd-boot, sdboot-manage upstream.

## §8. Packages

**PKGS_ADD (18):** nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa, mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, git-delta, lm_sensors, rtkit, realtime-privileges, ddcutil, nftables, pacman-contrib (supplies pactree `PACTREE_TIMEOUT_S=60` + paccache -rk2/-ruk0).
**PKGS_DEL (9, `-Rns`, rdep-aware via pactree):** plymouth, cachyos-plymouth-bootanimation, cachyos-plymouth-theme, breeze-plymouth, plymouth-kcm, micro, cachyos-micro-settings, cachy-update, kdeconnect. **AUR:** none. **Vulkan (chwd, verify-only):** vulkan-radeon, lib32-vulkan-radeon.

- Confirm the gaming metas supply RADV/Proton/gamescope/MangoHud/GameMode; **GameMode omission — KEEP** (governor/EPP/DPM pinned profile-wide); confirm the meta's MangoHud does not clash with the shipped conf.
- `-D --asexplicit` post-`Syu` re-mark = orphan protection for PKGS_ADD members that pre-existed as dependencies (idempotent; failure warns).
- rtkit + realtime-privileges for PipeWire priority (rtkit-daemon socket-activated); lib32-mesa still needed beside lib32-vulkan-radeon; PKGS_DEL fallout tracked (`_RY_PKG_REMOVE_SKIPS`).
- Advisory: znver/x86-64-v4 (AVX-512) repo benefit over v3 for this build.
- Sources: wiki.cachyos.org, wiki.archlinux.org Gaming + PipeWire + RealtimeKit, archlinux.org/packages.

## §9. systemd units & time-sync

**Mask (12):** ananicy-cpp, power-profiles-daemon, NetworkManager-wait-online, ufw, modemmanager, avahi-daemon.service + .socket, sleep/suspend/hibernate/hybrid-sleep/suspend-then-hibernate targets. **Enable (5):** fstrim.timer, NetworkManager, cpupower, nftables, bluetooth. **Untouched:** oomd (intentional), NetworkManager-dispatcher + rtkit-daemon (socket-activated), iwd; ufw flushed after nftables live.
**NTP unconditional:** `_ry_check_time_sync` scans chronyd/ntpd/openntpd; refuses to enable timesyncd if any is active; else enables timesyncd, re-checks after 2 s, runs `_ry_rtc_writeback` (`--systohc --utc`; RTCInLocalTZ defer branch) on sync. Escape = mask timesyncd (no opt-out env, by design).

- Each mask safe on current CachyOS: ananicy-cpp + ppd (§2); modemmanager (no cellular); avahi (§6); sleep targets = no suspend (always-on mini-PC).
- nftables-first-then-ufw-flush leaves no unfirewalled window (mask skipped if nft not live); oneshot judged by live ruleset; `nft -c`-gated at deploy + `_post_nft`.
- fstrim.timer vs continuous discard; cpupower service vs CachyOS freq management; logind Handle*Key=ignore (8 keys incl. LongPress) — no lockout.
- RTC write-back safety; no ownership conflict with timesyncd.
- Sources: man.archlinux.org systemd.unit + logind.conf + hwclock + timesyncd, wiki.archlinux.org Bluetooth + System_time.

## §10. Version floors, firmware & known issues

**Advisory floor 6.18.4 (comment-only, NOT enforced; validator + KERNEL_MIN removed 7.105.x).** Legs: RTL8127 r8169 support `f24f7b2f3af9` + suspend/shutdown-hang fix `ae1737e7339b`. **gfx1151 GPU-hang fix re-anchored to firmware: linux-firmware MES 0x86 (GC 11.5.1), not kernel.**

- **Verify both advisory legs:** cite the exact mainline releases for `f24f7b2f3af9` and `ae1737e7339b`; consequence of error is doc-only (floor unenforced) but the regression baseline must be accurate.
- **Verify the firmware anchor:** shipping linux-firmware contains the MES 0x86 (or later) GC 11.5.1 revision with the hang fix; name the minimum linux-firmware release; confirm kernel version is genuinely irrelevant to the fix (prior audit generations bounced between kernel-6.19/post-0x83/0x86 labels — settle it with git.kernel.org/kernel-firmware evidence, ROCm #5724, Launchpad #2129150).
- **Floor-removal posture:** deploy now runs on any kernel — confirm acceptable given both legs are hardware-enablement (RTL8127) rather than safety; no bypass env exists to misuse.
- **MT7925:** §6 upstream-fix status; 6.17+ panic/deauth fixes assumed present on any current CachyOS kernel.
- **amdxdna:** `-EINVAL (ret -22)` probe failure under `amd_iommu=off` — errno still current; watch for a kernel making the NPU IOMMU-optional (obsoletes the blacklist).
- **Strix Halo ACP:** internal-mic ASoC/UCM still open upstream as of mid-2026; nothing to ship; upstream report is the only action.
- Mesa soft floor 26.0 vs current RADV guidance; enumerate open gfx1151 RADV issues.
- Prefer kernel/firmware floors over DKMS for any landed fix.
- Sources: git.kernel.org linux-firmware + r8169 + mt76, gitlab.freedesktop.org/drm/amd + mesa, bugzilla.kernel.org, discuss.cachyos.org, docs.kernel.org accel/amdxdna, wiki.cachyos.org.

## §11. Security & safety deltas (quantify, ordered — no auto-FIX)

nftables IPv4-only default-deny-inbound (ufw masked; ipv6.disable=1): policy drop; lo accept; ct established/related accept; ct invalid drop; IPv4 ICMP {echo-request, destination-unreachable, time-exceeded, parameter-problem} accept (inbound ping ALLOWED); forward drop; output accept; no ICMPv6/NDP. `RY_REMOTE_PLAY_PORTS` (default false) appends TCP {47984,47989,48010,27036,27037} + UDP {47998-48010,27031-27036}. Rendered ruleset passes `nft -c -f` before commit; `_post_nft` re-validates before reload; restart failure downgrades to applies-at-boot (warned).

1. **UMIP off** (`clearcpuid=umip`) — descriptor-table base leak, kernel tainted; headline open reduction.
2. **AMD-Vi fully disabled** (`amd_iommu=off`) — no DMA isolation/remapping (USB4/TB, NVMe, NIC DMA unmediated). Named casualty: XDNA 2 NPU blacklisted. Opt-back-in is one validator pair (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`) restoring isolation and NPU together; coupling asymmetry intended (`amd_iommu=on` + blacklist-true valid; false-without-IOMMU refuses).
3. **IPv6 disabled + inbound IPv4 ping accepted** — net LAN delta = +ping −mDNS (avahi masked unit+socket + resolved MulticastDNS=no close multicast discovery entirely). `_vss_nft` hard-fails on missing echo-request (regression guard); `_vrsv_nft_assert_ping` warns live; do NOT flag ping-accept as a regression.
4. **split_lock_detect=off** — a misbehaving app can degrade the system.
5. **Plaintext DNS** (DNSOverTLS=no, DNSSEC=allow-downgrade; reverts CachyOS DoH) — observable/spoofable on-path.
6. **Remote-play ports** (default OFF) — validate the TCP/UDP sets against current Sunshine/Moonlight/Steam docs; default-OFF correct.
7. **Default-deny-inbound ships** — net positive; `flush ruleset` blast radius vs docker/libvirt/podman; no ICMP/new-conn rate limit (trusted-LAN assumption — state it).

## §12. HUD & Bluetooth

**MangoHud.conf (19 active + 1 commented, 0600):** horizontal · legacy_layout=0 · position=top-left · toggle_hud=Shift_R+F12 · fps · frametime · frame_timing · gpu_stats · gpu_temp · gpu_core_clock · gpu_power · cpu_stats · `# cpu_temp intentionally disabled — enable if you want CPU temperature in the HUD` · cpu_mhz · cpu_power · vram · ram · font_size=20 · text_outline · background_alpha=0.4. Enabled via MANGOHUD=1.
**bluetooth main.conf:** FastConnectable=true · AutoEnable=true · ReconnectAttempts=3.
**Companion parity (measured, v1.17.0 ↔ generator script L925-948):** 19/19 active directives identical in set AND order; byte delta = 2 comment lines only (repo identity header vs installer managed-file header; bare `# cpu_temp` vs installer's expanded comment) — functionally nil, comments are inert to MangoHud. Repo policy: installer is source of truth (repo CHANGELOG 1.14.0 realignment). **HUD-scoped floors per repo:** MangoHud ≥ 0.8.4 (Steam-Overlay fix), kernel ≥ 6.14, Mesa 24+ — HUD floors only, do not conflate with profile floors (§10).

- **cpu_power live target:** confirm it populates from Zen 5 RAPL/hwmon under Wayland; blank/zero ⇒ FIX-to-investigate. **cpu_temp stays dormant — reconcile three records:** installer README (#1794: cpu_power reads 0 on Zen 5 while cpu_temp active), repo README v1.17.0 (k10temp Tctl present but MangoHud pickup unreliable depending on kernel/hwmon layout), repo CHANGELOG 1.15.0 ("CPU package sensor is not reported on the GTR9 Pro"). Settle the current failure mode on MangoHud ≥ 0.8.4 / kernel 6.18.x; if #1794 is fixed AND k10temp is picked up, cpu_temp exits dormancy into CPU-block slot 2 (load, temp, freq, power — repo 1.17.0 layout).
- Byte-exact checks must use the full commented string; `grep -c '^# cpu_temp'` = 1, `grep -c '^cpu_temp'` = 0, `grep -c '^cpu_power'` = 1.
- Confirm all 19 directives valid on current MangoHud; gpu_temp/gpu_core_clock/vram/cpu_mhz populate from amdgpu under Wayland; overhead near-zero with gamescope. `vram` on this UMA part reports the BIOS carveout only — `ram` is the load-bearing figure (§14 special case); a low `vram` reading is not a finding.
- BlueZ keys current; ReconnectAttempts=3 + backoff sane; AutoEnable fixes adapter-off-at-boot; complements btusb.enable_autosuspend=n.
- Sources: github flightlessmango/MangoHud (#1794, #1825), wiki.archlinux.org MangoHud + Bluetooth, mangohud-gtr9-pro v1.17.0 archive (README + CHANGELOG — lockstep policy, floors, directive history).

## §13. Candidate enhancements (absent knobs — gaming-first)

Knobs the profile does NOT set. Anchor every call to gfx1151 / Zen 5 / RDNA 3.5 / current Mesa + Proton-CachyOS. Reserve ADD-as-default for a clear, low-risk frametime/throughput win; never invent a flag; bias KEEP-omitted — the profile is intentionally lean.

**13a. Kernel cmdline**
- `mitigations=off` — KEEP-omitted. Zen 5 unaffected by Inception/SRSO-class issues at hardware/microcode level; no measured gaming benefit. Re-open as ADD-opt-in only on a published gfx1151 Proton frametime delta > ~2%. IMPACT Low · RISK Med.
- `amdgpu.ppfeaturemask=0xffffffff` — KEEP-omitted. Undervolt/OC unimplemented on gfx1151 (overdrive/power-cap unsupported, ROCm #5750); CPU undervolt via ryzenadj is the real lever (out of scope). IMPACT Low · RISK Med.
- `preempt=full` — KEEP-omitted, redundant (CachyOS boots full; CONFIG_PREEMPT_DYNAMIC=y).
- `nvme_core.io_timeout` / `pcie_port_pm=off` — KEEP-omitted unless §5 ASPM research shows `pcie_aspm=off` leaves port PM active in a way that matters; else redundant beside ps_max_latency=0.

**13b. RADV / Mesa env**
- `RADV_PERFTEST` — KEEP-omitted (gpl default-on since 23.1; sam auto-on when all VRAM CPU-visible) / UNCERTAIN (nggc — no gfx1151 benchmark).
- `RADV_DEBUG` correctness toggles — KEEP-omitted unless a live gfx1151 rendering bug requires one.
- `MESA_VK_WSI_PRESENT_MODE` / `vblank_mode` / `mesa_glthread=true` — KEEP-omitted (per-game / GL-only).

**13c. DXVK / VKD3D-Proton**
- dxvk.conf — KEEP-omitted (GPL default-on; numCompilerThreads auto). Legacy DXVK_ASYNC superseded (gplAsyncCache removed in DXVK 2.7) — never recommend the old async patch.
- Additional `VKD3D_CONFIG` flags beyond the shipped `descriptor_heap` — evaluate per §1; per-game flags stay per-game.
- Upscaler envs beyond §1 — KEEP-omitted (`PROTON_FSR4_RDNA3_UPGRADE=1` + per-title DXIL workaround is the shipped scope).

**13d. Firmware / platform (verify-only)**
- Resizable BAR / SAM — verify-only, auto-on (all VRAM CPU-visible; RADV auto-enables sam). Optional INFO via rocminfo / lspci BAR.
- BIOS UMA carveout vs GTT — KEEP-omitted for gaming (GTT ~62 GiB never bottlenecks a game; carveout is compute-oriented).
- BIOS power ceiling — verify-only: README prescribes flat SPL=fPPT=sPPT=85 W + TjMax 90 (gains flatten past ~85 W; STAPM zeroed). Installer-external; the only action is consistency — every §2/§13 power statement names its assumed budget.

**13e. Scheduler / memory**
- `read_ahead_kb` / `nr_requests` — KEEP-omitted, defaults optimal absent evidence (§4).
- `vm.max_map_count` — KEEP (sufficient; SteamOS value).
- CPU isolation (`isolcpus`, `nohz_full`, `rcu_nocbs`) — KEEP-omitted (hurts a 16C/32T gaming desktop).

## §14. Scope, protected items, special cases

**Scope:** recommendations only — do not emit a modified script. Out of scope: dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS, laptops, UKI, BIOS flashing (README link-out only). Per-game Proton tuning secondary to system-wide config.

**Protected (deliberately removed/disabled — do not recommend reinstating unless current upstream directly contradicts the rationale; then flag, not FIX):**
- `pcie_aspm.policy=performance` (superseded 7.102.x by `pcie_aspm=off` — evaluate the semantics in §5, do not blind-revert).
- `mt7925e disable_aspm=1` module option / standalone `60-ry-mt7925e.conf` (removed 7.102.x — covered by global ASPM-off).
- `KERNEL_MIN` + `_ir_validate_kernel_floor` hard floor (removed 7.105.x — do not recommend re-enforcement; advisory comment is the mechanism).
- `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` (removed 7.98.x) · `RY_NO_NTP_REMEDIATION` (removed 7.96/97 — escape is masking timesyncd) · `clearcpuid=514` numeric form (renamed 7.94/95) · `archlinux-contrib` (removed 7.101.0) · `60-ry-blacklist-amdxdna.conf` standalone (merged 7.99.0).
- `amdgpu.ppfeaturemask`, `--country` flag, TTM/GTT cap, RADV drirc, MangoHud repo-history removals — `fps_metrics` (added 1.10.0, dropped 1.13.0), `gpu_junction_temp` (hotspot mirrors the edge value), `throttling_status`(+`_graph`) (removed twice), `gpu_mem_clock` + `swap` (meaningless on a shared-memory APU) — do not re-propose without new evidence, `vm.page-cluster`/`vm.vfs_cache_pressure` (vendor-provided), ntsync autoload conf (assert-only), baloofilerc, `_kb_*` subs + `_ry_check_umip_disabled`, ICMPv6/NDP rules (do NOT re-add without restoring IPv6), the linux-firmware version advisory.

**Live config to evaluate KEEP-or-FIX-to-remove (not protected):** `VKD3D_CONFIG=descriptor_heap`, `PROTON_FSR4_RDNA3_UPGRADE`, `vm.watermark_boost_factor=0`, MangoHud gpu_power/text_outline/toggle_hud/cpu_power, `ipv6.disable=1`, inbound-ping accept, `BLACKLIST_AMDXDNA=true` default (evaluate the NPU-off default, not the mechanism). `cpu_temp` stays a user opt-in.

**Special cases:**
- **IOMMU:** ships `amd_iommu=off`. Do NOT recommend `iommu=pt`/`amd_iommu=on` as default unless ROCm on gfx1151 provably requires it (it does not) OR a DMA-isolation requirement is established; opt-in is per-user and validator-enforced.
- **PCIe ASPM:** ships `pcie_aspm=off`. Do NOT flag the change from policy=performance as a regression without link-state evidence (§5, question a); if `off` leaves links ASPM-enabled on this board, that is a FIX with lspci proof.
- **IPv6/nftables:** ships `ipv6.disable=1` + IPv4-only ruleset accepting inbound ping. Do NOT flag ping-accept (asserted regression guard); do NOT re-add ICMPv6/NDP without restoring IPv6.
- **Governor/EPP:** powersave + balance_performance is the EPP-honoring config under amd_pstate=active — do not flag powersave without proving `performance` would honor the hint.
- **GPU_DPM_LEVEL:** `auto` is deliberate; flag only with gfx1151 frametime/1%-low evidence for `high` under the named budget; pre-v7.94/95 observations are void.
- **HUD `vram` on UMA:** reports only the BIOS carveout, never the 128 GB pool — do NOT flag a low `vram` reading as misconfiguration; `ram` carries the shared-pool load (§12, companion repo).

## §15. VERIFY block (post-reboot)

```fish
cat /proc/cmdline
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver          # amd-pstate-epp
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor        # powersave
cat /sys/devices/system/cpu/cpu0/cpufreq/energy_performance_preference   # balance_performance
cat /sys/devices/system/cpu/amd_pstate/status                    # active
cat /sys/devices/system/cpu/amd_pstate/dynamic_epp               # disabled (absent pre-6.16)
cat /sys/devices/system/cpu/amd_pstate/prefcore                  # enabled
cat /sys/devices/system/cpu/cpufreq/boost                        # 1
cat /sys/devices/system/clocksource/clocksource0/current_clocksource   # tsc — INFORMATIONAL ONLY
cat /sys/block/nvme0n1/queue/scheduler                           # [none] (adjust node)
cat /sys/class/drm/card*/device/power_dpm_force_performance_level      # auto (GPU_DPM_LEVEL)
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l  # 0 — INFORMATIONAL ONLY
rg -o 'amd_iommu=\S+' /proc/cmdline                              # amd_iommu=off
rg -o 'ipv6\.disable=\S+' /proc/cmdline                          # ipv6.disable=1
rg -o 'clearcpuid=\S+' /proc/cmdline                             # clearcpuid=umip
rg -o 'pcie_aspm=\S+' /proc/cmdline                              # pcie_aspm=off
rg -o 'processor.max_cstate=\S+' /proc/cmdline                   # 1
rg -o 'fsck\S+' /proc/cmdline                                    # fsck.mode=force fsck.repair=yes
sudo lspci -vv | rg 'LnkCtl:.*ASPM'                              # §5 (a): audit per-link ASPM state under pcie_aspm=off
ls -l /dev/ntsync                                                # present (assert-only)
lsmod | rg -c '^amdxdna'                                         # 0 (blacklisted; loaded = verify FAIL)
sudo dmesg | rg -i 'AMD-Vi|DMAR'                                 # expect NO "AMD-Vi: Enabled"
ip -6 addr                                                       # expect no IPv6 addresses
cat /etc/modprobe.d/60-ry-modules.conf                           # header + amdxdna blacklist (default; NO mt7925e line)
ls /etc/modprobe.d/60-ry-mt7925e.conf /etc/modprobe.d/60-ry-blacklist-amdxdna.conf 2>/dev/null   # ENOENT both (pre-7.99 leftovers; ZERO in-script/README guard — §6 gap)
pacman -Q linux-firmware                                         # §10 MES 0x86 currency check (no version gate)
pacman -Q pacman-contrib                                         # present
vulkaninfo | rg -i 'driverName|deviceName'                       # RADV / Radeon 8060S; confirm uma heap
systemctl --user show-environment | rg 'VKD3D_CONFIG|PROTON_FSR4|MANGOHUD'   # descriptor_heap / 1 / 1
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc vm.max_map_count vm.compaction_proactiveness vm.swappiness vm.watermark_boost_factor
findmnt -no OPTIONS /                                            # noatime,lazytime,commit=10
swapon --show; zramctl                                           # zram active (advisory; not managed)
iw reg get | rg -i country                                       # US
cat /etc/iw-regdomain                                            # COUNTRY=US
sudo nft list chain inet filter input                            # policy drop + lo + est/rel + IPv4 ICMP incl echo-request; +ports IFF RY_REMOTE_PLAY_PORTS=true; no ICMPv6/NDP
sudo nft -c -f /etc/nftables.conf                                # syntax-valid
stat -c '%a %U:%G' /etc/NetworkManager/system-connections/*      # 0600 root:root
systemctl is-enabled bluetooth.service                           # enabled
systemctl is-enabled avahi-daemon.service avahi-daemon.socket    # masked masked
grep -c '^cpu_temp' ~/.config/MangoHud/MangoHud.conf             # 0
grep -c '^cpu_power' ~/.config/MangoHud/MangoHud.conf            # 1
grep -c '^# cpu_temp' ~/.config/MangoHud/MangoHud.conf           # 1 (byte-exact checks use the full string)
grep -c '^[a-z]' ~/.config/MangoHud/MangoHud.conf                # 19 active (companion v1.17.0 parity: set + order identical)
cat /sys/class/hwmon/hwmon*/name | rg -c '^k10temp$'             # ≥1 — P0 #15 sensor presence — INFORMATIONAL ONLY
sensors 2>/dev/null | rg -i -A1 '^k10temp'                       # Tctl populates? — §12 cpu_temp pickup evidence — INFORMATIONAL ONLY
```

**Hard `--verify` asserts (mismatch → exit 1/3):** every KERNEL_PARAMS token + `rw` in /proc/cmdline (`_vrk_cmdline` generic loop); scaling_driver/governor/EPP/amd_pstate status/prefcore/boost/`dynamic_epp=disabled`; GPU `power_dpm_force_performance_level=$GPU_DPM_LEVEL` (comparison QUOTED); usbcore.autosuspend=-1, nvme_core ps_max_latency=0, zswap∈{N,0}, nmi_watchdog=0, NVMe `[none]`; managed modprobe blacklist entries NOT loaded (`_vrkm_blacklist_modprobe`); live mkinitcpio COMPRESSION/_OPTIONS match; regdom; nftables echo-request present (`_vss_nft` hard guard) + live warn (`_vrsv_nft_assert_ping`); NM system-connections 0600 root:root; PKGS_ADD 18 + Vulkan pkgs (`_vsp_required`). Presence checks comment-proof (`_chk_grep` strips inline comments).
**REMOVED asserts — do NOT verify:** `_vrkm_iommu`, `_vrk_clocksource`, `_vre_zram`, `_vre_tcp` (gone since 7.90.0); no THP, KSM, `ttm.*`, drirc, `iommu=pt`, ICMPv6/NDP, baloo, `_kb_*`, kernel-floor, or mt7925e-option assert exists.

---

# Appendix A — install-phase model (validate sequence, not prose)

```
1 Preflight     _install_preflight          — _ir_* gates (counts 21, keys incl BLACKLIST_AMDXDNA + charsets/metachar, post-hooks, root UUID); mesa soft floor. NO kernel-floor gate.
2 Packages      _install_packages           — mkinitcpio.conf pre-deployed → pacman -Syu; PKGS_ADD re-marked -D --asexplicit; chwd Vulkan
3 Configuration _install_system_files       — render+deploy 17 files (atomic tmp+rename); format-validate pre-write; nftables additionally nft -c
4 Services      _install_configure_services — fstab → resolved → PKGS_DEL (-Rns) → mask (nft-first, ufw flush; MASK 12) → iwd handoff → enable → regdom → NTP (chronyd/ntpd/openntpd guard) → RTC write-back
5 Boot          _install_rebuild_boot       — taint-gate → mkinitcpio -P → sdboot-manage gen/update (gated on boot-critical writes)
6 Finalize      _install_finalize           — user daemon-reload → paccache (-rk2, -ruk0) → NetworkManager restart
```

- Firewall handoff lives in Phase 4 (nftables live before ufw flushed/masked); Phase-5 regeneration fires only when a `_RY_BOOT_CRITICAL_DSTS` member changed. Flag any recommendation moving a cmdline/mkinitcpio change outside the Phase-5 gate.
- `_RY_BOOT_CRITICAL_DSTS` (4) = `_RY_BACKUP_TARGETS` (derived, count-asserted): /boot/loader/loader.conf, /etc/kernel/cmdline, /etc/sdboot-manage.conf, /etc/mkinitcpio.conf — all get `.ry.bak` + post-write verify/restore (plus fstab during rewrite). Preflight refuses a side-effecting generator in the backup set (sysctl.d guard).
- `_RY_POST_HOOKS` (17 entries, 16 distinct tags; dispatch FIRST-MATCH-WINS by list order — ordering is load-bearing): `/boot/*|loader`, `/etc/kernel/cmdline|cmdline`, `/etc/sdboot-manage.conf|boot`, `/etc/mkinitcpio.conf|boot`, `*/resolved.conf.d/*|resolved`, `*/logind.conf.d/*|logind`, `*/NetworkManager-dispatcher.service.d/*|nmdispatch`, `*/NetworkManager/conf.d/*|nm`, `/etc/iw-regdomain|regdom`, `/etc/bluetooth/main.conf|bluetooth`, `/etc/nftables.conf|nft`, `/etc/default/cpupower-service.conf|cpupower`, `*/sysctl.d/*|sysctl`, `/etc/udev/rules.d/*|udev`, `*/modprobe.d/*|modprobe`, `*/environment.d/*|envd`, `*/MangoHud/MangoHud.conf|mangohud`. `_ir_validate_post_hooks` refuses any tag lacking `_post_<tag>`.
- Boot family shares `_post_boot_apply <target> <skip_mki>`: `_post_boot` → skip_mki=false (full mkinitcpio -P); `_post_cmdline`/`_post_loader` → true (sdboot regen only). Notify-only: `_post_logind`, `_post_modprobe` (reboot), `_post_envd`/`_post_mangohud` (session). `_post_nm` DEFERS the NM restart when Wi-Fi is the active route — confirm the deferred restart lands (Phase 6) and is surfaced.
- All destinations + `--install-file` values canonicalized via `realpath -m` at load (failure warns, falls back literal). Unmatched patterns log `POST_HOOK_NONE`; unchanged bytes log `POST_HOOK_SKIP_UNCHANGED`; `_post_udev` runs `udevadm verify` (systemd ≥254) before reload + retrigger (block AND cpu).

# Appendix B — exact rendered bodies (validate content, not paraphrase)

Every generator emits a leading `#` header line — byte-exact/checksum comparisons include it.

### B1. /etc/kernel/cmdline + /etc/sdboot-manage.conf + /etc/mkinitcpio.conf
```
rw root=UUID=<_ROOT_UUID> 8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm=off processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0
```
```
# sdboot-manage configuration — changes require: sudo sdboot-manage gen && sudo sdboot-manage update
LINUX_OPTIONS="8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm=off processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0"
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
- `clearcpuid=umip` (and every token) ships in BOTH bootloader paths — confirm no conflict and state which one CachyOS drives.
- Fallback entry strips ALL params (B1 exposure: kernel-default IOMMU + IPv6 on; amdxdna blacklist remains) — §7.

Scalar map: `root=UUID` ⇐ `_ROOT_UUID` (machine-specific — placeholder retained) · `LINUX_OPTIONS` ⇐ `KERNEL_PARAMS` join (literal above).

### B2. /boot/loader/loader.conf
```
# systemd-boot loader configuration
default @saved
timeout 0
console-mode keep
editor no
```
- Confirm `@saved` resolves; failed-boot menu reachability vs live-USB recovery; loader.conf changes regenerate entries only.

### B3. /etc/nftables.conf (validate rule-by-rule)
```
#!/usr/bin/nft -f
# ry-install: default-deny-inbound, IPv4-only (ufw masked; ipv6.disable=1). Add inbound ports below.
flush ruleset
table inet filter {
    chain input {
        type filter hook input priority filter; policy drop;
        iif "lo" accept
        ct state established,related accept
        ct state invalid drop
        # IPv4 ICMP: inbound ping (echo-request) + error/PMTUD types (replies match ct established)
        icmp type { echo-request, destination-unreachable, time-exceeded, parameter-problem } accept
        # [IFF RY_REMOTE_PLAY_PORTS=true:]
        # ry-install: remote-play inbound (RY_REMOTE_PLAY_PORTS=true)
        tcp dport { 47984, 47989, 48010, 27036, 27037 } accept
        udp dport { 47998-48010, 27031-27036 } accept
    }
    chain forward { type filter hook forward priority filter; policy drop; }
    chain output { type filter hook output priority filter; policy accept; }
}
```
- Rule order lo → established/related → invalid-drop cannot drop valid traffic; echo-request accept is the regression guard; destination-unreachable preserves PMTUD; `flush ruleset` blast radius vs docker/libvirt/podman; no rate limit (trusted LAN — state it); `nft -c` pre-commit + `_post_nft` re-validate close the malformed-ruleset window (same-binary check-then-load); restart failure surfaces as applies-at-boot warn.

### B4. /etc/udev/rules.d/99-ry-perf.rules
```
# ry-install: udev performance rules (managed file, do not edit by hand)
# NVMe scheduler none (lowest tail latency; diverges from CachyOS kyber default)
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
# AMD P-State EPP balance_performance (perf-leaning CPPC hint)
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="balance_performance"
# GPU performance level (gfx1151 clock-floor; optional)
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="auto"
```
- GPU rule is `add`-only (no re-assert after GPU reset — robustness argument for `auto`); EPP value enum-bounded; filename 99- sorts after vendor 60-ioschedulers.rules (last ATTR wins) — confirm CachyOS still defaults kyber.

Scalar map: EPP ⇐ `EPP_PREFERENCE` (=balance_performance) · DPM ⇐ `GPU_DPM_LEVEL` (=auto) — literals above are the rendered defaults.

### B5. /etc/systemd/resolved.conf.d/99-cachyos-resolved.conf
```
# systemd-resolved: plaintext DNS, mDNS/LLMNR off (diverges from CachyOS DoH default)
[Resolve]
MulticastDNS=no
LLMNR=no
DNSOverTLS=no
DNSSEC=allow-downgrade
```
- Same-basename replace (not merge) caution if CachyOS ships the identical filename; restarts skipped when bytes unchanged; §11 privacy flag stands.

### B6. NetworkManager 99-cachyos-nm.conf + dispatcher logging.conf
```
# NetworkManager configuration - wpa_supplicant backend
[device]
wifi.backend=wpa_supplicant

[connection]
wifi.powersave=2

[logging]
level=WARN

# LogLevelMax drops info-level dispatcher lines (journald-logged; StandardError=null ineffective)
[Service]
LogLevelMax=notice
```
- Same basename-override caution as B5; confirm LogLevelMax=notice remains the correct journald-noise fix.

### B7. /etc/iw-regdomain
```
# ry-install: wireless regulatory domain (managed file, do not edit by hand)
COUNTRY=US
```
- Most version-fragile external: confirm `cachyos-iw-set-regdomain` (or successor) still exists and reads this file at boot; if dropped, the file is inert and the profile must switch mechanisms.

### B8. /etc/bluetooth/main.conf + /etc/default/cpupower-service.conf
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
GOVERNOR='powersave'
```
- Single verify: that consumer script path + `GOVERNOR` var name on current CachyOS cpupower packaging (if moved, the file is inert and the governor falls to kernel default; the udev EPP rule still applies). `_vrsv_chk_cpupower_governor` asserts the running governor.

### B9. ~/.config/MangoHud/MangoHud.conf (19 active + 1 commented + file header)
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
Parity vs companion `mangohud-gtr9-pro` v1.17.0 `MangoHud.conf` (measured): identical except line 1 (repo identity header) and the `# cpu_temp` comment wording — all 19 active directives byte- and order-identical.

### B10. /etc/modprobe.d/60-ry-modules.conf (CHANGED 7.102.x — amdxdna-only)
Default (`BLACKLIST_AMDXDNA=true`):
```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# blacklist amdxdna: XDNA NPU needs IOMMU, probes -EINVAL (ret -22) under amd_iommu=off
blacklist amdxdna
```
NPU path (`BLACKLIST_AMDXDNA=false`, validator-coupled to IOMMU on):
```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# no directives: BLACKLIST_AMDXDNA=false (NPU path) and MT7925 ASPM now covered by pcie_aspm=off
```
- `_grep_modprobe_entry` accepts a comment-only file (7.102.x); `_vss_modprobe` greps the blacklist only when default-true; `_vrkm_blacklist_modprobe` asserts amdxdna NOT loaded. **The §6 leftover gap applies here: generator-sourced checks cannot see stale pre-7.99 drop-ins, and no README guard remains.**

### B11. /etc/systemd/logind.conf.d/99-cachyos-logind.conf (LOGIND_IGNORE_KEYS 8)
```
# systemd-logind configuration - desktop power handling
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

### B12. /etc/sysctl.d/95-ry-overrides.conf (SYSCTL_VALUES 10 — §3/§6)
```
# ry-install sysctl tunables (priority 95 — loaded after CachyOS vendor 70-cachyos-settings.conf)
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

### B13. ~/.config/environment.d/10-environment.conf (ENV_VARS 12 — §1; 0600)
```
# Environment for systemd --user services and graphical sessions (Plasma, Flatpak, D-Bus apps)
AMD_VULKAN_ICD=RADV
DXVK_LOG_LEVEL=none
DXVK_LOG_PATH=none
MANGOHUD=1
MESA_SHADER_CACHE_MAX_SIZE=16G
PROTON_ENABLE_WAYLAND=1
PROTON_FSR4_RDNA3_UPGRADE=1
PROTON_LOCAL_SHADER_CACHE=1
VKD3D_CONFIG=descriptor_heap
VKD3D_DEBUG=none
VKD3D_SHADER_DEBUG=none
WINEDEBUG=-all
```

# Appendix C — verify surface (assert ownership per recommendation)

A recommendation that changes a value MUST state which sub asserts it (hard-fail vs warn).

- **Static boot** `_verify_static_boot` → `_vsb_loader` · `_vsb_sdboot` (LINUX_OPTIONS token set + keys) · `_vsb_cmdline` (/etc/kernel/cmdline token set — amd_iommu=off / ipv6.disable=1 / clearcpuid=umip / pcie_aspm=off / tsc=reliable byte-asserted here) · `_vsb_mkinitcpio` (HOOKS/MODULES + live COMPRESSION/_OPTIONS via `_ry_mkinitcpio_array`, multi-line join, last-wins warn) · `_vsb_entries` (BLS entries + count; loader-entry paths realpath-canonicalized, WARNED textual-join downgrade when realpath absent). All hard-fail.
- **Static system** `_verify_static_system` → `_vss_logind` · `_vss_nmdispatch` · `_vss_nm` · `_vss_sysctl` · `_vss_regdom` · `_vss_bluetooth` · resolved inline `_chk_grep` loop · cpupower inline `_chk_grep GOVERNOR` · `_vss_udev` (all 3 rules; EPP from `$EPP_PREFERENCE`; GPU_DPM-aware) · `_vss_modprobe` (blacklist grep iff default-true) · `_vss_nft` (hard-fail on missing echo-request).
- **Static user** — ENV_VARS per-var `_chk_grep` + MangoHud `_chk_file` + `_chk_grep "fps"`. `_chk_grep` is comment-proof (awk strips inline comments, skips comment-only lines before `grep -wF`); "no non-comment lines" is a distinct FAIL; mid-read sudo lapse is a distinct warn.
- **Static packages** → `_vsp_required` (PKGS_ADD 18 + Vulkan, pacman-db-lock guard) · `_vsp_removed` · `_vsp_pacman_conf` (sudo-read fallback; grep rc>1 → warn-skip) · `_verify_static_services` (MASK 12) · `_verify_static_syntax` · `_verify_static_checksum` → `_vsc_check_one` (embedded SHA256 == installed; graceful skip on EXIT_GEN_NOUUID).
- **Pre-deploy format validators** (`_ry_validate_configs` → `_rvc_dispatch`): `_grep_kv`, `_grep_kparam`, `_grep_sysctl_kv`, `_grep_modprobe_entry` (comment-only OK; else every non-comment line ∈ options/blacklist/install/alias/softdep/remove), `_grep_regdomain_entry`, `_grep_udev_entry`, `_grep_nft_entry`, `_grep_envd_entry`, `_grep_cpupower_entry`, `_grep_mangohud_entry`, `_grep_ini_header`; mkinitcpio case REQUIRES `MODULES=(`, `HOOKS=(`, `COMPRESSION="` lines; `_vmh_*` = mkinitcpio hook validators (NOT MangoHud); nftables additionally `nft -c -f` on the rendered tmpfile.
- **Runtime kernel** `_verify_runtime_kparams`: `_vrk_cmdline` (every token + rw; preemption INFO from one cached `sudo -n dmesg`) · `_vrk_gpu_state` (QUOTED compare) · `_vrk_cpu_state` (driver/governor/EPP/status/dynamic_epp/prefcore/boost) · `_vrk_module_state` → `_vrkm_amdgpu` (hex-aware, no-op without amdgpu.*) · `_vrkm_blacklist` (module_blacklist= cmdline scan — currently no-op) · `_vrkm_blacklist_modprobe` (managed-content parse, `-`→`_` normalize, lsmod check; amdxdna LOADED ⇒ FAIL; lsmod absent ⇒ warn) · usbcore/nvme_core/zswap/nmi_watchdog/NVMe-none asserts.
- **Runtime services** `_verify_runtime_services`: `_vrsv_chk_active_enabled` · `_vrsv_nft_assert_ping` (warn) · `_vrsv_chk_nftables` (oneshot judged by live policy drop) · `_vrsv_chk_resolved` · `_vrsv_chk_cpupower_governor` · `_vrsv_sys_units` · `_vrsv_wifi_nm_backend` · `_vrsv_wifi` (no iwd path; skips when `_RY_PROFILE_USES_WIFI_BACKEND=false`; wlan via /sys/class/net/*/wireless; closes with firewall-posture INFO) · `_vrsv_masked_inactive` (covers the avahi pair).
- **Runtime env** `_verify_runtime_env`: `_vre_envvars` (systemctl --user show-environment) · `_vre_sysctl_runtime` (/proc/sys) · `_vre_fstab` (ext4 noatime,lazytime,commit=10) · `_vre_ntsync` · `_vre_regdom` (iw reg get).
- **Runtime session** `_verify_runtime_session`: `_vrs_nm_perms` (0600 root:root) · `_vrs_installed_file_perms` (system 0644 / user 0600) · `_vrs_parent_dirs` → `_vpd_dir_perm_check` (0755/0700); `_vrs_vfat_skip` guards BOTH loops (vfat/undetermined $BOOT counted-skipped with INFO).
- **Aggregation** `_ry_verify_all`/`_verify_summary`: static first, runtime second; per-stage summaries summed; runtime preflight bail restores static totals; a static FAIL outranks the runtime bail code. Confirm no path zeroes static counters after a runtime bail.
- **Actionables:** (a) leftover-file blindness of `_vrkm_blacklist_modprobe` (§6 — CONFIRMED, now unguarded); (b) COMPRESSION multi-line/duplicate tolerance without false FAIL; (c) comment-strip safety holds while no managed value contains `#` (boot-scalar metachar gate forbids it) — re-check on any new value; (d) removed effect-asserts leave directive-level coverage intact; iwd narrowing deliberate (Low/Low).

# Appendix D — fstab rewrite (`_install_fstab_opts`)

- Adds `noatime,lazytime,commit=10` to ext4 field 4 only; every other column and non-ext4 row byte-preserved; purely-numeric $4 rows pass through to the malformed guard — confirm they are then caught, not shipped.
- Verify-side conflict list exact: `defaults`, `relatime`, `atime`, `strictatime` (presence = rewrite-pending FAIL); existing `commit=` rewritten to 10; non-10 overrides tracked in `_RY_FSTAB_COMMIT_OVERRIDES` (surfaced).
- Gates: line-count parity + size floor + mandatory `findmnt --verify`; symlinked or whitespace-split /etc/fstab refused, not corrected.
- Confirm: ext4-only (not vfat ESP / btrfs / xfs); idempotent; atomic (tmp+rename, `.ry.bak`); commit=10 vs every-boot fsck coherence (§4).

# Appendix E — preflight gates & exit codes

Init-time capability probes FIRST: `id` (hard-require, non-numeric `id -u` refuses) → `timeout --foreground --kill-after` → `find -maxdepth/-printf` → `mv -T` live-probe (two mktemp files, /tmp — vfat semantics untested by design) → `stat` → `date %z`; each rejects busybox/uutils (exit 3). TMPDIR erased (tmp pinned /tmp); umask set as the VARIABLE; `--check` silence pinned pre-argparse. Dependency gate: 33-command GNU set + `df --output` probe + systemd ≥250; optional tools warn-listed. Destinations canonicalized (`realpath -m`, literal fallback).

- `_ir_resolve_root_uuid` → EXIT_GEN_NOUUID 12; mode-scoped: `--install-file` FATAL only when target IS /etc/kernel/cmdline; else warn-continue; `--verify` warn-continues with generic root=UUID check.
- Hardware gate (CPU match; sole override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed unreadable; `--verify` warns).
- `_ir_validate_counts` (21 tripwires) → `_ir_validate_keys` (bool/yes-no/int enums; ISO-3166 COUNTRY with reserved-range rejection; GPU_DPM ∈ `_RY_DPM_LEVELS`; EPP ∈ `_RY_EPP_LEVELS`; governor regex; nftables↔ipv6.disable coupling; BLACKLIST_AMDXDNA=false↔IOMMU-on coupling; non-empty scalars; boot-scalar metachar gate; MKINITCPIO_COMPRESSION_OPTIONS charset `^-?[A-Za-z0-9]+$`; KERNEL_PARAMS charset `^[A-Za-z0-9._,=-]+$`) → `_ir_validate_post_hooks`. **No kernel-floor validator exists.**
- Confirm: (a) counts/keys run BEFORE any disk write; (b) bypass inventory is exactly ONE env; (c) `PACTREE_TIMEOUT_S=60`, `BOOT_SPACE_CRIT/WARN` 200/500 MB, `ROOT_AVAIL_CRIT/WARN` 2/5 GiB sane vs multiple kernels + fallback + zstd -1 image (§7).

Exit-code contract (audit for discipline — no bare `exit 1` collapsing 3/4/5/10):

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

11–14/250/251/255 are annotated in-script as internal-only sentinels (never a process exit; checksum verify maps NOUUID to graceful skip). Confirm none can reach a process exit.

# Appendices G–L — robustness & correctness (safety invariants; FIX applies normally)

Audit whether the installer is safe to run at all on current fish (3.6 floor) / CachyOS. Flag any TOCTOU, fail-open, or partial-write window. Any GAP in G/H/J is release-blocking and outranks every tuning finding.

**G. Atomic writes (`_awf_*`):** render-to-tmp (tee `$pipestatus` split-checked; generator fail → EXIT_GEN_*) → (nftables: `nft -c` tmpfile gate) → symlink-swap probe (rc 0/1/2; abort on swap) → chmod → `sudo -n true` re-assert → `mv -T` (tmp in dst parent, same-FS; capability probed on /tmp — vfat /boot rename semantics stand) → backup targets add `.ry.bak` pre + post-write re-read/restore (tri-state rc; `string collect --no-trim-newlines --allow-empty` preserves trailing newlines; cmdline UUID-dependence → NOUUID graceful skip). Non-backup files detect-only on mismatch (bounded by nft -c for the ruleset; a semantically-wrong ruleset still deploys — residual, state it). Confirm no probe pipes a fish builtin into an external reader (SIGPIPE capture-drop) and generator determinism for all four backups.

**H. Instance lock (`_acquire_lock*`):** atomic `mkdir` (umask 0077, 0700) + pidfile via mktemp+`mv -Tf` (0600); in-script rationale comment: mkdir+pidfile, not flock — atomic on any fs, no fd inheritance into sudo children, stale-pid reclaim — confirm each leg holds on fish 3.6+. PID-recycle: /proc/PID/stat field-22 starttime + /proc/stat btime; divisor `getconf CLK_TCK` fallback USER_HZ=100 (correct by ABI); reclaim only if holder start > pidfile mtime + 2 s; unparseable ⇒ live, refuse; garbage pidfile settle 0.2 s → re-read → refuse; bounded 3 attempts, fail-closed. Re-read-before-rm guard; symlinked LOCK_DIR refused; `--preserve-root`; kill -0 EPERM ⇒ /proc liveness. Confirm the `^.*\) ` comm-strip survives `) ` in comm and field indexing after it.

**I. Privilege handling (`_as`, `_run`):** `sudo -n` everywhere; one TTY-gated `sudo -v` prompt, non-TTY refuses (no mid-run hang); credential re-asserted before each critical write — confirm the once-prompt cannot recur. Tri-state rc 0/1/2 (drift vs lapse) in `_is_symlink`/`_installed_bytes`/`_ry_content_bytes` — audit every caller branches on 2 (incl. `_vsp_pacman_conf`). `_run` timeout: default 3600 s, >9-digit clamp 2147483647, invalid → default, 0 disables; long ops (pacman, mkinitcpio, sdboot-manage, paccache, updatedb, pkgfile; PATH-resolved, sudo value-flag skip list, `env`/`VAR=` prefixes skipped) FLOORED to 7200 s — confirm 7200 covers worst-case `-Syu` and a cap-kill is fatal-with-rollback. Capture hygiene: argv redacts /tmp/ry-* → [REDACTED]; overflow inlines bytes+sha256 + awk keyword scan of the elided middle (≤10 hits, ≤2000 chars; nothing on disk).

**J. Boot-wipe gate & rollback:** `_irb_taint_gate` (taint OR failed mkinitcpio revert ⇒ skip rebuild, exit 4; `_taint` sets INSTALL_HAD_ERRORS + _RY_BOOT_TAINTED together — confirm every boot-critical write failure routes through it) → `mkinitcpio -P` fail ⇒ exit 4 → `$BOOT` resolved BEFORE the vfat gate; unresolved or non-vfat + REMOVE_EXISTING=yes ⇒ refuse wipe (exit 4) → `_preflight_boot_sanity` (vmlinuz + initramfs + entries or exit 4). Confirm: no path to REMOVE_EXISTING=yes with unverified $BOOT; the Phase-3 cmdline-write → Phase-5 rebuild window is covered by the param-stripped fallback (B1 asymmetry); EXIT_BOOT_CRIT terminal (no Finalize). mkinitcpio rollback: pre-Syu snapshot /run/ry-install (0700, mktemp, tagged) → `_mkinitcpio_revert` same-FS mktemp + byte-exact `cmp` + atomic mv; duplicate KEY= resolves last (matches `_ry_mkinitcpio_array`); tmpfs snapshot same-boot-only (acceptable). Flag if a pacman partial transaction can desync mkinitcpio.conf from installed modules without triggering revert.

**K. Signal & exit teardown:** INT/TERM/HUP/QUIT/ABRT with explicit 128+N map (HUP:129 INT:130 QUIT:131 TERM:143 ABRT:134; unknown→130); idempotent; SIGPIPE marks output broken, JSONL continues; fish_exit prefers _INTENDED_EXIT_CODE → _RY_INSTALL_LAST_EXIT → $status. `--check` stderr-silence holds through the PRE-ARGPARSE window (root + --check-only ⇒ silent exit 3) — confirm no early-init path prints to stderr under a pure --check argv. umask set as the VARIABLE (autoload-race safe) — confirm fish ≥3.6 honors it for children and mkdir. Cleanup order: kill children (TERM to -P $fish_pid descendants; 0.5 s grace, 10 s under db.lck; then KILL; missing pgrep degrades flat 0.5 s) → mkinitcpio revert → tmpfile sweep (`_RY_TMPDIR_GLOBS` 6, PID-scoped ry-*.$fish_pid.* — confirm glob set == created set) → fs sweep → lock release LAST → erase globals. Log lifecycle: mv -T with cp -pT + rm recovery; both-fail keeps old path (warn); symlinked LOG_FILE removed, recreated 0600; root-guard `@@LEFT@@`/`@@IF@@` display sentinels never leak unstripped. SIGKILL is the only cleanup bypass (stale-reclaim §H recovers).

**L. pacman transaction safety:** full `-Syu --needed` only; retry `-Syyu --needed`; second failure fatal; SYSTEM_UPGRADED via `pacman -Q | sha256sum` fingerprint (empty fails open to true). db.lck pre-checked, never removed; teardown reaper is $fish_pid-scoped (peer pacman untouchable) with 10 s grace under db.lck. PKGS_DEL `-Rns` rdep-aware (external dependants skip + `_RY_PKG_REMOVE_SKIPS`); paccache -rk2/-ruk0 separate; pactree-absent pre-Phase-2 warns only (`PACTREE_MISSING`). Confirm no `-S <pkg>` runs outside `-yu` context and db.lck is checked before any package op.

**ROBUSTNESS verdict (required, separate):** per §G–§L PASS / GAP / UNCERTAIN; GAPs in G/H/J surface first; correctness has no "deliberate trade-off" defense.

---

## Sources

docs.kernel.org (kernel-parameters, PCIe/ASPM, amd-pstate, sysctl/vm, networking, block, ext4, UMIP, AMD-Vi, accel/amdxdna, /proc/stat) · git.kernel.org (linux-firmware, r8169, mt76, wireless-regdb) · gitlab.freedesktop.org (mesa, drm/amd) · docs.mesa3d.org · github.com (HansKristian-Work/vkd3d-proton, CachyOS/proton-cachyos, flightlessmango/MangoHud, LizardByte/Sunshine, moonlight-stream) · wiki.archlinux.org (AMDGPU, IOMMU, fsck, Gaming, PipeWire, Zram, SSD, Ext4, Sysctl, NetworkManager, Wireless, nftables, Security, Mkinitcpio, systemd-boot, Bluetooth, System_time, CPU_frequency_scaling, MangoHud, pacman) · wiki.cachyos.org · discuss.cachyos.org · bugzilla.kernel.org · man.archlinux.org (nft, avahi-daemon, systemd.unit, logind.conf, hwclock, timesyncd) · man7.org (mkdir/rename(2), proc(5), sysconf) · fishshell.com/docs · amd.com ROCm · archlinux.org/packages · mangohud-gtr9-pro v1.17.0 companion archive (MangoHud.conf + README + CHANGELOG — HUD lockstep, floors, directive history). Cite access dates + exact versions in the methodology block.
