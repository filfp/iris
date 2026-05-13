# /relic.scan

Bootstrap the Relic knowledge layer from an existing codebase. Run this **once** when
adopting Relic on a project that already has code. It generates the shared artifact layer
(`.relic/shared/`) so every subsequent spec starts with a populated brain.

> This is an expensive operation — it reads many files. That cost is paid once. After this,
> each spec runs faster because the domains, contracts, and rules are already defined.

---

## Step 0 — Get the project manifest

```bash
relic scan --json
```

The manifest tells you the tech stack, key files, and what artifacts already exist.
Do not explore the filesystem manually — use the manifest to navigate efficiently.

---

## Step 1 — Read entry points and type files

From the manifest's `key_files`, read all files with role `entry_point`, `types`, or `schema`.
These reveal the domain language and contracts fastest.

If the project has no schema or type files, read the top-level `src/` or equivalent directory
and identify the main packages or modules.

---

## Step 2 — Read route and service files

From `key_files`, read roles `routes`, `services`, `middleware`.
These show what the project exposes and what cross-cutting logic exists.

---

## Step 3 — Sample business logic

Read 5–10 files from the core business logic directories (not tests, not generated code).
Focus on: validation logic, calculation, workflows, state machines.

If the project has >500 files, limit to the first 3 levels of the main source directory.

---

## Step 4 — Write domain artifacts

For each bounded context you identified, write:

```
.relic/shared/domains/<DomainName>.md
```

Use this format:

```markdown
# <DomainName>

**Type:** domain
**Inferred from:** <comma-separated file paths>
**Confidence:** high | medium | low

## Description
What this bounded context is responsible for.

## Key Entities
- Entity: description

## Relationships
How this domain relates to others.

## Owned by
(unowned — assign when a spec takes responsibility)
```

---

## Step 5 — Write contract artifacts

For each API surface, event schema, or shared data interface, write:

```
.relic/shared/contracts/<ContractName>.md
```

```markdown
# <ContractName>

**Type:** contract
**Inferred from:** <file paths>
**Confidence:** high | medium | low

## Description
What this contract defines.

## Shape
Key fields, endpoints, or event structure (no need to be exhaustive — capture the intent).

## Consumers
Which parts of the codebase depend on this contract.

## Owned by
(unowned — assign when a spec takes responsibility)
```

---

## Step 6 — Write rule artifacts

For each cross-cutting business rule or invariant (validation, access control, pricing logic,
rate limits, etc.), write:

```
.relic/shared/rules/<RuleName>.md
```

```markdown
# <RuleName>

**Type:** rule
**Inferred from:** <file paths>
**Confidence:** high | medium | low

## Description
The rule in plain language.

## Enforcement
Where this rule is enforced in the codebase.

## Exceptions
Any known exceptions or edge cases.

## Owned by
(unowned — assign when a spec takes responsibility)
```

---

## Step 7 — Write assumption artifacts

For each thing that is true today but could change (third-party dependencies, infrastructure
choices, data volume assumptions, external API contracts), write:

```
.relic/shared/assumptions/<AssumptionName>.md
```

```markdown
# <AssumptionName>

**Type:** assumption
**Inferred from:** <file paths>
**Confidence:** high | medium | low

## Description
What we are assuming to be true.

## Risk if wrong
What breaks if this assumption changes.

## Staleness signal
How you would know this assumption is no longer valid.

## Owned by
(unowned — assign when a spec takes responsibility)
```

---

## Step 8 — Register artifacts in toon manifests

For every artifact you created, register it in the appropriate manifest using `relic write`:

```bash
# For a domain artifact:
relic write --knowledge-domains --payload '{"name":"<ArtifactName>","file":"<ArtifactName>.md","description":"<one-sentence tldr>","tags":["<tag1>","<tag2>","<tag3>"]}'

# For a contract artifact:
relic write --knowledge-contracts --payload '{"name":"<ArtifactName>","file":"<ArtifactName>.md","description":"<one-sentence tldr>","tags":["<tag1>","<tag2>","<tag3>"]}'

# For a rule artifact:
relic write --knowledge-rules --payload '{"name":"<ArtifactName>","file":"<ArtifactName>.md","description":"<one-sentence tldr>","tags":["<tag1>","<tag2>","<tag3>"]}'

# For an assumption artifact:
relic write --knowledge-assumptions --payload '{"name":"<ArtifactName>","file":"<ArtifactName>.md","description":"<one-sentence tldr>","tags":["<tag1>","<tag2>","<tag3>"]}'
```

Do not open or edit any `manifest.toon` file directly.

Each payload must have:
- `name` — matches the `# Heading` in the artifact file
- `file` — filename only (e.g. `UserAuth.md`)
- `tags` — 4–8 lowercase keywords
- `description` — one sentence summary (becomes `tldr` in the index)

Run `relic validate` after this step — it will warn if any `.md` files are unregistered.

---

## Step 9 — Changelog

`scan` creates new artifacts — it does not amend existing ones. Do not write a changelog entry.

---

## Guardrails

- Do NOT modify artifacts that already have an owner set.
- Mark confidence as `medium` for inferences — a human should review before a spec claims ownership.
- Do NOT generate a spec or plan — this command is discovery only.
- Do NOT write anything inside `specs/` — all output goes into `shared/`.
- If you are unsure whether something is a domain vs a contract, prefer domain for entities and
  contract for interfaces/APIs.

---

## After scan completes

Tell the user:
- How many artifacts were generated (domains / contracts / rules / assumptions)
- Which ones have low confidence and need human review

## Next: Generate the Constitution

Run `/relic.constitution` to complete the bootstrap.

- `/relic.scan` → **WHAT** the system does (domains, contracts, rules, assumptions)
- `/relic.constitution` → **HOW** the team works (coding standards, tech stack, architecture)

Once both are done, run `/relic.specify` to create the first feature spec — it will start
with a fully populated brain and a set of hard rules governing every decision.
