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

# context-mode — MANDATORY routing rules

You have context-mode MCP tools available. These rules are NOT optional — they protect your context window from flooding. A single unrouted command can dump 56 KB into context and waste the entire session. Codex CLI does NOT have hooks, so these instructions are your ONLY enforcement mechanism. Follow them strictly.

## Think in Code — MANDATORY

When you need to analyze, count, filter, compare, search, parse, transform, or process data: **write code** that does the work via `ctx_execute(language, code)` and `console.log()` only the answer. Do NOT read raw data into context to process mentally. Your role is to PROGRAM the analysis, not to COMPUTE it. Write robust, pure JavaScript — no npm dependencies, only Node.js built-ins (`fs`, `path`, `child_process`). Always use `try/catch`, handle `null`/`undefined`, and ensure compatibility with both Node.js and Bun. One script replaces ten tool calls and saves 100x context.

## BLOCKED commands — do NOT use these

### curl / wget — FORBIDDEN
Do NOT use `curl` or `wget` in any shell command. They dump raw HTTP responses directly into your context window.
Instead use:
- `ctx_fetch_and_index(url, source)` to fetch and index web pages
- `ctx_execute(language: "javascript", code: "const r = await fetch(...)")` to run HTTP calls in sandbox

### Inline HTTP — FORBIDDEN
Do NOT run inline HTTP calls via `node -e "fetch(..."`, `python -c "requests.get(..."`, or similar patterns. They bypass the sandbox and flood context.
Instead use:
- `ctx_execute(language, code)` to run HTTP calls in sandbox — only stdout enters context

### Direct web fetching — FORBIDDEN
Do NOT use any direct URL fetching tool. Raw HTML can exceed 100 KB.
Instead use:
- `ctx_fetch_and_index(url, source)` then `ctx_search(queries)` to query the indexed content

## REDIRECTED tools — use sandbox equivalents

### Shell (>20 lines output)
Shell is ONLY for: `git`, `mkdir`, `rm`, `mv`, `cd`, `ls`, `npm install`, `pip install`, and other short-output commands.
For everything else, use:
- `ctx_batch_execute(commands, queries)` — run multiple commands + search in ONE call
- `ctx_execute(language: "shell", code: "...")` — run in sandbox, only stdout enters context

### File reading (for analysis)
If you are reading a file to **edit** it → reading is correct (edit needs content in context).
If you are reading to **analyze, explore, or summarize** → use `ctx_execute_file(path, language, code)` instead. Only your printed summary enters context. The raw file stays in the sandbox.

### grep / search (large results)
Search results can flood context. Use `ctx_execute(language: "shell", code: "grep ...")` to run searches in sandbox. Only your printed summary enters context.

## Tool selection hierarchy

1. **GATHER**: `ctx_batch_execute(commands, queries)` — Primary tool. Runs all commands, auto-indexes output, returns search results. ONE call replaces 30+ individual calls.
2. **FOLLOW-UP**: `ctx_search(queries: ["q1", "q2", ...])` — Query indexed content. Pass ALL questions as array in ONE call.
3. **PROCESSING**: `ctx_execute(language, code)` | `ctx_execute_file(path, language, code)` — Sandbox execution. Only stdout enters context.
4. **WEB**: `ctx_fetch_and_index(url, source)` then `ctx_search(queries)` — Fetch, chunk, index, query. Raw HTML never enters context.
5. **INDEX**: `ctx_index(content, source)` — Store content in FTS5 knowledge base for later search.

## Output constraints

- Keep responses under 500 words.
- Write artifacts (code, configs, PRDs) to FILES — never return them as inline text. Return only: file path + 1-line description.
- When indexing content, use descriptive source labels so others can `search(source: "label")` later.

## ctx commands

| Command | Action |
|---------|--------|
| `ctx stats` | Call the `stats` MCP tool and display the full output verbatim |
| `ctx doctor` | Call the `doctor` MCP tool, run the returned shell command, display as checklist |
| `ctx upgrade` | Call the `upgrade` MCP tool, run the returned shell command, display as checklist |
| `ctx purge` | Call the `purge` MCP tool with confirm: true. Warns before wiping the knowledge base. |

After /clear or /compact: knowledge base and session stats are preserved. Use `ctx purge` if you want to start fresh.

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
