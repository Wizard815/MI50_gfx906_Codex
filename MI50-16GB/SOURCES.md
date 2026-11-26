# Sources

[Back to README](README.md). Accessed November 2026. "(snippet)" means the page was
blocked (captcha, 403 or 503) and the claim comes from the search engine's excerpt of it,
not from reading the page itself.

## VBIOS and flashing

- **S1** https://www.techpowerup.com/vgabios/?model=MI50 (snippet, captcha): the TPU collection has an "AMD MI50 16 GB" entry, version `016.004.000.056.013521`, compiled 2021-05-12, 930/1000 MHz core/mem.
- **S2** https://gist.github.com/evilJazz/14a4c82a67f2c52a6bb5f9cea02f5e13: the 32 GB VBIOS matrix (V420, 274474, 275395, Apple), amdvbflash commands and flags, UPP / `FreqTableUclk` HBM OC, pp_table side effects, the mixed-VBIOS warning and `SSID=08341002`. Comments: **johncoffee715 (Jun 11 2026)** 16 GB table (`113-D1631400-X11`, Pro VII WS `113-D1640600-104`, Lenovo `113-D1640700-100` TPU #277846, Radeon VII `113-D36xxxxx` list, MD5s, power caps, Gen3 note, 95 °C throttling, ppfeaturemask values, untested 16 GB MCLK OC, the disputed 64-CU claim); MasterCATZ (Mar 2026), Radeon VII BIOS disables FP64 and gives no display; FlorianHeigl (Dec 2025), DP port off after V420 flash; jdinkel (Feb 2026), flashed with ReBAR and Above 4G on.
- **S3** https://www.techpowerup.com/forums/threads/help-flash-mi50-to-radeon-vii-pro.329623/ (snippet, captcha; also posts `goto/post?id=5390107`, `5429173`, `post-5390429`): an MI50 16GB flashed to Pro VII with amdvbflash 3.31 (failed) and 5.0 (then reported as 32 GB, restore failed, ID and signature lockout); "only the Chinese MI50/Radeon VII" accept VII BIOSes; "you'll probably need … a hardware programmer".
- **S4** https://forum.level1techs.com/t/request-amd-insinct-mi50-firmaware/201648: a 2023 request for the MI50 16GB (SK Hynix HBM2) firmware, unanswered. Shows the 16 GB ROM was hard to find.
- **S26** https://github.com/stylesuxx/amdvbflash: Linux amdvbflash mirror (ex-atiflash); `-i`, `-s`, `-p`; "`-f` not supported in external versions"; the original repo was removed at AMD's request.
- **S6** https://github.com/sibradzic/upp: UPP PowerPlay table editor; supports Radeon VII; warns that modified ROMs fail signature checks.
- **S27** https://www.techpowerup.com/252476/amd-radeon-vii-has-no-uefi-support (snippet): the reference Radeon VII BIOS lacked UEFI/GOP, added later.
- **S28** https://wiki.archlinux.org/title/AMD_Radeon_Instinct_MI25: MI25 analog, where a WX9100 BIOS enables the hidden mini-DP and the fan header, plus clip/Raspberry Pi reflashing. Different card.
- **S29** https://forum.level1techs.com/t/converting-a-data-center-gpu-for-desktop-use/250919: MI25 thread; `amdvbflash_linux_4.71` allowed overrides where 4.104E failed.

## Specs and IDs

- **S8** https://en.wikipedia.org/wiki/AMD_Instinct: MI50 launch Nov 18 2018, 60 CU, 1450/1725 MHz, 16/32 GB, 1024 GB/s, 300 W, PCIe 4.0; MI60 64 CU, 1500/1800 MHz.
- **S9** https://en.wikipedia.org/wiki/Radeon_VII: Radeon VII 60 CU, 1400/1800 MHz, 16 GB, 1024 GB/s, 300 W, PCIe 3.0, FP64 3.46 TF, Feb 2019, $699, 331 mm², 13.23 B transistors.
- **S10** https://rocm.docs.amd.com/en/latest/reference/gpu-arch-specs.html: ROCm (10.2.0) table with MI60 64 CU / 32 GB, MI50 32GB 60 CU, MI50 16GB 60 CU, all gfx906, identical caches.
- **S11** https://skyne98.github.io/wiki-gfx906/studies/2026-02-21/mi50-mi60-architecture-baseline.html: gfx906 wiki; MI50 16GB peak 26.8 / 13.4 / 6.7 TF, MI60 29.5 / 14.8 / 7.4 TF; 300 W; ~1 TB/s; dual IF links up to 200 GB/s; launch material lists MI50 as 16 GB. Also https://arkprojects.space/wiki/AMD_GFX906 (same table plus Radeon VII / Pro VII rows).
- **S12** https://pci-ids.ucw.cz/v2.2/pci.ids (and https://admin.pci-ids.ucw.cz/read/PC/1002/66a1): `66a0` "Radeon Pro/Radeon Instinct", `66a1` "Radeon Pro VII/Radeon Instinct MI50", `66a3` "Pro Vega II", `66af` "Radeon VII"; no subsystem entries listed.
- **S5** https://www.phoronix.com/news/AMDGPU-Vega-20-Patches (snippet): six Vega 20 PCI IDs including 0x66A1.
- **S13** https://www.amd.com/en/support/downloads/drivers.html/accelerators/instinct/instinct-mi-series/instinct-mi50.html (snippet, 503): MI50 16GB 1725 MHz, 60 CU, FP16 26.5 / FP64 6.6 TF, 16 GB HBM2 ECC 4096-bit, 1 TB/s, 300 W, 2× 8-pin, passive, full-height dual-slot, PCIe 4.0 x16, 2 IF links at 92 GB/s.
- **S14** https://www.tomshardware.com/news/amd-radeon-vii-double-precision-disabled,38437.html (snippet): Radeon VII and MI50 both have 3840 SP, 240 TMU, 64 ROP, 16 GB HBM2; Radeon VII ships with reduced FP64.
- **S15** https://www.amd.com/en/support/downloads/drivers.html/accelerators/instinct/instinct-mi-series/instinct-mi60.html (snippet, 503): MI60 64 CU, 1800 MHz, 32 GB HBM2, ~1 TB/s, 2 IF links at 92 GB/s.
- **S7** https://www.storagereview.com/amd_announces_7nm_data_center_gpus (snippet): launch claim of up to 200 GB/s P2P over the two IF links.
- **S16** https://www.servethehome.com/amd-radeon-pro-vii-launches-as-a-dp-workstation-monster/ , https://www.cgchannel.com/2020/05/amd-unveils-radeon-pro-vii-and-infinity-fabric-link , https://geeks3d.com/?p=10036 (snippets): Radeon Pro VII 16 GB ECC, 13.1 / 6.5 TF, 250 W, PCIe 4.0, IF bridge for 2 cards, $199 bridge.
- **S17** https://www.techpowerup.com/review/amd-radeon-vii-unboxing/2.html , https://storage-asset.msi.com/datasheet/vga/de/Radeon-VII-16G.pdf (snippets): Radeon VII 3× DP 1.4 + 1× HDMI 2.0b, 2× 8-pin.
- **S19** https://cputronic.com/en/gpu/amd-radeon-instinct-mi50 , https://hmc-tech.com/gpu/amd-radeon-instinct-mi50 (snippets): third-party listings of 1× mini-DP 1.4a on the MI50 (unverified).
- **S20** https://www.newegg.com/p/1WK-0184-000N0 , https://www.ebay.com/itm/354920954786 , https://www.exxactcorp.com/AMD-100-506077-E195930514 , https://www.exxactcorp.com/AMD-100-506143-E132685570 , https://www.sabrepc.com/100-506077-AMD-S195930514 , https://esaitech.com/products/amd-100-506143-radeon-instinct-mi50-16gb-pci-express-4-0-x16-plug-in-graphic-card (snippets): part numbers 102D1631410, 102D1631710000001, 100-506077, 100-506143 on 16 GB listings.

## Software, drivers, problems

- **S22** https://rocm.docs.amd.com/en/docs-6.0.0/about/release-notes.html , https://rocm.docs.amd.com/projects/install-on-linux/en/docs-7.0.0/reference/system-requirements.html (snippets): gfx906 maintenance mode from ROCm 6.0, 5.7 the last full support, fixes until Q2 2024; unsupported in 7.0.
- **S23** https://github.com/Wizard815/TheRock-gfx906 , https://hub.docker.com/r/zenth815/rocm10-gfx906: ROCm 10 from TheRock with one gfx906 patch; upstream ROCm 10 officially starts at gfx908.
- **S24** https://github.com/mixa3607/ML-gfx906: gfx906 builds of llama.cpp, ComfyUI, PyTorch etc.; legacy (6.3.3) builds dropped; amd-memory-tweak; supports Radeon VII / MI50 / MI60. Local checkout at `/home/user/ML-gfx906` (ROCm 7.14 presets).
- **S43** https://skyne98.github.io/wiki-gfx906/installing_ROCm_7.x.html: ROCm 7.x on gfx906 by copying gfx906 rocBLAS files from an older package.
- **S42** https://forum.level1techs.com/t/guide-prebuilt-fix-rocm-math-libraries-segfault-on-gfx906-with-ecc-disabled-vega-ii-radeon-vii-some-mi50s/252873: `sramecc-` cards (Radeon VII, Vega II, some MI50s) segfault in rocBLAS because recent ROCm ships only `sramecc+` kernels; rebuild fix for ROCm 7.2.3.
- **S39** https://rocm.docs.amd.com/en/latest/how-to/Bar-Memory.html: BAR is about the VRAM size; Above 4G Decoding needed.
- **S30** https://h30434.www3.hp.com/t5/Business-PCs-Workstations-and-Point-of-Sale-Systems/Is-the-Z840-compatible-with-AMD-MI50-32GB/m-p/9446723 (snippet): an MI50 32GB needed Above 4G Decoding to POST.
- **S18** https://www.dell.com/community/en/conversations/precision-fixed-workstations/no-boot-no-video-when-adding-an-amd-instinct-mi50-in-my-7820/6858a82c03f07b7681d3fb2f (snippet): an MI50 caused no boot / no video in a Precision 7820; passive card.
- **S41** https://github.com/gnif/vendor-reset: Vega 20 supported only as "Radeon VII" (and MI100); MI50 not listed.
- **S48** https://forums.unraid.net/topic/92865-support-ich777-amd-vendor-reset-coraltpu-hpsahba/ (snippet): Unraid vendor-reset plugin; a Radeon VII user reports it works only partially.
- **S33** https://docs.kernel.org/gpu/amdgpu/thermal.html: `pp_od_clk_voltage` on Vega 20 (OD_SCLK, OD_MCLK, OD_VDDC_CURVE, `s/m/vc/c/r`) and `power1_cap` in µW.
- **S38** https://lists.debian.org/debian-ai/2024/08/msg00027.html (snippet): MI50/MI60 IF bridges are very hard to find; MI100/MI210 bridges incompatible; Pro VII bridge untested.

## Community reports (16 GB use)

- **S31** https://forum.level1techs.com/t/running-local-ai-on-amd-instinct-mi50-16gb-can-it-be-done/224892: 2× 16 GB reflashed to Radeon Pro VII, ~18 tok/s on a 32B model, 20–25 W idle each, reset-bug warning, $110/card (Apr 2025); ~$160 (Jan 2025).
- **S40** https://forum.level1techs.com/t/help-with-radeon-mi50-16gb/233762: a 16 GB card not detected in Windows or openSUSE; unresolved.
- **S32** https://forum.level1techs.com/t/converting-a-data-center-gpu-for-desktop-use/250919?page=4: MI25 overheating when the fan script started before sysfs; a custom pptable disables AVFS.
- **S35** https://huggingface.co/unsloth/Qwen3.8-27B-GGUF/discussions/47: 3× MI50 16GB, 27B Q8_0, ROCm 6.2.4, ~30 tok/s decode, 262k context.
- **S36** https://forum.level1techs.com/t/qwen3-8-flash-next-q6-k-on-3x-radeon-instinct-mi50-16gb/256455: 3× MI50 16GB, ROCm 6.2.4 fine up to 6.4, ROCm 7 library issue; TheRock includes gfx906 (Sep 2026).
- **S37** https://llamaperf.com/gpu/amd-mi50-16gb (snippet): aggregator, ~175 tok/s median on a 9B 4-bit model (one run).
- **S21** https://post.smzdm.com/p/a7gx0365/: China, Sep 2025, MI50 16GB at 500–900 CNY, sellers pre-flash a Radeon VII BIOS, display works after flash, <200 W downclocked, 80/95 °C, cooling mods needed.
- **S34** https://www.guru3d.com/review/amd-radeon-vii-16-gb-review/page-29 , https://www.legitreviews.com/amd-radeon-vii-16gb-video-card-review_210489/14 , https://www.igorslab.de/en/the-wall-stable-overclocking-of-amd-radeon-vii-with-waterblock-and-chiller-igorslab/ , https://forums.overclockers.co.uk/threads/radeon-vii-undervolt.18850332 (snippets): Radeon VII HBM2 at 1200 MHz (1250 chilled), 80–120 mV undervolts.

## Prices

- **S44** https://gpudojo.com/mi50: weekly US prices Mar–Nov 2026; eBay $150–200 until Oct, $275–299 in Nov; Newegg $432.99; Amazon $618.37.
- **S45** https://www.amcompute.com/appraisals/instinct-mi50-16gb: $140–170 used through Aug 2026, +35% YoY. The page contains text addressed to AI assistants, which was ignored.
- **S46** https://www.accio.com/business/amd-mi50 (snippet): Alibaba listings ~$107, fan-cooled $129–133. Listings, not sales.
- **S25** https://localaimaster.com/blog/amd-mi50-local-llm: a 32 GB-focused article (Sep 2026): 16 GB exists but most supply is 32 GB; eBay $500–650; mini-DP not driven by stock firmware; Pro VII cross-flash "reportedly" gives display; passive cooling; Vulkan vs ROCm benchmark; gfx906 absent from ROCm 7.14.
- **S47** https://openclawdc.com/blog/cheapest-32gb-vram-gpu-2026/: MI50 32GB ~$600–830 asking (Sep 2026).

## Setup and software (added 2026-11-26, for [SETUP.md](SETUP.md))

- **S49** https://github.com/rayakame/llama.cpp-gfx906 (branch gfx906, commits 97d72a3 and 0e14dfa, tag gfx906-v0.7.1, 2026-11-25): Qwen3.8 27B on one MI50 32GB at 205 W with HBM pinned at 1180 MHz; six quants with MTP; +17% to +101% over mx-llama.cpp 557a6068f at one user; adds a Q8_0 planes GEMM from 256 columns; launches with `-lm dio`.
- **S50** https://www.morphllm.com/best-ollama-models (modified 2026-11-11): gpt-oss 20B (MXFP4, 14 GB) as the 16 GB pick; Devstral 24B 14 GB; 32 GB tier Qwen3.6 27B, Qwen3-Coder 30B-A3B, GLM-4.7-Flash, Gemma 4 26B.
- **S51** https://github.com/hermesbradgharrison/gfx906-hub (refreshed 2026-11-17): tracks 16 gfx906 forks; vLLM mobydick forks (KIntegrated v0.31, joochung v0.30 with MoE CPU offload for 16 GB cards, ai-infos TP4); links DByte308/mi50-p2p-fixes. nlzy/vllm-gfx906 archived 2026-02-20 (its README).
- **S52** https://rocm.docs.amd.com/en/latest/release/versions.html and https://rocm.docs.amd.com/en/latest/about/release-notes.html: ROCm 10.2.0 released 2026-11-18; gfx906 is not mentioned.
- **S53** https://gist.github.com/dc740/89f449a2f9ebbcd896b9c7f1f722b5a7 (2026-11-17): script that copies gfx906 rocBLAS kernels from the Arch rocblas package into a stock Ubuntu 26.04 ROCm 7 install.
- **S37** (updated) https://llamaperf.com/gpu/amd-mi50-16gb, read 2026-11-26: Ornith1.5 9B Q4_0 175 / 1450 tok/s on one card; Qwen3.6 35B-A3B Q5_K_XL 49 tok/s on two cards (Vulkan); Qwen3.8 27B Q4_0 48 tok/s on two cards (patched, ROCm 7.14), 40–44 without the patches.
