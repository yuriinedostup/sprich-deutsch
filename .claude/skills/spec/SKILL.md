---
name: spec
description: Use when a Sprich! spec must be created or changed - a new feature or work package needs a feature spec, requirements changed mid-work, an open question was answered, implementation diverged from the spec, or the user asks to write, update or review a spec in specs/.
argument-hint: "[specs/<file>.md | #<issue> | new <topic>]"
---

# Writing and updating specs

A spec is the contract between intent and code. Your job is to turn intent into a spec the author has reviewed, in **our** format, in `specs/`. You write spec files only — no code.

**Read first:** `specs/README.md` (conventions), `specs/product.md` (umbrella), any spec the target links to, and the issue if one was given (`gh issue view <n> --comments`).

## 1. Find the target

| Argument | Target |
|---|---|
| `specs/<file>.md` | Update that spec |
| `#<issue>` | The spec the issue links to; if the issue needs a new one, a new `specs/<feature>.md` |
| `new <topic>` | New `specs/<feature>.md` — first check the index in `specs/README.md`: if a spec already covers the area, **update it instead** |
| none | Ask what the spec is for |

Branch: if on `master`, create `spec/<topic>` first. On a WP branch, stay on it.

## 2. Interview

Ask one question at a time, until you can write requirements without guessing:

- Why does the learner need this? What does success look like?
- Challenge assumptions: is there a simpler version? What breaks if we skip it?
- Edge cases: wrong input, repeated commands, failures, other interface languages (`en`/`uk`/`ru`/`de`).
- What is explicitly **not** part of this?

Skip what the umbrella spec or issue already answers. If superpowers `brainstorming` is available, you may use it to run this interview — but stop before its design-doc step: the output is the spec below, written to `specs/`, never to `docs/superpowers/`.

If the direction is still unclear after the interview, say so and suggest a throwaway spike before writing the spec.

## 3. Write

New spec: copy `specs/_template.md`. Fill every section:

- **Requirements** in EARS, one behaviour each. IDs continue the area's numbering — find the highest used ID with `grep -rhoE '\*\*<AREA>-[0-9]+' specs/` and add one.
- **Out of scope** — what you heard is *not* included.
- **Acceptance criteria** — checkable, each mapped to requirement IDs.
- **Open questions** — each with a working assumption and a **Blocks** value (requirement IDs it blocks, or `no`). Never silently assume: an unanswered question goes here.
- **Message keys** — for any new user-facing text (every shipped catalog — `en`, `uk`, `ru`, `de` — will need them).
- **Changelog** — first line: date, "Initial draft", why.

Updating a spec:

- Never renumber. Removed requirement → strike through with date and reason.
- Add a Changelog line: date, what changed, **why**.
- Answered open question → move the answer into requirements/background, delete or mark the row resolved. If the answer weighed real alternatives, record it with `/adr` first and link the ADR.
- Umbrella touched? Keep `specs/product.md` consistent too.

Update the index table in `specs/README.md` for a new spec or a status change.

## 4. Privacy check

Specs are public. Before showing the draft, check it contains no secrets, prompt text from `PROMPTS_DIR`, real learner data, personal details or infra URLs. Use made-up examples.

## 5. Review gate

Show the author the spec (or the diff for an update) and list:
- assumptions you made,
- open questions that block requirements (Blocks ≠ `no`).

Wait for approval. Then set **Status: Approved** (if the author agrees it's ready), commit:

```
docs(spec): <what> (#<issue>)
```

Blocking open questions left? Status stays **Draft** and implementation of what they block must not start — tell the author which ones.

## Common mistakes

- Writing two behaviours in one requirement ("… and …") — split them.
- Vague criteria ("works well") — make them observable ("`/stats` lists categories highest first").
- Creating `specs/writing-v2.md` instead of updating `specs/writing-practice.md`.
- Editing the spec silently after code diverged — always add the Changelog line.
