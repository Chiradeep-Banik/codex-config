---
name: planner
description: Use for explicit top-level project planning requests like "$planner <requirement>". Clarifies requirements, creates project-plans/ if missing, writes broad phase files, and updates status.md. Do not use for implementation.
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

Take a project requirement and turn it into durable broad-phase planning artifacts on disk.

## Workflow

1. Read repo `AGENTS.md` if present.
2. Check whether `project-plans/` exists.
3. If missing, create it.
4. Inspect `project-plans/status.md` if present.
5. **Ask clarification questions to the user for ANY ambiguity or missing details:**
   - Ask about scope, priorities, constraints, integrations, or non-functional requirements
   - Use AskUserQuestion tool for explicit user clarification
   - Wait for user responses before proceeding
6. Collect clarified requirement summary from user answers.
7. Explicitly delegate to `plan_builder`.
8. Write or update:
   - `project-plans/status.md`
   - `project-plans/NN-phase-name.md`
9. Do not write detailed execution plans.
10. Return a concise summary of created phases and active phase.

## Output Rules

- broad phases only
- phase sequencing should be outcome-driven
- phase names should be stable and readable
- leave implementation detail for `$vik`

## Required Disk Writes

- `project-plans/status.md`
- one broad markdown file per phase

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
- create execution chunks here
- mix broad planning and implementation planning
- keep planning state only in memory
