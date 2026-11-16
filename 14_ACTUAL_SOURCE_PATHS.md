# Tatsächliche Quellpfade und Patch-Reihenfolge

Diese Liste verhindert, dass der Implementer selbst raten muss. Die Pfade stammen aus den öffentlichen Repository-Bäumen und den veröffentlichten Patchserien.

## Lesilva NVFP4-KV (Pennyroyal 3.1.5 overlay)

Quelle: `Lesilva/qwen38-flashnext-nvfp4-kv`, Produktions-Overlay-Commit `215fe89e0feee9a5b3a988df8f97affb0ee87168`.

```text
overlay/pennyroyal-3.1.5/nvfp4-kv/python/sglang/srt/layers/attention/flashinfer_backend.py
overlay/pennyroyal-3.1.5/nvfp4-kv/python/sglang/srt/layers/attention/qsa/nvfp4_kv.py
overlay/pennyroyal-3.1.5/nvfp4-kv/python/sglang/srt/layers/attention/qwen_sparse_attn_backend.py
overlay/pennyroyal-3.1.5/nvfp4-kv/python/sglang/srt/layers/quantization/fp4_kv_cache_quant_method.py
overlay/pennyroyal-3.1.5/nvfp4-kv/python/sglang/srt/mem_cache/kv_cache_configurator.py
overlay/pennyroyal-3.1.5/nvfp4-kv/python/sglang/srt/server_args.py
```

Optionale Änderungen sind getrennt und dürfen nicht automatisch übernommen werden:

```text
overlay/pennyroyal-3.1.5/optional-mamba-pingpong-fix/python/sglang/srt/managers/scheduler.py
overlay/pennyroyal-3.1.5/optional-mamba-pingpong-fix/python/sglang/srt/mem_cache/unified_radix_cache.py
overlay/pennyroyal-3.1.5/optional-qsa-indexer-splitk/python/sglang/srt/layers/attention/qsa/mqa.py
```

Die Patchserie `patches/pennyroyal-3.1.5/0001-qsa-nvfp4-kv.patch` bis `0003-optional-qsa-indexer-splitk.patch` dokumentiert die beabsichtigte Reihenfolge. Die 3.1.5-Patches dürfen nicht blind auf 3.1.6 angewendet werden; erst `git apply --check`, dann Drei-Wege-Semantikvergleich und eigener 3.1.6-Rebase.

## Hyperburst

Hyperburst ist kein vollständiger Basis-Fork für dieses Projekt. Der aktuelle Stand `8ff73f41696828b184ff21abbce02e03809947d3` dokumentiert nachträgliche Änderungen gegenüber der Basis `16e5682aad`. Der Agent muss die Patchquellen unter `patches/sglang/`, `patches/flashinfer/`, `launch/`, `bench/` und `docs/` einzeln inventarisieren und nur den Runtime-Code portieren. Benchmarks, private MTP-Heads und Tokenmaps sind keine Runtime-Patches.

## Entscheidungsreihenfolge

1. Pennyroyal `202f266be2d46453e098792fa39057e859fb8095` als Lesebaseline.
2. NVIDIA-FP4-Local mit FP8-KV starten.
3. Lesilva `nvfp4-kv` nur in den sechs genannten Runtime-Dateien isoliert portieren.
4. Optionale Scheduler-/Radix-/Split-K-Patches einzeln prüfen.
5. Hyperburst-FlashInfer und Graph-Patches einzeln portieren.
6. Erst danach Overlay und Streaming-Restore.
