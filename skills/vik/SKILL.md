---
name: vik
description: Use for explicit top-level execution requests like "$vik <phase>". Reads current phase state, creates detailed execution plan, delegates bounded implementation chunks, delegates review, and updates status.md.
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
2. bounded implementation
3. review
4. status update

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
   - `review.md`
6. If no detailed plan exists or phase changed materially, explicitly delegate to `phase_planner`.
7. Write `project-plans/NN-phase-name/execution-plan.md`.
8. Decide chunking:
   - small phase = one `implementer`
   - medium/large phase = multiple bounded `implementer` runs
9. Each implementer gets explicit ownership.
10. After each major chunk, delegate to `reviewer`.
11. Update:
   - `progress.md`
   - `review.md`
   - `status.md`
12. If phase is complete, advance `current_phase` or mark project done.

## Delegation Rules

- `phase_planner` for detailed plan only
- `implementer` for code only
- `reviewer` for review only

Do not let child agents redefine product scope.

## Chunking Rules

Split work when:

- write scope is large
- multiple subsystems involved
- testing burden is high
- review finds unresolved issues

Each chunk should have:

- explicit file/module ownership
- clear completion condition
- clear validation step

## Required Disk Writes

- `project-plans/NN-phase-name/execution-plan.md`
- `project-plans/NN-phase-name/progress.md`
- `project-plans/NN-phase-name/review.md`
- `project-plans/status.md`

## Anti-Patterns

Do not:

- keep plan only in chat memory
- ask broad product questions during implementation unless blocked
- let one implementer own whole repository without reason
- skip review before marking phase done
