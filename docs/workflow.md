# How we work

Sprich! is built **spec-first**, in **work packages** tracked as GitHub issues, through **PRs**, with Claude Code in one of three **working modes**. The author builds it to learn, so the default mode is **Guide**.

## Lifecycle

```
Intent ──▶ Spec ──▶ Work packages ──▶ Start + Plan ──▶ Implement ──▶ Review ──▶ Validate + PR ──▶ Merge
(brief,    /spec    /wp-create         /wp-start        (by mode)     /wp-review  /wp-finish
 idea,      │        │                  │                              │           │
 bug)       ▼        ▼                  ▼                              ▼           ▼
          specs/   GitHub issues     docs/plans/                     fixes,      PR: AC evidence,
          *.md     (label: wp)       <n>-<slug>.md                   ticked      traceability
                                     + branch                        tasks
```

Every arrow is a **review gate**: nothing moves on until the author has checked it.

1. **Intent** — what we want, in plain words (the brief, an idea, a bug report).
2. **Spec** — intent turned into a written contract: EARS requirements, out of scope, acceptance criteria, open questions. See [specs/README.md](../specs/README.md).
3. **Work packages** — the spec cut into issues, ordered by dependency.
4. **Plan** — one WP broken into ordered tasks, shaped by the working mode, reviewed before any code.
5. **Implement** — against the spec, in the chosen mode.
6. **Validate** — each acceptance criterion checked against real evidence (tests, manual check), not "it runs".
7. **Review** — every meaningful change in the diff traces to a spec clause. If it doesn't, either the code has scope creep or the spec is incomplete — fix whichever is wrong.

## Where things live

| Artifact | Location | Notes |
|---|---|---|
| Umbrella spec | [specs/product.md](../specs/product.md) | Product-wide source of truth |
| Feature specs | `specs/<feature>.md` | One living file per capability |
| Work packages | GitHub issues, label `wp`, grouped by milestone | Link to spec + requirement IDs |
| Plans | `docs/plans/<issue>-<slug>.md` | Committed on the WP branch; survives across sessions; reviewed in the PR |
| Code review | Pull requests | Body from [.github/pull_request_template.md](../.github/pull_request_template.md) |
| Rules for Claude Code | [CLAUDE.md](../CLAUDE.md) | Loaded every session |

## Work packages

- **One WP = one issue = one branch = one PR**, sized to one milestone task (ideally a day or less of work).
- **Milestones:** `v0.1 Foundation`, `v0.2 Writing + mistakes`, `v0.3 Deploy & dogfood`, `v0.4 Vocabulary`, `v0.5 Conversation`, `v0.6 Voice`, `v0.7 Adaptive planner`, `v1.0 Public release`.
- **Labels:** `wp` (every work package), `needs-spec` (a feature spec must be written or updated first), `blocked` (waiting on another WP or a decision).
- **Dependencies:** `Depends on #N` lines in the issue body. A WP starts only when all of them are closed.
- **Body:** from [.github/ISSUE_TEMPLATE/work-package.md](../.github/ISSUE_TEMPLATE/work-package.md) — goal, spec links with requirement IDs, acceptance criteria, dependencies, suggested mode, size.
- Issues are written for a reader who has the repo open: link to spec sections, don't paste them.

## Working modes

The author switches modes explicitly ("switch to Pair"). The mode is chosen at `/wp-start` and recorded in the plan header; switching updates the header.

### Guide (default)

- Claude explains the approach and trade-offs, points to the relevant docs, and answers questions.
- Claude does **not** write implementation code unless asked for a specific piece.
- **Reviewing the author's code:** Claude fixes **small** mistakes in place (typo, wrong import, off-by-one, missing `await`) and explains each one. **Design-level** mistakes (wrong layer, wrong data model, missing case, wrong abstraction) are explained with the reasoning; the author redoes them.

### Pair

- Claude writes scaffolding, interfaces (signatures, Pydantic models, protocols) and tests — tests first, failing.
- The author writes the implementation that makes the tests pass.
- Reviews of the author's code follow the Guide rule.

### Delegate

- Claude implements the whole WP end to end: code, tests, migrations, catalogs.
- Then walks the author through it **by spec clause**: what each requirement became, key decisions, what to read first.

### In every mode

- Claude may draft specs, plans and docs.
- Keep each change to **one WP**.
- Explain **why**, not just what.
- **Flag over-engineering** — the simplest thing that meets the spec wins.
- Never commit secrets or private prompts.

## Using superpowers (if installed)

The project skills own the flow, the gates and the **output formats**. Superpowers is a tool used inside them:

| Situation | Superpowers use | Output goes to |
|---|---|---|
| Writing or updating a spec (any mode) | `brainstorming` may drive the interview and design | `specs/<feature>.md` in our template — never `docs/superpowers/` |
| **Delegate** mode | Any skill when useful: `writing-plans`, `executing-plans`, `subagent-driven-development`, `test-driven-development`, `systematic-debugging`, `verification-before-completion` | Plans in `docs/plans/<n>-<slug>.md` in our format |
| **Guide** / **Pair** mode | No skill that writes implementation code or drives execution (`executing-plans`, `subagent-driven-development`, TDD as implementer) | — |

Whatever tool produced it, a spec looks like [specs/_template.md](../specs/_template.md) and a plan looks like the format in [docs/plans/README.md](plans/README.md).

## Skills

| Command | Use it to | Who triggers it |
|---|---|---|
| `/spec` | Create or update a spec (umbrella or feature) | You or Claude |
| `/wp-create` | Cut a spec or milestone into WP issues | You only — posts to GitHub |
| `/wp-start` | Pick up a WP: choose mode, settle spec needs, plan, branch | You |
| `/wp-review` | Review the branch so far against plan and spec | You or Claude |
| `/wp-finish` | Validate acceptance criteria, check traceability, open the PR | You only — posts to GitHub |

Skills live in [.claude/skills/](../.claude/skills/).

## Git & PRs

- Never commit to `master`. Branch per WP: `wp/<issue>-<slug>`. Non-WP work: `chore/<slug>`, `docs/<slug>`, `spec/<slug>`.
- **Conventional Commits** with the issue number: `feat(onboarding): ask interface language first (#12)`.
- PR body from the template, with `Closes #N`. Every acceptance criterion has evidence; every change traces to a requirement.
- **Squash merge.** Delete the branch after merge.

## Privacy (public repo)

Issues, PRs, specs, plans and commits are **all public**. Never put any of these in them:

- secrets, tokens, `.env` values, connection strings;
- anything from `PROMPTS_DIR` (tuned prompts, curated content);
- real learner data: messages, mistakes, Telegram user IDs, usernames;
- personal details (names, emails, phone numbers, addresses);
- infrastructure URLs, project IDs, dashboards.

Use made-up examples (`user 42`, `"Ich habe gestern in Berlin gearbeitet"`). Skills that post to GitHub scan for these first.

## Localization

- Never hard-code user-facing text. Add a message key to **every** shipped catalog (`en`, `uk`, `ru`, `de`) in the same PR.
- Store language-neutral codes, never translated labels.
- German learning content stays German; explanations use the user's interface language.

See [specs/product.md](../specs/product.md#architecture) for the full principle.
