# MI50 16GB: cooling, power, OC/UV, LLM fit, problems, prices

[Back to README](README.md). Source numbers like [S2] refer to [SOURCES.md](SOURCES.md).

The tuning tools and sysfs paths are the same as on your 32 GB cards. UPP and the
`smcPPTable/FreqTableUclk` HBM overclock are documented in the
[evilJazz gist](https://gist.github.com/evilJazz/14a4c82a67f2c52a6bb5f9cea02f5e13) and
not repeated here. This page lists only what differs for 16 GB.

## "Upgrades" that are not possible

- **VRAM.** The HBM2 sits on the GPU package (interposer), so a 16 GB card can't become 32 GB. Flashing a 32 GB ROM only makes the card report the wrong size, and that risks a brick [S3].
- **CUs.** Every MI50 is 60 CUs. Unlocking MI60's 64 CUs through firmware has not been reported.
- **Firmware** is the only "upgrade". See [BIOS.md](BIOS.md). The realistic options are the stock BIOS or a Pro VII image.

## Cooling

- The card is passive and was designed for server airflow (front-to-back, about 300 W TDP) [S13]. Everyone who uses it in a desktop or tower case adds airflow [S25][S31].
- Common fixes: a ducted fan or a 3D-printed shroud per card [S25]. Blower fans taped on with aluminium tape also work (MI25 report) [S32]. Chinese buyers report 80 °C core / 95 °C hotspot in games with improvised fans [S21].
- Throttling: above ~95 °C junction, SCLK drops from 1700 to ~860 MHz, about half the throughput (one report, 16 GB) [S2].
- Fan control: drive the fans from the motherboard or a script that reads `temp2_input` (junction). Start the script **after** amdgpu has created sysfs. A user's card overheated because the script started before the sysfs interface was ready (MI25 report, same failure mode) [S32]. On the **ML350 Gen10**, the chassis fans and GPU baffle are how a passive card is meant to be cooled, the same as your 32 GB cards.

## Power cap and undervolting

| BIOS | Default cap |
|---|---|
| Stock 16 GB `113-D1631400-X11` | unconfirmed, ~225 W? [S2] |
| Pro VII (WS / Lenovo) | 198 W driver default; Lenovo hard max 350 W [S2] |
| for reference: your V420 (32 GB) | 178 W, costs about 1% performance per the gist [S2] |

LLM token generation is memory-bound, so lowering the power cap costs little. It
mostly slows prompt processing.

```bash
# cap (microwatts); allowed range in power1_cap_min / power1_cap_max   [S33]
echo 180000000 | sudo tee /sys/class/drm/card1/device/hwmon/hwmon*/power1_cap
# or: rocm-smi -d 1 --setpoweroverdrive 180
```

Undervolting on Vega 20 uses the 3-point `OD_VDDC_CURVE` in `pp_od_clk_voltage` [S33]:

```bash
# needs overdrive enabled: amdgpu.ppfeaturemask=0xfff7ffff  (default 0xfff7bfff, per [S2])
echo manual | sudo tee /sys/class/drm/card1/device/power_dpm_force_performance_level
cat /sys/class/drm/card1/device/pp_od_clk_voltage        # shows OD_SCLK, OD_MCLK, OD_VDDC_CURVE
echo "vc 2 1700 1050" | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage   # example only
echo c | sudo tee /sys/class/drm/card1/device/pp_od_clk_voltage                  # commit ("r" = reset)
```

- On the Pro VII Lenovo BIOS, overdrive is off by default. The commenter enabled it with UPP: `-s 'OverDrive8Table/ODFeatureCapabilities[0]:1'` (one report) [S2].
- Radeon VII owners commonly run an 80–120 mV undervolt (consumer card, one forum thread) [S34]. Each card differs, so test with a long llama-bench run.
- Writing `pp_table` with UPP disabled AVFS on an MI25 [S32]. On the V420 it added about 3 W idle and disabled PCIe link power management until suspend/resume [S2]. Expect the same on the 16 GB card.
- Idle power: about 20–25 W per card for two MI50 16GB flashed to Pro VII (one report) [S31]. The 32 GB V420 idles at 15–21 W [S2].

## HBM overclock on 16 GB

- **Untested on the MI50 16GB.** The gist commenter lists "MCLK OC above 1000 MHz on 16GB: untested (4-Hi HBM2 speed bin unknown)" [S2].
- The **Radeon VII**, which uses the same die and the same 4 × 4 GB HBM2, commonly runs **1200 MHz**. Guru3D and LegitReviews used 1200 MHz, and Igor's Lab hit 1250 MHz on a chilled card [S34]. That makes ~1150–1200 MHz plausible on the MI50 16GB, but it is **unverified**.
- **Check the starting point first** with `cat pp_dpm_mclk`. If the stock BIOS really runs **600 MHz** (one report) [S2], getting to 1000 MHz is worth far more (about +65% bandwidth) than going from 1000 to 1180 MHz. Use a Pro VII BIOS, or the same `FreqTableUclk` UPP edit you used on the 32 GB cards with a value of 1000.
- Go up in 20–40 MHz steps. Test with `vkpeak` or llama-bench `tg` plus a perplexity run, since HBM errors can show up as garbage tokens instead of crashes. Check ECC counters if your BIOS exposes them (`ras` sysfs).
- mixa3607/ML-gfx906 ships **amd-memory-tweak** for HBM2 timings [S24]. It is advanced and was not tested for this page.

## What fits in 16 GB (LLM inference)

Rule of thumb: GGUF size ≈ parameters (B) × bits-per-weight / 8. Q4_K_M is about 4.8 bpw,
Q6_K about 6.6, Q8_0 about 8.5. Add the KV cache (grows with context) and about 0.5–1 GB of
overhead. These are estimates, not measurements.

| Model size | Q4_K_M | Q8_0 | One 16 GB card | Two 16 GB cards | 16 GB + 32 GB |
|---|---|---|---|---|---|
| 7–9B | ~5 GB | ~8–9 GB | Q8 with long context | — | — |
| 12–14B | ~7–9 GB | ~13–15 GB | Q4/Q6 easily; Q8 with short context | Q8 | Q8 |
| 22–24B | ~13–14.5 GB | ~24–25 GB | Q4 with small context, or Q3 | Q6/Q8 | Q8 |
| 27–32B | ~16.5–20 GB | ~29–34 GB | no (or IQ3 / CPU offload) | Q4/Q5 | Q6/Q8 |
| 30–35B MoE (3B active) | ~18–21 GB | ~32–36 GB | with `--n-cpu-moe` / `-ot` experts on CPU | Q4 | Q6 |
| 70B dense | ~42 GB | ~75 GB | no | no | Q3/IQ3 only, tight |

Reported numbers:

- 2× MI50 16GB with Ollama: about 18 tok/s on a 32B DeepSeek distill (one report, Apr 2025) [S31].
- 3× MI50 16GB, Qwen3.8-27B **Q8_0**, mainline llama.cpp on ROCm 6.2.4, `--split-mode tensor`, FA on: about 29–31 tok/s decode and about 51 tok/s prompt at short context; about 18 tok/s decode at 8k–216k context; 37 tok/s with MTP draft. Uses about 44–49 GB of the 48 GB (one report) [S35].
- 3× MI50 16GB, Qwen3.8-Flash-Next Q6_K, partly on CPU: about 30 tok/s prefill and 6.5 tok/s generation at 21.5k context. The owner said ROCm works well up to 6.4, but ROCm 7 had a library issue for them (one report, Sep 2026) [S36].
- Single card: an aggregator lists a median of about 175 tok/s for a 9B model at 4-bit (one run) [S37].

Ways to stretch 16 GB:

- Quantize the KV cache (`-ctk q8_0 -ctv q8_0` with `-fa`). Your fork's turbo KV types (`turbo3`/`turbo4`) give about 3.3× KV capacity; see `/home/user/mx-llama.cpp-Rocm10/GFX906-ROCM10.md`. Multi-GPU split with turbo is unverified there.
- Mixing a 16 GB and a 32 GB card: put the 32 GB card first and use `-ts 2,1` or `--tensor-split` by VRAM. Use `-sm layer` across PCIe. Use the same VBIOS family on every card [S2].

## Multi-card and P2P

- The MI50 has two Infinity Fabric (xGMI) link connectors [S13]. MI50/MI60 bridge cards are "very hard to find", and MI100/MI210 bridges are not compatible. One builder bought a Radeon Pro VII bridge hoping it would work, untested (Debian ROCm CI, Aug 2024) [S38].
- Without a bridge, P2P runs over PCIe. For llama.cpp layer split the inter-card traffic is small (general guidance, not measured here). `--split-mode tensor` exchanges much more data. To allow direct P2P, disable ACS in the BIOS or kernel [S38].

## Common problems

| Problem | Notes |
|---|---|
| **No POST / card not detected** | Enable **Above 4G Decoding** (and ReBAR if offered). An MI50 32GB needed it to POST in an HP box, and AMD's docs say it is required when the BAR sits above 4 GB [S30][S39]. Some consumer and old boards lack the option. A 16 GB card has a 16 GB BAR and still needs it |
| Not seen in Windows | One 16 GB owner saw nothing in Windows or openSUSE; the only reply was "use Linux" (unresolved) [S40] |
| **Reset bug** (VM passthrough) | An owner of 2× 16 GB warns against virtualizing them because of the GPU reset bug [S31]. gnif/vendor-reset lists Vega 20 only as "Radeon VII" (and MI100); **the MI50 is not listed** [S41]. On Unraid, pass `/dev/kfd` + `/dev/dri` into Docker containers instead of VMs |
| **rocBLAS segfault, `sramecc-`** | gfx906 has `sramecc+` and `sramecc-` ISA variants. Recent ROCm ships math kernels only for `sramecc+`, so Radeon VII, Vega II and "some MI50s" segfault in rocBLAS/rocSPARSE. Fix by rebuilding those libraries for `gfx906:sramecc-` [S42]. A replier suggests checking sramecc on a different VBIOS. Flashed-to-Radeon-VII 16 GB cards are a likely candidate (**unverified**) |
| Missing gfx906 Tensile files | Newer ROCm drops gfx906 rocBLAS kernels. Copy them from an older rocblas package (wiki guide) or use the mixa3607 / TheRock-gfx906 images [S43][S24] |
| PCIe stuck at Gen3 | Pro VII and Radeon VII BIOSes cap at Gen3 [S2]. One 32 GB user also saw Gen3 on stock (unresolved) [S2] |
| Vulkan sees 16 GB on a 32 GB card | 32 GB-specific (275395 BIOS BAR); not relevant to 16 GB cards [S2] |

## Prices and availability (2026)

| Date | Where | MI50 16GB | Source |
|---|---|---|---|
| Jan–Apr 2025 | US used | ~$110–160 | L1T [S31] |
| Sep 2025 | China used (Xianyu/Taobao) | 500–900 CNY, usually pre-flashed to Radeon VII | SMZDM [S21] |
| Mar–Oct 2026 | US eBay used, weekly low | $150–200 | GPUDojo [S44] |
| through Aug 2026 | resale sales | $140–170, +35% YoY | amcompute appraisal [S45] |
| Mar–Oct 2026 | Newegg | $432.99 (stale listing) | GPUDojo [S44] |
| Nov 13–24 2026 | US eBay used | **$275–299** (90-day high) | GPUDojo [S44] |
| Nov 24 2026 | Amazon "new" | $618.37 | GPUDojo [S44] |
| 2026 | Alibaba listings | ~$107; fan-cooled ~$129–133 (listings, not sales) | search snippet only [S46] |

For comparison, the 32 GB card was ~$500–650 on eBay in Aug 2026 and ~$600–830 asking
in Sep 2026 [S25][S47]. The "$110 era" of 2024–25 is over [S25].

**Trend:** 16 GB prices were flat at ~$150–200 for most of 2026, then jumped about 50%
in November 2026, apparently with the general memory and GPU shortage. Per GB, a 16 GB
card at ~$300 (~$19/GB) now costs about the same as a 32 GB card at $600 (~$19/GB).
