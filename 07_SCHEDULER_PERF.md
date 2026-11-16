# Scheduler und Performanceprofile

## Profile

| Profil | Zweck | NEXTN | KV |
|---|---|---:|---|
| C1-fast | einzelner Implementer | breiteste validierte Breite | maximal GPU-resident |
| C2-balanced | zwei Hauptagenten | mittlere Breite | keine unnötige Preemption |
| C4-production | vier Implementer | konservativ | Fairness + TTFT |
| C6/C8-elastic | Explorer/Reviewer | adaptive Breite | Admission nur bei Reserve |

MTP-/Draft-Breite wird anhand von Acceptance, VRAM und TPOT gewählt. Keine pauschale C8-Behauptung.

## Admission

1. Hauptimplementer pinned/resident halten.
2. Kurze Requests nur in freie Graph-/KV-Slots einlassen.
3. Geparkte Sessions zuerst auf SSD evicten, aber nur mit vollständigem Zustandsmanifest.
4. Bei NVFP4/HiCache-Unsicherheit auf FP8 oder Recompute zurückfallen.
5. C8 nur aktivieren, wenn C4-TTFT p95 und KV-Preemption-Grenzen eingehalten werden.

## Messung

Immer identische Promptsets, Sampling, Modellrevision, GPU-Power, Treiber und Clientpfad. Client-t/s und serverseitige Kernelzeit getrennt loggen. Private Hyperburst-Artefakte als nicht reproduzierbar markieren.
