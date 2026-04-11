# Storage And Disk Usage

## Short Answer

Disk usage is small if you keep artifacts markdown-only.

For most projects:

- `status.md`: under 5 KB
- broad phase files: 2 KB to 10 KB each
- execution plans: 5 KB to 25 KB each
- progress and review files: 2 KB to 15 KB each

Even with 20 phases, total usually stays under a few MB.

## Biggest Disk Risks

Not markdown. Usually these:

- logs copied into phase files
- screenshots or binary attachments
- duplicated long code blocks
- repeated full transcripts

Avoid those.

## Good Storage Pattern

Store:

- summaries
- decisions
- references to files
- review findings
- checkpoints

Do not store:

- full terminal logs unless required
- large diffs pasted repeatedly
- long chat transcripts
- binaries inside `project-plans/`

## Practical Retention Policy

Keep:

- `status.md`
- broad phase files
- latest execution plan
- latest review
- latest progress

Optional archive:

- old review iterations into `project-plans/archive/`
- old chunk plans into `project-plans/NN-phase-name/archive/`

## If Project Gets Large

Use this split:

```text
project-plans/
  active/
  completed/
  archive/
```

Or:

```text
project-plans/
  01-phase-name/
    execution-plan.md
    progress.md
    review.md
    archive/
```

## Recommended Compression Rule

Every file should answer one question fast:

- `status.md` = what is current state?
- phase file = what is this phase?
- execution plan = how are we building it?
- progress = what happened?
- review = what is wrong or approved?

If file starts answering many questions, split it.

## Schema Design Tradeoff

### Flat only

Pros:

- simple

Cons:

- phase detail mixes with phase summary

### Folder per phase

Pros:

- better isolation
- easier VIK context loading
- cleaner archives

Cons:

- more files

Best choice for your use case: folder per phase.

## Recommendation

Start with markdown-only schema.

Do not add database.
Do not add JSON state store unless automation later needs machine-precise parsing.

If needed later, add optional `status.json` mirrored from `status.md`.
