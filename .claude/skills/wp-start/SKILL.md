---
name: wp-start
description: Use when starting work on a Sprich! work package (GitHub issue), when the user asks what to work on next, or says "let's start", "pick up", "begin" or names an issue number to work on.
argument-hint: "[#<issue>]"
---

# Starting a work package

Pick up one WP, settle mode and spec needs with the author, write a mode-shaped plan, get it approved, and start. **No code is written in this skill** — implementation begins after the plan is approved.

**Read first:** `docs/workflow.md` (modes, conventions), `docs/plans/README.md` (plan format).

## 1. Choose the WP

No argument:

```bash
gh issue list --label wp --state open --json number,title,milestone,body
```

Take the earliest open milestone. A WP is **unblocked** when every `Depends on #N` in its body is closed (`gh issue view N --json state`). Show unblocked WPs and suggest one. Wait for the author to pick.

## 2. Load context

- `gh issue view <n> --comments`
- Every spec it links to — read the referenced sections and requirement IDs, plus the related parts of `specs/product.md` (principles, decisions).
- Check: all dependencies closed; `git status` clean; `master` up to date (`git switch master && git pull`). Report problems instead of working around them.
- Check the spec's **open questions**. Any that block this WP → resolve with the author and update the spec via `/spec` before planning.

## 3. Ask (one question at a time; skip what's already known)

1. **Mode** — "Guide, Pair or Delegate?" Suggest one with a reason (Guide is the default; Delegate fits boilerplate the author doesn't want to learn). If they already said, don't ask.
2. **Learning focus** *(Guide and Pair only)* — "What do you already know about ⟨core concept of this WP⟩, and what do you want to get out of it?" Shapes the explanations and doc links in the plan.
3. **Spec** — propose whether this WP needs a feature spec (new or update), using the rule in `specs/README.md`, with your reason. Confirm with the author. If yes: create the branch (step 4), then run the `/spec` flow and get the spec approved **before** the plan.

## 4. Branch and claim

```bash
git switch -c wp/<n>-<slug>
gh issue edit <n> --add-assignee @me
```

## 5. Write the plan

`docs/plans/<n>-<slug>.md`, in the format from `docs/plans/README.md`. Required: header (issue, spec + requirement IDs, mode, status `Planned`, branch), goal, approach + why + one rejected alternative, ordered task list with dependencies, **owner per task**, acceptance criteria → verification table, watch-outs, mode log.

Shape it by mode:

| Mode | Tasks look like | Owner | Also include |
|---|---|---|---|
| **Guide** | Small learning steps: what to build, why, which docs to read, pitfalls, a "done when" check per step | You (author) for all code; Claude explains and reviews | Learning notes with official doc links, tied to what the author said they want to learn |
| **Pair** | Pairs of steps: Claude writes interface + failing tests → author implements until green | Split per step | Which test proves each step; learning notes |
| **Delegate** | Implementation tasks, test-first | Claude | Walkthrough outline by spec clause (filled in at the end) |

In **Delegate** you may use superpowers `writing-plans` to produce the task breakdown — render the result into this format and location, not `docs/superpowers/`.

Every task traces to requirement IDs. Flag anything not required by the spec as scope creep; keep the simplest approach that meets the spec.

## 6. Review gate

Show the plan. Wait for approval. Then:

```bash
git add docs/plans/<n>-<slug>.md
git commit -m "docs(plan): <WP title> (#<n>)"
```

Set plan status to `In progress` when work begins.

## 7. Start task 1 in the chosen mode

- **Guide** — explain task 1: the concept, the approach, the docs to read. Then hand over: the author writes the code. Do not write implementation code unless they ask for a specific piece. Don't use superpowers skills that write code or drive execution.
- **Pair** — write the interface and failing tests for task 1, explain them, then hand over the implementation. Same superpowers restriction as Guide.
- **Delegate** — implement. Superpowers `executing-plans`, `subagent-driven-development`, `test-driven-development` are allowed. Keep the plan's checkboxes current.

If the author switches mode later: update the plan header and add a line to the mode log.

## Common mistakes

- Planning before blocking open questions are answered — the plan will be rebuilt.
- Writing code in Guide mode "to save time" — the author is here to learn; explain instead.
- A plan that restates the spec — link requirement IDs, don't copy text.
- Starting a WP whose dependency is still open.
