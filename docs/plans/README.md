# Plans

One plan per work package, written by `/wp-start`, named `<issue>-<slug>.md` (e.g. `12-onboarding-language.md`). It is committed on the WP branch as the first commit, so it survives across sessions and is reviewed in the PR.

Plans are public — same privacy rule as specs and issues.

## Format

```markdown
# Plan: <WP title> (#<issue>)

| | |
|---|---|
| **Issue** | #<issue> |
| **Spec** | specs/<file>.md — <requirement IDs> |
| **Mode** | Guide | Pair | Delegate |
| **Status** | Planned | In progress | Done |
| **Branch** | wp/<issue>-<slug> |

## Goal
One or two sentences: what is true when this WP is done.

## Approach
What we'll do and why. One rejected alternative and why it lost.

## Tasks
Ordered; each task lists what it depends on and who does it.
- [ ] 1. <task> — owner: <you|Claude> — reqs: <IDs>
- [ ] 2. <task> (after 1) — owner: … — reqs: …

## Acceptance criteria → verification
| Criterion | Requirement | How we verify |
|---|---|---|
| <criterion> | <ID> | <test name / manual check> |

## Watch out for
Risks, pitfalls, over-engineering to avoid.

## Learning notes  <!-- Guide and Pair -->
What you wanted to learn; concepts per task; links to official docs.

## Mode log
- <YYYY-MM-DD> — started in <mode>.
```

### Owner per task by mode

| Mode | Claude | You |
|---|---|---|
| Guide | explanations, doc pointers, reviews | all code |
| Pair | scaffolding, interfaces, failing tests | implementation |
| Delegate | everything; then a walkthrough | review |
