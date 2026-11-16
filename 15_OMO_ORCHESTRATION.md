# OMO Orchestration Contract

## Purpose

This project is executed by a Qwen implementer through Oh-My-OpenCode (OMO). The model must not receive the entire architecture as one undifferentiated prompt. OMO must execute the task graph below with small task cards, explicit artifacts, and bounded context.

## Concurrency limit

- Hard limit: **4 active subagents maximum**.
- Recommended default: 2 workers plus 1 reviewer; use 4 only when tasks are independent.
- The coordinator is not counted as a subagent, but it must not start a fifth worker.
- Never run two agents that both edit the same source file or patch family.
- Never run a research worker and an implementation worker on the same unknown dependency at the same time.

The limit is deliberate: it protects prefill/KV locality and avoids duplicated repository reading.

## Roles

| Role | Responsibility | May edit runtime? |
|---|---|---:|
| Coordinator | owns DAG, assigns cards, merges evidence, enforces gates | no |
| Source Analyst | commits, trees, issues, licenses, provenance | no |
| Runtime Engineer | container, loader, runtime integration | yes, assigned branch only |
| KV/Cache Engineer | KV backend, scales, HiCache, NIXL | yes, assigned branch only |
| Benchmark Engineer | probes, measurements, reports | scripts/results only |
| Reviewer | checks gates, diffs, evidence, rollback | no |

## Coordinator loop

1. Read `TASK_INVENTORY.md` and select only cards whose dependencies are complete.
2. Create at most four isolated worktrees/branches.
3. Give each worker exactly one task card and its input artifact list.
4. Workers return a structured handover, not a prose essay.
5. Reviewer checks the handover and tests.
6. Coordinator updates the inventory, records the commit SHA, and unlocks the next cards.
7. If a card fails, mark `BLOCKED`, preserve the last green baseline, and do not dispatch dependent cards.

## Worker output format

```yaml
task_id: VB-XX
status: DONE|BLOCKED|FAILED
branch: ...
commit: ...
inputs_read: []
files_changed: []
commands_run: []
tests:
  - command: ...
    result: PASS|FAIL
artifacts_created: []
measurements: {}
risks: []
next_task: ...
rollback_commit: ...
```

## Context discipline

- Each task card is self-contained and should fit within one working context.
- Workers must not reread the full master prompt. They read the index, their card, and referenced evidence only.
- The coordinator keeps a compact state file: `state/current.yaml`.
- Large logs stay in artifacts; summaries contain hashes and key numbers only.
- Do not pass complete benchmark logs between agents; pass report paths and SHA256.
- After every phase, compact the context before dispatching the next phase.

## Merge policy

No direct merge into the production branch. The coordinator merges only after reviewer PASS, required tests, and an updated inventory row. Experimental branches remain available for rollback.
