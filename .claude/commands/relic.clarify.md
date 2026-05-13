# /relic.clarify

> **Before proceeding:** Read `.relic/preamble.md`. It defines where artifacts belong.
> Violating those rules cannot be undone by a changelog entry.

## Before you begin — run these first

```bash
# 1. Resolve paths and check what exists
relic context --spec <your-spec-id>

# 2. Validate shared brain integrity
relic validate
```

Do not proceed if `relic validate` reports `"valid": false`.

You are appending details, changing contracts, or adding behaviors to an existing spec.

## Before you begin

1. Read `.relic/constitution.md`.
2. Read `specs/{{SPEC_ID}}/spec.md` fully.
3. Read `specs/{{SPEC_ID}}/artifacts.json`.
4. Load all referenced shared artifacts.

## Intersection check (mandatory)

Before making any changes, check all other specs' `artifacts.json` files:
- Does the change you're about to make affect an artifact owned by another spec?
- Does it add files to `touches_files` that another spec already owns?

If yes, flag the intersection explicitly and do not proceed until resolved.

## Your task

Apply the user's clarification to `spec.md`:
- Update requirements, user stories, scope, or decisions as needed.
- If a shared artifact changes, update it.
- Update `artifacts.json` if ownership or file touches change.

## After changes — changelog (cross-artifact mutations only)

Only write a changelog entry if a shared artifact **owned by this spec** was amended in this
clarify session. Do not write one for: spec.md edits, open question resolution, new artifact
creation, or artifacts.json updates.

If a cross-artifact mutation occurred, run:
```bash
relic write --changelog --payload '{"name":"<spec-id>: <what changed>","slash_command":"/relic.clarify","description":"<why it changed>"}'
```

Do not open or edit `changelog.md` directly.
