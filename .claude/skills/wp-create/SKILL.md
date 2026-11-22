---
name: wp-create
description: Use when an approved Sprich! spec or milestone needs to be broken into work packages as GitHub issues, or when the user asks to create, cut or plan WPs, tickets or issues for a milestone or spec.
argument-hint: "<specs/<file>.md | v0.N>"
disable-model-invocation: true
---

# Creating work packages

Turn a spec (or one milestone of `specs/product.md`) into GitHub issues, one per work package, in dependency order. Nothing is posted until the author approves the breakdown.

**Read first:** `docs/workflow.md` (WP conventions), `specs/README.md`, the target spec, `.github/ISSUE_TEMPLATE/work-package.md`.

## 1. Check readiness

- Target spec exists and its **Status** is `Approved` (or the author explicitly says to proceed with `Draft`).
- List open questions whose **Blocks** column names requirements in scope. Don't stop for them — they are settled at `/wp-start` of the WP that touches them. Note each on the affected WP (`Resolve Q-n at start`). Stop and offer `/spec` only if an answer would change how the work is cut into WPs.
- Check what already exists so you don't duplicate: `gh issue list --label wp --milestone "<milestone>" --state all`.

## 2. Propose the breakdown

Each WP: **one issue = one branch = one PR**, ideally ≤ 1 day. Cut along requirement groups and natural layers (e.g. config → DB → handler), never "frontend half / backend half" of one behaviour.

Show the author a table, ordered so dependencies come first:

| # | Title | Goal | Requirements | Depends on | Needs spec | Mode | Size |
|---|---|---|---|---|---|---|---|

Then the dependency graph as an indented list or mermaid. For each WP:

- **Needs spec** = yes when it adds user-visible behaviour not yet in EARS, changes a data model/contract, or touches a blocking open question (see `specs/README.md`).
- **Mode** = a suggestion only (Guide unless it's pure boilerplate the author doesn't want to learn).
- Acceptance criteria are copied from the spec's criteria for those requirements — reference, don't invent.

Flag anything over-sized or over-engineered. Wait for approval; apply changes and re-show if asked.

## 3. Ensure labels and milestone exist

Idempotent; run after approval:

```bash
gh label create wp --color 1d76db --description "Work package" --force
gh label create needs-spec --color fbca04 --description "Feature spec must be written or updated first" --force
gh label create blocked --color b60205 --description "Waiting on another WP or a decision" --force
gh api repos/{owner}/{repo}/milestones --jq '.[].title'   # create only if missing:
gh api repos/{owner}/{repo}/milestones -f title="v0.1 Foundation"
```

Milestone titles are listed in `docs/workflow.md`.

## 4. Privacy scan

Issues are public. Every body must be free of secrets, `.env` values, `PROMPTS_DIR` content, real learner data, personal details and infra URLs. Fix before posting.

## 5. Create issues

In dependency order, so `Depends on #N` can reference real numbers. Body follows the issue template exactly; link the spec by relative URL `../blob/master/specs/<file>.md#<anchor>` and list requirement IDs.

```bash
gh issue create --title "<title>" --label wp [--label needs-spec] --milestone "<milestone>" --body-file <tmpfile>
```

Write bodies to temp files in the scratchpad, not inline strings. Add `blocked` to WPs whose dependencies are still open only if the author asks — the dependency lines already say it.

## 6. Report

List created issues (`#n — title — depends on`) and suggest the first unblocked one for `/wp-start`.

## Common mistakes

- Posting before approval, or posting a body with a real Telegram ID or chat excerpt.
- WPs that each implement half a requirement — split by behaviour instead.
- Restating the spec inside the issue — link to it; the spec is the source of truth.
