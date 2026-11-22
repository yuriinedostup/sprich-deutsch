---
name: wp-review
description: Use when the user asks for a review of their code or of the current Sprich! work-package branch, says "check my code", "review this", "is this right", finishes a plan task, or before running /wp-finish.
argument-hint: "[task number | path]"
---

# Reviewing work on a WP branch

Review the branch against **its plan and spec**, not just for bugs. Every change should trace to a requirement; everything required should be there.

**Read first:** the plan `docs/plans/<n>-<slug>.md` (branch name gives `<n>`), the spec sections it links, `CLAUDE.md` architecture principles.

## 1. Collect the change

```bash
git diff master...HEAD     # committed on the branch
git diff HEAD              # uncommitted
git status --short         # untracked files
```

Argument given → focus on that plan task or path, but still note anything else that's wrong.

## 2. Check, in this order

1. **Correctness** — bugs, unhandled errors, async mistakes, wrong SQL/ORM use, failing edge cases from the spec.
2. **Traceability** — each change maps to a requirement ID or plan task. Untraceable change → scope creep (remove it) or a spec gap (`/spec` update). Required behaviour with no code → missing.
3. **Architecture** — `core/` imports no Telegram/aiogram code; handlers stay thin; analyzer runs after the reply; mistakes are append-only; DB stores codes, not translated labels.
4. **Localization** — no hard-coded user-facing text; every new key exists in every shipped catalog (`en`, `uk`, `ru`, `de`).
5. **Tests** — each acceptance criterion touched has a test or a stated manual check.
6. **Over-engineering** — abstractions, options or layers the spec doesn't need.
7. **Decisions** — code that commits to a significant choice (new dependency, provider, data model, cross-module pattern) with no ADR → flag it and suggest `/adr`. Code that contradicts an accepted ADR → design-level finding.
8. **Privacy** — no secrets, `.env` values, prompt text, real learner data or Telegram IDs in code, tests, logs or comments.

## 3. Report and fix — by who wrote the code

**Code the author wrote (Guide, Pair):**

| Finding | Action |
|---|---|
| **Small** — typo, wrong import, off-by-one, missing `await`, wrong variable | Fix it in place, then explain each fix: what was wrong, why |
| **Design-level** — wrong layer, wrong data model, missing case, wrong abstraction, untraceable feature | Do **not** fix. Explain the problem, why it matters, point to the principle/requirement/docs. The author redoes it |

**Code Claude wrote (Delegate, Pair scaffolding):** fix everything found, then summarize what changed.

Report as a short list, most important first: `file:line — finding — why — fixed / your turn`. End with what's good, briefly — what the author should keep doing.

## 4. Update the plan

Tick tasks that are now complete and verified. Note any new task the review uncovered (with its requirement ID).

For a deeper bug hunt on a large diff, suggest the built-in `/code-review`.

## Common mistakes

- Rewriting the author's design-level mistake yourself — they learn by redoing it.
- Reviewing only style while a requirement is unimplemented — traceability comes before polish.
- Praise padding. One line of what's good is enough.
