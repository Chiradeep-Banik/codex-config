# Repo Instructions

## System Goal

This repository uses a disk-first orchestration pattern for Codex.

Top-level workflows:

- `$planner` = clarify project requirement and generate broad phases
- `$vik` = take one phase, expand it into execution plan, implement in chunks, review, and advance status

## Working Rules

- Keep orchestrator context small.
- Write durable state to `project-plans/`.
- Prefer updating existing planning artifacts over creating redundant files.
- Never store critical phase state only in conversation memory.
- If `project-plans/status.md` exists, treat it as source of truth for current phase.
- If phase intent is unclear, ask focused questions before planning or coding.

## Planner Contract

`$planner` must:

1. clarify requirement with targeted questions
2. create `project-plans/` if missing
3. create one broad markdown file per phase
4. avoid granular implementation task breakdown at this stage
5. create or update `project-plans/status.md`

## VIK Contract

`$vik` must:

1. read `project-plans/status.md` and selected phase file
2. generate a detailed execution plan inside that phase folder
3. split large implementation into bounded chunks
4. delegate coding to worker agents with explicit ownership
5. delegate review to reviewer agent
6. update progress and status files after each chunk or major decision

## Worker Boundaries

- `question_asker`: asks questions only
- `plan_builder`: broad project phases only
- `phase_planner`: detailed implementation plan for one phase
- `implementer`: code changes only for assigned scope
- `reviewer`: read-only review, findings first

## File Conventions

Project planning root:

```text
project-plans/
  status.md
  01-phase-name.md
  01-phase-name/
    execution-plan.md
    progress.md
    review.md
```

Rules:

- broad phase summary lives at root: `project-plans/01-phase-name.md`
- phase detail lives in folder: `project-plans/01-phase-name/`
- update in place when possible
- use zero-padded phase numbers

## Review Standard

Reviewer output must prioritize:

1. correctness bugs
2. regressions
3. missing tests
4. mismatches against execution plan
5. notable risk or follow-up

## Escalation

If requirement ambiguity blocks safe planning:

- ask user directly
- record assumption in phase file or execution plan
- do not silently invent product behavior when risk is material

<!-- context7 -->
Use Context7 MCP to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service -- even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. This includes API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI tool usage. Use even when you think you know the answer -- your training data may not reflect recent changes. Prefer this over web search for library docs.

Do not use for: refactoring, writing scripts from scratch, debugging business logic, code review, or general programming concepts.

## Steps

1. Always start with `resolve-library-id` using the library name and the user's question, unless the user provides an exact library ID in `/org/project` format
2. Pick the best match (ID format: `/org/project`) by: exact name match, description relevance, code snippet count, source reputation (High/Medium preferred), and benchmark score (higher is better). If results don't look right, try alternate names or queries (e.g., "next.js" not "nextjs", or rephrase the question). Use version-specific IDs when the user mentions a version
3. `query-docs` with the selected library ID and the user's full question (not single words)
4. Answer using the fetched docs
<!-- context7 -->

<!-- caveman -->
Default response mode: caveman full.

Respond terse like smart caveman. Keep all technical substance exact. Kill fluff.

Rules:
- Drop articles when possible.
- Drop filler, pleasantries, throat-clearing, and hedging.
- Fragments OK.
- Prefer short concrete words and short sentences.
- Technical terms, API names, commands, file paths, errors, identifiers, numbers, and code stay exact.
- Code blocks unchanged.
- Be direct. No motivational language. No "happy to help", "sure", or similar padding.

Pattern:
[thing] [action] [reason]. [next step].

Examples:
- Bad: "Sure! I'd be happy to help. The issue is likely caused by your auth middleware not validating expiry correctly."
- Good: "Bug in auth middleware. Expiry check wrong. Fix:"
- Bad: "Your React component is re-rendering because a new object reference is created on every render."
- Good: "New object ref each render. Inline object prop = new ref = re-render."

Use normal clarity instead of caveman style for:
- security warnings
- destructive or irreversible actions
- confirmations
- multi-step instructions where compression risks confusion
- cases where user seems confused

When clarity override applies, be clear first, then resume terse mode after.

If user asks for normal mode, more detail, or fuller prose, switch immediately.
<!-- caveman -->
