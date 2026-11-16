# Quellen- und Evidenzbericht

Stand: 2026-11-16. Jede Zahl ist als Quelle oder als Ziel/Probe gekennzeichnet.

## Hyperburst

Quelle: https://github.com/aiueo52/hyperburst-rtxpro6000

- 121 Commits auf Pennyroyal-Basis, dokumentierte Basis `16e5682aad`.
- C1-Optimierungen: NEXTN/MTP, reduzierte Tokenmaps, CUDA Graphs, FlashInfer-Patches, OEA, PLE-/HiCache-Anpassungen.
- Öffentliche Ergebnisse sind workload- und artefaktabhängig; private MTP-Heads und Tokenmaps sind nicht verfügbar.
- Übernahmeentscheidung: nur einzelne Patches nach `git show`, Dateimatrix und Regressionstest; OEA zunächst aus.

## Pennyroyal

Quelle: https://github.com/jpezzulli/sglang-rtxpro6000

- dokumentiert 524K Kontext, HiCache/NIXL und SM120.
- gemeldete native NEXTN/FR-Spec-Profile und automatisch aufgelöster gemeinsamer KV-Pool.
- aktuelle Runtime ist der Correctness-Anker für Mamba/GDN, Cache und Recovery.

## Lesilva NVFP4-KV

Quelle: https://github.com/Lesilva/qwen38-flashnext-nvfp4-kv

Repository enthält getrennte `overlay`, `patches`, `tests` und Dokumentation. Das wird als Modulquelle behandelt, nicht als pauschaler Cherry-Pick.

## NVIDIA-Modell

Quelle: https://huggingface.co/nvidia/Qwen3.8-Flash-Next-FP4-Local

Die Modellkarte nennt ModelOpt/QAD, neu komprimierte NVFP4-Scales, ca. 96 GiB Diskverbrauch, PLE-Offload auf RTX PRO 6000 und ca. 850K FP8-KV-Tokens. Die Scale-Kompression und QAD-Abstimmung sind proprietäre Eigenschaften des Checkpoints; der Loader darf sie nicht stillschweigend in ein anderes Format umwandeln.

## ThinkingCap

Quelle: https://huggingface.co/bottlecapai/ThinkingCap-Qwen3.8-Flash-Next-NVFP4

Die Karte weist NVFP4/Compressed-Tensors aus und nennt Qwen Community License 1.0. Das beweist keine Kompatibilität mit NVIDIAs ModelOpt/QAD-Checkpoint. Kompatibilität muss per Tensor-Manifest nachgewiesen werden.

## Pflicht-Issues

| Issue | Befund | Konsequenz |
|---|---|---|
| #43239 | NVFP4-Row-Pitch im Kernel-HiCache falsch; K/V-Scales werden nicht transportiert; MTP-Draft wird korrupt | `kernel`-Backend sperren; direct/segmentierter Pfad und Regressionstest |
| #42369 | FP8-Checkpoint-Scales werden fälschlich als NVFP4-Global-Scales verwendet | Loader muss Scheme/Dtype abgleichen und bei mismatch abbrechen |
| #45030 | `buffer_only` verwirft Storage-Hits, die größer als Host-Arena sind | Chunk-Streaming implementieren oder bewusst Recompute melden |

## Evidenzregeln

Jeder Agentenbericht muss URL, Commit/Revision, Pfad/Zeile, Beobachtung, Interpretation und Konfidenz enthalten. Benchmarkzahlen ohne Prompt, Sampling, Kontext, Concurrency und Artefaktmanifest gelten nicht als vergleichbar.
