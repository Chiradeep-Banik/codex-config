---
name: planner
description: Use for explicit top-level project planning requests like "$planner <requirement>". Explores the codebase first, uses grill-me for all clarifying questions, then delegates broad planning to plan_builder and detailed phase planning to phase_planner. Do not implement.
---

# Planner Skill

Top-level planning workflow for Codex.

## Trigger

Explicit invocation only:

- `$planner <requirement>`

Examples:

- `$planner Build a B2B ticketing platform with SSO, billing, admin analytics, and audit logs.`
- `$planner Plan a mobile-first AI note-taking app with offline sync and team sharing.`

## Goal

Take a project requirement and turn it into durable broad and phase execution planning artifacts on disk while keeping the main model context small.

## Workflow

1. Explore the codebase and understand what needs to be done:
   - Read repo `AGENTS.md` if present.
   - Inspect project structure, existing implementation, and relevant files.
   - Check whether `project-plans/` exists.
   - If missing, create it.
   - Inspect `project-plans/status.md` if present.
2. Use `$grill-me` for every clarification:
   - Do not assume missing product, design, technical, sequencing, or validation details.
   - Ask one question at a time.
   - Include the recommended answer with each question.
   - If a question can be answered by exploring the codebase, explore the codebase instead.
   - Resolve dependent decisions before moving to the next branch.
3. Collect the confirmed requirement summary, decisions, assumptions explicitly accepted by the user, and non-goals.
4. Delegate to `plan_builder` using `agents/plan_builder.toml` to create the broad multi-phase plan.
5. For each broad phase, one by one, delegate to `phase_planner` using `agents/phase_planner.toml` to create the detailed implementation plan.
6. Write or update:
   - `project-plans/status.md`
   - `project-plans/NN-phase-name.md`
   - `project-plans/NN-phase-name/execution-plan.md`
   - `project-plans/NN-phase-name/progress.md`
7. Return a concise summary of created phases, execution plans, and active phase.

## Output Rules

- broad phases plus phase execution plans only
- phase sequencing should be outcome-driven
- phase names should be stable and readable
- execution plans should be chunkable by file or module ownership
- leave code changes and verification runs for `$vik`

## Required Disk Writes

- `project-plans/status.md`
- one broad markdown file per phase
- one `execution-plan.md` per phase folder
- one initial `progress.md` per phase folder

## `status.md` Minimum Fields

Include:

- `last_updated`
- `current_phase`
- `overall_status`
- `next_action`
- phase checklist

## Anti-Patterns

Do not:

- create code-level task lists here
- implement code here
- mix broad planning and implementation planning
- keep planning state only in memory
- leave phase execution planning for `$vik`
