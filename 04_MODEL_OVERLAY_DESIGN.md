# Modell- und Tensor-Overlay

## Grundsatz

Der NVIDIA-Checkpoint ist unveränderliche Referenz. Das Overlay erzeugt einen neuen Verzeichnisbaum außerhalb von Git:

```text
models/
  nvidia-fp4-local/        # read-only source
  thinkingcap-source/       # read-only source
  hybrid-vb/<manifest-id>/  # generated output
```

## Pipeline

1. `snapshot.py`: Revision, Dateien, SHA256, Lizenz und `config.json` erfassen.
2. `inspect_tensors.py`: alle Tensoren mit Name, Shape, dtype, Quantisierung und Scale-Referenzen auslesen.
3. `compatibility_matrix.py`: exakte Namens-/Shape-/Dtype-Matrix erzeugen.
4. `overlay.py --mode=copy|requantize`: standardmäßig nur `copy` erlauben.
5. `validate_hybrid.py`: safetensors header, tensor counts, scales, config und loader dry-run prüfen.
6. `quality_ab.py`: NVIDIA-Baseline gegen Hybrid mit identischen Prompts vergleichen.

## Zulässige Entscheidungen

- `KEEP_NVIDIA`: PLE, MoE-Experten, MTP und alle nicht eindeutig kompatiblen Tensoren.
- `COPY_THINKINGCAP`: nur dense Tensoren bei exakter Kompatibilität.
- `REQUANTIZE`: standardmäßig verboten; nur nach eigenem numerischem Fehlerbudget, Kalibrierung und Review.
- `BLOCK`: bei Shape-/Scale-/Architecture-Mismatch.

## Abnahmekriterien

- Kein Originalfile verändert.
- Manifest mit beiden Revisionen und SHA256.
- Loader dry-run ohne unbekannte Tensoren.
- Logits-Korrelation und deterministischer A/B-Vergleich.
- Coding-/Tool-/JSON-Test ohne relevante Regression.
- Bei Fehler: NVIDIA-Baseline bleibt startbar.
