# Implementierungs-Roadmap

## Phase 00 – Quellen und Baseline

**Ziel:** Commits, Versionen, Lizenz und reale Pfade festnageln.

**Dateien:** `manifests/sources.json`, `manifests/versions.json`, `docs/research/phase-00.md`, `scripts/inspect_checkpoint.py`.

**Tests:** Git SHA prüfen; Modell-Header lesen; keine Gewichte committen.

**Gate:** Baseline-Report vollständig; sonst STOP.

## Phase 01 – Pennyroyal reproduzierbar starten

**Ziel:** NVIDIA FP4-Local auf Pennyroyal mit FP8-KV, ohne Overlay/OEA/NVFP4.

**Änderungen:** Containerfile, pinned requirements, `deploy/profile-baseline.env`, smoke API, 8K/32K/128K prompt tests.

**Gate:** deterministischer API- und Tool-Call-Test, GPU-Pool report, kein silent fallback.

## Phase 02 – Hyperburst selektiv importieren

**Ziel:** C1-Optimierungen einzeln übernehmen.

**Reihenfolge:** FlashInfer patch → CUDA graph patch → NEXTN profile → tokenmap optional → OEA zuletzt.

**Gate:** jede Änderung eigener Commit, A/B gegen Phase 01, kein Patch mit unbekannter Dateisemantik.

## Phase 03 – NVFP4-KV Adapter

**Ziel:** Lesilva-Backend hinter `KVBackend`.

**Änderungen:** `runtime/kv/nvfp4.py`, scale validation, packed stride helpers, tests.

**Gate:** Eager round-trip und MTP acceptance; bei Fehler FP8 default bleibt aktiv.

## Phase 04 – HiCache Streaming

**Ziel:** 4–8 GiB Bounce-Puffer ohne oversized-hit rejection.

**Änderungen:** `runtime/cache/stream_restore.py`, manifest ABI, chunk scheduler, direct NIXL path.

**Gate:** 350K- und 500K-Restore mit kleinerem Puffer, Neustart, Crash-injection, checksum mismatch.

## Phase 05 – Overlay-Probe

**Ziel:** NVIDIA vs selektives ThinkingCap-Overlay.

**Änderungen:** `tools/overlay/`, tensor matrix, generated manifest.

**Gate:** nur bei exakter Kompatibilität; ansonsten BLOCK mit NVIDIA-Baseline.

## Phase 06 – C4/C8 Produktion

**Ziel:** Admission und Profile.

**Gate:** C1/C4 correctness first, dann C6/C8; Erfolgsmetrik ist gelöste Coding-Aufgabe pro Stunde.
