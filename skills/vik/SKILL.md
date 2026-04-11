---
name: vik
description: Use for explicit top-level execution requests like "$vik <phase>". Reads current phase state, creates detailed execution plan, delegates bounded implementation chunks with full testing/linting checks, and updates status.md.
---

# VIK Skill

VIK is head-of-tech style execution orchestrator.

## Trigger

Explicit invocation only:

- `$vik Build current active phase`
- `$vik Build phase 02 from project-plans/02-auth-foundation.md`

## Goal

Take one broad phase and move it through:

1. detailed planning
2. bounded implementation with testing and linting
3. status update

Keep parent context clean by loading only needed files and writing progress back to disk.

## Workflow

1. Read repo `AGENTS.md` if present.
2. Read `project-plans/status.md`.
3. Resolve target phase:
   - explicit phase input wins
   - otherwise use `current_phase`
4. Read broad phase file.
5. Read existing phase folder files if present:
   - `execution-plan.md`
   - `progress.md`
6. If no detailed plan exists or phase changed materially, explicitly delegate to `phase_planner`.
7. Write `project-plans/NN-phase-name/execution-plan.md`.
8. Decide chunking:
   - small phase = one `implementer`
   - medium/large phase = multiple bounded `implementer` runs
9. Each implementer gets explicit ownership and must:
   - Write/update code
   - Run tests and ensure they pass
   - Run linting and fix issues
   - Verify node checks pass
10. Update:
   - `progress.md`
   - `status.md`
11. If phase is complete, advance `current_phase` or mark project done.

## Delegation Rules

- `phase_planner` for detailed plan only
- `implementer` for code, testing, linting, and verification

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
- ask broad product questions during implementation unless blocked
- let one implementer own whole repository without reason
- mark phase done without implementer running tests, linting, and node checks
