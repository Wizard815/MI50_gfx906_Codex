# MI50_gfx906_Codex

One place for everything about running AMD Instinct **MI50** (gfx906, Vega 20)
cards for local AI: the repos, the Docker images, the 16 GB and 32 GB card
research, and what has been learned so far.

The server is an HPE ML350 Gen10 with **2x MI50 32GB** (V420 BIOS), managed by
Argos. Notes for adding a **16 GB** card are in [MI50-16GB/](MI50-16GB/README.md).

Status on 2026-11-27.

## Start here

| I want to... | Go to |
|---|---|
| Know what the 32 GB cards in this server run and how they're tuned | [MI50-32GB/README.md](MI50-32GB/README.md) |
| Run LLMs on a 16 GB card, or next to the 32 GB cards | [MI50-16GB/SETUP.md](MI50-16GB/SETUP.md) |
| Compare MI50 16GB / 32GB / MI60 / Radeon VII / Pro VII | [MI50-16GB/SPECS-AND-DIFFERENCES.md](MI50-16GB/SPECS-AND-DIFFERENCES.md) |
| Flash a BIOS | [MI50-16GB/BIOS.md](MI50-16GB/BIOS.md), [evilJazz 32GB gist](https://gist.github.com/evilJazz/14a4c82a67f2c52a6bb5f9cea02f5e13) |
| Read what we found (ROCm version, kernels, engines, P2P, power) | [docs/findings.md](docs/findings.md) |
| See what still needs testing on the real cards | [docs/llm-checklist.md](docs/llm-checklist.md), [findings: untested](docs/findings.md#merged-but-not-yet-tested-on-the-mi50s) |
| Find a Docker image | [Docker images](#docker-images) below |
| Run AI agents safely on the box | [docs/agent-stack.md](docs/agent-stack.md), [guardian/](guardian/README.md) |

## The stack

```
 Odysseus (chat, agents, tools) ──► Argos /v1 ──► mx-llama.cpp-Rocm10 llama-server
                                     │   │            (also whisper.cpp, Kokoro, sd.cpp)
                                     │   └─ power profiles ──► fan-bridge /api/power
                                     │                           (ilofanmonitorgpu)
                                     ▼
                   ROCm 10 for gfx906, built from TheRock-gfx906
                   (zenth815/rocm10-gfx906 image; RockComp builds on it)
```

## Repos

| Repo | What it is for | Status |
|---|---|---|
| [argos](https://github.com/Wizard815/argos) | One OpenAI-compatible `/v1` endpoint for all local models. Replaces llama-swap: launch command builder, profiles, GPU scheduler, benchmarks, per-profile power caps. | Daily driver. Power profiles merged (PR 1), System RAM card (PR 2). Many features only tested against the simulator. |
| [mx-llama.cpp-Rocm10](https://github.com/Wizard815/mx-llama.cpp-Rocm10) | llama.cpp fork (mx-llama.cpp base) with gfx906 kernel tuning: weight repack, MMQ/MMVQ tuning, MTP verify-pass mat-vec, tensor parallel, custom P2P AllReduce, turbo KV. See [GFX906-ROCM10.md](https://github.com/Wizard815/mx-llama.cpp-Rocm10/blob/master/GFX906-ROCM10.md). | Daily engine behind Argos. Upstream synced to c9433313 + PR 6 fixes. Open: [PR 7](https://github.com/Wizard815/mx-llama.cpp-Rocm10/pull/7) (FA prefetch defaults), [PR 8](https://github.com/Wizard815/mx-llama.cpp-Rocm10/pull/8) (vega/slop ports). |
| [ilofanmonitorgpu](https://github.com/Wizard815/ilofanmonitorgpu) | **fan-bridge**: drives HPE iLO fans from GPU temperatures (iLO can't see the MI50s), writes power caps for Argos, and runs JBOD fans on an ESP32. | In use on iLO 5 / ML350 Gen10. Power API merged (PR 1). |
| [odysseus](https://github.com/Wizard815/odysseus) | Self-hosted AI workspace (chat, agents, research, docs, mail). Fork of odysseus-dev/odysseus with an Argos connection suite. | In use; `docker-compose.argos.yml` points it at Argos. |
| [whisper.cpp-rocm10-gfx906](https://github.com/Wizard815/whisper.cpp-rocm10-gfx906) | whisper.cpp fork for gfx906 / ROCm 10 with a llama-server-style model lifecycle (`--lazy-load`, `/unload`, `--idle-timeout`). | Work in progress. Lifecycle verified on CPU; HIP path and `max-ilp` not benchmarked on the MI50s. |
| [colibri-gfx906](https://github.com/Wizard815/colibri-gfx906) | Mirror of JustVugg/colibri: CPU-first pure-C engine that streams MoE experts from disk to run very large models at all. | Pure upstream mirror (2.2.0). HIP tier untested on MI50. See [verdict](docs/findings.md#engines-other-than-llamacpp). |
| [ML-gfx906](https://github.com/Wizard815/ML-gfx906) | Mirror of mixa3607/ML-gfx906: ROCm, PyTorch, vLLM, ComfyUI, llama.cpp and Project Maya builds for gfx906. | Pure upstream mirror. Source of the `mixa3607/*-gfx906` images. |
| [RockComp](https://github.com/Wizard815/RockComp) | Build/deploy recipe for mx-llama.cpp on the ROCm 10 image, plus the `bench/` container (GPU/BAR/P2P report, llama-bench sweeps, rocBLAS tuning). | Private repo. `p2p` check measures access + bandwidth only, no data check yet. |
| [TheRock-gfx906](https://github.com/Wizard815/TheRock-gfx906) | ROCm/TheRock fork that builds ROCm 10 for gfx906 only. Recipe in [GFX906.md](https://github.com/Wizard815/TheRock-gfx906/blob/main/GFX906.md). | Stay on `main` (10.3 dev), not the 10.2 release. Docs PR 3 merged. |

## Docker images

Images the repos above actually pull or build. Tags are the ones the compose files
and docs use.

**Our gfx906 stack (Docker Hub)**

| Image | What | Built from / used by |
|---|---|---|
| [zenth815/rocm10-gfx906](https://hub.docker.com/r/zenth815/rocm10-gfx906) `:sdk`, `:runtime` | ROCm 10 for gfx906: full toolchain / libs only | Built from TheRock-gfx906. Base for RockComp, whisper.cpp fork, mx-llama self-build |
| [zenth815/mx-llama-rocm10-gfx906](https://hub.docker.com/r/zenth815/mx-llama-rocm10-gfx906) `:latest` | mx-llama.cpp on a trimmed ROCm 10 runtime (~1.9 GB) | RockComp `Dockerfile`. Default base in mx-llama.cpp-Rocm10 `docker-compose.yml` (`GFX906_BASE_IMAGE`) and `deploy/` |
| [mxxm/mx-llama.cpp](https://hub.docker.com/r/mxxm/mx-llama.cpp) | Upstream mx-llama.cpp images | Linked from the mx-llama.cpp-Rocm10 banner |

To build these yourself instead of pulling: [mx-llama.cpp-Rocm10 deploy/self-build](https://github.com/Wizard815/mx-llama.cpp-Rocm10/blob/master/deploy/self-build/README.md)
and the [TheRock-gfx906 SDK image](https://github.com/Wizard815/TheRock-gfx906/blob/main/GFX906.md#sdk-image-from-the-build).

**Community gfx906 images (mixa3607, built in ML-gfx906)**

| Image | What |
|---|---|
| [mixa3607/rocm-gfx906](https://hub.docker.com/r/mixa3607/rocm-gfx906) | ROCm base (TheRock 7.x/10.x); quartz builds on `7.14-complete` |
| [mixa3607/rocm-toolkit-gfx906](https://hub.docker.com/r/mixa3607/rocm-toolkit-gfx906) | ROCm toolkit |
| [mixa3607/pytorch-gfx906](https://hub.docker.com/r/mixa3607/pytorch-gfx906) | PyTorch |
| [mixa3607/llama.cpp-gfx906](https://hub.docker.com/r/mixa3607/llama.cpp-gfx906) | Upstream llama.cpp |
| [mixa3607/vllm-gfx906](https://hub.docker.com/r/mixa3607/vllm-gfx906) | vLLM |
| [mixa3607/comfyui-gfx906](https://hub.docker.com/r/mixa3607/comfyui-gfx906) | ComfyUI |
| [mixa3607/maya-gfx906](https://hub.docker.com/r/mixa3607/maya-gfx906) | Project Maya (giant-MoE engines) |
| [mixa3607/vega20-metrics-exporter](https://hub.docker.com/r/mixa3607/vega20-metrics-exporter) | Prometheus metrics for Vega 20 |

**Other images in the stack**

| Image | Where |
|---|---|
| `ghcr.io/rayakame/llama.cpp-gfx906:b12230-v0.7.1` | rayakame's gfx906 llama.cpp, for the A/B in the [checklist](docs/llm-checklist.md) |
| `ghcr.io/llaminar/llaminar:master` | Llaminar engine ([verdict](docs/findings.md#engines-other-than-llamacpp)) |
| `ghcr.io/odysseus-dev/odysseus:latest` | Default `ODYSSEUS_IMAGE` in the odysseus compose files |
| `fan-bridge:latest`, Argos, `rockcomp:gfx906` | Built locally from ilofanmonitorgpu, argos and RockComp; not published |

## Basics

BIOS upgrades
- 32 GB: https://gist.github.com/evilJazz/14a4c82a67f2c52a6bb5f9cea02f5e13
  (personally using lower TDP): https://katastrophos.net/downloads/InstinctMI50/V420.rom
- 16 GB: [MI50-16GB/BIOS.md](MI50-16GB/BIOS.md). Never flash a 32 GB ROM onto a 16 GB card.

Guide (6/1): https://youtu.be/xcI0pyE8VN8?si=-sa-YsBmRRSWKBRL

Ubuntu: https://ubuntu.com/download/server (22.04)

## Outside trackers worth watching

- [hermesbradgharrison/gfx906-hub](https://github.com/hermesbradgharrison/gfx906-hub): bot-updated list of gfx906 forks, and the maintained vLLM forks
- [rayakame/llama.cpp-gfx906](https://github.com/rayakame/llama.cpp-gfx906): the most active gfx906 llama.cpp fork
- [mixa3607/ML-gfx906](https://github.com/mixa3607/ML-gfx906) and its [wiki](https://arkprojects.space/wiki/AMD_GFX906)
