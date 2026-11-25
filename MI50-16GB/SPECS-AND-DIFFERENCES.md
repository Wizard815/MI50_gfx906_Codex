# Specs and differences: MI50 16GB vs 32GB vs MI60 vs Radeon VII

[Back to README](README.md). Source numbers like [S3] refer to [SOURCES.md](SOURCES.md).

All of these cards use the same **Vega 20** die: 7 nm GCN 5.1, LLVM target **gfx906**,
13.2 B transistors, 331 mm² [S8][S9]. ROCm's spec table lists the MI60, MI50 (32GB) and
MI50 (16GB) with identical cache and register-file sizes. The CU count and VRAM are the
only differences [S10][S11].

## Main table

| | MI50 16GB | MI50 32GB | MI60 | Radeon VII | Radeon Pro VII |
|---|---|---|---|---|---|
| Launch | Nov 2018 [S8] | later; the 2018 launch material lists only 16 GB [S11] | Nov 2018 | Feb 2019, $699 [S9] | 2020 |
| PCI device ID | `1002:66a1` [S12][S2] | `1002:66a1` [S2] | `66a0` likely, unverified: pci.ids names 66a0 "Radeon Pro/Radeon Instinct" | `1002:66af` [S12] | `1002:66a1` (pci.ids lists "Radeon Pro VII/Radeon Instinct MI50") [S12] |
| Subsystem ID | **not found** | `0834` (amdvbflash `SSID=08341002`) [S2] | not found | not found | not found |
| CUs / stream processors | 60 / 3840 [S10][S13] | 60 / 3840 [S10] | 64 / 4096 [S10] | 60 / 3840 [S9] | 60 / 3840 |
| TMUs / ROPs | 240 / 64 [S2 comment][S14] | 240 / 64 | 256 / 64 | 240 / 64 | 240 / 64 |
| Clock | base 1450, peak 1725 MHz [S8][S13] | same | base 1500, peak 1800 MHz [S8] | base 1400, boost 1800 MHz [S9] | n/a |
| Memory | 16 GB HBM2 ECC, 4096-bit | 32 GB HBM2 ECC, 4096-bit | 32 GB HBM2 ECC | 16 GB HBM2 (ECC not exposed) | 16 GB HBM2 ECC [S16] |
| HBM stacks | 4 × 4 GB, 4-Hi (one report) [S2] | 4 × 8 GB; 8-Hi on V420 boards, possibly 4-Hi on server boards (one report) [S2] | 4 × 8 GB (implied by 32 GB on 4 stacks) | 4 × 4 GB (not sourced here) | 4 × 4 GB (not sourced here) |
| Memory clock / bandwidth | 1000 MHz (2.0 Gbps), ~1 TB/s (1024 GB/s) [S8][S13] | same | ~1 TB/s [S15] | 1024 GB/s [S9] | ~1 TB/s [S16] |
| FP32 / FP64 peak | 13.4 / 6.7 TF (launch) [S11]; AMD support page says FP16 26.5, FP64 6.6 TF [S13] | same | 14.8 / 7.4 TF [S11] | 13.8 / 3.46 TF (FP64 1/4 rate) [S9] | 13.1 / 6.5 TF (1/2 rate) [S16] |
| TDP / board power | 300 W, 2× 8-pin [S13] | 300 W | 300 W [S11] | 300 W, 2× 8-pin [S17] | 250 W [S16] |
| BIOS power cap (Linux) | unconfirmed, ~225 W? (one report) [S2] | 225 W stock; 178 W V420; 300 W Apple Pro VII BIOS [S2] | n/a | n/a | 198 W driver default (one report) [S2] |
| PCIe | 4.0 x16, backward compatible with 3.0 [S13] | 4.0 x16 | 4.0 x16 | 3.0 x16 [S9] | 4.0 x16 per STH [S16]; the Pro VII **BIOS** reports Gen3 max on MI50 (one report) [S2] |
| Infinity Fabric | 2 links, 92 GB/s peak per AMD; launch PR said up to 200 GB/s P2P [S13][S15][S11] | same | same | none | 2 links, bridge sold separately, 2 cards max [S16] |
| Cooling | passive, full height, dual slot [S13] | passive | passive | triple axial fan (not sourced here) | blower (not sourced here) |
| Display | stock BIOS: none. Some listings mention 1× mini-DP 1.4a (unverified) [S18][S19] | stock BIOS: none (legacy only); V420 and Apple BIOS: yes [S2] | none | 3× DP 1.4 + 1× HDMI 2.0b [S17] | 6× mini-DP (not sourced here) |

### Conflicts in the sources

- **CU count of the 32 GB card.** One gist comment (johncoffee715, Jun 2026) says an "original AMD" 32 GB MI50 has 64 CUs and 14.75 TF FP32. AMD's ROCm table says **60 CUs** for both MI50 variants [S10]. The gist author's own `amdgpu_top` output from a 32 GB card also shows 60 CUs [S2]. Treat the 64-CU claim as **unverified**. Cards with 64 CUs would be MI60s or relabeled cards.
- **FP32 of the 16 GB card.** The same comment gives 13.06 TF, which works out to about 1700 MHz × 3840 × 2. AMD's figure is 13.4 TF at 1725 MHz. The difference is only the clock used in the calculation.
- **Memory clock of the stock 16 GB BIOS.** The comment says `113-D1631400-X11` runs HBM2 at **600 MHz**. TechPowerUp lists the same version string (`016.004.000.056.013521`) at **930 / 1000 MHz** core/memory [S1][S2]. Unresolved. Check your own card (see below).

## Part numbers

These are **listing data, not AMD documentation**. Use them to help identify a board, not as proof of what it is.

| Number | What / where | Notes |
|---|---|---|
| `100-506077` | AMD MPN, "Radeon Instinct MI50 16 GB" (Exxact, SabrePC) [S20] | retail SKU |
| `100-506143` | AMD MPN, "MI50 16GB PCIe 4.0" (Exxact, esaitech) [S20] | second 16 GB SKU, perhaps an OEM or region variant (unverified) |
| `102D1631410` | board P/N on a Newegg refurb listing, "MI50 16GB" [S20] | `102-D16314…` lines up with the BIOS P/N `113-D1631400` (my inference) |
| `102D1631710000001` | board P/N in an eBay listing title, "MI50 16GB" [S20] | `D16317` is also the board family of the 32 GB BIOS `113-D1631700-111` (274474). Ask the seller for a photo of the label and `lspci`/`amdgpu_top` output |
| `113-D1631400-X11` | VBIOS P/N, stock MI50 16GB server BIOS (one report) [S2] | version `016.004.000.056.013521` |
| `113-D1631700-111`, `113-D1631711-100` | **32 GB** VBIOS P/Ns [S2] | not for 16 GB cards |
| `113-D1640200-043` | **32 GB** Radeon Pro V420 BIOS (the one you run) [S2] | not for 16 GB cards |

## How to tell what you have

```bash
lspci -nn | grep -i 1002                  # 66a1 = MI50 / Pro VII, 66af = Radeon VII, 66a0 = MI60 family
sudo ./amdvbflash -i                      # shows SSID, BIOS P/N (113-...) and version
amdgpu_top -d | grep -B4 -A1 ROP          # CU and ROP count; MI50 should show 60 CUs
cat /sys/class/drm/card*/device/mem_info_vram_total   # ~17.1e9 bytes = 16 GB card, ~34e9 = 32 GB
cat /sys/class/drm/card*/device/pp_dpm_mclk           # memory clock states (stock BIOS: 600 or 1000 MHz?)
rocminfo | grep -o 'gfx906[^ ]*' | head -1            # sramecc+ or sramecc-
```

`amdvbflash` is covered in [BIOS.md](BIOS.md). The `amdgpu_top` command is from the gist comment [S2].

### "Radeon VII" labeling on 16 GB cards

- Chinese resellers usually sell 16 GB MI50s **already flashed to a Radeon VII BIOS**, which "unlocks consumer features" and display output (article plus comments, SMZDM, Sep 2025) [S21]. A card that shows up as "Radeon VII" in Windows or `amdgpu_top` is therefore probably an MI50 16GB with a Radeon VII BIOS.
- Whether such a card also changes its PCI ID to `66af` is **unverified**. No `lspci` dump from a cross-flashed MI50 was found. Real Radeon VIIs (`66af`) have fans, display outputs and a PCIe 3.0 interface, so a passive card showing `66af` is almost certainly a flashed MI50.
- A real Radeon VII cannot become an MI50 32GB, and an MI50 16GB cannot become one either: **the HBM is on the package, so VRAM can't be upgraded**.

## Practical differences, 16 GB vs 32 GB

| Topic | 16 GB | 32 GB |
|---|---|---|
| VRAM | 16 GB: up to ~24B at Q4 or ~14B at Q8 per card | 32 GB: ~32B Q6/Q8, 70B Q4 across 2 cards |
| BAR | 16 GB BAR; still needs Above 4G Decoding | 32 GB BAR on most BIOSes. The original AMD 275395 BIOS exposes only a 16 GB BAR, which breaks Vulkan [S2] |
| ReBAR | stock BIOS unknown; Pro VII Lenovo BIOS: yes (one report) [S2] | yes on V420, 274474 and Apple; no on 275395 [S2] |
| Cross-flash options | Pro VII 16GB (WS/Lenovo), Radeon VII (no display per one report, display works per another) | V420, Apple Pro VII 32GB, 274474 |
| HBM OC | untested on MI50; Radeon VII commonly reaches 1200 MHz | 1180 MHz on your cards |
| Price, Nov 2026 | ~$300 used on US eBay | ~$500–830 used on US eBay |
| Supply | less common in the West; common in China | the main used-market card in 2026 |

## Software support (both variants)

- **Official ROCm:** gfx906 entered maintenance mode in ROCm 6.0, with ROCm 5.7 the last fully supported release. Fixes ended around Q2 2024, and ROCm 7.0 shows gfx906 as unsupported [S22]. ROCm 10 officially starts at gfx908 [S23].
- **Community:** mixa3607/ML-gfx906 provides llama.cpp, ComfyUI and PyTorch images. Its pre-TheRock legacy builds (6.3.3) were dropped in 2026, and current presets use ROCm 7.14 [S24]. Wizard815/TheRock-gfx906 builds ROCm 10 with one patch [S23]. Your local checkouts: `/home/user/ML-gfx906`, `/home/user/TheRock-gfx906`, `/home/user/mx-llama.cpp-Rocm10/GFX906-ROCM10.md`.
- **Vulkan (Mesa RADV)** works regardless of AMD's support list. One article cites a Mar 2026 benchmark where Vulkan was faster on short-context dense models and ROCm faster on MoE and long context (one report) [S25].
- None of this depends on 16 vs 32 GB. The ISA is the same, so binaries and images are identical.
