# AGENTS.md

Du bist der Implementer. Die Architekturentscheidungen in den Dokumenten sind verbindliche Vorgaben. Lies zuerst `00_START_HERE.md`, dann die aktuelle Phase und den zugehörigen Prompt.

## Regeln

- Pennyroyal ist die Basis; Hyperburst wird nur selektiv und mit Semantikvergleich übernommen.
- NVIDIA FP4-Local bleibt die unveränderte Referenz.
- ThinkingCap-Overlay nur bei exakter Tensor-/Shape-/Dtype-/Scale-Kompatibilität.
- FP8-KV ist der Referenzpfad; NVFP4-KV bleibt hinter einem Feature-Flag.
- NVFP4-HiCache über den Kernel-Pfad ist bis zur Behebung von Issue #43239 gesperrt.
- Kein OEA, kein Megakernel und keine privaten MTP-Artefakte in den ersten Correctness-Phasen.
- Keine Gewichte, Keys oder proprietären Dateien in Git.
- Jede Phase endet mit Tests, Messprotokoll, Definition of Done und Commit.
- Bei Mismatch, Korruption, OOM oder fehlender Evidenz: STOP, Baseline erhalten, Blocker dokumentieren.

## Commit-Regel

Ein Commit behandelt genau eine Phase oder einen atomaren Patch. Commit-Nachricht, geänderte Dateien, Testbefehle, Messwerte und Rückfallpfad in `docs/` dokumentieren.
