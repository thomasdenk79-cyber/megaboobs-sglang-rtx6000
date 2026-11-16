# Atomic Task Inventory

Status values: `LOCKED`, `READY`, `RUNNING`, `REVIEW`, `DONE`, `BLOCKED`.

| ID | Task | Owner | Depends on | Output | Parallel group |
|---|---|---|---|---|---|
| VB-00 | Freeze source revisions and licenses | Source Analyst | — | source manifest | A |
| VB-01 | Inventory Pennyroyal runtime tree | Source Analyst | VB-00 | runtime file map | A |
| VB-02 | Inventory Hyperburst patch families | Source Analyst | VB-00 | patch map | A |
| VB-03 | Inventory Lesilva 3.1.5 NVFP4 files | Source Analyst | VB-00 | NVFP4 file map | A |
| VB-04 | Inspect NVIDIA checkpoint metadata | Source Analyst | VB-00 | NVIDIA tensor manifest | B |
| VB-05 | Inspect ThinkingCap metadata | Source Analyst | VB-00 | ThinkingCap tensor manifest | B |
| VB-06 | Decide overlay compatibility | Runtime Architect | VB-04,VB-05 | compatibility ADR | C |
| VB-07 | Build FP8 baseline container | Runtime Engineer | VB-01,VB-04 | baseline image/report | C |
| VB-08 | Baseline correctness suite | Benchmark Engineer | VB-07 | baseline evidence | D |
| VB-09 | Port FlashInfer patch family | Runtime Engineer | VB-08,VB-02 | isolated commit | E |
| VB-10 | Port NEXTN/graph patch family | Runtime Engineer | VB-09 | isolated commit | E |
| VB-11 | Implement KVBackend interface | KV Engineer | VB-08 | backend contract | F |
| VB-12 | Port NVFP4 packed KV path | KV Engineer | VB-03,VB-11 | NVFP4 backend | F |
| VB-13 | Add scale scheme validation | KV Engineer | VB-12 | loader guard | F |
| VB-14 | NVFP4 eager round-trip tests | Reviewer | VB-12,VB-13 | test report | G |
| VB-15 | Integrate direct HiCache path | KV Engineer | VB-14 | direct cache profile | H |
| VB-16 | Implement chunked SSD restore | KV Engineer | VB-15 | streaming restore | H |
| VB-17 | Crash/corruption recovery tests | Reviewer | VB-16 | recovery report | I |
| VB-18 | Generate overlay copy plan | Runtime Architect | VB-06 | tensor decision matrix | J |
| VB-19 | Generate hybrid checkpoint tool | Runtime Engineer | VB-18 | overlay tool | J |
| VB-20 | Overlay A/B quality suite | Benchmark Engineer | VB-19 | quality report | K |
| VB-21 | Scheduler profiles C1/C4/C8 | Runtime Engineer | VB-10,VB-17 | profiles | L |
| VB-22 | C1/C4 performance qualification | Benchmark Engineer | VB-21 | performance report | M |
| VB-23 | C6/C8 admission qualification | Benchmark Engineer | VB-22 | concurrency report | M |
| VB-24 | Packaging, deployment, rollback | Coordinator | VB-20,VB-23 | release bundle | N |

Only tasks in `READY` may be dispatched. A task becomes `DONE` only after reviewer approval.
