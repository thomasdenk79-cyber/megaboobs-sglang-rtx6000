# Start Here – Variante B

Dieses Repository ist ein **Implementierungs-Handover**, kein allgemeiner Rechercheauftrag. Die Architektur ist vorgegeben; der lokale Agent soll sie verifizieren und in kleinen Phasen umsetzen.

## Verbindliche Reihenfolge

1. `AGENTS.md` lesen.
2. `01_EXECUTIVE_SUMMARY.md`, `02_SOURCE_RESEARCH.md` und `03_ARCHITECTURE.md` lesen.
3. Phase 00 aus `08_IMPLEMENTATION_ROADMAP.md` ausführen.
4. Erst nach bestandenem Phase-00-Gate mit Phase 01 fortfahren.
5. Für jede Phase den zugehörigen Prompt unter `prompts/` verwenden.

## Hauptentscheidung

Pennyroyal/SGLang `jpezzulli/sglang-rtxpro6000` ist die Runtime-Basis. Hyperburst wird als selektive Patchquelle verwendet. Lesilvas NVFP4-KV wird als gekapseltes KV-Backend importiert. Der NVIDIA-FP4-Local-Checkpoint bleibt unverändert; ein ThinkingCap-Overlay ist zunächst nur ein kontrolliertes Experiment.

## Harte Stop-Regeln

- Keine Gewichte in Git.
- Kein Overlay bei abweichenden Tensor-Shapes, Quantisierung oder Scale-Layouts.
- Kein `kernel`-HiCache-Backend für NVFP4, solange Scale-Transport und Packed-Row-Geometrie nicht durch Tests bewiesen sind.
- Kein C8-Tuning vor C1/C4-Correctness.
- Kein Megakernel als Pflichtabhängigkeit.

## Erwartete Endprodukte

Der Agent erzeugt Architektur-ADRs, Patch-Matrix, reproduzierbare Probe-Skripte, getrennte Baseline-/Overlay-Model-Manifeste, NVFP4-KV-Tests, Streaming-Restore-Tests und versionierte Container-Deployments.
