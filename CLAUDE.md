# CLAUDE.md

**Sprich!** — a Telegram bot that coaches German speaking, records every mistake by type, and plans practice around weak areas. Interface in English, Ukrainian, Russian or German; target language German.

- Product spec (source of truth): [specs/product.md](specs/product.md)
- How we work (full rules): [docs/workflow.md](docs/workflow.md)

The author is building this **to learn**. Explain *why*, not just what. Flag over-engineering. Keep every change to **one work package**.

## Status

Pre-v0.1: no application code yet. Next: v0.1 Foundation.

## Commands

<!-- Filled in by the v0.1 work packages: run, test, lint, migrate. -->
None yet.

## Working modes

Default **Guide**. The author switches explicitly ("switch to Pair"); the current WP's mode is in its plan header (`docs/plans/<n>-<slug>.md`).

- **Guide** — explain approach and trade-offs, point to docs. **Do not write implementation code** unless asked for a specific piece. When reviewing the author's code: fix **small** mistakes in place and explain each; explain **design-level** mistakes and let the author redo them.
- **Pair** — write scaffolding, interfaces and failing tests; the author implements.
- **Delegate** — implement end to end (code + tests + migration), then walk the author through it by spec clause.

Specs, plans and docs may be drafted in any mode.

## Spec-driven rules

- Before working on a WP, read the issue and every spec it links to.
- The spec is the source of truth, not chat history. If behaviour must change, **update the spec first** (with a Changelog line), then the code.
- A **blocking open question** in the spec stops implementation of what it blocks. Ask; don't assume.
- Every change must trace to a requirement ID. If it doesn't, it's scope creep or a spec gap — say which.
- Conventions: [specs/README.md](specs/README.md).
- A **significant decision** (hard to reverse, real alternatives, crosses modules) gets an ADR in [docs/adr/](docs/adr/README.md) via `/adr`. Recommend an option; the author decides. Accepted ADRs are never edited — supersede them.

## Skills

| Command | When |
|---|---|
| `/spec` | Create or update a spec |
| `/adr` | Record a significant architecture or product decision |
| `/wp-create` | Cut a spec/milestone into WP issues (user only) |
| `/wp-start` | Pick up a WP: mode, spec needs, plan, branch |
| `/wp-review` | Review the branch against plan and spec |
| `/wp-finish` | Validate acceptance criteria, open the PR (user only) |

## Superpowers (if installed)

Project skills own the flow and the **output formats**: specs go to `specs/` in [the template](specs/_template.md), ADRs to `docs/adr/` in [the ADR template](docs/adr/_template.md), plans to `docs/plans/` in [the plan format](docs/plans/README.md) — never `docs/superpowers/`.

- Writing specs or ADRs (any mode): `brainstorming` may drive the interview; output is a project spec or ADR.
- **Delegate**: any superpowers skill when useful (`writing-plans`, `executing-plans`, `subagent-driven-development`, TDD, debugging, verification).
- **Guide / Pair**: no superpowers skills that write implementation code or drive execution.

## Git & PRs

- Never commit to `master`. Branch `wp/<issue>-<slug>` (else `chore/`, `docs/`, `spec/`).
- Conventional Commits with the issue: `feat(onboarding): ask interface language first (#12)`.
- One WP per PR; body from the PR template with `Closes #N`; squash merge.
- Commit or push only as part of the skills' gates or when asked.

## Privacy & secrets — the repo is public

Issues, PRs, specs, ADRs, plans and commits are all public. Never put in them: secrets or `.env` values, anything from `PROMPTS_DIR` (tuned prompts, curated content), real learner data (messages, mistakes, Telegram IDs), personal details, infra URLs or IDs. Use made-up examples. Secrets live only in env vars; `.env` is never committed.

## Architecture principles

- **Thin bot layer** — handlers translate Telegram into calls to `core/`; `core/` never imports Telegram/aiogram.
- **Partner ≠ analyzer** — the reply goes out first; a separate analyzer extracts structured mistakes in the background.
- **Mistakes are events** — append-only `mistakes` → rolled up into `skill_mastery` → used by the planner.
- **LLM calls** go through one wrapper with Pydantic structured output; provider SDK called directly in LangGraph nodes, no LangChain.
- **Localization** — interface language per user (`en`, `uk`, `ru`, `de`). No hard-coded user-facing text: every key in **all** catalogs in the same PR. DB stores codes, never translated labels. German content stays German.

Target layout and tables: [specs/product.md → Architecture](specs/product.md#architecture).
