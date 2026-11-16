# Initial Prompt for OMO

Clone and execute this architecture repository:

```bash
git clone https://github.com/thomasdenk79-cyber/megaboobs-sglang-rtx6000.git
cd megaboobs-sglang-rtx6000
```

Read `19_INDEX.md` first, then `00_START_HERE.md`, `AGENTS.md`, `15_OMO_ORCHESTRATION.md`, `16_TASK_INVENTORY.md`, and `tasks/MASTER_REQUIREMENTS_EN.md`.

You are the OMO coordinator. Do not ask a weak local Qwen model to invent the architecture or perform the full project from one mega-prompt. Use the existing architecture, dependency graph, source manifests, task cards, patch instructions, tests, and review gates as the single source of truth.

Start with task `VB-00`. Dispatch only tasks whose dependencies are complete. Use at most four active subagents; normally use two workers and one reviewer. Keep heavy repository research and prefill-intensive work bounded to preserve context and KV locality. Never assign two agents to the same files or patch family.

Each worker receives exactly one atomic task card and must return the YAML handover format defined in `15_OMO_ORCHESTRATION.md`. Record every branch, commit SHA, changed file, command, test result, measurement, risk, and rollback point in the inventory and reports.

Do not silently change the architecture, substitute model checkpoints, copy private MTP artifacts, enable OEA, enable the NVIDIA megakernel, or make NVFP4 the production default. Stop and mark the task `BLOCKED` on any tensor, scale, state, cache, license, reproducibility, or correctness mismatch. Preserve the last green baseline.

Advance to the next task only after the reviewer returns `PASS` and the task definition of done is complete. The final result must be a reproducible, rollback-capable implementation of Variant B, not a collection of unverified patches.