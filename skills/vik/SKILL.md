---
name: vik
description: Use for explicit top-level execution requests like "$vik <phase>". Reads planner-created phase execution plans, delegates bounded implementation chunks only to implementer, runs verification through implementer, and updates status.md.
---

# VIK Skill

VIK is head-of-tech style execution orchestrator.

## Trigger

Explicit invocation only:

- `$vik Build current active phase`
- `$vik Build phase 02 from project-plans/02-auth-foundation.md`

## Goal

Take one broad phase and move it through:

1. bounded implementation with testing and linting
2. status update

Keep parent context clean by loading only needed files and writing progress back to disk.

## Workflow

1. Read repo `AGENTS.md` if present.
2. Read `project-plans/status.md`.
3. Resolve target phase:
   - explicit phase input wins
   - otherwise use `current_phase`
4. Read broad phase file.
5. Read phase folder files:
   - `execution-plan.md`
   - `progress.md`
6. If `execution-plan.md` is missing, stop and ask for `$planner` to create the phase execution plan.
7. Decide next implementation chunk from `execution-plan.md` and `progress.md`.
8. Delegate only to `implementer`:
   - small phase = one `implementer`
   - medium/large phase = multiple bounded `implementer` runs
9. Each implementer gets explicit ownership and must:
   - Write/update code
   - Run tests and ensure they pass
   - Run linting and fix issues
   - Verify node checks pass
10. Integrate implementer results without broadening scope.
11. Update:
   - `progress.md`
   - `status.md`
12. If phase is complete, advance `current_phase` or mark project done.

## Delegation Rules

- `implementer` only
- no other child agents

Do not let child agents redefine product scope.

## Chunking Rules

Split work when:

- write scope is large
- multiple subsystems involved
- testing burden is high

Each chunk should have:

- explicit file/module ownership
- clear completion condition
- clear validation step (tests, linting, node checks)

## Required Disk Writes

- `project-plans/NN-phase-name/execution-plan.md`
- `project-plans/NN-phase-name/progress.md`
- `project-plans/status.md`

## Anti-Patterns

Do not:

- keep plan only in chat memory
- call `phase_planner`
- ask broad product questions during implementation unless blocked
- let one implementer own whole repository without reason
- mark phase done without implementer running tests, linting, and node checks
