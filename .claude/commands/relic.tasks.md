# /relic.tasks

> **Before proceeding:** Read `.relic/preamble.md`. It defines where artifacts belong.
> Violating those rules cannot be undone by a changelog entry.

## Before you begin — run this first

```bash
relic scaffold --spec <your-spec-id>
```

This ensures the spec folder and all files exist, creating any that are missing.
Check `files_created` in the output — if `plan.md` was just created, run `/relic.plan` first.

You are generating a task list from the current implementation plan.

## Before you begin

1. Read `.relic/constitution.md`.
2. Read `specs/{{SPEC_ID}}/spec.md`.
3. Read `specs/{{SPEC_ID}}/plan.md` — this is your source of truth.
4. Check other specs' `tasks.md` files for overlap (same files being modified in parallel).

## Task overlap check

If another spec's tasks touch the same files:
- Flag the overlap explicitly in the **Notes** section of `tasks.md`.
- Do not block progress — flag it so the implementer is aware.

## Writing tasks

Fill in `specs/{{SPEC_ID}}/tasks.md`:
- Break each implementation phase into concrete, atomic tasks.
- Each task should be independently completable (one file or one concern).
- Order tasks so dependencies come first.
- Prefix tasks that depend on other specs with `[blocked by: <spec-id>]`.

## What NOT to do

- Do not write code.
- Do not modify `plan.md` — if the plan is wrong, run `/relic.plan` again.
