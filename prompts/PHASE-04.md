# Prompt Phase 04

Implementiere chunked HiCache/NIXL-Restore mit 4–8 GiB page-aligned Double Buffering. Ein Cache-Hit wird erst nach vollständigem Target-/Draft-KV-, Scale-, QSA-, Mamba/GDN- und Metadaten-Check sichtbar. Teste 350K/500K, Neustart, Crash und beschädigte Scales. Bei Fehler kontrollierter Recompute.
