# What we found

[Back to README](../README.md). Research and MI50 test results from November 2026,
newest first within each section. Each item links to the deeper doc or PR.

## ROCm: stay on TheRock main, not ROCm 10.2

- **ROCm 10.2.0 (2026-11-18) is older than TheRock-gfx906 `main`.** The 10.2 tag
  split from main on 2026-10-18 and lacks the amd-llvm scheduler spill-rejection
  fix (5eaa4e7e0). gfx906 spills much more on ROCm 10.x than on 7.2, so that fix
  matters. Nothing on `release/therock-10.2` is gfx906-specific.
- Keep the pin on `main` (10.3 dev). The vega20 rocBLAS Tensile logic is intact.
- **Never put gfx906 in `ROCBLAS_TENSILE_ARCH_SKIPLIST`**: GEMMs would fall back to
  slow source kernels, since hipBLASLt has no gfx906 kernels.
- Next lever: measure the next amd-llvm bump (amd-staging 60f85815 and a02af9f6
  change scheduling under register pressure) for spills and the `max-ilp` gap.
- Details: [TheRock-gfx906 GFX906.md](https://github.com/Wizard815/TheRock-gfx906/blob/main/GFX906.md#stable-vs-development-sources).

## mx-llama.cpp kernels

### FA tile rows and K/V prefetch (mx-llama.cpp-Rocm10 PR 7)

MI50 A/B on 2026-11-27 with `scripts/gfx906-fa-prefetch-ab.sh`, 2x MI50 32GB,
Qwen3.8-27B UD-Q6_K_XL (GQA 6):

- All 503 FA tests pass.
- Only win: 16 tokens / 32 columns, **-11% FA time**.
- GQA 2 with the tile rows: **5-33% slower** from 1024 K/V rows.
- `GGML_CUDA_FA_GQA_PAD=0`: decode **16-31% slower** at 16K+ rows, so padding stays on.
- End to end: within about 1%.

Result: [PR 7](https://github.com/Wizard815/mx-llama.cpp-Rocm10/pull/7) turns
`GGML_FA_TILE_GFX906` off by default and sets `GGML_FA_KV_PREFETCH=32`
(`62` gives the old behaviour). Don't turn the rows back on without a new MI50 A/B.

### Ports from llama-of-vega.cpp and slop.cpp (PR 8)

[PR 8](https://github.com/Wizard815/mx-llama.cpp-Rocm10/pull/8) ports:

- vega 6943a105: HIP graph churn fix (2 lines).
- vega 210b13ec plus the IQ3_XXS vec-dot mask: IQ2_S / IQ3_XXS sign fix.
- slop's pinned async upload ring as `GGML_CUDA_UPLOAD_RING=1`, **off by default**.

Builds for gfx906; not GPU-tested yet. Still worth porting later from vega: MoE id
counting sort, top-k MoE 512x10, sparse tile FA. slop's Q8_0 MMVQ tuning conflicts
with the fork's fixed Q8_0 layout for exact MTP verify, so it isn't ported.

### Earlier (merged)

- PR 2: rayakame's MTP verify-pass row-group mat-vec, exact top-k sampler, padded
  MTP drafts. Compiled only; A/B it with `GGML_MMVQ_GCN_MC=0 GGML_MMVQ_GCN_TABLE=0
  LLAMA_SAMPLER_SPARSE_TOPK=0 LLAMA_ARG_SPEC_DRAFT_P_MIN_PAD=0`.
- PR 6: upstream ROCm fixes (FA head size 576 tile pick, MMF overflow, top-k MoE
  fusion) and tensor-split MTP (`--spec-type draft-mtp -sm tensor`), verified on CPU.
- Held back: upstream #32901 (FA refactor) and what builds on it; needs MI50 numbers.

## Engines other than llama.cpp

**llama.cpp (mx-llama.cpp-Rocm10 behind Argos) stays the daily engine** for
anything that fits 64 GB VRAM plus system RAM (`--n-cpu-moe`, `--moe-cache-mib`).

| Engine | Verdict |
|---|---|
| **Maya** (mw00/project-maya, built in [ML-gfx906/project-maya](https://github.com/Wizard815/ML-gfx906/tree/master/project-maya), image `mixa3607/maya-gfx906`) | Optional sidecar for 500B+ MoEs (DeepSeek V4.1 Flash, GLM, MiniMax) with experts read from disk and TP across GPUs. Upstream's DeepSeek engine wants V4.1-Flash (Maya-XS 277 GB); GLM-5.3-Flash Maya-S (96.5 GB) is the fallback that fits. Reported: ~19.5 tok/s on 2x MI50 for DS-V4-Flash IQ3_S on a community build. |
| **Strata** | Dead for gfx906: mixa3607 removed the build on 2026-11-22. Community forks are single-card only. Skip, or at most one A/B on a mid-size MoE. |
| **colibri** ([colibri-gfx906](https://github.com/Wizard815/colibri-gfx906)) | CPU-first, about 10 GB resident for a 744B model. For "run a huge model at all", not for serving. HIP tier untested on MI50 (`make hip-test HIP=1 HIP_ARCH=gfx906`). |
| **gfx906-quartz** (iacopPBK) | Tiny 2-card engine, Q8_0 only, Qwen3.8-27B (MTP) and Inkling-2-Mini, one request at a time, no auth. ~62-138 tok/s decode reported at 135 W. Worth a sidecar test if you run Qwen3.8-27B Q8. No image: build on `mixa3607/rocm-gfx906:7.14-complete`. |
| **Llaminar** | Alpha multi-backend engine with gfx906 kernels, TP/PP, image `ghcr.io/llaminar/llaminar:master`. Watch it and maybe A/B it against mx-llama on Qwen3.8-27B; not a replacement yet. A P2P data-check patch for it is written but needs a Wizard815 fork to push. |
| **vLLM** | nlzy/vllm-gfx906 is archived. The maintained forks are the "mobydick" ones, listed in [gfx906-hub](https://github.com/hermesbradgharrison/gfx906-hub). |

## Multi-GPU and P2P

From the [DByte308/mi50-p2p-fixes](https://github.com/DByte308/mi50-p2p-fixes)
write-up and our own checks:

- **Keep the GPU BARs below 16 TiB.** With BARs placed above the 44-bit range,
  peer traffic fails. The fix there was a UEFI setting that moves the BARs below
  16 TiB.
- **Keep the native 44-bit DMA mask.** Don't force a wider mask to "fix" P2P.
- **`hipDeviceCanAccessPeer = 1` can still lie.** A pair can report peer access and
  still corrupt data or hang. Test with a real byte-for-byte copy check, not just
  the capability flag and bandwidth. RockComp's `bench p2p` only checks the flag
  and bandwidth today; a bidirectional, re-seeded compare should be added.
- Check `iommu=pt` in `/proc/cmdline`: a missing `iommu=pt` is RCCL's documented
  cause of hangs, and RCCL only warns about it with `NCCL_DEBUG` set.
- When P2P works, tensor split gave +22-26% decode over layer split in that
  write-up. mx-llama's custom peer-write AllReduce (`GGML_ENABLE_CUSTOM_AR`) beats
  the RCCL ring over PCIe.
- Before trying `-sm tensor` on the ML350: run RockComp `bench info` (BAR size per
  GPU, topology, P2P matrix) and `rocm-bandwidth-test`, and check ACS.

## Power profiles

- Argos sets a **power cap and top SCLK per profile** (Profiles -> Power), with an
  idle setting and a main switch. Benchmarks save watts, tokens per watt and
  prompt tokens per watt.
- Argos's container has a read-only `/sys`, so the writes go through **fan-bridge**
  (`/api/power` in [ilofanmonitorgpu](https://github.com/Wizard815/ilofanmonitorgpu)),
  which needs its writable `/gpu/cardN` mounts.
- Junction guard 95 °C: the card halves SCLK above about 95 °C.
- Decode is bandwidth-bound, so the cap mostly costs prompt speed. The V420 32 GB
  BIOS loses about 1% at 178 W. Compare stock, 205 W, 180 W, 150 W and 135 W per model.
- Still to measure: real tokens per watt per model, and whether the V420 default cap
  really is 178 W.

## Merged but not yet tested on the MI50s

In rough order of value (survey of all repos, 2026-11-27):

1. mx-llama PR 2 MTP verify-pass kernels (A/B switches above).
2. rayakame v0.7.1 vs our build, same GGUF, 205 W.
3. Turbo KV (`turbo3`/`turbo4`) under 2-GPU layer split.
4. whisper.cpp fork HIP path and `max-ilp`.
5. colibri 2.2.0 HIP tier.
6. mx-llama build-time A/Bs: `FA_VEC_MIN_BLOCKS`, `DPP_REDUCE`, `HIP_MAX_ILP`,
   GCN MMQ legacy.
7. RockComp `bench info` / `p2p`.
8. Argos features tested only against the simulator, including power cap writes.
9. mx-llama PR 7 and PR 8 once merged.

Note: GDN `rs_replay` is hard-coded off in mx-llama and needs a switch before any
A/B. The running checklist is [llm-checklist.md](llm-checklist.md).
