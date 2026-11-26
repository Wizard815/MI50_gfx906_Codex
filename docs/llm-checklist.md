# MI50 LLM: what's still left to check

Status on 2026-11-26 for the 2x MI50 32GB (V420 BIOS) in the ML350, plus a
possible 16 GB card. Tick these off on the real hardware; nothing here has been
measured from a cloud session.

## Already known and pending on your side

- [ ] TheRock-gfx906 rebuild
- [ ] `scripts/gfx906-fa-prefetch-ab.sh` in mx-llama.cpp-Rocm10 (FA tile rows and
      K/V prefetch vs `GGML_CUDA_FA_GQA_PAD=0`); send the `summary.md`
- [ ] fan-bridge writable `/gpu/cardN` mounts, so Argos power profiles can write caps
- [ ] Argos `/v1` API key test

## New since 2026-11-25

- [ ] **A/B rayakame v0.7.1 against your build.** v0.7.1 (2026-11-25) adds a
      Q8_0 planes GEMM for prompts of 256+ tokens and is measured +17% to +101%
      over mx-llama.cpp at one user on Qwen3.8 27B with MTP. Run
      `ghcr.io/rayakame/llama.cpp-gfx906:b12230-v0.7.1` on one card with the same
      GGUF, 205 W and the same flags, and compare with your mx-llama.cpp-Rocm10
      build. That tells you whether porting more of it (planes, 9f96ebe39 onward)
      is worth it.
- [ ] **Pin HBM at 1180 MHz and measure.** rayakame now benchmarks with the
      memory clock pinned at 1180 MHz. Check what your V420 cards run
      (`pp_dpm_mclk`), then try 1100 and 1180 MHz with a perplexity run to catch
      silent HBM errors.
- [x] **Upstream llama.cpp since the last sync (c9433313):** e38ea71 (#33045,
      ROCm: no tile FA kernel for head size 576 when the GQA ratio is not a
      multiple of 4, for MLA models like DeepSeek/GLM), the MMF/MMVF offset
      overflow fixes (#33044, #32965) and rayakame's router Connection-header
      fix are in mx-llama.cpp-Rocm10 master since PR #6 (2026-11-26).
      rayakame's Q8_0 planes GEMM is mx-llama.cpp's own repacked GEMM, which
      the fork already has.
- [ ] **#32901** (FA data-loading refactor) is still the one upstream change
      that conflicts with the gfx906 tile rows and needs MI50 numbers before
      merging.
- [ ] **ROCm 10.2.0** came out 2026-11-18 without gfx906. Whether the next
      TheRock-gfx906 rebuild should move its pin to the 10.2 tag is being
      looked at in its own thread.
- [ ] **vLLM:** nlzy/vllm-gfx906 is archived. If you want vLLM for
      multi-user serving, the maintained forks are the "mobydick" ones
      (KIntegrated, v0.31 base). Tracked at
      https://github.com/hermesbradgharrison/gfx906-hub.
- [ ] **P2P:** DByte308/mi50-p2p-fixes has a PCIe P2P write-up for 4x MI50.
      Check `rocm-bandwidth-test` / ACS on the ML350 before trying `-sm tensor`.

## Still open from earlier research

- [ ] Real tokens per watt per model at stock, 180 W and 150 W (Argos
      Profiles -> Power), and whether the V420 default cap is really 178 W
- [ ] Turbo KV (`turbo3`/`turbo4`) under multi-GPU layer split: unverified
- [ ] GDN replay vs the fork's snapshot ring (`rs_replay`): VRAM and speed
- [ ] Bonsai one-binary port (parked on `mx-llamacpp-bonsai-wip`)

## If you add a 16 GB card

See [MI50-16GB/SETUP.md](../MI50-16GB/SETUP.md). Check first: `sramecc+`,
`pp_dpm_mclk`, `power1_cap_max`, and that it runs the same BIOS family as the
other cards.
