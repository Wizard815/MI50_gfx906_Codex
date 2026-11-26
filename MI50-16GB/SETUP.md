# MI50 16GB: LLM setup

[Back to README](README.md). Source numbers like [S2] refer to [SOURCES.md](SOURCES.md).

A starting setup for running LLMs on one MI50 16GB card, or a 16 GB card next to
the 32 GB cards, with **mx-llama.cpp-Rocm10** launched by **Argos**. Sizes are
estimates (parameters × bits per weight / 8, plus KV cache and about 1 GB of
compute buffers), not measurements on this server. Check each one with
`llama-bench` and Argos's **Profiles -> Power** before you rely on it.

Written 2026-11-26.

## 1. Check the card first

```bash
lspci -nn | grep -i 66a1                 # MI50 16GB and 32GB are both 1002:66a1
rocminfo | grep -o 'gfx906[^ ]*'         # gfx906:sramecc+ is what the libraries expect
cat /sys/class/drm/card*/device/pp_dpm_mclk          # HBM clock the BIOS really sets
cat /sys/class/drm/card*/device/hwmon/hwmon*/power1_cap_max   # BIOS power limit (µW)
cat /sys/class/drm/card*/device/mem_info_vram_total  # about 17.1e9 bytes on a 16 GB card
```

- If `rocminfo` shows `gfx906:sramecc-`, rocBLAS from ROCm 7+ segfaults. Use the
  TheRock-gfx906 / ML-gfx906 images or rebuild rocBLAS for `sramecc-`
  ([common problems](UPGRADES-AND-TUNING.md#common-problems)).
- If `pp_dpm_mclk` tops out at 600 MHz, fix that before anything else (see
  [HBM overclock](UPGRADES-AND-TUNING.md#hbm-overclock-on-16-gb)). Decode speed
  follows memory bandwidth almost one for one.
- Write the `power1_cap_max` value down: the stock 16 GB BIOS's limit is still
  unconfirmed, and this answers it ([open questions](README.md#open-questions)).

## 2. Power cap and clocks

Token generation is bandwidth-bound, so the cap mostly costs prompt speed.

| Setting | Start at | Then compare | Why |
|---|---|---|---|
| Power cap | **180 W** | 150 W, 205 W, BIOS default | rayakame's fork benchmarks a 32 GB MI50 at 205 W [S49]; your V420 32 GB cards lose about 1% at 178 W [S2] |
| Idle | Argos idle setting on | — | 16 GB cards idle at 20–25 W each [S31] |
| HBM (MCLK) | BIOS value, 1000 MHz | 1100, 1180 MHz in 20–40 MHz steps | rayakame pins 1180 MHz on the 32 GB card from v0.7.1 [S49]; untested on 16 GB (4-Hi) stacks |
| Junction guard | 95 °C (fan-bridge default) | — | the card halves SCLK above ~95 °C [S2] |

In Argos, add a power entry per model profile (Profiles -> Power): one at 180 W
and one at 150 W, run the same `llama-bench` on both, and keep the one with the
best tokens per watt that still meets the speed you want. Argos writes the cap
through fan-bridge's `/api/power`, so fan-bridge needs its writable `/gpu/cardN`
mounts.

## 3. Models that fit one 16 GB card

Leave about 1–1.5 GB free for compute buffers and the desktop/ROCm runtime. The
context column assumes `-fa on` with a `q8_0` KV cache.

| Use | Model | Quant | File (est.) | Context to start with | Notes |
|---|---|---|---|---|---|
| Fast general / agent | Ornith1.5 9B | Q4_0 or Q8_0 | 5 / 9.5 GB | 32k (Q8) – 64k (Q4) | 175 tok/s decode, 1450 tok/s prompt reported at Q4_0 on one 16 GB card [S37] |
| Strong all-round, dense | Qwen3.8 27B | UD-IQ3_XXS | ~11 GB | 16k–24k | does not fit at Q4 (~16.5 GB). Has an MTP head: add `--spec-type draft-mtp`. rayakame measures IQ3_XXS at 45 tok/s with MTP on a 32 GB card [S49] |
| Long context MoE | gpt-oss 20B | MXFP4 (native) | ~13 GB | 32k–64k | sliding-window attention keeps the KV small; listed as the 16 GB pick by Ollama rankings [S50] |
| Coding | Devstral 24B | Q3_K_M / IQ4_XS | ~11.5 / 12.8 GB | 16k | Q4_K_M is 14 GB and leaves almost no room for context [S50] |
| Big MoE, partly on CPU | Qwen3.6 35B-A3B | Q4_K_M | ~21 GB | 32k | needs `--n-cpu-moe` (start at 12–16) so the experts that don't fit sit in system RAM. 49 tok/s was reported on two 16 GB cards with the whole model on GPU [S37] |
| Small / draft | Qwen3 4B or similar | Q8_0 | ~4.5 GB | — | only as a `--model-draft` for a bigger model on another card |

Models that need two 16 GB cards, or a 16 GB + 32 GB pair: Qwen3.8 27B at
Q4_0–Q8_0 (48 tok/s at Q4_0 on two 16 GB cards with gfx906 patches [S37]),
Qwen3-Coder 30B-A3B and GLM-4.7-Flash at Q4–Q6, Gemma 4 26B at Q6–Q8.

## 4. Launch flags

One 16 GB card, dense model (Qwen3.8 27B IQ3_XXS with MTP):

```bash
HIP_VISIBLE_DEVICES=2 llama-server \
  -m /models/Qwen3.8-27B-UD-IQ3_XXS.gguf \
  -ngl 99 -fa on -ctk q8_0 -ctv q8_0 \
  -c 16384 -np 1 -b 1024 -ub 512 \
  --spec-type draft-mtp --spec-draft-n-max 3 --spec-draft-p-min 0.3 \
  -lm dio --jinja --host 0.0.0.0 --port 8082
```

One 16 GB card, MoE with experts in system RAM (Qwen3.6 35B-A3B):

```bash
HIP_VISIBLE_DEVICES=2 llama-server \
  -m /models/Qwen3.6-35B-A3B-Q4_K_M.gguf \
  -ngl 99 --n-cpu-moe 14 -fa on -ctk q8_0 -ctv q8_0 \
  -c 32768 -np 1 -b 2048 -ub 1024 -t 16 \
  --jinja --host 0.0.0.0 --port 8082
```

Lower `--n-cpu-moe` one step at a time until the load fails with an
out-of-memory error, then go back one.

What the 16 GB-specific choices do:

| Flag | On 16 GB | Why |
|---|---|---|
| `-ctk q8_0 -ctv q8_0` | always | halves the KV cache against f16 at almost no quality cost |
| `-ctk q8_0 -ctv turbo4` | when context is the limit | the fork's turbo V types give about 3.3× KV capacity at a decode-speed cost; one GPU only, multi-GPU split is unverified |
| `-ub 512` | for dense models | a smaller ubatch shrinks the compute buffer by a few hundred MB; it slows long-prompt prefill |
| `-np 1` | unless you need parallel users | each slot needs its own share of context |
| `--fit on` | to find a starting point | lets llama.cpp shrink unset context/offload to fit; set the values explicitly in Argos once known |
| `-lm dio` | large GGUFs | direct I/O load, avoids filling the page cache twice; rayakame's fork uses it [S49] |
| `--spec-type draft-mtp` | Qwen3.8 27B | the built-in MTP head; the fork has the MTP verify-pass mat-vec kernels from rayakame |

In Argos these map to the builder's GPU, context, KV type, System RAM
(`--n-cpu-moe` slider) and speculative fields, so set them in a profile
instead of a hand-written command line.

## 5. A 16 GB card next to the 32 GB cards

- Order the cards largest first and split by VRAM: `-sm layer -ts 2,2,1` for
  32 + 32 + 16, `-ts 2,1` for 32 + 16.
- Don't use `-sm tensor` with mixed sizes: each layer is split evenly, so the
  16 GB card caps everyone's share.
- Good uses for a lone 16 GB card in this server: a second, smaller model
  (Ornith1.5 9B, gpt-oss 20B) that runs while the 32 GB cards serve the big
  one, a draft model for speculative decoding, or Whisper / embeddings.
- Keep the same BIOS family on every card; mixed VBIOS versions caused ROCm
  problems on 32 GB cards [S2].

## 6. Software to run it on (as of 2026-11-26)

- **mx-llama.cpp-Rocm10** (this server's build) runs unchanged on 16 GB.
- **rayakame/llama.cpp-gfx906 v0.7.1** (2026-11-25) is the fastest measured
  fork for Qwen3.8 27B on a 32 GB MI50 with MTP: +17% to +101% over
  mx-llama.cpp at one user [S49]. It ships a Docker image, so it is easy to A/B
  against your build on the 16 GB card before porting more of it.
- **vLLM**: nlzy/vllm-gfx906 was archived on 2026-02-20. The maintained forks
  are the "mobydick" ones tracked by gfx906-hub; joochung's has CPU offload of
  big MoE models for 16 GB cards [S51].
- **ROCm**: official releases are at 10.2.0 (2026-11-18) and do not list
  gfx906 [S52]. Keep using the TheRock-gfx906 build; on a stock Ubuntu 26.04
  ROCm 7 install, dc740's script adds the gfx906 rocBLAS kernels [S53].
