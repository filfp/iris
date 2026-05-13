# /relic.plan

> **Before proceeding:** Read `.relic/preamble.md`. It defines where artifacts belong.
> Violating those rules cannot be undone by a changelog entry.

## Before you begin — run these first

```bash
# 1. Ensure spec folder and files exist (creates missing files from templates)
relic scaffold --spec <your-spec-id>

# 2. Validate shared brain integrity — no conflicts before planning
relic validate
```

Do not proceed if `relic validate` reports `"valid": false`. Resolve conflicts first.

You are creating an implementation plan for this spec.

## Before writing the plan

1. Read `.relic/constitution.md`.
2. Read `specs/{{SPEC_ID}}/spec.md` fully.
3. Discover relevant shared artifacts using the two-step cascade:

   **Step A — targeted search (preferred):**
   Extract up to 10 keywords from the spec (feature domain, entities, verbs, technical concepts). Run:
   ```bash
   relic search <keyword1> <keyword2> ...
   ```
   Load full files for high-score results. If the returned candidates are sufficient context for planning, stop here.

   **Step B — full brain scan (fallback, only if Step A is insufficient):**
   ```bash
   relic search --deep
   ```
   Read only `tldr` fields from results. Select and fully read only the artifacts with clear relevance to this plan.

4. Load ALL other specs' `artifacts.json` files from `specs/*/artifacts.json`.
5. Run an intersection check:
   - Which shared artifacts does this plan need to create or modify?
   - Which files does this plan touch (`touches_files`)?
   - Compare against all other specs' `owns` and `touches_files`.
6. **Report any conflicts BEFORE writing the plan.** Conflicts must be resolved first.

## Writing the plan

Fill in `specs/{{SPEC_ID}}/plan.md`:
- **Architecture Overview** — high-level approach.
- **Implementation Phases** — concrete ordered steps.
- **File Changes table** — every file to be created or modified.
- **Shared Artifact Changes** — new artifacts to create, existing ones to update.
- **Intersection Notes** — any intersections detected and how they are resolved.

## After the plan is written — changelog (cross-artifact mutations only)

Only write a changelog entry if this plan **amends an existing shared artifact** owned by this
spec. Do not write one when the plan is first created, when only spec.md or plan.md change, or
when only new artifacts are being defined.

If a cross-artifact mutation occurred, run:
```bash
relic write --changelog --payload '{"name":"<spec-id>: Plan updated — <what changed>","slash_command":"/relic.plan","description":"<what changed and why>"}'
```

Do not open or edit `changelog.md` directly.

## What NOT to do

- Do not write a plan that claims ownership of an artifact already owned by another spec.
- Do not skip the intersection check.
- Do not write code.
