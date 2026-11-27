# MI50 32GB

[Back to README](../README.md). The cards in this server: **2x MI50 32GB**
(V420 BIOS, `113-D1640200-043`) in an HPE ML350 Gen10. This page collects what is
specific to them; the deeper research lives in the linked docs.

## The card

- Same GPU as the 16 GB card: Vega 20 / gfx906, 60 CUs, 4096-bit HBM2 at about
  1 TB/s, PCIe 4.0 x16, passive cooling. Full table:
  [SPECS-AND-DIFFERENCES.md](../MI50-16GB/SPECS-AND-DIFFERENCES.md).
- PCI ID `1002:66a1` (same as the 16 GB card). `rocminfo` should show
  `gfx906:sramecc+`.
- 32 GB BAR on the V420, 274474 and Apple BIOSes. The original AMD 275395 BIOS
  only exposes 16 GB, which breaks Vulkan.
- Turn on Above 4G Decoding: one MI50 32GB needed it to POST in an HP box.

## BIOS

- In use: **V420** (lower TDP): https://katastrophos.net/downloads/InstinctMI50/V420.rom
- The 32 GB VBIOS matrix (V420, 274474, 275395, Apple), flashing commands and HBM
  overclock: [evilJazz gist](https://gist.github.com/evilJazz/14a4c82a67f2c52a6bb5f9cea02f5e13).
- Use the same BIOS on every card in a ROCm box. Never flash a 32 GB ROM onto a
  16 GB card ([BIOS.md](../MI50-16GB/BIOS.md)).

## Cooling and power

- iLO can't see the MI50 temperatures, so [fan-bridge](https://github.com/Wizard815/ilofanmonitorgpu)
  raises iLO's fan minimum from the GPU junction and HBM temperatures.
- Power caps per model are set from Argos (Profiles -> Power) through fan-bridge.
  The V420 BIOS loses about 1% at 178 W; rayakame benchmarks at 205 W with HBM
  pinned at 1180 MHz. See [findings: power](../docs/findings.md#power-profiles).
- Try HBM at 1100 and 1180 MHz with a perplexity run to catch silent errors
  (check `pp_dpm_mclk` first).

## What runs on them

- **Engine:** [mx-llama.cpp-Rocm10](https://github.com/Wizard815/mx-llama.cpp-Rocm10)
  behind [Argos](https://github.com/Wizard815/argos), on ROCm 10 from
  [TheRock-gfx906](https://github.com/Wizard815/TheRock-gfx906) `main`.
- **Fits:** 64 GB VRAM across the two cards, plus system RAM for MoE experts
  (`--n-cpu-moe`, `--moe-cache-mib`). Qwen3.8-27B at Q6/Q8 fits with room for
  context. For 500B+ MoEs see Maya and colibri in
  [findings: engines](../docs/findings.md#engines-other-than-llamacpp).
- **Two cards:** layer split works today. Tensor split (`-sm tensor`, MTP drafts
  included) needs working P2P first: see
  [findings: P2P](../docs/findings.md#multi-gpu-and-p2p).
- **Measured here (2026-11-27):** the FA tile rows / K/V prefetch A/B on
  Qwen3.8-27B ([findings](../docs/findings.md#fa-tile-rows-and-kv-prefetch-mx-llamacpp-rocm10-pr-7)).

## Adding a 16 GB card

See [MI50-16GB/SETUP.md](../MI50-16GB/SETUP.md#5-a-16-gb-card-next-to-the-32-gb-cards).
Check `sramecc+`, `pp_dpm_mclk`, `power1_cap_max`, and run the same BIOS family.

## Still to check

The running list is [docs/llm-checklist.md](../docs/llm-checklist.md), and the
merged-but-untested features are in
[findings](../docs/findings.md#merged-but-not-yet-tested-on-the-mi50s).
