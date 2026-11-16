# MASTER-AUFTRAG FÜR ASTRA: Architektur, Deep Research und Implementierungspaket für einen kombinierten Qwen3.8-Flash-Next-Inference-Stack

## DEINE ROLLE

Du bist der **Lead Software Architect, Principal GPU Inference Engineer und technische Reviewer** für ein anspruchsvolles Open-Source-Integrationsprojekt.

Du sollst nicht lediglich Empfehlungen formulieren. Du sollst die relevanten GitHub-Repositories, Implementierungen, Commits, Pull Requests, Checkpoints, Quantisierungsformate, Schnittstellen und bekannten Fehler **selbst gründlich untersuchen** und daraus einen realistischen, detaillierten, implementierbaren Architekturplan entwickeln.

Dein Ergebnis soll ein vollständiges **ZIP-Implementierungspaket** mit Architekturunterlagen, konkreten Arbeitspaketen, Abhängigkeiten, Teststrategien, Integrationsplänen und detaillierten Prompts für meinen lokalen **Qwen3.8-Flash-Next-Agenten mit Oh-My-OpenCode (OMO)** sein.

Du übernimmst die schwierigen Architekturentscheidungen und zerlegst die Implementierung so präzise, dass der lokale Agent nicht selbst die Gesamtarchitektur erfinden muss.

**WICHTIG: In diesem Auftrag sollst du zunächst recherchieren, entwerfen und das ZIP-Paket erstellen. Noch keinen produktiven Inference-Fork installieren, meine existierenden Repositories verändern oder Hardwarekonfigurationen umstellen.**

Arbeite selbstständig und liefere in dieser Sitzung das bestmögliche vollständige Ergebnis. Stelle keine Rückfragen, die durch Recherche oder begründete technische Annahmen vermieden werden können. Kennzeichne Unsicherheiten und offene Validierungen ausdrücklich.

---

# 1. PROJEKTZIEL: VARIANTE B

Entwickle eine Architektur für folgende Kombination:

**NVIDIA Qwen3.8-Flash-Next-FP4-Local als Modellbasis + selektives ThinkingCap-Overlay + optimierter Hyperburst-Inference-Pfad + aktuelle Pennyroyal-Korrektheit und Cache-Funktionalität + Lesilva-NVFP4-GPU-KV + kleiner RAM-Puffer und SSD-basiertes HiCache/NIXL.**

Ich möchte einen Produktionsserver für mehrere parallel arbeitende Coding-Agenten. Hauptanwendung ist Softwareentwicklung mit OMO, unter anderem umfangreiche Oracle→PostgreSQL-Migrationsprojekte.

Meine Priorität ist **maximale Zahl erfolgreich abgeschlossener Coding-Aufgaben pro Zeiteinheit**, nicht der höchste künstliche Kurzbenchmark.

Insbesondere möchte ich:

1. Extrem schnelle einzelne Agenten (C1).
2. Sehr hohe Leistung bei vier aktiven Implementern (C4).
3. Dynamisch auf C6–C8 skalieren, wenn zusätzliche Explorer, Reviewer und Subagenten sinnvolle Arbeit leisten.
4. Einen möglichst großen GPU-residenten KV-Cache.
5. Keine unnötige KV-Preemption oder erneute Prompt-Verarbeitung aktiver Agenten.
6. Nicht aktive Sessions effizient auf NVMe halten und bei Wiederaufnahme korrekt und schnell restaurieren.
7. Möglichst geringen dauerhaft reservierten Host-RAM.
8. Korrekte Mamba-/GDN-/QSA-/MTP-Zustände bei jedem Cache- und Decode-Vorgang.
9. Hohe Stabilität unter langen, verzweigten Coding-Agent-Workloads.
10. Einen langfristig wartbaren Fork, bei dem upstream Fixes übernommen werden können.

## Zielwerte, keine behaupteten Ergebnisse

| Eigenschaft | Angestrebter Wert |
|---|---|
| C1 Decode | Mindestens ca. 370–400 tok/s |
| C4 Decode gesamt | Mindestens ca. 700–710 tok/s |
| C8 Decode gesamt | Möglichst 900–1.000+ tok/s |
| GPU-KV-Pool | Ziel 1,2–1,5 Mio. Tokens |
| Kontext je Hauptagent | 200K–500K Tokens |
| Mixed C8 | Beispielsweise 4 × 230K + 4 × 24K |
| Gleichzeitige Requests | Dynamisch C1–C8, nicht statisch C8 |
| HiCache Host-Puffer | Ziel 4–8 GiB als anfängliches Experiment |
| Cache-Persistenz | NVMe über NIXL/geeigneten SSD-Backend |
| Wiederherstellung | Ohne unnötigen vollständigen Re-Prefill |
| Qualität | Keine relevante Regression bei Coding, Tools und Reasoning |

**Diese Zahlen sind Abnahmekriterien für Experimente, keine garantierten Leistungswerte. Prüfe zuerst, ob sie mit der Hardware und den jeweiligen Komponenten realistisch sind.**

---

# 2. MEINE HARDWARE UND BETRIEBSUMGEBUNG

Primäre GPU:

- NVIDIA RTX PRO 6000 Blackwell Workstation Edition
- 96 GB VRAM
- Single GPU, TP=1
- eGPU-Anbindung über Thunderbolt 4, damit eine erhebliche mögliche PCIe-Transferbeschränkung
- Nicht von einem direkten PCIe-5.0-x16-Steckplatz ausgehen

Hosts:

**Lenovo ThinkPad P16 Gen 2**
- Intel Core i9-13950HX
- 192 GB RAM
- NVMe-Laufwerke
- Linux/CachyOS oder Ubuntu 24.04
- Teilweise WSL-Experimente

**Zweiter Host**
- Nur 64 GB RAM
- Derselbe Inference-Stack soll möglichst auch dort funktionieren
- Besonders wichtig: Keine obligatorische 48–64-GiB-PLE-Pinned-RAM-Allokation plus große Host-KV-Pools

Betriebsanforderungen:

- Linux als bevorzugte Produktionsplattform
- Podman-Container bevorzugt
- Zentrale Daten- und Projektstruktur unter `/srv`
- Reproduzierbare Installationen und Builds
- Logs und Metriken für Prometheus/Grafana
- OpenAI-kompatible API
- OMO/OpenCode als hauptsächlicher Client
- Keine globale Veränderung bestehender Python-/CUDA-Umgebungen
- Rollback-fähige, versionsgebundene Deployments

---

# 3. VERBINDLICHE QUELLREPOSITORIES

Untersuche mindestens die folgenden Originalquellen und deren aktuellen Zustand. Benutze die tatsächlich verfügbaren Git-Branches, Tags und Commits. Prüfe Datum, Version, Wartungsstatus, Issues, Tests und die konkreten Implementierungen.

## A. Hyperburst: Performance und ThinkingCap-Hybrid

https://github.com/aiueo52/hyperburst-rtxpro6000

https://github.com/aiueo52/sglang-rtxpro6000

Besonders untersuchen:

- `normal` und `hyperpro`
- `serve-fast.sh` und die Startkonfigurationen
- Adaptive NEXTN/MTP
- Spekulative Draft-Breiten
- Reduced-Vocabulary-Token-Maps
- QSA-/GDN-/MoE-Optimierungen
- CUDA Graphs
- FlashInfer-Patches
- OEA
- PLE-Offload
- HiCache-Verhalten
- Mamba-State-Management
- C4-Benchmarks und bisherige C8-Limitierungen
- `tools/thinkingcap_overlay/overlay.py`
- Exakte Liste modifizierter Quelldateien
- Fork-Basis und Patch-Historie

**Wichtig:** OEA ist ein approximativer MoE-Pfad. Für initiale Korrektheits- und Qualitätsvergleiche muss OEA deaktiviert sein. OEA ist eine eigene, optionale Optimierung, die später unabhängig gemessen werden soll.

## B. Pennyroyal: aktuelle Runtime und Correctness

https://github.com/jpezzulli/sglang-rtxpro6000

Mindestens Version 3.1.6 und aktuellere relevante Entwicklungen untersuchen.

Besonders:

- Hybrid-Radix-Cache
- QSA-Pending-Index-Key-Ring
- NEXTN und MTP
- Mamba-/GDN-State-Recovery
- HiCache
- NIXL POSIX/FILE
- SSD-Persistenz
- Interleaved Prefill
- Scheduler
- Prefill-Fixes für lange gecachte Prefixe
- Cache-Hit-Admission
- GPU-/Host-/Disk-Transfer
- 3.1.5- und 3.1.6-Änderungen
- Aktuelle offene Korrektheitsfehler

## C. Lesilva: NVFP4-QSA-KV

https://github.com/Lesilva/qwen38-flashnext-nvfp4-kv

Besonders:

- NVFP4-GPU-KV-Speicherformat
- Packed E2M1-Daten
- Per-Block-Scales
- Globale KV-Scales
- QSA-Gather und Dequantisierung
- FP8- und BF16-Scratch-Buffer
- Prefix-Gather
- Decode-Gather
- CUDA-Graph-Kompatibilität
- MTP-Draft-KV-Datentyp
- Manuelle GPU-KV-Kapazitätsplanung
- Auswirkungen auf die Speicherreservierung
- Pennyroyal-3.1.5-Rebase
- Optionaler Mamba-Ping-Pong-Fix
- Split-K-Indexer-Patch
- Tests und ihre tatsächliche Reichweite
- Fehlende Kompatibilität mit neueren Pennyroyal-Versionen

Untersuche, ob sich der Patch als klar gekapseltes Modul übernehmen lässt und welche Hyperburst-Performanceoptimierungen denselben QSA-/KV-Pfad verändern.

## D. NVIDIA-Modell: Grundlage von Variante B

https://huggingface.co/nvidia/Qwen3.8-Flash-Next-FP4-Local

Untersuche:

- Exakte Checkpoint-Revision
- Architektur und vollständige Tensorliste
- Dense- und MoE-Gewichte
- QAD-Gewichte
- NVIDIA-PLE
- Tatsächliches PLE-Speicherformat
- Speicherverbrauch und Host-RAM-Bedarf
- CSF-/ModelOpt-Serialisierung
- NVFP4-Skalierung
- Layerweise Quantisierung
- MTP-Head
- NVFP4-MTP-Head-Format
- Benötigte Loader-Patches
- SGLang- und FlashInfer-Kompatibilität
- Qualitätsbenchmarks und Lizenzbedingungen

Prüfe explizit, welche Tensoren von der NVIDIA-QAD-Distillation verändert wurden.

Verwechsle nicht:
- NVFP4-Modellgewichte
- NVFP4-PLE
- NVFP4-GPU-KV
- NVFP4-MTP-Head

Diese Komponenten haben unterschiedliche Speicherformate und Runtime-Pfade.

## E. ThinkingCap: Dense-Overlay

https://huggingface.co/bottlecapai/ThinkingCap-Qwen3.8-Flash-Next-NVFP4

Untersuche:

- Exakte Revision
- Unterschiede gegenüber dem ursprünglichen Modell
- Tensor-Namen, Shapes und Dtypes
- Dense-Tensoren
- MoE-Experten
- PLE
- MTP
- Reasoning-Verhalten
- Qualitäts-/Thinking-Benchmarks
- Quantisierung
- Lizenzbedingungen

**Kritische Architekturfrage:** Lässt sich Hyperbursts bestehendes ThinkingCap-Dense-Overlay auf NVIDIAs FP4-Local-Checkpoint übertragen?

Prüfe das auf Tensor- und Ladeebene, nicht nur konzeptionell.

Wenn NVIDIA- und ThinkingCap-Tensoren unterschiedliche Quantisierungsschemata verwenden, entwickle einen präzisen Konvertierungs-/Requantisierungsplan oder erkläre, warum eine sichere Kombination nicht möglich ist.

Nicht stillschweigend inkompatible Gewichte austauschen.

## F. Weitere Referenzmodelle

https://huggingface.co/RadixArk/Qwen3.8-Flash-Next-NVFP4

https://huggingface.co/primitive-ai/Qwen3.8-Flash-Next-PLE-quant

Diese Modelle dienen nur der Analyse, technischen Referenz und gegebenenfalls kontrollierten Validierung.

**Keine eigene alternative Produktvariante A planen. Das Ziel bleibt ausschließlich Variante B auf Basis von NVIDIA FP4-Local.**

## G. Upstream SGLang und Megakernel

https://github.com/sgl-project/sglang

https://github.com/sgl-project/sglang/pull/50156

https://github.com/sgl-project/sglang/pull/50157

https://github.com/sgl-project/sgl-nvidia-mega/pull/60

https://github.com/sgl-project/sglang/pull/50194

Untersuche den aktuellen Merge- und Implementierungsstand, Abhängigkeiten, realen Hardware-Support und Performance-Nachweise.

Wichtige Fragestellungen:

- Lässt sich der neue NVIDIA-Megakernel überhaupt auf SM120/RTX PRO 6000 betreiben?
- Welche Optimierungen sind für Rubin statt Blackwell ausgelegt?
- Kann er NVIDIAs FP4-Local-Checkpoint und unseren ThinkingCap-Hybrid laden?
- Unterstützt er NVFP4-QSA-KV oder nur FP8?
- Welche Hyperburst-Kernel ersetzt er?
- Welche Hyperburst-Patches verlieren dadurch ihre Wirkung?
- Welche SGLang-Runtime-Optimierungen lassen sich unabhängig vom Megakernel übernehmen?
- Ist ein optionaler Execution-Backend-Schalter realistisch?

**Nicht voraussetzen, dass der Megakernel schneller ist. Kein obligatorischer Bestandteil der ersten funktionsfähigen Version.**

---

# 4. BEKANNTE FEHLER UND PFLICHTPRÜFUNGEN

Analysiere mindestens:

https://github.com/sgl-project/sglang/issues/43239

NVFP4-KV + HiCache: Fehler bei Packed-Row-Geometrie und Scale-Transport.

https://github.com/sgl-project/sglang/issues/42369

KV-Scale-Kompatibilität zwischen FP8-kalibrierten Checkpoints und NVFP4-KV.

https://github.com/jpezzulli/sglang-rtxpro6000/issues/115

DFlash2-/Mamba-Checkpoint-Restore-Korrektheit.

https://github.com/jpezzulli/sglang-rtxpro6000/issues/123

QSA-Pending-Key-Ring.

https://github.com/jpezzulli/sglang-rtxpro6000/issues/80

NVIDIA-QAD-MTP-Head und mögliche höhere MTP-Akzeptanz.

https://github.com/sgl-project/sglang/issues/45030

Probleme bei HiCache-Hits, die größer als der verfügbare Host-Transferpuffer sind.

Kläre für jedes Issue:

- Betroffene Versionen
- Root Cause
- Existierender Fix
- Upstream-Status
- Relevanz für Variante B
- Konkreter Patchbedarf
- Benötigter Regressionstest
- Ob es überhaupt auf die gewählte Ausführungskonfiguration zutrifft

Keine Vermutungen als bestätigte Fehler ausgeben.

---

# 5. ZENTRALE ARCHITEKTURFRAGE

Prüfe gründlich zwei mögliche Implementierungsstrategien:

**Strategie 1:** Pennyroyal 3.1.6 oder neuer als Basis, darauf selektiv Hyperburst-Performancepatches, Lesilva-NVFP4-KV und NVIDIA-Modell-Support.

**Strategie 2:** Hyperburst als Basis, darauf aktuelle Pennyroyal-Correctness-Patches, Lesilva-NVFP4-KV und NVIDIA-Modell-Support.

Bewerte objektiv:

- Codeänderungsumfang
- Patch-Konflikte
- Testbarkeit
- Performance-Risiko
- Wartbarkeit
- Upstream-Rebase-Kosten
- Cache-Architektur
- NVIDIA-Modell-Kompatibilität
- Unterstützung für C4–C8

Entscheide dich für **eine konkrete Hauptstrategie**. Dokumentiere die Begründung als Architecture Decision Record (ADR).

Die andere Strategie dient nur dem Vergleich, nicht als zweite auszuarbeitende Produktvariante.

---

# 6. CHECKPOINT-ARCHITEKTUR FÜR VARIANTE B

Dies ist ein zentrales Arbeitspaket.

Ziel:

**NVIDIA FP4-Local als unveränderte Modellbasis mit seiner QAD-PLE, seinen Experten und vorzugsweise seinem MTP-Head. Darauf ausschließlich die technisch kompatiblen ThinkingCap-Dense-Gewichte übertragen.**

Erstelle eine vollständige Tensor-Kompatibilitätsmatrix.

Untersuche für jede Tensorfamilie:

- Tensorname
- Ursprung NVIDIA oder ThinkingCap
- Shape
- dtype
- Quantisierung
- Scale-Layout
- Loader
- Erlaubte Konvertierung
- Möglicher numerischer Fehler
- Test
- Entscheidung: übernehmen, konvertieren oder unverändert lassen

Insbesondere:

1. NVIDIA-MTP-Head möglichst erhalten.
2. NVIDIA-QAD-PLE möglichst erhalten.
3. NVIDIA-MoE-Experten möglichst erhalten.
4. Selektiv ThinkingCap-Dense-Tensoren übernehmen.
5. Keine stillschweigenden Requantisierungen.
6. Kein komplettes Dequantisieren aller Gewichte in GPU-RAM.
7. Kein Überschreiben der Original-Checkpoints.
8. Reproduzierbare Generierung des Hybrid-Checkpoints.
9. SHA256-Manifest und Versionsmetadaten.
10. Automatische Tensor- und Quantisierungsvalidierung.

Beurteile ausdrücklich, ob diese Modellmischung fachlich sinnvoll ist oder ob die QAD-Abstimmung durch das Overlay erheblich gestört werden kann.

Definiere daher mindestens zwei kontrollierte Vergleichsläufe **innerhalb von Variante B**:

- NVIDIA FP4-Local unverändert als Referenz.
- NVIDIA FP4-Local + selektives ThinkingCap-Dense-Overlay.

Wenn die Hybridisierung die Qualitätsgrenzen nicht erfüllt, darf der Agent **nicht** automatisch auf eine andere, ungeprüfte Modellmischung wechseln. Er muss den Blocker dokumentieren und die unveränderte NVIDIA-Basis als Rückfallkonfiguration erhalten.

---

# 7. GPU-KV-ARCHITEKTUR

Entwerfe einen konkreten Integrationsplan für Lesilvas NVFP4-QSA-KV.

Pflichtbereiche:

- Packed E2M1-Layout
- FP8-Block-Scales
- Globale Scales
- Write-Pfad
- Decode-Gather
- Prefix-Gather
- Prefill
- Target-KV
- Draft-KV
- MTP-Verifikation
- CUDA-Graph-Capture
- Graph-Replay
- Cache-Accounting
- GPU-Pool-Kapazitätsplanung
- Memory-Fragmentation
- Interleaved Prefill
- Paging
- Radix-Cache
- Korrekte Eviction und Wiederherstellung

Berücksichtige, dass Hyperburst bereits eigene FP8-spezifische KV-Optimierungen besitzt.

Diese dürfen bei NVFP4 nicht unkontrolliert wiederverwendet werden.

Definiere einen verpflichtenden **KV-Backend-Abstraktionsvertrag** mit überprüfbaren Operationen für:

- Allocieren
- Schreiben
- Lesen
- Gather
- Dequantisieren
- Evicten
- Restaurieren
- Validieren

Lege fest, welche Komponente die alleinige Verantwortung für Quantisierungsformat und Scale-Metadaten besitzt.

Definiere FP8 als Referenzpfad und NVFP4 als zunächst optionalen neuen Pfad.

---

# 8. HICACHE: MINI-RAM-PUFFER + NVME

Dies ist neben C1-/C4-Performance die wichtigste Funktion.

Ich möchte keinen zwingenden 32-GB-Host-KV-Cache.

Wunscharchitektur:

```text
OMO / OpenCode
      |
OpenAI-Compatible API
      |
SGLang Scheduler + Radix Cache
      |
GPU HOT KV
NVFP4 QSA + vollständiger Hybrid-State
      |
Small Host Transfer Buffer
zunächst 4–8 GiB, dynamisch überprüfbar
      |
NIXL / SSD KV Backend
      |
NVMe /srv/...
Persistent Cold/Warm Sessions
```

Entwerfe ausdrücklich ein **Streaming-Restore-Verfahren**.

Ein 350K- oder 500K-Prefix muss aus SSD gelesen werden können, obwohl der verfügbare RAM-Transferpuffer kleiner als der gesamte Cache-Eintrag ist.

Prüfe:

- HiCache `cache` vs `buffer_only`
- Host-Pool-Accounting
- NIXL POSIX/FILE
- O_DIRECT
- Alignment
- Page-Größe 64
- Chunked I/O
- Double Buffering
- Async Prefetch
- DMA und Pinned Memory
- NVFP4-KV-Daten und Scale-Sidecars
- Mamba-/GDN-Checkpoints
- QSA-Indexdaten
- MTP-Draft-Zustand
- Metadaten und Versionskompatibilität
- Cache-Invalidation
- Atomic Write und Crash Recovery
- Wiederaufnahme nach Serverneustart
- Mehrere parallel laufende Sessions
- PCIe-/Thunderbolt-4-Transfergrenzen
- Linux Page Cache und optional ZFS ARC

**Keine Annahme, dass ein vollständiger SSD-Hit automatisch korrekt ist.**

Definiere explizite Validierungsregeln: Ein Cache-Hit darf nur bestätigt werden, wenn alle erforderlichen Komponenten und Skalen vollständig und korrekt restauriert wurden.

Bei unvollständiger oder inkompatibler Wiederherstellung muss ein kontrollierter Cache-Miss beziehungsweise Recompute erfolgen, niemals stille Datenkorruption.

Prüfe den Einfluss von NIXL-Backend und NVFP4-KV aufeinander.

Die PLE soll bei dieser Variante primär aus NVIDIAs kompaktem Checkpoint kommen. Ein eigenständiges PLE-SSD-Streaming-System ist **nicht Teil der ersten Zielarchitektur**, sofern die NVIDIA-PLE mit 64 GB Host-RAM zuverlässig betreibbar ist.

---

# 9. SCHEDULER UND PERFORMANCE

Entwickle einen Scheduler-/Admission-Plan für dynamische Concurrency:

- C1
- C2
- C4
- C6
- C8

Nicht alle Agenten sind gleich wichtig.

Mindestens unterscheiden:

- Lang laufende Implementer
- Explorer
- Reviewer
- Kurze Tool-/Analyse-Requests
- Inaktive, geparkte Sessions

Prioritäten:

1. Aktive Hauptagenten nicht unnötig preempten.
2. GPU-KV-Residency beachten.
3. Große gecachte Prefixe möglichst wiederverwenden.
4. Kurze Requests zulassen, ohne große Implementer zu verdrängen.
5. C8 nur nutzen, wenn genügend unabhängige Arbeit und Ressourcen existieren.
6. TTFT und Gesamtdurchsatz gemeinsam optimieren.
7. Keine hohen Mamba-Slot-Reservierungen ohne realen Nutzen.

Analysiere Hyperbursts adaptive MTP-Breiten, FR-Spec und CUDA-Graph-Konfigurationen.

Berücksichtige, dass eine extrem große Draft-Breite zusätzliche VRAM-Belegung verursachen und bei C4/C8 langsamer sein kann.

Definiere getrennte Performanceprofile und eine Methode, diese dynamisch oder anhand von Request-Klassen zu wählen.

Keine pauschale Aussage, dass C8 besser als C4 oder C4 besser als C1 ist.

---

# 10. BENCHMARK- UND QUALITÄTSSTRATEGIE

Entwerfe eine reproduzierbare Test-Suite.

Nicht nur synthetische Prompts messen.

Pflichtmessungen:

- C1/C2/C4/C6/C8
- 8K, 32K, 128K, 200K, 350K und 500K Kontext, soweit unterstützt
- Kleine und große Outputs
- Thinking an/aus
- ThinkingCap-Overlay an/aus
- OEA an/aus
- FP8-KV gegen NVFP4-KV
- Kalter Prefill
- Warmer GPU-Radix-Hit
- SSD-Cache-Restore
- Parallel laufende Prefills und Decodes
- MTP-Akzeptanzrate und Akzeptanzlänge
- GPU-/Host-RAM
- Energie und GPU-Auslastung
- Tool-Calls und Structured Output
- JSON-Schema
- Agentischer Multi-Turn-Betrieb
- Wiederaufnahme nach Cache-Eviction
- Wiederaufnahme nach Serverneustart

Entscheidende Kennzahlen:

- Aggregate Decode Throughput
- Per-Request Decode Throughput
- TTFT p50/p95/p99
- TPOT p50/p95/p99
- Prefill-Tokens/s
- Cache-Hit-Rate
- KV-Preemption
- Recompute-Tokens
- Restore-Bytes und Restore-Zeit
- MTP Acceptance
- Host-RAM-Anforderung
- Tatsächlich erfolgreich gelöste Coding-Aufgaben pro Stunde
- Gesamttokens pro erfolgreich gelöster Aufgabe

Vergleiche nur Testläufe mit identischen Eingaben, Sampling-Einstellungen, Hardwarebedingungen und möglichst gleicher Modellkonfiguration.

Ein C8-Durchsatz darf nicht direkt mit einem anders definierten C4-Benchmark verglichen werden.

Für Coding-Qualität entwickle Tests mit echten Repositories:

- Änderungen an mehreren zusammenhängenden Dateien
- Refactoring
- Fehlerbehebung
- Schreiben und Ausführen von Tests
- SQL-Migration
- API-Anpassungen
- Längere Aufgaben mit mehreren Agententurns

Bei den abschließenden Bewertungskriterien muss ein implementiertes Feature erst nach bestandenen Tests als erfolgreich gelten.

---

# 11. ARCHITEKTURARTEFAKTE

Erstelle ein professionelles, vollständiges Architekturpaket.

Verpflichtende Inhalte:

1. Executive Summary.
2. Quellen- und Evidenzübersicht.
3. Vollständiger Architekturentwurf.
4. Komponentenübersicht.
5. Systemkontextdiagramm.
6. Komponenten- und Datenflussdiagramme.
7. Sequenzdiagramme für Prefill, Decode, MTP und SSD-Restore.
8. Modell-/Tensor-Overlay-Architektur.
9. Quantisierungs- und Speicherformat-Spezifikation.
10. Patch- und Fork-Abhängigkeitsgraph.
11. Konfliktmatrix für veränderte Quelldateien.
12. Klare Entscheidung über den Basis-Fork.
13. ADRs für sämtliche wesentlichen Architekturentscheidungen.
14. GPU-/RAM-/SSD-Memory-Budget.
15. Risiken und Gegenmaßnahmen.
16. Sicherheits-, Lizenz- und Wartungsanalyse.
17. Implementierungsphasen.
18. Konkrete Aufgaben für den OMO-Agenten.
19. Test- und Abnahmeplan.
20. Rollback- und Deployment-Strategie.

Die Dokumentation soll **auf Deutsch** verständlich sein. Codebezeichner, Schnittstellen, CLI-Flags, Dateinamen und Quellcodekommentare dürfen beziehungsweise sollen auf Englisch bleiben.

Nutze Mermaid-Diagramme, wo sinnvoll. Lege außerdem eine gut lesbare HTML-Übersicht an, mit Navigation zwischen Architektur, Tasks, Risiken und Quellen.

---

# 12. IMPLEMENTIERUNGSPAKET FÜR QWEN + OMO

Mein lokaler Agent soll damit möglichst wenig selbst interpretieren müssen.

Jede Implementierungsphase braucht:

- Ziel
- Ausgangszustand
- Exakte Repositories und Commits
- Betroffene Dateien
- Abhängigkeiten
- Konkrete Änderung
- Erwartete Schnittstellen
- Technische Constraints
- Testbefehle
- Definition of Done
- Häufige Fehler
- Stop-/Rollback-Bedingungen
- Handover an den nächsten Agenten

Die Arbeitspakete sollen klein genug sein, dass ein Qwen-Agent sie zuverlässig in überschaubaren Kontextfenstern bearbeiten kann.

**Wichtig: Keine riesigen, unpräzisen „implementiere das gesamte System“-Prompts.**

Erstelle für jede Phase einen direkt kopierbaren Agenten-Prompt und ein eigenes `AGENTS.md` oder eine entsprechend klar referenzierte Agentenanweisung.

Definiere:

- Wer die Architektur ändern darf.
- Was der Implementer strikt nach Vorgabe umsetzt.
- Wann ein Architekturproblem eskaliert werden muss.
- Was nach jedem Commit dokumentiert wird.
- Wie der Agent Tests protokolliert.
- Wie Änderungen mit Git committed werden.
- Wie ein abgebrochener Agentenlauf fortgesetzt wird.

Jede Phase soll auf einer überprüfbaren Basis aufbauen.

Keine Änderung darf als abgeschlossen gelten, solange die vorher festgelegten Abnahmetests fehlschlagen.

---

# 13. GIT-/REPOSITORY-STRATEGIE

Entwickle eine konkrete Repository-Struktur für den späteren kombinierten Fork.

Berücksichtige:

- Klare Single Source of Truth.
- Baseline-Commit.
- Upstream-Remotes.
- Hyperburst-Patch-Import.
- Pennyroyal-Upstream-Fixes.
- Lesilva-NVFP4-Patch.
- NVIDIA-Modell-Loader.
- ThinkingCap-Overlay-Generator.
- Cache-Backend.
- Benchmark-Harness.
- Testdaten.
- Dokumentation.
- Build-System.
- CI.
- Container.
- Versioniertes Deployment.

Beispielhafte Struktur, die du prüfen und verbessern sollst:

```text
project/
├── AGENTS.md
├── README.md
├── ARCHITECTURE.md
├── docs/
│   ├── research/
│   ├── architecture/
│   ├── adr/
│   ├── compatibility/
│   ├── performance/
│   ├── security/
│   └── operations/
├── patches/
│   ├── hyperburst/
│   ├── pennyroyal/
│   ├── lesilva-nvfp4-kv/
│   ├── nvidia-loader/
│   └── upstream/
├── models/
│   └── thinkingcap-overlay/
├── runtime/
│   ├── cache/
│   ├── scheduler/
│   └── quantization/
├── configs/
│   ├── c1/
│   ├── c4/
│   └── c8/
├── benchmarks/
├── tests/
├── deploy/
├── scripts/
├── tasks/
│   ├── phase-00/
│   ├── phase-01/
│   └── ...
└── manifests/
```

**Das ist ein Architekturvorschlag, keine Anweisung, alle SGLang-Dateien in diese Struktur umzubauen.** Der tatsächliche Inference-Fork muss seine Python-Importpfade und Buildstruktur beibehalten, sofern eine Änderung nicht ausdrücklich begründet ist.

Unterscheide zwischen Quellcode-Fork, Integrationsrepository und Modellartefakten.

Modellgewichte gehören nicht ins Git-Repository.

---

# 14. WARTBARKEIT UND UPSTREAM-KONFLIKTE

Erstelle eine präzise Patch-Matrix:

| Patch | Ursprung | Basis-Commit | Geänderte Dateien | Abhängigkeiten | Konflikte | Tests | Übernahmeentscheidung |
|---|---|---|---|---|---|---|---|

Besonders wichtig:

- Hyperburst QSA-Patches vs Lesilva QSA-Patches
- Hyperburst FP8-KV-Stores vs NVFP4-KV
- Hyperburst Scheduler vs Pennyroyal-3.1.6-Scheduler
- Mamba-State-Checkpoint-Fixes
- HiCache-Memory-Pool-Layouts
- NVIDIA-Megakernel-Bindings
- CUDA-Graph- und Draft-Kernel-Änderungen
- FlashInfer-Versionen
- NVIDIA CSF/ModelOpt Loader

Verwende keine pauschalen Empfehlungen wie „einfach cherry-picken“.

Prüfe für jeden wesentlichen Patch, ob seine Funktion bereits upstream oder in der gewählten Runtime vorhanden ist.

Bereits integrierte Fixes nicht erneut portieren.

Bei Konflikten die Semantik vergleichen, nicht nur Text-Hunks.

---

# 15. RISIKOANALYSE

Erstelle eine Risikomatrix mit mindestens:

- Inkompatible NVIDIA-/ThinkingCap-Gewichte
- Qualitätsverlust durch QAD-/ThinkingCap-Mischung
- Unterschiedliche NVFP4-Scale-Formate
- Falsche KV-Global-Scales
- Defekte MTP-Draft-KV-Wiederherstellung
- Unvollständige Mamba-/GDN-Zustände
- Fehlerhafte QSA-Indexer-Restores
- GPU-OOM bei langen Prefixen
- Zu große CUDA-Graph-Reservierungen
- C8-Scheduler-Slot-Erschöpfung
- SSD-Prefetch ohne ausreichenden RAM-Puffer
- O_DIRECT-Alignment
- Fehlende Tensor- oder Sidecar-Daten
- NVIDIA-Megakernel ohne SM120-Support
- Performance-Regressions durch Patch-Kombinationen
- Lizenz-/Nutzungsbedingungen
- Host-RAM-Probleme auf dem 64-GB-System

Zu jedem Risiko gehören konkrete Verifikation, Wahrscheinlichkeit, Auswirkung, Gegenmaßnahme und Go-/No-Go-Kriterium.

---

# 16. ARCHITEKTURREGELN

Diese Regeln gelten für dich und anschließend für den lokalen Implementer:

**Keine Architektur auf unüberprüften Behauptungen aufbauen.**

**Keine Optimierung ohne Vergleich gegen eine definierte Baseline.**

**Keine stillschweigende Modell- oder KV-Korruption akzeptieren.**

**Keine komplette Umstellung der produktiven Runtime in einem Schritt.**

**Keine Patches übernehmen, deren Wirkung bereits vorhanden oder durch neuere Implementierungen überholt ist.**

**Keine Performancewerte verschiedener Benchmarks ohne Methodikprüfung vermischen.**

**Keine GPU-/RAM-Kapazität annehmen, ohne Gewichte, CUDA Graphs, MTP, Scratch und State-Buffers einzurechnen.**

**Keine Behauptung, dass der NVIDIA-Megakernel bereits auf der RTX PRO 6000 schneller ist, solange entsprechende Messungen fehlen.**

**Keine Behauptung, dass NVFP4-PLE automatisch qualitativ kompatibel mit ThinkingCap ist.**

**Kein Schreiben in Originalmodell-Dateien oder unkontrolliertes Überschreiben von Quellrepositories.**

**Keine nicht lizenzkonforme Weiterverteilung von Modellgewichten, proprietären Dateien oder veränderten Drittanbieterartefakten.**

---

# 17. KONKRETE ZIP-AUSGABE

Erzeuge ein tatsächliches ZIP-Archiv, beispielsweise:

`qwen38-nvidia-thinkingcap-hyperburst-architecture.zip`

Das Archiv muss mindestens enthalten:

```text
00_START_HERE.md
01_EXECUTIVE_SUMMARY.md
02_SOURCE_RESEARCH.md
03_ARCHITECTURE.md
04_MODEL_OVERLAY_DESIGN.md
05_NVIDIA_LOADER_COMPATIBILITY.md
06_NVFP4_KV_DESIGN.md
07_HICACHE_NIXL_SSD_DESIGN.md
08_SCHEDULER_C1_C8_DESIGN.md
09_MEMORY_BUDGET.md
10_PATCH_CONFLICT_MATRIX.md
11_IMPLEMENTATION_ROADMAP.md
12_TEST_STRATEGY.md
13_BENCHMARK_SPEC.md
14_RISK_REGISTER.md
15_DEPLOYMENT_ROLLBACK.md
16_LICENSE_SECURITY.md
17_ARCHITECTURE_DECISIONS.md
18_OPEN_QUESTIONS.md
19_REFERENCES.md

AGENTS.md
ARCHITECTURE.md
README.md

tasks/
  phase-00/
  phase-01/
  phase-02/
  ...

prompts/
  OMO_MASTER_PROMPT.md
  IMPLEMENTER_PHASE_PROMPTS/
  REVIEWER_PROMPTS/
  HANDOVER_TEMPLATE.md

manifests/
  sources.json
  versions.json
  patch-matrix.csv
  model-tensors.csv
  acceptance-criteria.json

diagrams/
  system-context.mmd
  components.mmd
  checkpoint-overlay.mmd
  kv-dataflow.mmd
  cache-restore-sequence.mmd

report/
  index.html
```

Ergänze erforderliche Dateien selbstständig.

`00_START_HERE.md` muss so klar sein, dass ich das ZIP entpacke, meinem OMO-Implementer `AGENTS.md` und den ersten Task gebe und er weiß, was er zu tun hat.

Keine bloßen TODO-Platzhalter für wesentliche Architekturentscheidungen.

Wenn ein Problem nur durch Tests auf meiner GPU endgültig entschieden werden kann, definiere einen exakten reproduzierbaren Probe und eine bedingte Entscheidungsregel.

---

# 18. ARBEITSWEISE FÜR DEINE DEEP RESEARCH

Arbeite in dieser Reihenfolge:

**Schritt 1: Quellen prüfen**

GitHub-Repositories, Commits, Modellkarten, Dateistrukturen, relevanten Quellcode und aktuelle Issues untersuchen.

**Schritt 2: Machbarkeit von NVIDIA + ThinkingCap entscheiden**

Die Checkpoint- und Tensor-Kompatibilität ist ein möglicher harter Blocker. Kläre sie vor der detaillierten Implementierungsplanung so weit wie möglich.

**Schritt 3: Basis-Fork auswählen**

Entscheide Pennyroyal-basiert oder Hyperburst-basiert nach konkreten Patch-/Dateikonflikten.

**Schritt 4: Speicher- und Cache-Architektur entwerfen**

NVIDIA-PLE, GPU-NVFP4-KV, Mamba-State, HiCache, NIXL und Mini-Transferpuffer gemeinsam budgetieren.

**Schritt 5: Performancearchitektur entwerfen**

C1, C4 und C8 samt MTP, Graphs, Scheduling und realistischen Ressourcenlimits.

**Schritt 6: Implementierungsaufgaben formulieren**

Atomare Phasen mit konkreten Pfaden, Code-Schnittstellen, Tests und Rollback.

**Schritt 7: Qualitätssicherung des Architekturpakets**

Prüfe die eigene Dokumentation kritisch auf Widersprüche, fehlende Abhängigkeiten, unrealistische Kapazitäten, unklare Patches und nicht reproduzierbare Arbeitsanweisungen.

**Schritt 8: ZIP erstellen und prüfen**

Verifiziere Dateistruktur, alle internen Links, referenzierte Pfade, JSON-/CSV-Validität und die Vollständigkeit der Agentenprompts.

---

# 19. ABSCHLIESSENDE ERWARTUNG

Ich möchte am Ende keinen allgemeinen technischen Bericht, sondern eine **belastbare Blaupause für ein reales Entwicklungsprojekt**.

Deine wichtigste Aufgabe ist, dem lokalen Qwen-OMO-Implementer die Architekturarbeit abzunehmen.

Bitte liefere:

1. Deine eindeutige Architekturentscheidung für Variante B.
2. Eine konkrete Aussage zur Machbarkeit des NVIDIA-/ThinkingCap-Hybrids, mit Belegen und offenen Prüfungen.
3. Die wichtigsten technischen Risiken und Blocker.
4. Einen nachvollziehbaren Weg zu C1-/C4-Hyperburst-Geschwindigkeit, NVFP4-KV und SSD-HiCache.
5. Das vollständige ZIP mit allen Unterlagen und direkt nutzbaren Implementer-Prompts.
6. Einen Download-Link zum erzeugten ZIP.

**Führe die Recherche gründlich durch, prüfe echte Quellcode-Schnittstellen und entscheide wie ein verantwortlicher Principal Architect. Dein Ergebnis muss für die spätere Implementierung durch einen weniger starken lokalen Coding-Agenten geeignet sein.**

Beginne jetzt mit der technischen Untersuchung und liefere das Architektur- und Handover-Paket in dieser Sitzung.