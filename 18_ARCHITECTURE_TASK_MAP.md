# Architecture-to-Task Map

| Architecture decision | Implemented by | Verified by | Forbidden shortcut |
|---|---|---|---|
| Pennyroyal is the runtime base | VB-01, VB-07 | VB-08 | starting from Hyperburst wholesale |
| Hyperburst is selective | VB-02, VB-09, VB-10 | VB-22 | cherry-picking an entire branch |
| NVIDIA checkpoint is immutable reference | VB-04, VB-07 | VB-08 | editing source weights |
| Overlay is conditional | VB-05, VB-06, VB-18, VB-19 | VB-20 | silent tensor replacement |
| FP8 is reference KV | VB-07, VB-11 | VB-08 | making NVFP4 default before tests |
| NVFP4 owns its scales and packed layout | VB-12, VB-13 | VB-14 | reusing FP8 scales |
| Direct HiCache before kernel backend | VB-15 | VB-17 | enabling kernel path despite #43239 |
| Streaming restore is required for mini-buffer | VB-16 | VB-17 | accepting oversized-hit rejection as success |
| C1/C4 before C8 | VB-21, VB-22 | VB-23 | optimizing C8 first |
| OEA/Megakernel are optional | later experiment | separate benchmark | treating them as dependencies |
