# Test- und Abnahmestrategie

## Ebenen

1. **Format:** tensor names/shapes/dtypes/scales, safetensors headers.
2. **Kernel:** packed round-trip, scale transport, eager vs graph.
3. **State:** target/draft KV, QSA index, Mamba/GDN, MTP.
4. **Cache:** GPU hit, disk hit, streamed restore, restart, corruption.
5. **Serving:** API, tool calls, JSON schema, multi-turn.
6. **Performance:** C1/C2/C4/C6/C8, 8K–500K, cold/warm/SSD.
7. **Coding quality:** echte Oracle→PostgreSQL- und Multi-File-Aufgaben.

## Pflichtprotokoll

Jeder Lauf speichert config hash, model revision, runtime SHA, driver/CUDA, GPU power, prompt-set SHA, concurrency, TTFT p50/p95/p99, TPOT, aggregate/per-request tok/s, MTP acceptance, KV preemption, recompute tokens, restore bytes/time und RAM/VRAM.

## Abnahmeregel

Ein Feature ist nur bestanden, wenn Correctness-Tests, Regressionstests und der definierte Performancevergleich bestanden sind. Benchmarkwerte aus unterschiedlichen Promptsets dürfen nicht verglichen werden.
