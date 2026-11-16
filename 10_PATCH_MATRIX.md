# Patch- und Konfliktmatrix

| Bereich | Quelle | Basis | Konfliktrisiko | Entscheidung |
|---|---|---|---|---|
| Scheduler/HiCache | Pennyroyal | `jpezzulli/sglang-rtxpro6000` | niedrig innerhalb Basis | behalten |
| C1 Graph/NEXTN | Hyperburst | `16e5682aad` | mittel/hoch | einzeln portieren |
| QSA/NVFP4 KV | Lesilva | eigener Pennyroyal-Rebase | hoch mit Hyperburst-QSA | hinter Adapter kapseln |
| FlashInfer | Hyperburst | 0.6.17-Patches | hoch | exakt pinnen, ABI-Test |
| Mamba recovery | Pennyroyal/upstream | jeweilige SHA | hoch | Semantikvergleich, nicht blind cherry-pick |
| kernel HiCache | SGLang | Issue #43239 | sehr hoch | für NVFP4 sperren |
| Megakernel | sgl-nvidia-mega | PR #60 | unbekannt | optionales Experiment |

Für jede Übernahme: `git show`, geänderte Dateien, Abhängigkeiten, Test, Revert-Commit und Messvergleich dokumentieren.
