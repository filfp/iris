# /relic.implement

> **Before proceeding:** Read `.relic/preamble.md`. It defines where artifacts belong.
> Violating those rules cannot be undone by a changelog entry.

## Before you begin — run this first

```bash
relic context --spec <your-spec-id>
```

This returns all file paths and which files exist. Read only what it confirms is present.

You are implementing the tasks from the current plan.

## Before you begin

1. Read `.relic/constitution.md`.
2. Read `specs/{{SPEC_ID}}/spec.md`.
3. Read `specs/{{SPEC_ID}}/plan.md`.
4. Read `specs/{{SPEC_ID}}/tasks.md` — work through tasks in order.

## Constraints

- Implement exactly what the plan describes. Do not add features not in scope.
- If you discover the plan is wrong or incomplete, stop and run `/relic.plan` to update it first.
- If your implementation requires changing a shared artifact, check ownership in `artifacts.json`
  before modifying it. If you do not own it, flag it and do not modify.
- Write a changelog entry only if implementation requires amending a shared artifact owned by
  this spec (a cross-artifact mutation). Do not write one for standard task completion.

## When a task is done

Check it off in `tasks.md`:
```
- [x] Task description
```

## When a shared artifact is amended during implementation

```bash
relic write --changelog --payload '{"name":"<spec-id>: <what changed>","slash_command":"/relic.implement","description":"<why the artifact was amended during implementation>"}'
```

Do not open or edit `changelog.md` directly.
