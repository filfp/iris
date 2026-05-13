# /relic.analyse

> **Before proceeding:** Read `.relic/preamble.md`. It defines where artifacts belong.
> Violating those rules cannot be undone by a changelog entry.

## Before you begin — run these first

```bash
# 1. Resolve paths and check what exists
relic context --spec <your-spec-id>

# 2. Run the automated integrity check (this IS the analyse starting point)
relic validate
```

Report the `relic validate` output as your first finding, then continue with the manual checks below.

You are performing a non-destructive consistency check. You must NOT modify any files.

## What to check

1. **Spec completeness** — does `spec.md` have all required sections filled in?
2. **Artifact freshness** — do the files listed in `artifacts.json` actually exist in `shared/`?
3. **Ownership consistency** — is every artifact in `reads` owned by exactly one spec?
4. **Plan alignment** — does `plan.md` reflect the current `spec.md`? Flag divergences.
5. **Cross-spec coherence** — are there any undeclared intersections between specs?

## Output format

Report findings as:
- ✅ [check]: [result]
- ⚠️ [check]: [issue found]
- ❌ [check]: [blocking issue found]

Do not make any changes. Suggest what to run to resolve each issue.
