# NVFP4-QSA-KV-Design

## Format

Device rows sind packed: zwei FP4-Werte pro Byte. Block-Scales und globale Scales liegen separat. Deshalb ist der Device-Stride nicht automatisch `head_dim * itemsize`. Host-Pools müssen packed row bytes, scale bytes und MTP-Draft-Daten getrennt bilanzieren.

## Sicherheitsregeln

- `kv_cache_dtype=nvfp4` verlangt ein passendes `kv_cache_scheme`.
- FP8-kalibrierte `k_scale/v_scale` niemals als NVFP4 global scale verwenden.
- Für jede Layer-/Page-Gruppe werden Daten- und Scale-Sidecar gemeinsam geschrieben.
- Draft-KV und Target-KV werden getrennt validiert.
- CUDA Graph Capture erst nach erfolgreichem eager round-trip.

## Implementierung

1. Lesilva-Patch in isoliertem Branch übernehmen.
2. `KVBackend`-Adapter schreiben; keine Scheduleränderung in diesem Commit.
3. Eager write/gather/dequant test für 1, 64, 1024 und lange Prefixe.
4. Host-/Disk-Roundtrip mit byte-/checksum-validierten Scales.
5. MTP acceptance test: FP8 und NVFP4 identischer Prompt, acceptance darf nicht auf 1.0 festgenagelt sein.
6. Graph capture/replay erst nach allen Eager-Tests.

## Go/No-Go

Go nur wenn packed row pitch, block scale transport, global scale scheme und MTP draft KV in einem End-to-End-Test korrekt sind. Sonst FP8 als Produktionspfad behalten.
