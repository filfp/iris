# /relic.constitution

Extract project-specific coding principles, tech standards, and architecture decisions
into `.relic/constitution.md`. Run this **once** when adopting Relic — after `/relic.scan`
if the codebase exists, or standalone for new projects.

> The constitution is the HOW of your project. `/relic.scan` captures the WHAT (domains,
> contracts, rules). Together they give the AI complete context before every interaction.

---

## Step 0 — Detect mode

```bash
relic scan --json
```

- If `total_files > 10` and `tech_stack` is non-empty → **Mode A** (existing codebase)
- Otherwise → **Mode B** (new project — ask the user)

---

## Mode A — Existing codebase

### Step 1 — Read configuration files

Read these files if they exist (use the manifest's `key_files` to locate them):

- `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` — dependencies and scripts
- `tsconfig.json` — TypeScript strictness settings
- `.eslintrc*` / `eslint.config.*` — lint rules (reveals what's forbidden)
- `.prettierrc*` — formatting conventions
- `jest.config.*` / `vitest.config.*` — test framework and coverage thresholds
- `Dockerfile` / `docker-compose.yml` — runtime environment

These files contain **explicit** decisions with high confidence.

### Step 2 — Sample test files (5–10 files)

Read a cross-section of test files from the codebase. Look for:
- Testing approach: TDD? write-tests-after? mostly E2E?
- Mocking strategy: heavy mocks? integration-first? no mocks?
- What constitutes a "unit" in this codebase?
- Test naming conventions

### Step 3 — Read architecture files

From the manifest's `key_files`, read `entry_point`, `routes`, `services`, `middleware` files.
Look for:
- Layering: is there a service layer? repository pattern? controller-service-repository?
- Error handling: how are errors propagated?
- Naming conventions: camelCase? snake_case? PascalCase for what?
- Async patterns: callbacks? promises? async/await throughout?

### Step 4 — Read documentation

Read if they exist:
- `README.md` — often captures high-level decisions and required setup
- `CONTRIBUTING.md` — explicit workflow rules (treat as high confidence)
- Any `docs/adr/` or `docs/decisions/` folder — Architecture Decision Records

### Step 5 — Extract principles

From what you read, identify **3 to 7** project-specific principles.

A good principle is:
- **Verifiable** — an AI can tell if code violates it
- **Specific** — "use TypeScript strict mode" not "write good code"
- **Non-trivial** — not already covered by preamble.md's operational rules

Mark each principle with its evidence:
- `(explicit)` — found in documentation or config
- `(inferred)` — deduced from code patterns
- `(assumed)` — no clear evidence; needs confirmation

### Step 6 — Fill and write `.relic/constitution.md`

Replace all placeholder sections in the template with your extracted content.

Format each principle as:

```markdown
### I. Principle Name
[One sentence: the rule]
[One sentence: why it matters or what it prevents]
Source: tsconfig.json strict: true, enforced by eslint no-explicit-any (explicit)
```

Set `**Ratified**: [today's date]`.

### Step 7 — Report to user

Show:
- Each principle with its source and confidence
- Areas where evidence was weak or contradictory
- Anything that needs human decision (competing patterns, unclear conventions)

End with: *"Are these principles correct? Anything to add, remove, or correct before we start using this as a hard constraint?"*

---

## Mode B — New project

Ask the user for:

1. **Project name** — what should the constitution header say?
2. **Language and framework** — e.g. "TypeScript + Next.js" or "Python + FastAPI"
3. **Testing approach** — TDD? write tests after? what framework?
4. **Architecture** — any key decisions already made? (monorepo? event-driven? microservices?)
5. **Any non-negotiables** — things that must never happen in this codebase

Fill the constitution template with their answers. Use placeholder comments for anything
not yet decided. Set today's date as the ratification date.

---

## Guardrails

- Write **principles**, not code — the constitution describes rules, not implementations
- Do not copy code snippets into the constitution
- Each principle must be independently checkable — vague rules help no one
- Mark confidence level on every extracted principle
- Do not generate a spec or plan — this is governance only
- Do not modify `.relic/preamble.md` — that document is not amendable

---

## After constitution is written

Tell the user:
- How many principles were extracted (or defined)
- Which ones are `(inferred)` and need confirmation
- Recommended next step:
  - If scan has not been run: `/relic.scan` to populate the shared artifact brain
  - If scan is done: `/relic.specify` to create the first feature spec
