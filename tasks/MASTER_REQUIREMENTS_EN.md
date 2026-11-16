# Master Requirements – Variant B (English Implementer Edition)

## Mission

Design and implement a production-oriented Qwen3.8-Flash-Next stack for a single RTX PRO 6000 Blackwell 96 GB GPU connected through Thunderbolt 4. The stack serves OMO/OpenCode coding agents and must maximize completed coding work per unit time while preserving correctness.

## Required target architecture

- NVIDIA `Qwen3.8-Flash-Next-FP4-Local` is the immutable reference checkpoint.
- Pennyroyal `jpezzulli/sglang-rtxpro6000` is the runtime base.
- Hyperburst performance work is imported selectively and measured one patch family at a time.
- Lesilva NVFP4-QSA-KV is an optional backend behind a KV abstraction.
- FP8 KV remains the reference path until NVFP4 passes all correctness gates.
- HiCache uses GPU hot KV, a bounded 4–8 GiB host transfer arena, and NVMe/NIXL storage.
- Oversized 350K/500K storage hits must be restored by aligned streaming chunks; a whole-hit rejection is not an acceptable implementation.
- ThinkingCap is an experimental dense overlay only. It may never silently replace NVIDIA tensors.
- OEA and the NVIDIA megakernel are optional experiments, never first-release dependencies.

## Non-negotiable correctness rules

1. Never modify or overwrite source model files.
2. Never commit model weights, private MTP heads, tokenmaps, credentials, or proprietary artifacts.
3. Every cache hit must include target KV, draft KV, all scales, QSA index state, Mamba/GDN state, MTP state, and compatible ABI metadata.
4. FP8-calibrated scales must never be interpreted as NVFP4 global scales.
5. NVFP4 packed row bytes, block scales, and scale sidecars must be accounted for separately.
6. A partial restore is a cache miss/recompute, never a successful hit.
7. No C8 tuning before C1/C4 correctness and repeatable measurements.
8. Every patch must have a source SHA, changed-file list, dependency list, test, measurement, and rollback commit.

## Required architecture artifacts

- source and version manifests with verified SHAs
- runtime component and data-flow diagrams
- tensor compatibility matrix
- KV backend contract
- HiCache streaming design and crash-recovery protocol
- patch conflict matrix
- memory budget for GPU, pinned host memory, SSD, CUDA graphs, MTP and recurrent state
- scheduler profiles for C1/C2/C4/C6/C8
- benchmark and coding-quality suite
- risk register with Go/No-Go rules
- deployment profiles and rollback procedure
- atomic OMO task cards and reviewer reports

## Required validation matrix

Run identical prompts and sampling settings for:

- baseline NVIDIA FP4-Local + FP8 KV
- Pennyroyal baseline + selected Hyperburst patch families
- FP8 KV versus NVFP4 KV
- cold prefill, GPU radix hit, SSD restore and restart restore
- ThinkingCap overlay disabled versus enabled
- OEA disabled versus enabled only after correctness
- C1, C2, C4, C6 and C8
- 8K, 32K, 128K, 200K, 350K and 500K context where supported
- tool calls, JSON schema, multi-turn coding, SQL migration and multi-file refactoring

Record TTFT p50/p95/p99, TPOT, prefill rate, aggregate and per-request decode rate, MTP acceptance, KV preemption, recompute tokens, restore bytes/time, GPU/host memory, and successfully completed coding tasks per hour.

## Phase gates

- Phase 00: source and revision inventory complete.
- Phase 01: reproducible FP8 baseline passes API, tool and JSON tests.
- Phase 02: selected Hyperburst patches pass isolated A/B tests.
- Phase 03: NVFP4 eager round-trip and scale validation pass.
- Phase 04: streaming restore passes 350K/500K, restart and corruption tests.
- Phase 05: overlay matrix and A/B quality tests pass, otherwise overlay is blocked.
- Phase 06: C1/C4 production profiles pass before C6/C8 admission work.
