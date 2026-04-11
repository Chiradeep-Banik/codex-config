# Codex Scaffold

Codex scaffold for 2 top-level workflows:

- `$planner <requirement>`
- `$vik <phase-or-instruction>`

Goal: keep orchestration thin, push details to disk, keep parent context clean.

## Layout

```text
.codex-scafold/
  AGENTS.md
  config.toml
  README.md
  SCHEMA.md
  STORAGE.md
  agents/
    question_asker.toml
    plan_builder.toml
    phase_planner.toml
    implementer.toml
    reviewer.toml
  skills/
    planner/
      SKILL.md
    vik/
      SKILL.md
  templates/
    project-plans/
      status.md
      phase.md
      execution-plan.md
      review.md
      progress.md
```

## Intended Runtime Shape

- `planner` is a skill, not just a worker. It orchestrates requirement clarification and broad phase creation.
- `vik` is a skill, not just a worker. It orchestrates detailed planning, execution chunks, and review.
- Custom agents do narrow work only.

## Recommended Invocation

```text
$planner Build a multi-tenant internal tool for inventory tracking, with auth, role-based access, import/export, and admin analytics.
```

```text
$vik Build the current active phase from project-plans/status.md
```

```text
$vik Build phase 02 from project-plans/02-auth-foundation.md
```

## Copy Into Real Codex Project

1. Copy `AGENTS.md` to repo root.
2. Copy `config.toml` into `.codex/config.toml` or merge into existing config.
3. Copy `agents/*.toml` into `.codex/agents/`.
4. Copy `skills/` into `.codex/skills/`.
5. Copy `templates/project-plans/` into project docs area if you want starter files.
6. Create `project-plans/` in target repo when first planning run starts.

## Why This Shape

- skills = stable top-level entrypoints
- agents = narrow workers
- markdown files = persistent memory
- status file = single source of truth for phase state

Read [SCHEMA.md](/home/banik/Desktop/Code/Build/get-shit-done/.codex-scafold/SCHEMA.md) for on-disk design.
Read [STORAGE.md](/home/banik/Desktop/Code/Build/get-shit-done/.codex-scafold/STORAGE.md) for disk usage guidance.
