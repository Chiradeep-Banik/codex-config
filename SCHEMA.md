# Schema Design

## Goal

Keep parent agent memory clear by moving project state to disk.

Use `project-plans/` as durable source of truth.

## Recommended Disk Schema

```text
project-plans/
  status.md
  01-phase-name.md
  01-phase-name/
    execution-plan.md
    progress.md
    review.md
    notes.md
  02-phase-name.md
  02-phase-name/
    execution-plan.md
    progress.md
    review.md
```

## File Roles

### `status.md`

Single source of truth for workflow state.

Tracks:

- current active phase
- per-phase status
- last updated timestamp
- blockers
- next action

### `NN-phase-name.md`

Broad phase definition only.

Should include:

- objective
- scope
- dependencies
- deliverables
- exit criteria
- open questions

No implementation-level task tree here.

### `NN-phase-name/execution-plan.md`

Detailed build plan for one phase.

Should include:

- technical design
- file/module ownership
- chunking strategy
- validation plan
- unresolved technical assumptions

### `NN-phase-name/progress.md`

Execution heartbeat.

Should include:

- chunks completed
- chunks remaining
- decisions made during execution
- blockers
- handoff state

### `NN-phase-name/review.md`

Reviewer output.

Should include:

- findings by severity
- tests missing
- mismatch against plan
- go/no-go recommendation

## Why Two Phase Representations

Use both:

- root `NN-phase-name.md` = stable product-facing phase contract
- folder `NN-phase-name/` = execution workspace

Reason:

- broad phase file stays clean
- detailed execution artifacts do not bloat top-level view
- VIK can load narrow folder state only

## Status Values

Recommended phase status enum:

- `pending`
- `active`
- `detailed-planning`
- `in-progress`
- `review-needed`
- `blocked`
- `done`

## Suggested `status.md` Format

```md
# Project Status

last_updated: 2026-04-11
current_phase: 02
overall_status: in-progress
next_action: expand detailed execution plan for phase 02

## Blockers
- None

## Phases
- [x] 01 Discovery and setup | done
- [ ] 02 Auth and tenant foundation | active
- [ ] 03 Core application workflows | pending
- [ ] 04 QA, polish, and release | pending
```

## Naming Rules

- use zero-padded numeric prefix: `01`, `02`, `03`
- use lowercase kebab slug
- keep names stable after creation
- avoid renumbering unless scope changed heavily

## Minimum Write Set By Workflow

### `$planner`

Must write:

- `project-plans/status.md`
- `project-plans/NN-phase-name.md`

May write:

- `project-plans/README.md`

### `$vik`

Must write:

- `project-plans/NN-phase-name/execution-plan.md`
- `project-plans/NN-phase-name/progress.md`
- `project-plans/NN-phase-name/review.md`
- `project-plans/status.md`

## Design Rule

If parent agent would need to remember it later, write it to disk now.
