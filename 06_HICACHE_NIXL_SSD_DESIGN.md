# HiCache/NIXL/SSD mit Mini-Puffer

## Zustandsgruppe

Ein Storage-Eintrag ist nur gültig, wenn folgende Dateien/Segmente gemeinsam vorhanden sind:

- target KV packed data
- target block/global scales
- draft KV und Draft-Scales
- QSA index/pending ring
- Mamba/GDN recurrent state
- token range, model/runtime/kv ABI, page size, dtype, checksum

## Streaming-Restore

`buffer_only` darf einen 350K-/500K-Hit nicht als einen monolithischen Transfer behandeln. Implementiere page-aligned chunks von 256–1024 MiB mit zwei Bounce-Buffern:

1. Manifest atomar prüfen.
2. Chunk N asynchron von NIXL/FILE lesen.
3. Daten und Scales gemeinsam in freie GPU-Seiten materialisieren.
4. Checksumme und QSA-Pointer prüfen.
5. Chunk N freigeben, Chunk N+1 laden.
6. Erst am Endpoint Mamba/GDN/MTP state anhängen und den Hit als vollständig markieren.
7. Bei Fehler: alle Teilseiten verwerfen und kontrollierten Recompute auslösen.

Zwischensegmente dürfen nicht als vollständiger Cache-Hit sichtbar werden.

## Vorläufiger Backend-Entscheid

Für NVFP4 `direct`/segmentierten Pfad verwenden. `kernel` bleibt gesperrt, bis #43239 reproduzierbar behoben und getestet ist. O_DIRECT nur mit geprüfter Ausrichtung und Fallback verwenden. Host-RAM-Ziel 4–8 GiB ist ein Durchsatzparameter; wenn Streaming nicht implementiert ist, muss die Konfiguration den langen Hit ablehnen und re-prefillen.

## Crash Recovery

Schreibe in temporäre Datei, fsync Daten und Manifest, rename atomar. Lade nur ABI-kompatible Manifeste. Unvollständige Dateien werden gelöscht oder quarantänisiert.
