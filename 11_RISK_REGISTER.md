# Risiko-Register

| Risiko | Wkeit | Auswirkung | Verifikation | Gegenmaßnahme | Go/No-Go |
|---|---|---|---|---|---|
| NVIDIA/ThinkingCap inkompatibel | hoch | Modell unbrauchbar | Tensor-Matrix + loader dry-run | NVIDIA-Baseline behalten | No-Go Overlay |
| falsche NVFP4 global scales | hoch | stille Langkontext-Korruption | needle/long-prefix tests | scheme validation | No-Go |
| kernel HiCache pitch/scale bug | hoch | korrupter MTP/KV | #43239 probe | direct backend | No-Go kernel |
| oversized buffer_only hit | hoch | vollständiger Re-prefill | 350K/500K restore | streaming chunks | No-Go mini-buffer |
| private MTP artefacts | mittel | nicht reproduzierbare t/s | manifest audit | public baseline | nur Vergleich |
| CUDA graph OOM | mittel | Startup/Runtime fail | memory budget | profile-specific graphs | No-Go bei OOM |
| SM120 Megakernel fehlt | mittel | Build/Runtime fail | capability probe | optional backend | kein Blocker |
| 64-GB Host überschritten | mittel | swap/OOM | RSS/pinned monitor | cap host buffer/Ple | No-Go |
