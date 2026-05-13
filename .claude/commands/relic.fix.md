# /relic.fix

> **Before proceeding:** Read `.relic/preamble.md` and `.relic/constitution.md` in full.
> The preamble defines structural invariants that cannot be bypassed.
> If this prompt deviates from a constitution principle, a constitution amendment
> authorising the deviation must exist before you proceed.

`/relic.fix` is the **diagnosis stage** of the two-stage fix pipeline. It identifies the owning
spec, classifies the root cause, creates a fix document, and sets the active fix. It does **not**
apply code changes. Run `/relic.solve` after reviewing the fix document.

---

## Step 1 — Read session state

```bash
relic context
```

Note the `current_fix` field. If a fix is already active, ask the user whether to proceed with the
existing fix or discard it.

---

## Step 2 — Identify the owning spec

Scan all `specs/*/artifacts.json` files and read the `touches_files` arrays. Do prefix matching
against the file path or code area mentioned in the issue.

**Resolution rules:**
- **No match** → Stop. Report: *"This area is not owned by any spec. Run `/relic.specify` to
  create a spec for this feature before filing a fix."*
- **Single match** → Use that spec.
- **Multiple matches** → Longest prefix wins. If two prefixes are equal length, list them and ask
  the user to confirm.

---

## Step 3 — Load spec context

```bash
relic context --spec <owning-spec-id>
```

Read the following files in full:
- `specs/<owning-spec-id>/spec.md` — original intent
- `specs/<owning-spec-id>/plan.md` — architecture decisions
- All artifacts listed in `owns` and `reads` from `artifacts.json`
- `.relic/constitution.md` (already loaded)

---

## Step 4 — Classify the root cause

Assign exactly one classification:

| Classification | Meaning |
|---|---|
| `code-bug` | Implementation error; spec and contracts are correct |
| `misspecification` | The spec described the wrong behaviour |
| `misunderstanding` | The implementation diverged from a correct spec |
| `wrong-spec` | The spec's requirement itself is incorrect or has become stale |

---

## Step 5 — Generate a fix ID and write the fix document

Generate a fix ID: `YYYY-MM-DD-<slug>` where slug is max 6 words, hyphen-separated, derived from
the issue description (e.g. `2026-04-13-null-session-read-on-missing-file`).

Write `.relic/fixes/<fix-id>.md` using the `FixDocumentContract` schema exactly:

```markdown
# Fix: <fix-id>

**Date:** YYYY-MM-DD
**Owning spec:** <owning-spec-id>
**Status:** pending

---

## Issue

<The original issue description as reported by the user — verbatim or paraphrased.>

## Root Cause

**Classification:** code-bug | misspecification | misunderstanding | wrong-spec

<Explanation of why this classification was chosen, grounded in the spec context.>

## Proposed Changes

### Code changes
<List of files and what changes are needed. Not the actual code — the description.>

### Spec amendments
<Only present if classification is misspecification, misunderstanding, or wrong-spec.
Describe what needs to change in spec.md and/or plan.md.>

### Shared artifact changes
<Only present if a contract or domain artifact needs updating. List which artifacts
and what changes. Identify all specs in reads[] that will be affected.>

## Changelog entry (draft)
<Draft changelog entry for .relic/changelog.md. /relic.solve will write this verbatim.>
```

---

## Step 6 — Register the fix in `fixes/manifest.toon`

```bash
relic write --fixes --payload '{"name":"<fix-title>","file":"<fix-id>.md","description":"<one-sentence summary of the issue>","tags":["<classification>","<owning-spec-id>"]}'
```

Do not open or edit `fixes/manifest.toon` directly.

---

## Step 7 — Activate the fix

```bash
relic use --fix <fix-id>
```

---

## Step 8 — Report to the user

Output:
1. **Owning spec:** which spec owns the affected code area
2. **Classification:** one of the four categories with a brief rationale
3. **Fix document:** path to the created fix doc (`.relic/fixes/<fix-id>.md`)
4. **Next step:** *"Review the fix document, then run `/relic.solve` to apply the changes. If the
   classification is `misspecification` or `misunderstanding`, run `/relic.clarify` after solving
   to update the spec."*
