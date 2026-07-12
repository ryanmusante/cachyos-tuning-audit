# cachyos-tuning-audit — ry-install Tuning (CachyOS · Beelink GTR9 Pro)

**Target:** `ry-install.fish` **v7.101.0** (attached, 2026-07-12).
**Source of truth:** script > README > CHANGELOG. Every value below is re-derived from the script; where a prior prompt disagrees, the script wins and the disagreement is flagged.

**Platform:** Beelink GTR9 Pro · Ryzen AI Max+ 395 "Strix Halo" (Zen 5, 16C/32T, gfx1151) · Radeon 8060S (40 RDNA 3.5 CUs) · XDNA 2 NPU · 128 GB LPDDR5X-8000 unified (≤96 GB as VRAM) · dual M.2 NVMe (ext4) · dual 10 GbE (RTL8127) + Wi-Fi 7 (MT7925) + BT 5.4 · 140 W TDP · CachyOS · systemd-boot.

---

## 0. Read first — audit target moved v7.99.1 → v7.101.0

The prior revision of this document audited **v7.99.1**. The attached script is **v7.101.0**. Between the two releases the maintainer resolved nearly every open finding the v7.99.1 audit raised. This document is re-derived against v7.101.0 and the resolved items are marked **[CLOSED in-script]**; do not re-audit them.

| # | v7.99.1 finding | Status in v7.101.0 | Evidence (script line) |
|---|---|---|---|
| 1 | **MES-0x86 label** — script-only, unreconciled; the audit's top item | **[CLOSED]** relabelled `post-0x83 MES`, with `0x83 reverted upstream 2025-12-01`; RTL8127 + suspend fixes explicitly `land <=6.18` | 15, 797, 809 |
| 2 | `archlinux-contrib` — installed but never invoked (KEEP-vs-remove open) | **[CLOSED]** removed; `PKGS_ADD` 19 → **18** | 665, 733 |
| 3 | SYSCTL trailing comment `netdev=2.5GbE` vs 10 GbE platform | **[CLOSED]** comment now `netdev=10GbE (RTL8127)` | 644 |
| 4 | NTP conflict guard scans only chronyd/ntpd (`openntpd.service` gap) | **[CLOSED]** guard now scans all three incl. `openntpd.service` | 1706 |
| 5 | `_verify_runtime_session` `--description` still says "Vulkan packages" | **[CLOSED]** description now `NM connection perms, installed-file perms, parent dirs` | 3280 |

The remaining open items below are genuinely still open in v7.101.0 (chiefly the modprobe-leftover migration gap, §8, and the intentional trade-offs the profile makes by design).

### Counts (all hard-asserted by `_ir_validate_counts`, 21 tripwires)

KERNEL_PARAMS 17 · MKINITCPIO_HOOKS 11 · MKINITCPIO_MODULES 1 · LOGIND_IGNORE_KEYS 8 · ENV_VARS 11 · SYSCTL_VALUES 9 · **PKGS_ADD 18** · PKGS_DEL 9 · MASK 12 · EXPECTED_VULKAN_PKGS 2 · EXPECTED_SERVICES 5 · _RY_PKG_MANAGED_SERVICES 1 · _RY_POST_HOOKS 17 · _RY_ARGPARSE_SPEC 7 · _RY_BOOT_CRITICAL_DSTS 4 · _RY_PHASE_NAMES 6 · _RY_BACKUP_TARGETS 4 · _RY_TMPDIR_GLOBS 6 · SYSTEM_DESTINATIONS 15 · USER_DESTINATIONS 2 · MKINITCPIO_COMPRESSION_OPTIONS 2.

Managed files = **17** (15 system + 2 user; `_RY_MANAGED_FILE_COUNT`), recomputed at load — a mismatch against the constant refuses (exit 3).

### Hard floors

- **KERNEL_MIN 6.19 — UNCONDITIONAL** (deploy and `--check` exit 3; `--verify` warns and continues). No bypass exists (`RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` removed v7.98.x). Rationale: **gfx1151 post-0x83 MES amdgpu** stability. The RTL8127 r8169 support (`f24f7b2f3af9`) and suspend/shutdown-hang fix (`ae1737e7339b`) are stated to land ≤6.18 (below the floor) and are therefore guaranteed, not floor rationale.
- **CPU gate** `Ryzen AI Max` — override `RY_INSTALL_SKIP_HARDWARE_CHECK=1` (the sole skip env; fail-closed on unreadable model).
- **Mesa < 26.0** — soft warn only; `vercmp` output validated `^-?\d+$` before the compare (garbage logs `MESA_SOFT_FLOOR_SKIP`).

The **only** runtime env inputs to the script are `RY_RUN_TIMEOUT`, `RY_INSTALL_SKIP_HARDWARE_CHECK`, and `NO_COLOR`. Every profile toggle (`BLACKLIST_AMDXDNA`, `NM_WIFI_BACKEND`, `COUNTRY`, `GPU_DPM_LEVEL`, `EPP_PREFERENCE`, `RY_REMOTE_PLAY_PORTS`) is an embedded scalar set with unconditional `set -g` — an exported env var of the same name is clobbered. Opting in means editing the script, by design.

### MangoHud posture (unchanged)

19 active directives + 1 commented (`# cpu_temp intentionally disabled — enable if you want CPU temperature in the HUD`). Composition and order are byte-identical to v7.91.0; only the commented line's text differs from the bare `# cpu_temp` of older revisions. Prefix-greps still match; byte-exact checks must use the full string (§B9).

---

## Mission

Evaluate every config decision against current upstream for this exact silicon and return a prioritized, evidence-backed tuning report. The profile deliberately trades PCI-passthrough, the XDNA NPU, power-saving, IPv6, and host inbound-firewalling-of-ping for performance, latency, and a simpler IPv4-only ruleset. Confirm each choice is current and correct, surface anything superseded or harmful, and quantify safety deltas without second-guessing intentional design.

## Rules

1. Item-by-item, hardware-anchored to gfx1151 / Zen 5 / RDNA 3.5 / CachyOS / 128 GB unified / dual 10 GbE.
2. Respect deliberate trade-offs: **flag and quantify, do not auto-FIX.** Reserve FIX for incorrect, superseded, deprecated, or harmful values.
3. Rate IMPACT × RISK (High/Med/Low). Default KEEP when impact is marginal and risk is non-trivial.
4. Never invent params, flags, keys, options, or URLs. Cite a source or mark UNCERTAIN.
5. Flag every source conflict and name the trusted side. (No cross-source conflicts remain open in v7.101.0 — the MES label and the SYSCTL comment were both resolved in-script; see §0.)
6. Give exact versions (kernel / Mesa / linux-firmware / pkg) and exact before→after, mapped to the in-script global.

## Output

- **Findings matrix** (box-drawn, code-fenced, grouped by section): ITEM · CURRENT · CALL (KEEP/TUNE/FIX/UNCERTAIN) · RECOMMENDED · IMPACT · RISK · EVIDENCE.
- **Candidate-enhancement matrix** (§13, separate): ITEM · PRESENT?(no) · CALL (ADD-default/ADD-opt-in/KEEP-omitted) · IMPACT · RISK · EVIDENCE.
- **Before→after** for each TUNE/FIX/ADD: exact current string, exact replacement, in-script global.
- **VERIFY block** (post-reboot commands, below).
- **Security delta vs CachyOS defaults** (ordered, below).
- **Verdict:** one per section (OPTIMAL/TUNE/FIX) plus overall (PASS/PASS-WITH-FIXES/FAIL).
- **ROBUSTNESS verdict** (§G–§L, separate from tuning).
- **Methodology:** source list with access dates and versions; unknowns marked UNCERTAIN.

### VERIFY block

```fish
cat /proc/cmdline
cat /sys/.../cpu0/cpufreq/scaling_driver                    # amd-pstate-epp (EXPECTED_SCALING_DRIVER)
cat /sys/.../cpu0/cpufreq/scaling_governor                  # powersave (EPP-honoring under pstate=active)
cat /sys/.../cpu0/cpufreq/energy_performance_preference     # balance_performance (EPP_PREFERENCE)
cat /sys/.../amd_pstate/status                              # active
cat /sys/.../amd_pstate/dynamic_epp                         # disabled (else EPP pin -EBUSY; absent pre-6.16)
cat /sys/.../amd_pstate/prefcore                            # enabled
cat /sys/.../cpufreq/boost                                  # 1
cat /sys/.../clocksource0/current_clocksource               # tsc — INFORMATIONAL ONLY (no _vrk_clocksource since v7.90.0)
cat /sys/block/nvme0n1/queue/scheduler                      # [none] (adjust node)
cat /sys/class/drm/card*/device/power_dpm_force_performance_level   # auto (GPU_DPM_LEVEL; udev rule LIVE only since the v7.94/95 ENV{DEVTYPE} fix)
find /sys/kernel/iommu_groups -mindepth 1 -maxdepth 1 -type d | wc -l   # 0 — INFORMATIONAL ONLY (no _vrkm_iommu since v7.90.0)
cat /proc/cmdline | rg -o 'amd_iommu=\S+'                   # amd_iommu=off
cat /proc/cmdline | rg -o 'ipv6\.disable=\S+'               # ipv6.disable=1
cat /proc/cmdline | rg -o 'clearcpuid=\S+'                  # clearcpuid=umip
cat /proc/cmdline | rg -o 'processor.max_cstate=\S+'        # 1
cat /proc/cmdline | rg -o 'fsck\S+'                         # fsck.mode=force fsck.repair=yes
ls -l /dev/ntsync                                           # present (assert-only)
lsmod | rg -c '^amdxdna'                                    # 0 (blacklisted; loaded = verify FAIL, _vrkm_blacklist_modprobe)
sudo dmesg | rg -i 'AMD-Vi|DMAR'                            # expect NO "AMD-Vi: Enabled" (amd_iommu=off)
ip -6 addr                                                  # expect no IPv6 addresses (ipv6.disable=1)
cat /etc/modprobe.d/60-ry-modules.conf                      # options mt7925e disable_aspm=1 + blacklist amdxdna (merged, v7.99.0)
ls /etc/modprobe.d/60-ry-mt7925e.conf /etc/modprobe.d/60-ry-blacklist-amdxdna.conf 2>/dev/null   # ENOENT both (pre-7.99 leftovers — remove once, README note)
pacman -Q linux-firmware                                    # currency check only (no version gate)
pacman -Q pacman-contrib                                    # present (PKGS_ADD 18)
vulkaninfo | rg -i 'driverName|deviceName'                 # RADV / Radeon 8060S; confirm uma heap
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc vm.max_map_count vm.compaction_proactiveness vm.swappiness
findmnt -no OPTIONS /                                       # noatime,lazytime,commit=10
swapon --show; zramctl                                      # zram active (advisory; not managed, not asserted)
iw reg get | rg -i country                                 # US
cat /etc/iw-regdomain                                       # COUNTRY=US
sudo nft list chain inet filter input                      # policy drop + lo + established/related + IPv4 ICMP incl echo-request; +remote-play ports IFF RY_REMOTE_PLAY_PORTS=true. No ICMPv6/NDP.
sudo nft -c -f /etc/nftables.conf                          # syntax-valid (deploy/reload refuses a failing ruleset, v7.96/97)
stat -c '%a %U:%G' /etc/NetworkManager/system-connections/* # 0600 root:root
systemctl is-enabled bluetooth.service                     # enabled
systemctl is-enabled avahi-daemon.service avahi-daemon.socket   # masked masked (MASK 12)
printenv MANGOHUD                                          # 1
grep -c '^cpu_temp' ~/.config/MangoHud/MangoHud.conf       # 0 (commented)
grep -c '^cpu_power' ~/.config/MangoHud/MangoHud.conf      # 1 (live)
grep -c '^# cpu_temp' ~/.config/MangoHud/MangoHud.conf     # 1 (expanded comment; byte-exact checks use the full string, §B9)
```

**Hard `--verify` asserts** (mismatch → exit 1/3): every `KERNEL_PARAMS` token present in `/proc/cmdline` + `rw` (generic loop in `_vrk_cmdline`); scaling_driver=`$EXPECTED_SCALING_DRIVER`, scaling_governor=`$CPUPOWER_GOVERNOR`, EPP=`$EPP_PREFERENCE`, amd_pstate status/prefcore/boost, `dynamic_epp=disabled`; GPU `power_dpm_force_performance_level=$GPU_DPM_LEVEL` (comparison QUOTED); usbcore.autosuspend=-1, nvme_core ps_max_latency=0, zswap∈{N,0}, nmi_watchdog=0, NVMe `[none]`; managed modprobe blacklist entries NOT loaded (`_vrkm_blacklist_modprobe`); live mkinitcpio `COMPRESSION=`/`COMPRESSION_OPTIONS` match (`_vsb_mkinitcpio` via `_ry_mkinitcpio_array`, multi-line-join, last-wins); regdom; nftables IPv4 ping accept present (`_vss_nft` hard static regression guard) + live warn (`_vrsv_nft_assert_ping`); mt7925e `disable_aspm=1`; NM system-connections 0600 root:root; PKGS_ADD (18) + Vulkan pkgs via `_vsp_required`. Presence checks are comment-proof (`_chk_grep` strips inline comments).

**REMOVED asserts — do NOT verify** (gone since v7.90.0): `_vrkm_iommu` (0-iommu-groups), `_vrk_clocksource` (HPET-fail/TSC-demotion), `_vre_zram`, `_vre_tcp`. No THP, KSM, `ttm.*`, drirc, `iommu=pt`, ICMPv6/NDP, baloo, or `_kb_*` assert exists.

### Security delta (ordered)

1. **UMIP off** (`clearcpuid=umip`) — descriptor-table base leak, kernel tainted; headline open reduction. Name form is version-stable; exposure is identical to the numeric form.
2. **AMD-Vi fully disabled** (`amd_iommu=off`) — no DMA isolation/remapping; any DMA-capable device (USB4/Thunderbolt, NVMe, NIC) can in principle DMA over system RAM unmediated. Named functional casualty: the **XDNA 2 NPU is blacklisted** (amdxdna probes `-EINVAL` without the IOMMU; `BLACKLIST_AMDXDNA=true` default). Validator-enforced opt-back-in (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`) restores DMA isolation and the NPU together. Open reduction.
3. **IPv6 disabled + inbound IPv4 ping allowed** (net wash-to-slight-reduction) — `ipv6.disable=1` removes the whole IPv6 stack; the ruleset accepts inbound `echo-request` (discoverability up slightly). Counterweight: **avahi masked (unit+socket)** removes the second mDNS responder, so with resolved's `MulticastDNS=no` multicast discovery is closed entirely. Net LAN delta = +ping −mDNS.
4. **split_lock_detect=off** — a misbehaving app can degrade the system.
5. **Plaintext DNS** (`DNSOverTLS=no`, `DNSSEC=allow-downgrade`) reverting the CachyOS DoH default — DNS observable and spoofable on-path.
6. **Optional inbound remote-play ports** (`RY_REMOTE_PLAY_PORTS`, default OFF) — when enabled, opens TCP 47984/47989/48010/27036/27037 + UDP 47998-48010/27031-27036.
7. **Firewall default-deny-inbound ships** (nftables IPv4-only; lo + established/related + IPv4 diagnostic ICMP incl. inbound ping; all else dropped) — net positive, now with a `nft -c` pre-commit gate so a malformed managed ruleset cannot replace a working one.

---

## Investigation (§1–§12 by installer phase; §13 = candidate enhancements)

### 1. Platform baseline and version floors

**Current:** kernel floor 6.19 unconditional (deploy/`--check` exit 3, `--verify` warns; no override). CPU gate `Ryzen AI Max` (override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`). Soft Mesa < 26.0 warn. No firmware-version advisory.

- **6.19 floor — verify the single leg.** Sole rationale is gfx1151 post-0x83 MES amdgpu ≥6.19. Confirm (a) that claim upstream; (b) `f24f7b2f3af9` + `ae1737e7339b` land ≤6.18 as asserted; (c) the true gfx1151-stability floor isn't above 6.19. State per-subsystem floors.
- **MES label — post-0x83, resolved in-script.** Was "0x86" in v7.99.1; now `post-0x83` (`0x83 reverted upstream 2025-12-01`). Verify the shipping GC 11.5.1 MES revision matches (git.kernel.org/kernel-firmware, ROCm #5724, Launchpad #2129150) — do not re-open.
- **Floor-override removal:** a 6.18 snapshot cannot deploy (`--verify` still runs). Confirm the posture vs the misuse the override carried (README documents no workaround — correct).
- Confirm soft Mesa 26.0 matches current RADV guidance; enumerate open gfx1151 RADV issues. Confirm gfx1151 reports `uma:1` natively.
- **README BIOS posture:** flat `SPL=fPPT=sPPT=85 W` + `TjMax 90` (gains flatten past ~85 W). Installer-external, but every §6/§13d power call must name its budget (85 W README vs 140 W stock).
- Sources: wiki.cachyos.org, docs.kernel.org gpu/amdgpu, gitlab.freedesktop.org/mesa, git.kernel.org linux-firmware + r8169.

### 2. Packages

**PKGS_ADD (18):** nvme-cli, cachyos-gaming-meta, cachyos-gaming-applications, lib32-mesa, mkinitcpio-firmware, fd, sd, dust, procs, bottom, htop, git-delta, lm_sensors, rtkit, realtime-privileges, ddcutil, nftables, pacman-contrib.
**PKGS_DEL (9, `-Rns`, rdep-aware):** plymouth stack (×5) + micro + cachyos-micro-settings + cachy-update + kdeconnect.
**AUR:** none. **Vulkan (chwd):** vulkan-radeon, lib32-vulkan-radeon.

- **pacman-contrib — KEEP.** Script invokes `pactree` (`PACTREE_TIMEOUT_S=60`) + `paccache` (`-rk2`, `-ruk0`); declaring the provider closes a formerly-assumed dependency. (`archlinux-contrib` was removed in v7.101.0 — the v7.99.1 "never invoked" finding is closed.)
- **`-D --asexplicit` re-mark (install-reason, not ordering):** post-`Syu`, PKGS_ADD members are re-marked so the Phase-4 `-Rns` can't orphan-cascade one that pre-existed as a dependency. Idempotent; failure warns (`PKG_ASEXPLICIT_FAIL`).
- Confirm the gaming metas supply RADV/Proton/gamescope/MangoHud/GameMode. **GameMode omission — KEEP** (governor/EPP/DPM pinned profile-wide; its governor switch is redundant); confirm the meta's MangoHud doesn't clash with the shipped conf.
- `rtkit` + realtime-privileges for PipeWire priority (rtkit-daemon socket-activated); `lib32-mesa` still needed beside `lib32-vulkan-radeon`; PKGS_DEL dependency fallout (`_RY_PKG_REMOVE_SKIPS`).
- Advisory: does znver/x86-64-v4 (AVX-512) benefit this build over v3?
- Sources: wiki.cachyos.org, wiki.archlinux.org/Gaming + PipeWire + RealtimeKit, archlinux.org/packages.

### 3. Kernel cmdline (17)

```
8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0
```

- **`clearcpuid=umip`:** UMIP off, kernel tainted; name form version-stable (numeric bit-index is layout-fragile). Confirm `umip` name accepted on ≥6.19; README says drop if no `umip_printk` stutter. Asserted generically (`_vrk_cmdline`/`_vsb_cmdline`), no UMIP-specific check.
- **`amd_iommu=off`:** validator-paired to `BLACKLIST_AMDXDNA` (see §10). ROCm unaffected (`IOMMU Support: None`); NPU is the named casualty. Weigh marginal latency vs DMA-isolation loss vs NPU loss.
- **`ipv6.disable=1`:** hard-coupled to the IPv4-only nftables ruleset. LAN impact, Steam/Proton netcode fallback, README dual-stack opt-out.
- **processor.max_cstate=1:** idle power/thermal (name budget) vs wake-latency/jitter; conflict with boost headroom? is `1` right?
- **btusb.enable_autosuspend=n:** MT7925/BT reconnect fix; overlap with `usbcore.autosuspend=-1`?
- **fsck.mode=force + fsck.repair=yes:** every-boot fsck on ext4 — boot cost, auto-repair safety, hook handshake, durability vs periodic.
- **amd_pstate=active:** recommended on Zen 5; interaction with powersave + EPP + `dynamic_epp=disabled`.
- **split_lock_detect=off:** perf vs stability; blast radius.
- No `preempt=` — KEEP-omitted (CachyOS boots full; `_vrk_cmdline` INFOs the model, `_ok` on `full`). Zero amdgpu/ttm params — hands-off (`_vrkm_amdgpu` no-ops without `amdgpu.*`).
- Validate: tsc=reliable, nowatchdog, 8250.nr_uarts=0, usbcore.autosuspend=-1, nvme_core ps_max_latency=0, pcie_aspm.policy=performance, zswap.enabled=0.
- **Input hygiene:** tokens charset-gated `^[A-Za-z0-9._,=-]+$`; a new token outside it must also change the validator.
- Sources: docs.kernel.org kernel-parameters + amd-pstate + UMIP + IOMMU + ipv6-sysctl, wiki.archlinux.org/AMDGPU + IOMMU + fsck, amd.com ROCm.

### 4. Bootloader and initramfs

**loader.conf:** default @saved, timeout 0, console-mode keep, editor no.
**sdboot-manage:** DEFAULT_ENTRY manual, OVERWRITE/REMOVE_EXISTING/REMOVE_OBSOLETE yes, LINUX_FALLBACK_OPTIONS "quiet".
**mkinitcpio:** MODULES=(amdgpu), HOOKS (11) base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck, COMPRESSION zstd (-1 -T0), explicit BINARIES=()/FILES=(). Pre-deployed in Phase 2 → one rebuild at `-Syu`.

- Verify HOOKS order (systemd/microcode/kms/sd-vconsole/block); amdgpu early-KMS; fsck-hook handshake with no boot prompt.
- **COMPRESSION_OPTIONS=(-1 -T0):** zstd -1 all-threads. Quantify boot-decompress vs default-3 (sub-100 ms on NVMe) and image size vs ESP `BOOT_SPACE_*` (200/500 MB) with multiple kernels + fallback. TUNE to default-3 if size threatens the budget. Tokens charset-gated + count-asserted — any TUNE updates both.
- **Live verify:** `_vsb_mkinitcpio` compares live `COMPRESSION=`/`COMPRESSION_OPTIONS` (via `_ry_mkinitcpio_array`, multi-line join, last-wins) — drift caught. Confirm last-wins matches shell-sourcing.
- timeout 0 + manual + REMOVE_EXISTING=yes wipes foreign BLS entries; confirm recovery ergonomics (live-USB → chroot) intended. sdboot-manage current vs kernel-install/UKI (UKI out of scope).
- Sources: wiki.archlinux.org/Mkinitcpio + systemd-boot, sdboot-manage upstream.

### 5. GPU / Vulkan / gaming

No drirc (uma:1 native), no ttm/modprobe GPU params.
**ENV_VARS (11):** AMD_VULKAN_ICD=RADV, DXVK_LOG_LEVEL=none, DXVK_LOG_PATH=none, MANGOHUD=1, MESA_SHADER_CACHE_MAX_SIZE=16G, PROTON_ENABLE_WAYLAND=1, PROTON_FSR4_RDNA3_UPGRADE=1, PROTON_LOCAL_SHADER_CACHE=1, VKD3D_DEBUG=none, VKD3D_SHADER_DEBUG=none, WINEDEBUG=-all.

- **ntsync (assert-only):** no autoload conf; `_vre_ntsync` + `_ntsync_state` (builtin|loaded|loaded_nodev|missing) survive; README ≥6.14 + `PROTON_NO_NTSYNC=1` opt-out. Confirm: current vs esync/fsync; CachyOS `CONFIG_NTSYNC=y` (so `/dev/ntsync` without autoload); `loaded_nodev` still a real failure; Proton frametime benefit on 16C/32T; opt-out current. Floor subsumes ntsync's requirement.
- **PROTON_FSR4_RDNA3_UPGRADE=1:** confirm current Proton-CachyOS consumes it for FSR3.1→FSR4 on RDNA 3.5, the min version, and the `DXIL_SPIRV_CONFIG=wmma_rdna3_workaround` companion; no-op on non-FSR titles. Unverified ⇒ FIX-to-remove; verified ⇒ KEEP + cite.
- **RADV heap (drirc removed):** confirm uma:1 on current Mesa.
- **GTT (ttm removed):** kernel auto-sizes (~62 GiB); README routes >62 GiB single allocs to BIOS UMA carveout (≤96 GB), not deprecated `amdgpu.gttsize`; verify `/sys/module/ttm/parameters/pages_limit`. `amd_iommu=off` doesn't change the ceiling.
- PROTON_ENABLE_WAYLAND maturity/fallback; RADV vs VK_DRIVER_FILES; shader-cache sizing; MANGOHUD=1 overhead, clean with gamescope/GameMode.
- **XDNA NPU:** blacklisted default (§8/§10) — no gaming impact; future LLM/NPU work needs the opt-in.
- Sources: docs.mesa3d.org (RADV, APU heap), gitlab.freedesktop.org/mesa + drm/amd, github Proton-CachyOS, amd.com ROCm.

### 6. CPU performance and power

amd_pstate=active; governor **powersave** (`CPUPOWER_GOVERNOR`); EPP **balance_performance** via udev (`$EPP_PREFERENCE`, enum-gated `_RY_EPP_LEVELS`); **GPU_DPM_LEVEL=auto** (add-only udev, `ENV{DEVTYPE}=="drm_minor"`); `EXPECTED_SCALING_DRIVER=amd-pstate-epp`; `dynamic_epp=disabled` asserted. Masked: power-profiles-daemon, ananicy-cpp, modemmanager.

- **governor=powersave + EPP=balance_performance — special case, do not flag the governor.** Under amd_pstate=active, `powersave` honors EPP (dynamic) while `performance` pins max and ignores it — this triple is the documented EPP-honoring max-perf config on Zen 5. `balance_performance`→`performance` stays **UNCERTAIN** (no gfx1151/Zen-5 frametime data); any change is the `EPP_PREFERENCE` global only (enum-clean).
- **GPU_DPM_LEVEL=auto:** enum-gated `_RY_DPM_LEVELS`. Re-evaluate `auto` vs `high` for frametime/1%-lows on the shared budget — two inertia facts: rule is add-only (no re-assert after GPU reset) AND only began firing at the v7.94/95 matcher fix (pre-fix "high made no difference" is void). Name the budget.
- Confirm EPP live-applies (`_post_udev`: `udevadm verify` ≥254, reload + retrigger cpu/block) and GPU rule matches at enumeration. prefcore + boost=1 on Strix Halo; `dynamic_epp` node ≥6.16. Masks (ppd/ananicy-cpp/modemmanager) safe on current CachyOS.
- Sources: docs.kernel.org amd-pstate, wiki.archlinux.org/CPU_frequency_scaling + AMDGPU, freedesktop ppd.

### 7. Memory and storage

zswap.enabled=0; NVMe scheduler none (udev, sorts after vendor 60-ioschedulers.rules).
**SYSCTL_VALUES (9):** default_qdisc=fq, netdev_budget=600, netdev_budget_usecs=5000, tcp_congestion_control=bbr, tcp_notsent_lowat=16384, tcp_slow_start_after_idle=0, vm.compaction_proactiveness=0, vm.max_map_count=2147483642, vm.swappiness=150 (priority 95, after vendor 70-cachyos-settings.conf).
fstab ext4 noatime,lazytime,commit=10. THP/KSM/oomd left to CachyOS; vm.page-cluster + vm.vfs_cache_pressure dropped (vendor duplicates).

- (The v7.99.1 `netdev=2.5GbE` comment conflict is closed — now `netdev=10GbE (RTL8127)`, matching the platform.)
- **netdev_budget/usecs on 10 GbE:** confirm the 600/5000 pair is sized for dual 10 GbE, or propose values with driver evidence.
- **Vendor-duplicate drop a no-op?** Confirm 70-cachyos-settings.conf sets page-cluster=0 + vfs_cache_pressure=50 (a differing default makes the drop a silent change).
- zram: swappiness 150 on 128 GB — gratuitous or LLM-reclaim-helpful? zswap=0 vs CachyOS zram (no double compression).
- NVMe none vs mq-deadline/kyber; `nr_requests`/`read_ahead_kb` unset — propose ATTRs only with evidence, else defaults optimal; confirm 99- sorts after 60-.
- noatime+lazytime coexistence; commit=10 vs every-boot fsck (§3); fstrim.timer vs continuous discard. max_map_count (MAX_INT−5) Proton/anti-cheat; compaction_proactiveness=0 for large unified allocs; oomd disabled on 128 GB.
- Sources: docs.kernel.org (block, sysctl/vm), wiki.archlinux.org/Zram + SSD + Ext4.

### 8. Network and latency

sysctl net (§7); IPv6 disabled (§3); nftables IPv4-only (§10). NM: wifi.backend=wpa_supplicant, wifi.powersave=2 (off), logging WARN.
**Modprobe merged (v7.99.0):** `60-ry-modules.conf` = `options mt7925e disable_aspm=1` + conditional `blacklist amdxdna` (default on).
resolved: MulticastDNS=no, LLMNR=no, DNSOverTLS=no, DNSSEC=allow-downgrade (plaintext; diverges from CachyOS DoH). regdom US. Masked: NetworkManager-wait-online, modemmanager, avahi-daemon.service + .socket. Enabled: NetworkManager.

- **⚠ OPEN GAP (Low/Low) — modprobe-leftover migration.** The only material open finding in v7.101.0. Superseded `60-ry-mt7925e.conf` / `60-ry-blacklist-amdxdna.conf` have **zero in-script references**; `_vrkm_blacklist_modprobe` is generator-sourced (checks the intended blacklist, not on-disk), so a pre-7.99 leftover is invisible to every verify path. Only the README `sudo rm` note guards it — a stale `60-ry-blacklist-amdxdna.conf` keeps the NPU blacklisted after opt-in, undetected. ADD-check candidate.
- **avahi masked:** confirm no host dependency (printer/`.local` discovery) and that unit+socket is complete closure (no D-Bus resurrection).
- **mt7925e disable_aspm=1:** still the correct MT7925 mitigation (coredump/BT-reconnect/assoc); has an upstream mt76 fix landed (file comment: "drop when upstream fixes")? redundancy vs `pcie_aspm.policy=performance`.
- **amdxdna blacklist:** confirm `-EINVAL (ret -22)` is the real probe failure under `amd_iommu=off`; blacklist-vs-alternatives; the fail-closed coupling (§10).
- NM wpa_supplicant vs iwd parity; wifi.powersave=2 for mt76 latency; iwd opt-in intact. bbr+fq / BBRv3 status; 10 GbE netdev + tcp_rmem/wmem/ring for line rate. mDNS+plaintext DNS privacy reduction (§10); same-basename replace caution (§B5/B6). regdom US: MT7925 TX/channel on current wireless-regdb, 6 GHz AFC, non-US hand-edit.
- Sources: docs.kernel.org/networking, wiki.archlinux.org/Sysctl + NetworkManager + Wireless, git.kernel.org wireless-regdb + mt76, man.archlinux.org avahi-daemon.

### 9. systemd units, time-sync

**Mask (12):** ananicy-cpp, power-profiles-daemon, NetworkManager-wait-online, ufw, modemmanager, avahi-daemon.service + .socket, sleep/suspend/hibernate/hybrid-sleep/suspend-then-hibernate targets.
**Enable (5):** fstrim.timer, NetworkManager, cpupower, nftables, bluetooth.
**Not enabled:** oomd (intentional), NetworkManager-dispatcher + rtkit-daemon (socket-activated). iwd untouched; ufw flushed after nftables live.
**NTP unconditional** (`RY_NO_NTP_REMEDIATION` removed): `_ry_check_time_sync` scans chronyd/ntpd/openntpd and REFUSES to enable timesyncd if any is active ("two NTP clients would conflict"); else enables timesyncd, re-checks after 2 s, runs `_ry_rtc_writeback` on sync. (The v7.99.1 openntpd scan-gap is closed.)

- Each mask safe/beneficial on CachyOS: ananicy-cpp + ppd (§6); modemmanager (no cellular); avahi (§8); sleep/suspend masked = no suspend (always-on mini-PC).
- **NTP escape:** opt-out-env removal acceptable (remediation is warn-only, non-fatal; a user can mask timesyncd — state that escape).
- **RTC write-back:** `--systohc --utc` safe; `RTCInLocalTZ=yes` defer branch correct; no ownership conflict with timesyncd.
- nftables ufw-flush-then-mask leaves no unfirewalled window (mask skipped if nft not live); oneshot judged by live ruleset; `nft -c`-gated at deploy + `_post_nft`. fstrim.timer vs continuous discard; cpupower vs CachyOS freq mgmt. logind Handle*Key=ignore (8 keys incl LongPress) — intended, no lockout.
- Sources: man.archlinux.org (systemd.unit, logind.conf, hwclock, timesyncd, avahi-daemon), wiki.archlinux.org (Bluetooth, System time).

### 10. Security and safety (cross-cutting)

nftables **IPv4-only** default-deny-inbound (ufw masked; `ipv6.disable=1`): policy drop, lo accept (first), ct established/related accept, ct invalid drop, IPv4 ICMP `{ echo-request, destination-unreachable, time-exceeded, parameter-problem }` accept (inbound ping ALLOWED), forward drop, output accept. No ICMPv6/NDP. `RY_REMOTE_PLAY_PORTS` (default false) appends TCP `{47984,47989,48010,27036,27037}` + UDP `{47998-48010,27031-27036}`. amd_iommu=off, clearcpuid=umip, split_lock_detect=off. Hard gate: rendered ruleset passes `nft -c -f` before commit; `_post_nft` re-validates before reload.

- **amd_iommu=off (#2 reduction):** quantify DMA-isolation loss (USB4/TB, NVMe, NIC). Named cost: XDNA 2 NPU blacklisted. Opt-back-in is one validator pair (`BLACKLIST_AMDXDNA=false` + `amd_iommu=on iommu=pt`) restoring isolation AND NPU together. Confirm the coupling asymmetry is intended — `amd_iommu=on` + `BLACKLIST_AMDXDNA=true` is valid; only `false`-without-IOMMU refuses. ROCm unaffected.
- **IPv6 off + ping accepted:** quantify both directions; `_ir_validate_keys` coupling holds. Net LAN delta = +ping −mDNS (avahi masked).
- **RY_REMOTE_PLAY_PORTS:** validate TCP 47984/47989/48010 (Sunshine), 27036/27037 (Steam) + UDP vs current Sunshine/Moonlight/Steam; default-OFF right. Every toggle is an embedded scalar with no `set -q` guard — `set -g` clobbers exported values. Confirm the env-proof posture is intended/documented.
- **Inbound ping = REGRESSION GUARD:** `_vss_nft` hard-fails on missing `echo-request`; `_vrsv_nft_assert_ping` warns live. `destination-unreachable` preserves PMTUD.
- **`nft -c` gate:** confirm pre-commit + post-hook validate close the malformed-ruleset window (same binary → `nft -c` pass guarantees `nft -f` load); restart failure applies at boot — confirm surfaced (warn), not silent.
- `flush ruleset` blast radius vs docker/libvirt/podman. `ct invalid drop` ordering (after lo+established) can't drop valid traffic.
- Ordered security-delta (above): umip #1, amd_iommu=off+NPU #2, IPv6-off/ping-on/avahi-masked #3.
- Sources: wiki.archlinux.org (nftables, Security, IOMMU, IPv6), docs.kernel.org (split lock, UMIP, AMD-Vi), github Sunshine/Moonlight, man.archlinux.org nft(8).

### 11. Known issues and DKMS currency

- **MES page faults:** label now **post-0x83** in-script (v7.99.1 "0x86, unreconciled" resolved). Firmware-revision check is the §1 task.
- **RTL8127 throughput + suspend/shutdown hang:** in-tree r8169 (`f24f7b2f3af9`, `ae1737e7339b`); script states these land ≤6.18. Cite the exact releases; if either landed at 6.19+, the claim is FIX-doc and the floor regains a second leg. No DKMS.
- **MT7925 panics/deauth/coredump:** mitigated (`disable_aspm=1` + `btusb.enable_autosuspend=n` + wpa_supplicant); 6.17+ fixes covered by the floor. Check whether "drop when upstream fixes" is met.
- **amdxdna probe failure:** `-EINVAL (ret -22)` under `amd_iommu=off` every boot; profile blacklists it. Confirm the errno/ret current, and that no future kernel makes the NPU IOMMU-optional (obsoleting the blacklist).
- **Strix Halo ACP:** open upstream (no ACP70 internal-mic ASoC driver / UCM profile as of mid-2026); internal mic undetected; nothing to ship. Known gap; upstream board report is the action.
- Recommend a kernel/firmware floor over DKMS for any landed fix.
- Sources: gitlab.freedesktop.org/drm/amd, git.kernel.org linux-firmware + r8169 + mt76, bugzilla.kernel.org, discuss.cachyos.org, docs.kernel.org accel/amdxdna.

### 12. MangoHud, Bluetooth, and hygiene

**MangoHud.conf (19 active + 1 commented, 0600):** horizontal, legacy_layout=0, position=top-left, toggle_hud=Shift_R+F12, fps, frametime, frame_timing, gpu_stats, gpu_temp, gpu_core_clock, gpu_power, cpu_stats, `# cpu_temp intentionally disabled …`, cpu_mhz, cpu_power, vram, ram, font_size=20, text_outline, background_alpha=0.4. Enabled via MANGOHUD=1.
**bluetooth main.conf:** FastConnectable=true, AutoEnable=true, ReconnectAttempts=3. USER_DESTINATIONS = 2.

- **`cpu_power` — live target:** confirm it populates from Zen 5 RAPL/`power1_average` hwmon under Wayland; blank/zero ⇒ FIX-to-investigate. `cpu_temp` stays dormant (re-enabling re-trips MangoHud #1794, may need the sensor key).
- **Byte-exact:** checksum comparison must use the expanded comment string; `grep -c '^# cpu_temp'` still returns 1.
- Confirm all 19 directives valid; gpu_power/cpu_power populate from sensors; gpu_temp/gpu_core_clock/vram/cpu_mhz from amdgpu under Wayland; overhead near-zero with gamescope/GameMode.
- **Bluetooth:** BlueZ keys current; ReconnectAttempts=3 + backoff sane; AutoEnable fixes adapter-off-at-boot; complementary with `btusb.enable_autosuspend=n`.
- Sources: github flightlessmango/MangoHud (#1794, #1825), wiki.archlinux.org/MangoHud + Bluetooth.

### 13. Candidate enhancements (absent knobs — gaming-first)

Each item is a knob the profile does NOT set. Anchor every call to gfx1151 / Zen 5 / RDNA 3.5 / current Mesa+Proton-CachyOS. Reserve ADD-as-default for a clear, low-risk frametime/throughput win. Never invent a flag — cite upstream or mark UNCERTAIN. Bias toward KEEP-omitted; the profile is intentionally lean. (v7.92–7.101.0 added none of these.)

**13a. Kernel cmdline**
- **`mitigations=off` — KEEP-omitted.** Zen 5 unaffected by Inception/SRSO; no measured gaming benefit; residual mitigations HW/microcode-handled. Re-open as ADD-opt-in only on a published gfx1151 Proton frametime delta > ~2%. IMPACT Low · RISK Med (security).
- **`amdgpu.ppfeaturemask=0xffffffff` — KEEP-omitted.** Undervolt/OC not implemented on gfx1151 (overdrive/power-cap "Not supported", ROCm #5750); package cap shared. CPU undervolt via ryzenadj is the real lever (out of scope). IMPACT Low · RISK Med.
- **`preempt=full` — KEEP-omitted, redundant.** CachyOS desktop kernel boot-defaults full (`CONFIG_PREEMPT_DYNAMIC=y`). IMPACT none · RISK none.
- **`nvme_core.io_timeout` / `pcie_port_pm=off` — KEEP-omitted.** Redundant beside ps_max_latency=0 + aspm.policy=performance. IMPACT Low · RISK Low.

**13b. RADV / Mesa env**
- **`RADV_PERFTEST` — KEEP-omitted (gpl/sam) / UNCERTAIN (nggc).** gpl default-on since Mesa 23.1; sam auto-on when all VRAM CPU-visible (APU); nggc: no gfx1151 benchmark → UNCERTAIN. rtwave64 hurts RDNA2; ignore. IMPACT Low · RISK Low.
- **`RADV_DEBUG` correctness toggles — KEEP-omitted** unless a live gfx1151 rendering bug requires one; flag any open issue a toggle works around.
- **`MESA_VK_WSI_PRESENT_MODE` / `vblank_mode` — KEEP-omitted (per-game).**
- **`mesa_glthread=true` — KEEP-omitted.** GL-only; Proton is Vulkan-dominant. IMPACT Low · RISK Low.

**13c. DXVK / VKD3D-Proton**
- **dxvk.conf — KEEP-omitted (auto optimal).** GPL default-on; numCompilerThreads=0 auto-detects. Legacy DXVK_ASYNC superseded (gplAsyncCache removed in DXVK 2.7) — never recommend the old async patch. IMPACT Low · RISK Low.
- **Upscaler envs beyond §5 — KEEP-omitted.** PROTON_FSR4_RDNA3_UPGRADE=1 (+ per-title DXIL_SPIRV_CONFIG workaround) is the shipped scope.
- **`VKD3D_CONFIG` — KEEP-omitted (per-game).**

**13d. Firmware / platform (verify-only)**
- **Resizable BAR / SAM — verify-only, auto-on.** All VRAM CPU-visible on Strix Halo; RADV auto-enables sam. Optional INFO via rocminfo / lspci BAR / amdgpu dmesg.
- **BIOS UMA carveout vs GTT — KEEP-omitted (gaming).** Default GTT (~62 GiB) never bottlenecks a game; carveout is compute-oriented.
- **BIOS power ceiling (verify-only):** README prescribes `SPL=fPPT=sPPT=85 W` + `TjMax 90`. Installer-external; the only action is consistency — every §6/§13 power statement must name its assumed budget (85 W README vs 140 W stock), and any DPM=`high`/EPP=`performance` re-evaluation must use the 85 W case if the user follows the README. IMPACT doc-only · RISK none.

**13e. Scheduler / memory**
- **`read_ahead_kb` / `nr_requests` — KEEP-omitted, defaults optimal** absent game-load/LLM-read evidence (§7). IMPACT Low · RISK Low.
- **`vm.max_map_count` — KEEP (sufficient).** MAX_INT−5 (SteamOS value) satisfies Proton/anti-cheat.
- **CPU isolation (`isolcpus`, `nohz_full`, `rcu_nocbs`) — KEEP-omitted (wrong here).** Hurts a 16C/32T gaming desktop. IMPACT Low · RISK Med (if added).

---

## Scope and non-goals

- Recommendations only — do not emit a modified script.
- Out of scope: dotfiles, shells, editors, secrets, backups, multi-user, non-CachyOS, laptops, UKI, BIOS flashing (README link-out only).
- Per-game Proton tuning is secondary; prioritize system-wide config.

### Protected / special-case items (do not recommend reinstating)

Items deliberately removed or disabled — do not recommend reinstating unless current upstream directly contradicts the rationale (then flag, not FIX):

- `amdgpu.ppfeaturemask`, `--country` flag, TTM/GTT cap, RADV drirc, MangoHud `fps_metrics`, `vm.page-cluster`/`vm.vfs_cache_pressure` (vendor-provided), ntsync autoload conf (assert-only), baloofilerc, the `_kb_*` subs + `_ry_check_umip_disabled`, ICMPv6/NDP rules (do NOT re-add without restoring IPv6), the linux-firmware version advisory.
- `RY_INSTALL_SKIP_KERNEL_FLOOR_CHECK` (removed v7.98.x — do not recommend a floor bypass).
- `RY_NO_NTP_REMEDIATION` (removed v7.96/97 — the escape is masking timesyncd).
- `60-ry-mt7925e.conf` / `60-ry-blacklist-amdxdna.conf` as standalone files (merged v7.99.0).
- `clearcpuid=514` numeric form (renamed v7.94/95 — never revert to the bit index).
- `archlinux-contrib` (removed v7.101.0 — do not recommend re-adding).

Live config to evaluate as KEEP-or-FIX-to-remove (not protected): PROTON_FSR4_RDNA3_UPGRADE, MangoHud gpu_power/text_outline/toggle_hud/cpu_power, `ipv6.disable=1`, inbound-ping accept, `BLACKLIST_AMDXDNA=true` default (evaluate the NPU-off default, not the mechanism). `cpu_temp` stays a user opt-in.

**Special cases:**
- **IOMMU:** ships `amd_iommu=off`. Do NOT recommend `iommu=pt`/`amd_iommu=on` as default unless ROCm on gfx1151 is proven to require it (it is not) OR a DMA-isolation requirement is established. The opt-in is per-user and validator-enforced.
- **IPv6/nftables:** ships `ipv6.disable=1` + IPv4-only ruleset accepting inbound ping. Do NOT flag ping-accept as a regression (asserted regression-guard); do NOT re-add ICMPv6/NDP without restoring IPv6.
- **Governor/EPP:** powersave + balance_performance is the EPP-honoring config under amd_pstate=active — do not flag powersave without proving `performance` would honor the hint.
- **GPU_DPM_LEVEL:** `auto` is deliberate. Do not flag without gfx1151 frametime/1%-low evidence for `high` under the shared budget — and remember the rule only began firing at v7.94/95 (pre-fix observations are void).

---

# Deep-pass appendix — exact generated bodies + verify surface

§1–§13 are value-level. This appendix is artifact-level: the exact strings the script writes, the verify subsystem, and the install-phase model. Validate the **rendered content**, not a paraphrase. Every block is quoted from the generator functions in v7.101.0 (UUIDs/joins resolved at runtime). Every generator emits a leading `#` header-comment line — byte-exact/checksum comparisons must include it.

## A. Install-phase model (`_RY_PHASE_NAMES`)

Six ordered phases; recommendations must respect this sequence:

```
1 Preflight     _install_preflight          — _ir_* gates (counts=21, keys incl BLACKLIST_AMDXDNA + charsets/metachar, kernel floor NO-OVERRIDE, post-hooks, root UUID); mesa soft-floor
2 Packages      _install_packages           — mkinitcpio.conf pre-deployed (tagged pre-Syu seed) → pacman -Syu; PKGS_ADD re-marked -D --asexplicit post-Syu; chwd Vulkan
3 Configuration _install_system_files        — render+deploy all managed files (atomic tmp+rename); format-validate before write; nftables.conf additionally nft -c pre-validated
4 Services      _install_configure_services  — fstab rewrite + resolved + PKGS_DEL (-Rns) + mask (nft-first, then ufw flush; MASK 12) + iwd handoff + enable + regdom + NTP (always; chronyd/ntpd/openntpd guard) + RTC write-back
5 Boot          _install_rebuild_boot        — taint-gate → mkinitcpio -P + sdboot-manage gen/update (gated on boot-critical writes)
6 Finalize      _install_finalize            — user daemon-reload + paccache (-rk2 and -ruk0 separate) + NetworkManager restart
```

- Confirm the firewall handoff lives in Phase 4 (nftables live before ufw flushed/masked) and boot-critical regeneration (Phase 5) fires only when a `_RY_BOOT_CRITICAL_DSTS` member changed. Flag any recommendation moving a cmdline/mkinitcpio change outside the Phase-5 gate.
- **`_RY_BOOT_CRITICAL_DSTS` (4):** `/boot/loader/loader.conf`, `/etc/kernel/cmdline`, `/etc/sdboot-manage.conf`, `/etc/mkinitcpio.conf`. **`_RY_BACKUP_TARGETS` = the same 4** (derived): all four get `.ry.bak` + post-write verify/restore (plus fstab during its rewrite). Confirm the derived-equality is count-asserted (tripwire 4) so the sets cannot diverge. Preflight refuses if any backup target uses a side-effecting generator (the sysctl.d guard at line 852).
- **`_RY_POST_HOOKS` (17 entries, 16 distinct tags):** `/boot/*|loader`, `/etc/kernel/cmdline|cmdline`, `/etc/sdboot-manage.conf|boot`, `/etc/mkinitcpio.conf|boot`, `*/resolved.conf.d/*|resolved`, `*/logind.conf.d/*|logind`, `*/NetworkManager-dispatcher.service.d/*|nmdispatch`, `*/NetworkManager/conf.d/*|nm`, `/etc/iw-regdomain|regdom`, `/etc/bluetooth/main.conf|bluetooth`, `/etc/nftables.conf|nft`, `/etc/default/cpupower-service.conf|cpupower`, `*/sysctl.d/*|sysctl`, `/etc/udev/rules.d/*|udev`, `*/modprobe.d/*|modprobe`, `*/environment.d/*|envd`, `*/MangoHud/MangoHud.conf|mangohud`. `_ir_validate_post_hooks` refuses deploy on any tag lacking `_post_<tag>` (empty tags refuse).
  - **Dispatch is FIRST-MATCH-WINS by list order** (`_post_hook_for_target`) — pattern ordering is load-bearing for overlapping globs; a recommendation reordering or adding an entry must preserve it.
  - The boot family shares one body, `_post_boot_apply <target> <skip_mki>`: `_post_boot` passes `skip_mki=false` (full `mkinitcpio -P`); `_post_cmdline`/`_post_loader` pass `true` (sdboot regen only — cmdline/loader.conf are not initramfs inputs).
  - Notify-only handlers: `_post_logind` (restarting logind kills sessions → reboot), `_post_modprobe` (load-time options can't live-apply → reboot), `_post_envd`/`_post_mangohud` (session/next-launch).
  - `_post_nm` DEFERS the NetworkManager restart when WiFi is the active route; confirm the deferred restart still lands (Phase 6) and is surfaced.
  - All destinations + `--install-file` values are canonicalized via `realpath -m` at load (realpath-fail warns, falls back to the literal path) — managed-file matching is canonical-path based.
  - Unmatched patterns log `POST_HOOK_NONE` (deployed, live-apply skipped) vs `POST_HOOK_SKIP_UNCHANGED` (bytes identical); `_post_udev` runs `udevadm verify` (systemd ≥254) before reload+retrigger (block AND cpu).

## B. Exact rendered file bodies (validate content, not summary)

### B1. `/etc/kernel/cmdline` + `/etc/sdboot-manage.conf` + `/etc/mkinitcpio.conf`

```
rw root=UUID=<_ROOT_UUID> 8250.nr_uarts=0 amd_iommu=off amd_pstate=active btusb.enable_autosuspend=n clearcpuid=umip fsck.mode=force fsck.repair=yes ipv6.disable=1 nowatchdog nvme_core.default_ps_max_latency_us=0 pcie_aspm.policy=performance processor.max_cstate=1 quiet split_lock_detect=off tsc=reliable usbcore.autosuspend=-1 zswap.enabled=0
```
sdboot-manage.conf:
```
# sdboot-manage configuration — changes require: sudo sdboot-manage gen && sudo sdboot-manage update
LINUX_OPTIONS="<KERNEL_PARAMS join>"
LINUX_FALLBACK_OPTIONS="quiet"
DEFAULT_ENTRY="manual"
REMOVE_EXISTING="yes"
OVERWRITE_EXISTING="yes"
REMOVE_OBSOLETE="yes"
```
mkinitcpio.conf:
```
# mkinitcpio configuration — changes require: sudo mkinitcpio -P && sudo sdboot-manage update
MODULES=(amdgpu)
BINARIES=()
FILES=()
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)
COMPRESSION="zstd"
COMPRESSION_OPTIONS=(-1 -T0)
```
- **`clearcpuid=umip` ships in BOTH bootloader paths** (`/etc/kernel/cmdline` + `LINUX_OPTIONS`). Confirm the two aren't simultaneously active in conflict; state which CachyOS drives and whether maintaining both is redundant or a divergence risk.
- **`LINUX_FALLBACK_OPTIONS="quiet"` strips ALL params from the fallback entry:** a fallback boot runs kernel-default IOMMU (AMD-Vi ON) AND IPv6 ENABLED with the IPv4-only ruleset not covering it. The modprobe `amdxdna` blacklist (a modprobe.d file) REMAINS active in fallback, so the NPU stays off there — note the asymmetry vs the main entry. Confirm the fallback-only IPv6 exposure window is accepted or flag it.

### B2. `/boot/loader/loader.conf`
```
# systemd-boot loader configuration
default @saved
timeout 0
console-mode keep
editor no
```
- `@saved` + timeout 0 + editor no: confirm `@saved` resolves; failed-boot menu reachability vs live-USB recovery. loader.conf changes regenerate sdboot entries only.

### B3. `/etc/nftables.conf` (IPv4-only ruleset — validate rule-by-rule)
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
        # [IFF RY_REMOTE_PLAY_PORTS=true — gated block ships with its own marker comment:]
        # ry-install: remote-play inbound (RY_REMOTE_PLAY_PORTS=true)
        tcp dport { 47984, 47989, 48010, 27036, 27037 } accept
        udp dport { 47998-48010, 27031-27036 } accept
    }
    chain forward { type filter hook forward priority filter; policy drop; }
    chain output { type filter hook output priority filter; policy accept; }
}
```
- **Deploy-time `nft -c -f <tmpfile>` gate:** a rendered ruleset failing syntax check refuses deploy with live+installed unchanged (`NFT_PREVALIDATE_FAIL`); `_post_nft` re-validates the installed file before `systemctl restart nftables` and downgrades a restart failure to "applies at next boot" (warn). Confirm the check-then-load pair is same-binary-consistent and the degraded path is surfaced.
- IPv4-ONLY: no `ip6`/`icmpv6` types; safe only under the `ipv6.disable=1` coupling. Rule order lo → established/related → invalid-drop cannot drop valid loopback/established. `echo-request` accept is the regression guard; `destination-unreachable` preserves PMTUD; no echo-reply rule (ct established covers). `flush ruleset` blast radius vs docker/libvirt/podman. No ICMP/new-conn rate-limit — acceptable on a trusted LAN?

### B4. udev `99-ry-perf.rules` (3 rules; GPU matcher `ENV{DEVTYPE}`)
```
# ry-install: udev performance rules (managed file, do not edit by hand)
# NVMe scheduler none (lowest tail latency; diverges from CachyOS kyber default)
ACTION=="add|change", KERNEL=="nvme[0-9]*n[0-9]*", ENV{DEVTYPE}=="disk", ATTR{queue/scheduler}="none"
# AMD P-State EPP balance_performance (perf-leaning CPPC hint)
ACTION=="add|change", SUBSYSTEM=="cpu", KERNEL=="cpu[0-9]*", ATTR{cpufreq/energy_performance_preference}="<EPP_PREFERENCE>"
# GPU performance level (gfx1151 clock-floor; optional)
ACTION=="add", KERNEL=="card[0-9]*", SUBSYSTEM=="drm", ENV{DEVTYPE}=="drm_minor", DRIVERS=="amdgpu", ATTR{device/power_dpm_force_performance_level}="<GPU_DPM_LEVEL>"
```
- **GPU rule:** bare `DEVTYPE` is not a udev match key; the pre-v7.94/95 rule NEVER APPLIED (net-nil only because auto = kernel default). Confirm the rule now matches at enumeration; it remains `add`-only (no re-assert after GPU reset — a robustness argument for `auto`).
- **EPP rule value from `$EPP_PREFERENCE`** (enum-gated `_RY_EPP_LEVELS`; unquoted ATTR interpolation bounded by the enum). `_post_udev` retriggers cpu+block so it live-applies; `udevadm verify` gates the reload on systemd ≥254.
- Filename 99- sorts after vendor 60-ioschedulers.rules (last ATTR wins); confirm CachyOS still defaults kyber.

### B5. `/etc/systemd/resolved.conf.d/99-cachyos-resolved.conf`
```
# systemd-resolved: plaintext DNS, mDNS/LLMNR off (diverges from CachyOS DoH default)
[Resolve]
MulticastDNS=no
LLMNR=no
DNSOverTLS=no
DNSSEC=allow-downgrade
```
- Same-basename replace (not merge) caution: if CachyOS ships its own `99-cachyos-resolved.conf`, this REPLACES it — confirm intended, not a clash a vendor update re-overwrites. Restarts skipped when bytes unchanged. §10 privacy flag stands.

### B6. NetworkManager `99-cachyos-nm.conf` + dispatcher `logging.conf`
```
# NetworkManager configuration - wpa_supplicant backend
[device]
wifi.backend=wpa_supplicant

[connection]
wifi.powersave=2

[logging]
level=WARN
```
dispatcher: `# LogLevelMax drops info-level dispatcher lines (journald-logged; StandardError=null ineffective)` + `[Service]` + `LogLevelMax=notice`.
- Same basename-override caution as B5. Confirm LogLevelMax=notice remains the correct journald-noise fix.

### B7. `/etc/iw-regdomain`
```
# ry-install: wireless regulatory domain (managed file, do not edit by hand)
COUNTRY=US
```
- **Persistence dependency (most version-fragile external):** confirm `cachyos-iw-set-regdomain` (or its successor) still exists in current CachyOS and reads this file at boot; if dropped, the file is inert and the profile must switch mechanisms. Reserved/user-assigned ISO codes rejected at preflight.

### B8. `/etc/bluetooth/main.conf` + `/etc/default/cpupower-service.conf`
```
# ry-install: BlueZ daemon config (managed file, do not edit by hand)
[General]
FastConnectable=true

[Policy]
AutoEnable=true
ReconnectAttempts=3
```
cpupower-service.conf:
```
# cpupower-service.conf — sourced by /usr/lib/systemd/scripts/cpupower (cpupower.service)
GOVERNOR='powersave'
```
- **The consumer path is stated in-file** (`/usr/lib/systemd/scripts/cpupower`). Single verify: confirm that script path + `GOVERNOR` var name on current CachyOS cpupower packaging (if moved, the file is inert and the governor falls to kernel default; the udev EPP rule still applies). `_vrsv_chk_cpupower_governor` asserts the running governor.

### B9. `~/.config/MangoHud/MangoHud.conf` (19 active + 1 commented + 1 file-header)
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
- Byte-exact checks must use the full commented string (prefix greps unchanged). `cpu_power` remains the live target (§12); the file-header is not a directive.

### B10. `/etc/modprobe.d/60-ry-modules.conf` (merged destination, v7.99.0)
```
# ry-install: module options + blacklist (managed file, do not edit by hand)
# disable PCIe ASPM on MT7925 (coredump/BT-reconnect/assoc mitigation; drop when upstream fixes)
options mt7925e disable_aspm=1
# [IFF BLACKLIST_AMDXDNA=true (default):]
# blacklist amdxdna: XDNA NPU needs IOMMU, probes -EINVAL (ret -22) under amd_iommu=off
blacklist amdxdna
```
- Replaces `60-ry-mt7925e.conf` + the interim `60-ry-blacklist-amdxdna.conf`; pre-7.99 installs carry stale unmanaged copies until the README's one-time `sudo rm`. The conditional block renders ONLY under the default-true toggle; `BLACKLIST_AMDXDNA=false` (validator-coupled to the IOMMU being ON) yields a 3-line file. `_vss_modprobe` asserts the static content; `_vrkm_blacklist_modprobe` asserts amdxdna is NOT loaded. **The leftover-detection gap (§8) is the residual open item: `_vrkm_blacklist_modprobe` is generator-sourced, so a pre-7.99 leftover drop-in is invisible to it and to every verify path.**

## C. Verify subsystem (`--verify`) — orchestrator families

The top VERIFY block is the user-facing command set; `--verify` runs orchestrators across sub-families. A recommendation that changes a value MUST state which sub asserts it (hard-fail vs warn). Re-derived from v7.101.0 (289 fns). The three v7.91.0-flagged stale `--description` strings and the v7.99.1 runtime-session string are all now FIXED.

**Static (on-disk) — orchestrators + syntax + checksum:**
- `_verify_static_boot` → `_vsb_loader` · `_vsb_sdboot` (LINUX_OPTIONS token set + keys) · `_vsb_cmdline` (`/etc/kernel/cmdline` token set — `amd_iommu=off`/`ipv6.disable=1`/`clearcpuid=umip`/`tsc=reliable` byte-asserted here) · `_vsb_mkinitcpio` (HOOKS/MODULES + LIVE `COMPRESSION=` + `COMPRESSION_OPTIONS` via `_ry_mkinitcpio_array`, multi-line join, last-wins with a warn) · `_vsb_entries` (BLS entries + count). All hard-fail.
- `_verify_static_system` (description names: resolved, logind, NM, regdom, bluetooth, cpupower-service.conf, sysctl, udev, modprobe, nftables) → `_vss_logind` · `_vss_nmdispatch` · `_vss_nm` · `_vss_sysctl` · `_vss_regdom` · `_vss_bluetooth` · resolved (inline `_chk_grep` loop) · cpupower-service.conf (inline `_chk_grep GOVERNOR`) · `_vss_udev` (all 3 rules; EPP from `$EPP_PREFERENCE`; GPU_DPM_LEVEL-aware) · `_vss_modprobe` (merged file: mt7925e `disable_aspm=1` + optional amdxdna blacklist) · `_vss_nft` (hard-fail on missing `echo-request` — inbound-ping regression guard; IPv4-only).
- `_verify_static_user` — ENV_VARS (env.d, `_chk_grep` per var) + MangoHud (inline `_chk_file` + `_chk_grep "fps"`). `_chk_grep` is comment-proof: awk strips inline comments and skips comment-only lines before the `grep -wF` match — a commented-out `key=value` can no longer satisfy presence; "no non-comment lines" is a distinct FAIL; a mid-read sudo lapse is a distinct warn.
- `_verify_static_packages` → `_vsp_required` (PKGS_ADD 18 + folded Vulkan pkgs, pacman-db-lock guard) · `_vsp_removed` · `_vsp_pacman_conf` (sudo-read fallback for a 0600-hardened pacman.conf; grep rc>1 → warn-skip; rc 1 + sudo-lapse → warn-skip — never a false "not set") · `_verify_static_services` (MASK 12 state) · `_verify_static_syntax` (live mkinitcpio HOOKS presence) · `_verify_static_checksum` → `_vsc_check_one` per file (embedded SHA256 == installed; graceful skip on `EXIT_GEN_NOUUID`). Every body carries a header-comment line and mkinitcpio carries `BINARIES=()`/`FILES=()` — embedded and installed move together; any out-of-band byte comparison must use the §B bodies.
- **Config-format validators (`_ry_validate_configs` → `_rvc_dispatch`, pre-deploy):** `_grep_kv`, `_grep_kparam`, `_grep_sysctl_kv`, `_grep_modprobe_entry`, `_grep_regdomain_entry`, `_grep_udev_entry`, `_grep_nft_entry`, `_grep_envd_entry`, `_grep_cpupower_entry`, `_grep_mangohud_entry`, `_grep_ini_header` fallback; the mkinitcpio case REQUIRES `MODULES=(`, `HOOKS=(`, and `COMPRESSION="` lines. `_ry_validate_mkinitcpio_hooks` (`_vmh_existence_only` + `_vmh_order_checks`) + `_ry_validate_mkinitcpio_modules` validate the arrays — `_vmh_*` are mkinitcpio validators, NOT MangoHud. nftables.conf additionally passes `nft -c -f` on the rendered tmpfile before commit.

**Runtime-kernel — `_verify_runtime_kparams`** (description: "/proc/cmdline, hardware state, module params, blacklist"):
- Preemption INFO scaffold: caches `sudo -n dmesg` once, INFO-only, erased after.
- `_vrk_cmdline` — generic loop asserting EVERY KERNEL_PARAMS token + `rw` in `/proc/cmdline` + preemption INFO.
- `_vrk_gpu_state` — `power_dpm_force_performance_level == $GPU_DPM_LEVEL` across `card*` (comparison QUOTED — empty sysfs reads can't mis-evaluate).
- `_vrk_cpu_state` — scaling_driver=`$EXPECTED_SCALING_DRIVER` · scaling_governor=`$CPUPOWER_GOVERNOR` · EPP=`$EPP_PREFERENCE` · amd_pstate status · `dynamic_epp=disabled` · prefcore · boost=1 (expectations hoisted globals).
- `_vrk_module_state` → `_vrkm_amdgpu` (hex-aware; no-ops without `amdgpu.*`) · `_vrkm_blacklist` (`module_blacklist=` cmdline scan — currently no-op) · `_vrkm_blacklist_modprobe` (parses `blacklist <mod>` from the MANAGED modprobe.d content, normalizes `-`→`_`, `lsmod`-checks each — amdxdna LOADED ⇒ FAIL; `lsmod` absent ⇒ warn; generator failure defers to checksum verify) · usbcore.autosuspend, nvme_core ps_max_latency, zswap.enabled, nmi_watchdog, NVMe `[none]`. `_vrkm_iommu`/`_vrk_clocksource` remain removed (directives still cmdline-asserted).

**Runtime-services — `_verify_runtime_services`:** `_vrsv_chk_active_enabled` · `_vrsv_nft_assert_ping` (live input chain accepts inbound IPv4 ping; warn-only) · `_vrsv_chk_nftables` (oneshot judged by live policy-drop) · `_vrsv_chk_resolved` · `_vrsv_chk_cpupower_governor` · `_vrsv_sys_units` · `_vrsv_wifi_nm_backend` · `_vrsv_wifi` · `_vrsv_masked_inactive` (covers the avahi pair). `_vrsv_wifi` contains no iwd path: it skips when `_RY_PROFILE_USES_WIFI_BACKEND=false`, detects the wlan iface via `/sys/class/net/*/wireless`, calls `_vrsv_wifi_nm_backend`, reads nmcli radio/device state, and closes with a firewall-posture INFO line (ufw active-state + live nft rule count). Residual iwd coverage = the backend compare only; an iwd opt-in host has lost the process cross-check — a narrow, deliberate coverage reduction (Low/Low).

**Runtime-env — `_verify_runtime_env`** (description: "ENV_VARS, sysctl, fstab, ntsync, regdom runtime"): `_vre_envvars` (`systemctl --user show-environment`) · `_vre_sysctl_runtime` (`/proc/sys`) · `_vre_fstab` (ext4 `noatime,lazytime,commit=10`) · `_vre_ntsync` (state dispatch — survives) · `_vre_regdom` (`iw reg get`). `_vre_tcp`/`_vre_zram` remain removed.

**Runtime-session — `_verify_runtime_session`** (description FIXED — "NM connection perms, installed-file perms, parent dirs"): `_vrs_nm_perms` (system-connections 0600 root:root) · `_vrs_installed_file_perms` (system 0644 / user 0600) · `_vrs_parent_dirs` → `_vpd_dir_perm_check` (0755 system / 0700 user). `_vrs_vfat_skip` guards BOTH loops (vfat/undetermined `$BOOT` paths are counted-skipped with an INFO, not silently passed). The v7.99.1 "Vulkan packages" stale string is now removed.

**Aggregation (`_ry_verify_all` / `_verify_summary`):** static runs first (boot → system → user → packages → services → syntax → checksum), runtime second; each stage prints its own `_verify_summary` and `_ry_verify_all` sums the counters. A runtime preflight bail (sudo lapse) restores the static totals, and a static FAIL outranks the runtime bail code — the exit reflects the worse finding. Confirm no path zeroes the static counters after a runtime bail.

**Actionable for §C:**
- Confirm `_vrkm_blacklist_modprobe` closes the amdxdna live gap correctly (generator-sourced — checks the INTENDED blacklist, so a pre-7.99 leftover drop-in is invisible; §8/§B10 — CONFIRMED gap); confirm the `-`→`_` normalization and warn-on-no-lsmod.
- Confirm the live COMPRESSION/_OPTIONS compare tolerates vendor multi-line arrays (join) and duplicate assignments (last-wins warn) without false FAILs.
- Comment-strip false-negative closure: no managed value contains ` #` and the boot-scalar metachar gate FORBIDS `#`, so the greedy strip cannot bite a legitimate value; re-check if a future value adds `#`.
- Confirm the removed effect-asserts leave no coverage gap that matters (directive-level coverage holds); the iwd narrowing is deliberate and flagged.
- `_vsb_entries` canonicalization: loader-entry `linux` paths resolve via `realpath -m` with a WARNED textual-join downgrade when realpath is absent — confirm the downgrade is loud enough.

## D. fstab rewrite (`_install_fstab_opts`) — normalization, not just append

- Adds `noatime,lazytime,commit=10` to ext4 entries (field 4 only); every other column and non-ext4 row byte-preserved. An ext4 row whose options field is purely numeric (`$4 ~ /^[0-9]+$/`) passes through untouched — a malformed-column guard; confirm such a row is then caught by the malformed-filter/refusal path rather than silently shipped.
- Strips conflicting tokens — the verify-side conflict list is exact: **`defaults`, `relatime`, `atime`, `strictatime`** (presence = "installer removes it — rewrite pending" FAIL); an existing `commit=` is rewritten to `commit=10`, and non-10 values are tracked in `_RY_FSTAB_COMMIT_OVERRIDES` (surfaced, not silently replaced).
- Gates: line-count parity + size floor + mandatory `findmnt --verify`. Refused (not corrected): symlinked or whitespace-split/malformed `/etc/fstab`.
- Confirm: (a) ext4-only (not the vfat ESP, not btrfs/xfs); (b) idempotent; (c) atomic (tmp+rename, `.ry.bak` taken); (d) `commit=10` durability vs every-boot forced fsck (§3).

## E. Preflight gate ordering (`_init_runtime` / `_install_preflight` / `_ir_*`)

Order matters for exit-code semantics. **Init-time capability probes run BEFORE everything else:** `id(1)` is the FIRST external command (hard-require; non-numeric `id -u` refuses); `timeout(1)` probed for `--foreground --kill-after`; `find(1)` probed for `-maxdepth`/`-printf`; `mv -T` live-probed via two mktemp files; `stat(1)`; `date(1)` `%z`-probed — each rejects busybox/uutils explicitly (exit 3). `TMPDIR` erased (tmp pinned `/tmp`); umask set as the variable directly; `--check` silence pinned pre-argparse. The dependency gate hard-requires a 33-command GNU set (pacman, systemctl, mkinitcpio, sdboot-manage, findmnt, sha256sum, timeout, mktemp, awk, grep, curl, getent, id, sudo, head, df, mv, tee, stat, find, cp, chmod, chown, install, cat, rm, date, wc, tail, basename, dirname, mkdir, rmdir, touch, env, sleep, cmp) plus a `df --output` probe, systemd ≥250, and warn-lists optional tools. All destinations canonicalized at load (`realpath -m`; failure falls back to the literal path).

- `_ir_resolve_root_uuid` → `EXIT_GEN_NOUUID 12` if cmdline render finds no UUID. **Mode-scoped:** `--install-file` is FATAL only when the (canonicalized) target IS `/etc/kernel/cmdline` (the sole UUID-embedding file); any other target warn-continues; `--verify` warn-continues with a generic root=UUID presence check.
- Hardware gate (CPU match; override `RY_INSTALL_SKIP_HARDWARE_CHECK=1`; fail-closed on unreadable model; `--verify` warns, deploy exits 3).
- `_ir_validate_kernel_floor` (3) — ≥6.19, NO OVERRIDE; fail-closed on unreadable `uname -r`; `--verify` warns; deploy AND `--check` refuse.
- `_ir_validate_counts` (3) — 21 tripwires (incl `_RY_ARGPARSE_SPEC:7`, `_RY_BACKUP_TARGETS:4`, `MKINITCPIO_COMPRESSION_OPTIONS:2`, `PKGS_ADD:18`, `MASK:12`).
- `_ir_validate_keys` (3) — bool: BT_AUTO_ENABLE/BT_FAST_CONNECTABLE/RY_REMOTE_PLAY_PORTS/BLACKLIST_AMDXDNA; yes|no: SDBOOT_*/RESOLVED_MDNS/LLMNR/DOT; int: LOADER_TIMEOUT/NM_WIFI_POWERSAVE/BT_RECONNECT_ATTEMPTS; ISO-3166 COUNTRY incl reserved-range rejection; GPU_DPM_LEVEL ∈ `_RY_DPM_LEVELS`; EPP_PREFERENCE ∈ `_RY_EPP_LEVELS`; CPUPOWER_GOVERNOR `^[a-z][a-z0-9_-]*$`; the **nftables↔`ipv6.disable=1`** coupling; the **`BLACKLIST_AMDXDNA=false`↔IOMMU-required** coupling; non-empty scalar set (incl `EXPECTED_SCALING_DRIVER`); boot-scalar metachar gate (PCRE class with `\x27`) on MKINITCPIO_COMPRESSION/SDBOOT_DEFAULT_ENTRY/LOADER_*/CPUPOWER_GOVERNOR; `MKINITCPIO_COMPRESSION_OPTIONS` token charset `^-?[A-Za-z0-9]+$`; `KERNEL_PARAMS` token charset `^[A-Za-z0-9._,=-]+$`. Every validated scalar is an embedded value set unconditionally (`set -g`, no `set -q` guard) — exported env vars of the same name are clobbered.
- `_ir_validate_post_hooks` (3) — every tag has a `_post_<tag>` handler; empty tags refuse.
- Generator sentinels: 11/12/13/14; 250/251/255 never reach a process exit.
- Advisories (non-fatal): mesa < 26.0 soft floor — `vercmp` presence-checked and output-validated `^-?\d+$` before compare.

- Confirm: (a) counts/keys/floor run BEFORE any disk write; (b) the bypass inventory is exactly ONE (`RY_INSTALL_SKIP_HARDWARE_CHECK`); (c) `PACTREE_TIMEOUT_S` (60), `BOOT_SPACE_CRIT/WARN` (200/500 MB), `ROOT_AVAIL_CRIT/WARN` (2/5) remain sane vs multiple kernels + fallback + the `-1` zstd image (§4).

## F. Prioritized open items (v7.101.0)

Ranked by audit impact. The five items the v7.99.1 audit led with are resolved in-script (§0) and are NOT repeated here.

1. **⚠ modprobe-leftover migration gap (only material open finding, Low/Low).** Superseded filenames have 0 in-script references; pre-7.99 leftovers are invisible to `_vrkm_blacklist_modprobe` (generator-sourced) and to every verify path; the README `sudo rm` note is the only guard. A stale `60-ry-blacklist-amdxdna.conf` keeps the NPU blacklisted after opt-in. ADD-check candidate (§8/§B10).
2. **6.19 floor — verify the single remaining leg.** gfx1151 post-0x83 MES ≥6.19 is now the sole rationale; confirm against upstream and state per-subsystem floors (§1).
3. **RTL8127 landing kernels — verify the ≤6.18 claim.** Cite the exact mainline releases for `f24f7b2f3af9` + `ae1737e7339b`; if either landed at 6.19+, the claim is FIX-doc and the floor regains a second leg (§11).
4. **amdxdna blacklist + coupling (NEW mechanism).** Confirm `-EINVAL (ret -22)` current; blacklist-vs-alternatives; coupling asymmetry intended (true+IOMMU-on valid); NPU-off default acceptable (§10 #2).
5. **MES label — verify post-0x83 is current-correct.** Now resolved in-script; confirm the shipping GC 11.5.1 MES revision matches (§1/§11).
6. **Intentional trade-offs to quantify (not FIX):** `amd_iommu=off` DMA-isolation loss + NPU cost; `clearcpuid=umip` UMIP exposure; IPv6-off/ping-on/avahi-masked net delta; `split_lock_detect=off`; plaintext DNS.
7. **UNCERTAIN evaluations pending upstream evidence:** `EPP_PREFERENCE` balance_performance→performance; `GPU_DPM_LEVEL` auto→high (both under the stated package budget); `PROTON_FSR4_RDNA3_UPGRADE=1` consumption.
8. **Docs consistency:** every §6/§13 power call must name its assumed budget (85 W README vs 140 W stock).

---

# Deepest-pass appendix (§G–§L) — robustness & correctness

§1–§13 audit *what the profile configures*; §A–§F audit *what the script writes and asserts*. This layer audits *whether the installer is safe to run at all* — correctness, not tuning. Confirm each guarantee on current fish (3.6 floor) / CachyOS; flag any TOCTOU, fail-open, or partial-write window. Every mechanism is quoted from v7.101.0.

## G. Atomic-write guarantees (`_awf_*`)

Path per file: `_awf_render_to_tmp` → (nftables: `nft -c -f` tmpfile gate) → `_awf_symlink_check` → `_awf_finalize_mv`. Backup targets (4 boot-critical) add `_awf_make_backup` (pre) + `_awf_postwrite_verify_restore` (post).

- **tee-to-tmp with `$pipestatus`:** generator piped into `_as $use_sudo tee`; `pipestatus[1]`/`[2]` checked separately, generator failures → `EXIT_GEN_*`. Builtin→pipe captures dropped (SIGPIPE) — confirm no probe still pipes a fish builtin into an external reader.
- **Symlink-swap probe:** rc 0/1/2 = symlink/not/lapse, abort on swap — confirm the check→`mv -T` window is closed.
- **`mv -T` rename:** chmod → `sudo -n true` re-assert → `mv -T`; tmp in dst's parent (same-FS). Capability live-probed at init on /tmp — `/boot` vfat rename semantics stand (probe runs on /tmp, not vfat). `_ry_mkdir_0755` caps parent creation at umask 0022.
- **Post-write re-read + restore:** re-runs generator, compares bytes (tri-state rc), restores `.ry.bak` on mismatch across all 4 boot-critical files. Confirm generator determinism for all four (cmdline is UUID-dependent — `EXIT_GEN_NOUUID` skips gracefully); `string collect --no-trim-newlines --allow-empty` preserves trailing newlines.
- **Non-backup files (nftables.conf et al.):** detect-only on mismatch (no auto-restore). Bounded by the `nft -c` gate (syntactically-broken can't deploy) but a semantically-wrong ruleset still can — confirm the residual posture is stated and accepted.

## H. Instance lock & PID-recycle TOCTOU (`_acquire_lock*`, `_lock_pid_started_after`)

Lock = atomic `mkdir "$LOCK_DIR"` (umask 0077, 0700) + pidfile via mktemp+`mv -Tf` (0600). Stale reclaim bounded (3 attempts), fail-closed. `_RY_LOCK_MKDIR_OK` set beside the rc capture — no window where the dir exists but ownership is unrecorded.

- **PID-recycle detection:** `/proc/PID/stat` field 22 (starttime) + `/proc/stat` btime; divisor = `getconf CLK_TCK`, fallback USER_HZ=100 (correct by ABI — `starttime` is USER_HZ, fixed 100 on Linux, not the tick rate; the old `/proc/config.gz` recovery is gone, though `_kconfig_cache` survives for the `CONFIG_NTSYNC=y` check). Reclaim only if holder start > pidfile mtime + 2 s; unparseable ⇒ treat live, refuse; empty/garbage pidfile settles 0.2 s, re-reads, still garbage ⇒ refuse. Confirm the `^.*\) ` comm-strip survives `) ` in comm, field-22 indexing after it, and the +2 s slack.
- **Re-read-before-rm guard:** pidfile re-read vs decision-time value before `rm -rf`; changed ⇒ abort. Symlinked `$LOCK_DIR` ⇒ refuse; `--preserve-root`. `kill -0` EPERM ⇒ `/proc` liveness, not reclaimed.
- Fail-closed on every btime/starttime/CLK_TCK failure (the 100 fallback logs `LOCK_CLK_TCK_FALLBACK`).

## I. Privilege handling (`_as`, `_run`, `_is_symlink`, `_installed_bytes`)

- **`sudo -n` everywhere; prompt exactly once, TTY-gated:** cold cache on a TTY runs `sudo -v` once; non-TTY refuses ("pre-cache via 'sudo -v'") — no mid-run hang. Lapse banner names `timestamp_timeout=60`, a keepalive, or a SCOPED NOPASSWD drop-in (avoid ALL). Credential re-asserted before each critical write. Confirm the once-prompt cannot recur mid-run.
- **Tri-state rc 0/1/2 (drift vs lapse):** `_is_symlink`, `_installed_bytes`, `_ry_content_bytes` return 2 for lapse; every caller must branch on rc 2 (a 2→1 collapse misreports lapse as drift). Audit all callers incl. `_vsp_pacman_conf`.
- **`_run` timeout (floor, not exemption):** default 3600 s; >9-digit clamped to 2147483647; invalid → default; `0` disables. **Long ops (pacman, mkinitcpio, sdboot-manage, paccache, updatedb, pkgfile — PATH-resolved) FLOORED to `_RY_LONGOP_HARD_CAP=7200 s`** (raised when below, `TIMEOUT_LONGOP_CAP`; "short SIGKILL bypasses rollback"). The resolver skips the sudo value-flag set (`-h -u -g -p -C -D -R -T -U --user --group --prompt --close-from --chdir --chroot --command-timeout --other-user --host`), bare flags, `env`, `VAR=` prefixes before naming the command. `PACTREE_TIMEOUT_S` (60) governs pactree only. Confirm 7200 s covers worst-case `-Syu` and a cap-kill is fatal-with-rollback, not a silent skip. Capture hygiene: argv redacts `/tmp/ry-*` → `[REDACTED]`; on overflow, JSONL inlines bytes+sha256 then an awk scan of the elided middle for diagnostic keywords (≤10 hits, ≤2000 chars each; nothing on disk).

## J. Boot-wipe gate & boot-critical rollback (`_irb_taint_gate`, `_install_rebuild_boot`)

1. `_irb_taint_gate`: `_RY_BOOT_TAINTED=true` OR failed mkinitcpio.conf revert ⇒ SKIP `mkinitcpio -P`, exit 4. Taint set via the shared `_taint` helper (`INSTALL_HAD_ERRORS` + `_RY_BOOT_TAINTED` together) — confirm every boot-critical write-failure routes through it.
2. `mkinitcpio -P` failure ⇒ abort, exit 4.
3. `$BOOT` resolved BEFORE the vfat gate; unresolved OR non-vfat `/boot` + `REMOVE_EXISTING=yes` ⇒ refuse the wipe (exit 4) — never run `sdboot-manage` against an unverified target.
4. `_preflight_boot_sanity`: vmlinuz + initramfs + entries must exist or exit 4.

Confirm: (a) no path to `REMOVE_EXISTING=yes` with unverified `$BOOT`; (b) the Phase-3 cmdline-write vs Phase-5 rebuild window — a boot with new cmdline + old initramfs is covered by the param-stripped fallback (which re-enables IPv6 and kernel-default IOMMU; the modprobe amdxdna blacklist REMAINS active — B1 asymmetry); (c) `EXIT_BOOT_CRIT` is terminal (no Finalize after).

**mkinitcpio rollback:** pre-`Syu` snapshot to `/run/ry-install` (0700, mktemp, tagged log line); on pacman failure `_mkinitcpio_revert` restores via same-`/etc`-FS mktemp + byte-exact `cmp` + atomic mv. Duplicate `KEY=` resolves last (matches `_ry_mkinitcpio_array`). Tmpfs snapshot is same-boot-only (acceptable). Flag if a pacman partial transaction can desync mkinitcpio.conf from installed modules without triggering revert.

## K. Signal & exit teardown (`_cleanup`, `_teardown`, `_do_cleanup`)

- **Signal handlers:** INT/TERM/HUP/QUIT/ABRT with 128+N re-raise; idempotent (`_CLEANUP_DONE`); SIGPIPE marks output broken, JSONL-only continues; `fish_exit` prefers `_INTENDED_EXIT_CODE` → `_RY_INSTALL_LAST_EXIT` → `$status`. JSONL `ts` from the single `_RY_TS_FMT` global via `command date`. **`--check` stderr-silence holds through the PRE-ARGPARSE window** (`_RY_ARGV_CHECK_ONLY` scanned before MODE exists; root + `--check`-only ⇒ silent exit 3; `-V`/`-VV` compatible; other flags restore exit-2) — confirm no early-init path (dep probes, log-dir failures) prints to stderr under a pure `--check` argv. **umask set as the VARIABLE directly** (the autoloaded `umask` function could leak "Unknown command" on a mid-autoload signal) — confirm fish ≥3.6 honors the variable form for children and `mkdir`.
- **Cleanup order:** kill children → mkinitcpio revert → tmpfile sweep → filesystem sweep → lock release LAST → erase globals. Child reap: TERM to `-P $fish_pid` descendants, poll loop — 0.5 s grace normally, 10 s when db.lck exists — then KILL (descendants-only); missing pgrep degrades to flat 0.5 s. Signal→exit map explicit (HUP:129 INT:130 QUIT:131 TERM:143 ABRT:134; unknown → 130). `_RY_TMPDIR_GLOBS` (6, PID-scoped `ry-*.$fish_pid.*`): sudo-err, tee-err, run, argparse-err, fstab-tee-err, fstab-awk-err — confirm the glob set == the created set. `TMPDIR` erased at init (sweep and create targets can't diverge).
- **Log lifecycle:** `mv -T` with `cp -pT` + `rm` recovery (dir-squat safe); both-fail keeps the old path (warn); symlinked LOG_FILE removed, re-created 0600. Root-guard refusal emits one `@@LEFT@@` per leftover positional + `@@IF@@` for the install-file value (display sentinels) — confirm they never leak unstripped.

**Exit-code contract (14 distinct — audit for discipline, no bare `exit 1`):**

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
║ 11   ║ EXIT_GEN_NOFN     ║ generator fn not defined           ║
║ 12   ║ EXIT_GEN_NOUUID   ║ root UUID unresolved in generator  ║
║ 13   ║ EXIT_GEN_SYSCTL   ║ sysctl gen printed-count mismatch  ║
║ 14   ║ EXIT_GEN_ENVD     ║ env.d gen printed-count mismatch   ║
║ 250  ║ EXIT_AS_MISUSE    ║ _as w/o cmd or non-bool use_sudo   ║
║ 251  ║ EXIT_RUN_TMPFAIL  ║ _run tmpfile create failed         ║
║ 255  ║ EXIT_RUN_MISUSE   ║ _run no-args or dash-prefixed arg  ║
```

- Codes 11–14 internal (checksum verify maps NOUUID to graceful skip); 250/251/255 are BUG asserts that must never fire. Confirm no path collapses a structured code (3/4/5/10) into bare 1, and SIGKILL is the only cleanup bypass (stale-reclaim §H recovers a SIGKILLed holder).

## L. pacman transaction safety (`_ip_pacman_invoke`)

- **Full `-Syu --needed` only:** first `-Syu --needed --noconfirm`, retry `-Syyu --needed --noconfirm` (forced db re-sync; "will not resolve pkg conflicts"); second failure fatal. `SYSTEM_UPGRADED` from a `pacman -Q | sha256sum` fingerprint (identical ⇒ false + `PKG_STATE_UNCHANGED`; empty fails open to true). Post-`Syu` `-D --asexplicit` re-mark (idempotent) prevents the Phase-4 `-Rns` orphan-cascading a pre-installed-as-dependency member (`PKG_ASEXPLICIT_FAIL` on failure). Confirm no `-S <pkg>` runs without `-yu` context.
- **db.lck pre-check:** refuses if `/var/lib/pacman/db.lck` exists; never removes it. The teardown reaper honors an in-flight transaction: `-P $fish_pid`-scoped TERM with 10 s grace under db.lck (§K) — a peer pacman is untouchable by scope. Confirm checked before any package op.
- **PKGS_DEL via `-Rns` (rdep-aware):** external dependants skip + log (`_RY_PKG_REMOVE_SKIPS`); no cascade or partial-upgrade. `paccache -rk2`/`-ruk0` separate. Providers declared (pacman-contrib); Phase-4 `-Rns` runs after Phase-2 installs so the tool is present — confirm a pactree-absent pre-Phase-2 run only warns (`PACTREE_MISSING`).

## Robustness verdict (required, separate from §1–§13)

Emit a final **ROBUSTNESS** block:
- For each §G–§L: PASS / GAP / UNCERTAIN (cannot confirm against current fish/kernel without testing).
- Any GAP in §G/§H/§J (atomic-write, lock, boot-wipe) is release-blocking and outranks every tuning finding — surface it first.
- Correctness, not preference: no "deliberate trade-off" defense for a partial-write window or fail-open lock. FIX applies normally (flag-don't-FIX is for config values, not safety invariants).

Sources: man7.org (mkdir/rename(2), proc(5) stat + USER_HZ, sysconf CLK_TCK), fishshell.com/docs (`$pipestatus`, `string collect`, `--on-signal`, `--on-event fish_exit`, variable-umask), wiki.archlinux.org (pacman partial-upgrade, mkinitcpio, systemd-boot, nftables), docs.kernel.org (/proc/stat btime, accel/amdxdna), man.archlinux.org nft(8).
