# Deployment und Rollback

- Jeder Container pinnt Runtime SHA, FlashInfer SHA, CUDA/PyTorch-Version und Modellmanifest.
- Profile `baseline-fp8`, `nvfp4-experimental`, `overlay-experimental` getrennt starten.
- Produktivstandard bleibt `baseline-fp8`, bis NVFP4/Overlay-Gates bestanden sind.
- Rollback: altes Image + altes Manifest starten; Cache-Verzeichnisse nach ABI isolieren.
- Niemals einen Cache mit anderer KV-Dtype, page size, model revision oder QSA-layout wiederverwenden.
- Modelle liegen außerhalb Git unter `/srv/models`; Git enthält nur Manifeste und Skripte.
