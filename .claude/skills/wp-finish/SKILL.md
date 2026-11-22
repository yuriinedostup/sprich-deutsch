---
name: wp-finish
description: Use when a Sprich! work package is believed complete and needs validation and a pull request - the user says "finish", "ship", "open the PR", "done with this WP".
argument-hint: "[#<issue>]"
disable-model-invocation: true
---

# Finishing a work package

Prove the WP meets its spec, make the spec match reality, then open the PR. **Evidence before claims**: nothing is "done" until a command or check shows it.

**Read first:** the plan `docs/plans/<n>-<slug>.md`, its spec(s), `.github/pull_request_template.md`, `CLAUDE.md` (lint and test commands).

## 1. Verify

Run the lint and test commands listed in `CLAUDE.md`, fresh, and read the output. (If superpowers `verification-before-completion` is available, follow it.) No commands defined yet → say so in the PR and rely on the manual checks below.

Failures → stop. Report them; in Guide/Pair, the author fixes their code (small fixes per `/wp-review` rules).

## 2. Acceptance criteria

For each criterion in the plan's verification table, record the evidence: test name and result, command output, or a manual check the author confirms. **Any criterion without evidence → stop** and say which.

## 3. Traceability and review

Run the `/wp-review` checks on the whole branch diff (`git diff master...HEAD`). Build the table: requirement ID → files / tests. Any change without a requirement → remove it, or update the spec (step 4). Any significant decision made during the WP without an ADR → write it with `/adr` before the PR. ADRs on this branch must be `Accepted` before the PR is opened.

## 4. Sync the spec

If the implementation differs from the spec in any way, **ask the author** which is right:
- Code is right → update the spec via `/spec` (Changelog line: what changed, why).
- Spec is right → fix the code first.

When every requirement of a spec is shipped and verified, set its **Status: Implemented** and update `specs/README.md`.

## 5. Close the plan

Plan status → `Done`; all tasks ticked. **Delegate mode:** fill the walkthrough — each requirement → what was built, key decisions, where to start reading.

## 6. Privacy scan

Scan the full diff and the PR body for secrets, `.env` values, `PROMPTS_DIR` content, real learner data, Telegram IDs, personal details, infra URLs. Fix before pushing.

## 7. Open the PR

Write the body from `.github/pull_request_template.md`: `Closes #<n>`, summary, mode, spec + requirement IDs, ADRs added or superseded, acceptance criteria with evidence, traceability table, spec changes, how to test, checklist. Show it to the author and wait for the go-ahead. Then:

```bash
git push -u origin HEAD
gh pr create --base master --title "<type>(<scope>): <WP title> (#<n>)" --body-file <tmpfile>
```

Title follows Conventional Commits (PR is squash-merged into one commit). Write the body to a temp file in the scratchpad. Report the PR URL.

## Common mistakes

- "Tests pass" without having run them in this session.
- Ticking a criterion because the code "should" do it — evidence or it's not done.
- Quietly changing the spec to match the code without asking — the author decides which is right.
- Pushing before the author has seen the PR body.
