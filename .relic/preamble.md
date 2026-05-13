# Relic Architectural Invariants

> This document describes how Relic works. It is not project-specific and is not amendable.
> Read it before acting on any other context in this session.

---

## Two-Layer Architecture

Relic separates shared knowledge from spec-local context.

```
.relic/
  shared/                  ← THE ONLY LOCATION for reusable artifacts
    domains/               ← bounded context definitions, entity models
    contracts/             ← API shapes, event schemas, data interfaces
    rules/                 ← cross-cutting business and system rules
    assumptions/           ← declared assumptions about the world
  specs/
    <spec-id>/             ← spec-local context ONLY — four files, nothing else
      spec.md
      plan.md
      tasks.md
      artifacts.json       ← POINTER file — declares relationships, not content
  constitution.md          ← project governance (amendable)
  preamble.md              ← this document (not amendable)
  changelog.md
```

---

## The Invariant

**Artifacts ALWAYS live in `shared/`. They NEVER live inside a spec folder.**

This is not a style preference. It is a structural constraint. Violation breaks
cross-spec awareness and makes intersection detection impossible.

---

## What Belongs Where

### `shared/domains/`
Domain models: bounded contexts, entity definitions, ubiquitous language.
Example: `shared/domains/user.md`, `shared/domains/payment.md`

### `shared/contracts/`
Interface contracts: API shapes, event schemas, data interchange formats.
Example: `shared/contracts/checkout-api.md`, `shared/contracts/order-event.md`

### `shared/rules/`
Business rules and system constraints that govern behaviour across features.
Example: `shared/rules/pricing.md`, `shared/rules/access-control.md`

### `shared/assumptions/`
Declared assumptions about the environment, external systems, or user behaviour.
Example: `shared/assumptions/third-party-auth.md`

### `specs/<spec-id>/`
Exactly five files. No others.
- `spec.md` — feature intent, requirements, user stories
- `plan.md` — implementation decisions
- `tasks.md` — atomic task checklist
- `artifacts.json` — ownership and dependency declarations (pointers, not containers)
- `history.json` — session-local conversation history (gitignored; never read by cross-spec tooling)

**If you are about to create a sixth file inside a spec folder, stop.**
Whatever you are creating belongs in `shared/` instead.

---

## `artifacts.json` Is a Pointer File

`artifacts.json` declares **relationships**, not locations.

```json
{
  "owns": ["shared/contracts/checkout-api.md"],
  "reads": ["shared/domains/user.md"],
  "touches_files": ["src/checkout/handler.ts"]
}
```

- `owns` — this spec is the authoritative owner; only it may modify these artifacts
- `reads` — this spec depends on these artifacts but does not own or modify them
- `touches_files` — source code files this spec's implementation creates or modifies

The paths in `owns` and `reads` **MUST** point into `shared/`. A path pointing inside
`specs/` is invalid and will break intersection detection.

Ownership is a relationship declared in `artifacts.json`.
It does not determine where the artifact lives.
**The artifact always lives in `shared/`.**

---

## The Test

When you are about to create or place a file, ask:

> "Is this a domain model, a contract, a rule, or an assumption?"

**If YES** → it goes in the appropriate `shared/` subfolder.
             Then declare it in the owning spec's `artifacts.json` under `owns`.

**If NO** → it is a source file. It goes in the project source tree.
            Declare it in `touches_files`.

The only exceptions: `spec.md`, `plan.md`, `tasks.md`, `artifacts.json`, and `history.json` go in the spec folder. `history.json` is session-local runtime state and is gitignored — it is never read by cross-spec tooling.

---

## Prohibited Actions

- Creating a contract, domain, rule, or assumption file inside a spec folder
- Listing a `specs/` path in `owns` or `reads` in any `artifacts.json`
- Creating any file inside `specs/<spec-id>/` other than the five listed above
- Modifying a shared artifact whose path is not listed in your spec's `owns` array
- Claiming ownership of an artifact already listed in another spec's `owns` array

---

## Multiple Specs, One Artifact

A shared artifact may be **read** by many specs but **owned** by exactly one.

If you need to use an artifact already owned by another spec:
1. Declare it in your `reads` array — do not duplicate it.
2. Do not modify it — only the owning spec may do so.
3. If it needs to change, flag it in Open Questions and coordinate via `/relic.clarify`
   on the owning spec.

---

## relic search — Mandatory Context Entry Point

**Always run `relic search` before reading any artifact, spec, or fix file.**
Never traverse the filesystem or list directories to find relevant context.

**Command reference:**

| Command | When to use |
|---|---|
| `relic search <keywords>` | Default — scored results from all three index spaces |
| `relic search <keywords> --knowledge` | Shared artifacts only (domains, contracts, rules, assumptions) |
| `relic search <keywords> --spec` | Specs only |
| `relic search <keywords> --fix` | Fix documents only |
| `relic search --deep` | All entries, unfiltered — full triage when you have no keywords |
| `relic search --deep --knowledge` | All knowledge artifacts, unfiltered |
| `relic search --deep --spec` | All specs, unfiltered |
| `relic search --deep --fix` | All fix documents, unfiltered |
| `relic search <keywords> --deep` | Keywords filter even with `--deep` |

**Output format:** Each result is a 6-field toon line:
`<source> | <name> | <path> | <tags> | <tldr> | <score>`
Use `path` to open files directly. Use `score` to decide reading priority.

**Enforcement rules (non-negotiable):**
1. Never traverse the filesystem or list directories to find relevant context.
2. Always run `relic search <topic keywords>` before reading any artifact, spec, or fix file.
3. Use the `path` field from results to open files — never guess or derive paths manually.
4. Use `--deep` only when you have no keywords or need the full index for triage.
5. Use scope flags (`--knowledge`, `--spec`, `--fix`) to narrow when you know the type.
6. `relic search` without keywords AND without `--deep` is an error — do not invoke it that way.

---

## Relic Operational Rules

These rules govern every AI-assisted interaction in every Relic project. They are not
project-specific and are not amendable via the constitution.

**Respect ownership.** Never modify a shared artifact you do not own.
Check `artifacts.json` before editing anything in `.relic/shared/`.

**Flag intersections before acting.** If your implementation touches a file or artifact
that another spec owns, raise it explicitly. Do not proceed until the intersection is resolved.

**Never bypass the plan.** If the plan is wrong or incomplete, update `plan.md` and write
a changelog entry — do not silently deviate from it.

**Changelog is mandatory.** Every plan mutation, contract change, or significant
architectural decision must be written to `.relic/changelog.md`.

**Assumptions must be declared.** If you make an assumption not captured in the spec,
write it to `.relic/shared/assumptions/` before it influences any code.

**The spec is the source of truth.** If code and spec disagree, the spec is right —
unless a `clarify` has explicitly updated the spec to reflect the change.

---

## Manifest Registration

**Every file you create or modify in `shared/<subdir>/` MUST be registered in that
subdirectory's `manifest.json`.** The manifest is a flat JSON array:

```json
[
  {
    "name": "UserAuth",
    "file": "UserAuth.md",
    "tldr": "One sentence description accurate enough to decide whether to read the full file.",
    "tags": ["auth", "session", "token", "login", "user"]
  }
]
```

Rules:
- `name` matches the `# Heading` in the file.
- `file` is the filename only (no path).
- `tldr` is one sentence — precise enough to decide relevance without reading the file.
- `tags` are lowercase keywords, aim for 4–8 per artifact. Include domain terms, entity names, and technical concepts.
- If the manifest does not exist, create it as `[]` and add your entry.
- If you modify an artifact's scope or purpose, update its manifest entry.

`relic validate` will flag missing manifests and unregistered files as errors.
