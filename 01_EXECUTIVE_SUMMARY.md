# Executive Summary

## Ergebnis der Recherche

Die belastbare Produktbasis ist die Pennyroyal-Runtime von `jpezzulli/sglang-rtxpro6000`, nicht ein ungeprüfter Hyperburst-Snapshot. Die Runtime dokumentiert SM120, 524K-Kontext, HiCache/NIXL, native NEXTN/FR-Spec und einen gemeinsam genutzten KV-Pool. Hyperburst liefert wertvolle C1-Optimierungen, ist aber auf eine konkrete Basis (`16e5682aad`) und private MTP-/Tokenmap-Artefakte zugeschnitten.

NVIDIA `Qwen3.8-Flash-Next-FP4-Local` ist ein ModelOpt/QAD-Checkpoint mit neuem komprimiertem NVFP4-Scale-Format. Die Modellkarte nennt ca. 96 GiB auf Disk, RTX PRO 6000-Unterstützung mit PLE-Offload und ca. 850K FP8-KV-Tokens. Daraus folgt: das NVIDIA-Modell ist die Referenz, nicht ein frei austauschbarer Tensor-Pool.

## Hauptarchitektur

- Runtime: Pennyroyal-basierter SGLang-Fork.
- Performance: nur reproduzierbare Hyperburst-Patches übernehmen, zuerst ohne OEA.
- KV: FP8 als Referenz; NVFP4-QSA-KV als optionales, eigenes Backend.
- Cache: GPU-Hot-KV plus kleiner Host-Bounce-Puffer plus NVMe/NIXL. Oversized Restore muss chunkweise materialisiert werden.
- Modell: unverändertes NVIDIA-FP4-Local als Go/No-Go-Baseline; ThinkingCap-Dense-Overlay nur bei exakter Tensor-/Quantisierungs-Kompatibilität.
- NVIDIA-Megakernel: späterer optionaler Backend-Experimentpfad.

## Bekannte Blocker

1. SGLang Issue #43239 zeigt für NVFP4 mit `kernel`-HiCache falsche Host-/Device-Row-Geometrie und fehlenden Scale-Transport; der direkte Backend-Pfad ist der vorläufige Workaround.
2. Issue #42369 zeigt, dass FP8-kalibrierte KV-Scales nicht als NVFP4-Global-Scales wiederverwendet werden dürfen.
3. Issue #45030 beschreibt, dass `buffer_only` übergroße SSD-Hits derzeit komplett ablehnt; ohne Chunk-Streaming wird ein 4–8-GiB-Puffer das Ziel nicht erfüllen.
4. Hyperburst-Benchmarks enthalten private MTP-Head-/Tokenmap-Artefakte und sind daher nicht vollständig reproduzierbar.

## Entscheidung

Strategie 1: Pennyroyal als Basis, Hyperburst selektiv darüber. Diese Entscheidung minimiert Correctness- und Rebase-Risiken. Der lokale Agent darf die Strategie nur nach einem dokumentierten ADR ändern.
