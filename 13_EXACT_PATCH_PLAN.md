# Exakter Patch- und Prüfplan für den Implementer

## Repository-Workspace

```bash
export WORK=/srv/megaboobs-vb
mkdir -p "$WORK" && cd "$WORK"
git clone https://github.com/jpezzulli/sglang-rtxpro6000.git runtime
cd runtime
git remote add hyperburst https://github.com/aiueo52/sglang-rtxpro6000.git
git remote add lesilva https://github.com/Lesilva/qwen38-flashnext-nvfp4-kv.git
git fetch --all --tags --prune
git log --oneline --decorate -20
```

## Baseline-Snapshot

1. Den tatsächlich verwendeten Pennyroyal-Commit in `manifests/versions.json` eintragen.
2. `git worktree add ../baseline <SHA>` anlegen.
3. Baseline mit FP8-KV starten; `nvidia-smi`, Server-Args, Model-Revision und Promptset speichern.
4. Erst wenn Smoke-/Tool-/JSON-Tests bestehen, Patcharbeit beginnen.

## Hyperburst-Patches

Nicht den Hyperburst-Branch komplett übernehmen. Für jeden Kandidaten:

```bash
git log --oneline --all -- patches/sglang patches/flashinfer launch
 git show --stat <candidate-commit>
 git diff <pennyroyal-base>..<candidate-commit> -- sglang python
```

Reihenfolge:

1. FlashInfer-Patches mit exakt gepinnter Version.
2. CUDA-Graph-/NEXTN-Änderungen.
3. Adaptive Draft-Breite.
4. öffentliche Tokenmap optional.
5. OEA zuletzt und nur mit separater Qualitätsmessung.

Jeder Patch erhält einen eigenen Branch `integration/phase-02-<name>`, einen eigenen Commit und einen A/B-Report. Private MTP-Heads werden nicht angenommen.

## Lesilva-NVFP4-KV

```bash
git clone https://github.com/Lesilva/qwen38-flashnext-nvfp4-kv.git ../lesilva-kv
find ../lesilva-kv/overlay ../lesilva-kv/patches ../lesilva-kv/tests -type f | sort
rg -n "scale|packed|E2M1|KV|MTP|gather|hicache" ../lesilva-kv
```

Dann einen Adapter schreiben, statt den Scheduler direkt zu verändern:

- `runtime/kv/backend.py`: Interface.
- `runtime/kv/fp8.py`: vorhandener Referenzpfad.
- `runtime/kv/nvfp4.py`: Lesilva-Format.
- `runtime/kv/scale_scheme.py`: Dtype-/Scheme-Prüfung.
- `tests/kv/test_nvfp4_roundtrip.py`.

## Scale-Sicherheitsprobe

Für jeden Checkpoint `k_scale/v_scale` und `kv_cache_scheme` aus dem Safetensors-Header erfassen. Wenn Scheme-Bits und Server-Dtype nicht übereinstimmen: Start abbrechen oder dynamische Scales erzwingen. Eine erfolgreiche Antwort ohne diese Prüfung gilt als Fehler.

## HiCache-Probe

1. Mit `--hicache-io-backend direct` starten.
2. 64-, 1024-, 64K-, 350K- und 500K-Prefix schreiben.
3. Server stoppen, nur Cache und Manifest behalten, neu starten.
4. Restore mit 4 GiB, 8 GiB und absichtlich zu kleinem Puffer messen.
5. Daten, Scales, Draft-KV und Mamba/GDN getrennt checksum'en.
6. Teilrestore absichtlich unterbrechen; es darf kein Hit gemeldet werden.

Der `kernel`-Pfad darf erst wieder aktiviert werden, wenn die #43239-Probe bytegenau besteht.

## Overlay-Probe

```bash
python tools/overlay/snapshot.py --model nvidia/... --out manifests/nvidia.json
python tools/overlay/snapshot.py --model thinkingcap/... --out manifests/thinkingcap.json
python tools/overlay/compatibility_matrix.py manifests/nvidia.json manifests/thinkingcap.json
```

Nur Matrixzeilen mit identischem Namen, Shape, Dtype, quantization scheme und Scale-Layout dürfen `COPY_THINKINGCAP` werden. Sonst `KEEP_NVIDIA` oder `BLOCK`. Requantisierung ist eine eigene, spätere ADR.

## Abschluss-Gate

Der Implementer liefert `reports/phase-XX.md` mit SHAs, Befehlen, Logs, Messwerten, Fehlversuchen und Rollback. Ohne Report kein Merge in die nächste Phase.
