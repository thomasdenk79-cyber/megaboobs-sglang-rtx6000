# AGENTS.md

## Auftrag

Arbeite den Master-Auftrag unter `tasks/MASTER-AUFTRAG-VARIANTE-B.md` vollständig ab. Du bist für Recherche, Architektur, Implementierung, Tests und Dokumentation verantwortlich.

## Regeln

- Zuerst die echten Repositories, Branches, Tags, Commits, Issues und Quellcode-Schnittstellen prüfen.
- Keine unbestätigten Behauptungen als Fakten verwenden.
- Keine Gewichte in Git committen und keine Original-Checkpoints überschreiben.
- Keine Modell- oder KV-Hybridisierung ohne Tensor-, Quantisierungs- und Qualitätsvalidierung.
- NVIDIA FP4-Local bleibt die Referenz; bei einem Hybrid-Blocker bleibt die unveränderte NVIDIA-Basis lauffähig.
- Keine produktive Installation oder Hardwareänderung ohne explizite, reproduzierbare Abnahme.
- Jede Phase braucht Tests, Messprotokoll, Definition of Done, Rollback-Bedingung und Commit.
- Architekturänderungen werden in ADRs begründet; bei Unsicherheit STOP und eskalieren.

## Arbeitsweise

1. Quellen und aktuelle Baselines erfassen.
2. Architekturentscheidung mit Evidenz dokumentieren.
3. Kleine Phasen mit überprüfbaren Zwischenständen umsetzen.
4. Nach jedem Commit `git status`, relevante Tests und geänderte Dateien protokollieren.
5. Bei Fehlern zuerst reproduzieren, dann Ursache und minimale Korrektur dokumentieren.
