# Atomic Task Card Template

## Identity

- Task ID:
- Title:
- Owner role:
- Status:
- Branch:
- Parent task:

## Objective

One sentence describing the single observable result.

## Allowed scope

List exact files/directories that may change. Changes outside this list require coordinator approval.

## Inputs

- Source SHAs:
- Evidence documents:
- Environment assumptions:

## Procedure

Numbered commands and edits. No implicit research step. If a command fails, record the failure and stop.

## Tests

Every task must have at least one deterministic test or inspection command.

## Definition of done

- [ ] required artifact exists
- [ ] required test passes
- [ ] report written
- [ ] commit created
- [ ] inventory updated
- [ ] rollback commit recorded

## Stop conditions

List conditions that force `BLOCKED`.

## Handover

Use the YAML format in `15_OMO_ORCHESTRATION.md`.
