# MI50 16GB

Notes on the **16 GB** AMD Radeon Instinct MI50. The cards in this server are the
**32 GB** MI50 running the V420 BIOS (`113-D1640200-043`). Those notes are in the
[main README](../README.md) and the
[evilJazz 32GB gist](https://gist.github.com/evilJazz/14a4c82a67f2c52a6bb5f9cea02f5e13).
This folder covers only what is different about the 16 GB card, or what is still unknown.

Researched in November 2026. Labels used in these files:

- **AMD**: AMD's own spec or ROCm documentation.
- **one report**: one forum post or comment, not confirmed elsewhere.
- **unverified**: a claim with no first-hand evidence behind it, or one that conflicts with another source.

| File | What's in it |
|---|---|
| [SETUP.md](SETUP.md) | **Start here to run LLMs:** card checks, power cap, models and quants that fit, context sizes, launch flags, mixing with 32 GB cards |
| [SPECS-AND-DIFFERENCES.md](SPECS-AND-DIFFERENCES.md) | Full spec table for MI50 16GB, MI50 32GB, MI60, Radeon VII and Radeon Pro VII; IDs, part numbers, how to identify a card |
| [BIOS.md](BIOS.md) | Known 16 GB VBIOS images, what cross-flashing does, amdvbflash usage, risks, recovery |
| [UPGRADES-AND-TUNING.md](UPGRADES-AND-TUNING.md) | Cooling, power cap and undervolting, HBM overclocking, what fits in 16 GB, multi-card, common problems, 2026 prices |
| [SOURCES.md](SOURCES.md) | Every URL used, with one line on what it supports |

## Summary

- **The GPU is the same as the 32 GB card.** Both are Vega 20 / gfx906 with 60 CUs, up to 1725 MHz, a 4096-bit HBM2 bus at about 1 TB/s, 300 W TDP, PCIe 4.0 x16, two Infinity Fabric links, and passive cooling (AMD). The only official difference is 16 vs 32 GB of memory.
- **The PCI device ID is the same: `1002:66a1`.** A consumer Radeon VII is `1002:66af`. No source showed an MI50 16GB enumerating as `66af`. Run `lspci -nn` to see what your card is.
- **The stock 16 GB VBIOS is `016.004.000.056.013521`**, P/N `113-D1631400-X11`. TechPowerUp's collection and one gist comment both give this version string. It is not the same image as either 32 GB BIOS, so don't flash a 32 GB ROM (V420, 274474 or 275395) onto a 16 GB card.
- **Cross-flashing is riskier on the 16 GB card than on the 32 GB.** Reports conflict. One user runs two 16 GB cards flashed to Radeon Pro VII. Another bricked a 16 GB card with a Pro VII BIOS, and it then reported itself as 32 GB. Chinese-market cards are often sold already flashed to a Radeon VII BIOS. See [BIOS.md](BIOS.md).
- **mx-llama.cpp-Rocm10 runs unchanged on it.** The build targets gfx906 for both cards and nothing in it assumes 32 GB; only the model, context and `-ngl`/`-ts` choices change.
- **Software works the same as on your 32 GB cards.** gfx906 is dropped from official ROCm (maintenance mode in 6.0, removed by 7.0). It runs on community ROCm 7.x and ROCm 10 builds (mixa3607/ML-gfx906, TheRock-gfx906) or on Vulkan.
- **The 16 GB limit is the real constraint.** It fits about 14B at Q8 or about 24B at Q4 on one card. A 27–32B Q4 model needs two cards, or a 16 GB + 32 GB pair.

## Comparison

| | MI50 16GB | MI50 32GB (yours) | MI60 | Radeon VII |
|---|---|---|---|---|
| PCI ID | 1002:66a1 | 1002:66a1 | 66a0 (unverified) | 1002:66af |
| CUs / SPs | 60 / 3840 | 60 / 3840 (AMD) | 64 / 4096 | 60 / 3840 |
| Peak clock | 1725 MHz | 1725 MHz | 1800 MHz | 1800 MHz boost |
| HBM2 | 16 GB, 4 stacks | 32 GB, 4 stacks | 32 GB | 16 GB |
| Bandwidth | ~1 TB/s | ~1 TB/s | ~1 TB/s | 1 TB/s |
| FP64 | 1/2 rate (6.6–6.7 TF) | 1/2 rate | 1/2 rate (7.4 TF) | 1/4 rate (3.46 TF) |
| PCIe | 4.0 x16 | 4.0 x16 | 4.0 x16 | 3.0 x16 |
| IF links | 2 | 2 | 2 | none |
| TDP | 300 W | 300 W | 300 W | 300 W |
| Cooling | passive | passive | passive | triple fan |
| Display | mini-DP on some boards, not driven by stock BIOS | same | n/a | 3x DP + 1x HDMI |
| Used price, Nov 2026 (US eBay) | ~$275–300; was $150–200 most of 2026 | ~$500–830 | rare | n/a |

Sources and caveats for each row are in [SPECS-AND-DIFFERENCES.md](SPECS-AND-DIFFERENCES.md).

## Quick recommendations

1. **Buying for LLMs:** get the 32 GB card if the price gap is reasonable. Per GB of VRAM the 16 GB card is no longer cheaper in late 2026. Two 16 GB cards also need two slots, twice the cooling and about 2x idle power to match one 32 GB card.
2. **Adding a 16 GB card to the ML350:** it works next to the 32 GB cards with llama.cpp layer split weighted by VRAM (for example `-sm layer -ts 2,2,1`). Tensor split (`-sm tensor`) splits each layer evenly across the group, so the 16 GB card would cap every card's share. Keep the BIOS in mind: the gist author saw ROCm problems with mixed VBIOS versions across cards (one report, seen with 32 GB cards).
3. **Before flashing:** back up with `amdvbflash -s`, write down the `113-…` P/N, and check whether the card already carries a non-stock BIOS. See [BIOS.md](BIOS.md). Don't expect amdvbflash to recover a failed flash; plan on an SPI programmer.
4. **Check for `sramecc-`:** run `rocminfo | grep gfx906`. Cards that report `gfx906:sramecc-` segfault in the stock rocBLAS. Some MI50s do this, and it may be more likely on cards flashed to Radeon VII (unverified). See [UPGRADES-AND-TUNING.md](UPGRADES-AND-TUNING.md#common-problems).
5. **Before overclocking HBM:** first check what MCLK the BIOS actually sets with `cat /sys/class/drm/card*/device/pp_dpm_mclk`. Sources disagree on whether the stock 16 GB BIOS runs HBM at 600 or 1000 MHz.

## Open questions

- The stock 16 GB BIOS's power cap, ReBAR support and MCLK are unconfirmed.
- Subsystem IDs for 16 GB boards: none found. The only one seen is `0834` on a 32 GB card.
- Whether a stock 16 GB board has a usable mini-DP behind the bracket, and which BIOS drives it.
- Whether HBM overclocking above 1000 MHz works on the 16 GB (4-Hi) stacks. It is untested on MI50 16GB. Radeon VII cards often run 1200 MHz.
- Whether the "only Chinese MI50s accept Radeon VII / Pro VII BIOS" claim is true (one report).
