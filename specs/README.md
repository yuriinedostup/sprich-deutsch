# Specs

Specs are the written contract between intent and code. Claude Code and humans implement **against** them, and review diffs **against** them. They live in the repo and change through PRs, like code.

> **Specs are public.** Same rule as issues: no secrets, no prompt text from `PROMPTS_DIR`, no real learner data, no personal details, no infra URLs or IDs.

## Index

| Spec | Kind | Status | Covers |
|---|---|---|---|
| [product.md](product.md) | Umbrella | Draft | Whole product: intent, decisions, architecture, requirements v0.1–v0.3, milestones |

Add a row whenever a feature spec is created or changes status.

## Two kinds of spec

- **Umbrella** — [product.md](product.md). Product-wide intent, decisions, architecture principles, requirements, milestones.
- **Feature spec** — `specs/<feature>.md`, one **living** file per capability (e.g. `onboarding.md`, `writing-practice.md`). Refines umbrella requirements into precise behaviour, contracts and acceptance criteria. A follow-up change to a capability **updates the same file** instead of starting a new one, so the history of intent stays in one place.

Start a feature spec from [_template.md](_template.md), or run `/spec`.

## Status

`Draft` → `Approved` → `Implemented` (→ `Superseded`, with a link to what replaced it).

- **Draft** — being written; open questions allowed.
- **Approved** — reviewed and merged; implementation may start (if no blocking open questions).
- **Implemented** — every requirement in it is shipped and verified.

## Requirement IDs

- Format `AREA-n`, e.g. `ONB-3`. Area codes are listed in [product.md](product.md#requirements).
- IDs are **stable** and **never reused**. A removed requirement is struck through with a reason, not renumbered: `- ~~**WRT-3**~~ — removed 2026-12-01: merged into WRT-2.`
- A feature spec continues its area's numbering from the umbrella (next free number). It may refine an umbrella requirement by repeating its ID with more detail.
- Issues, plans, tests, commit messages and PRs reference requirements by ID.

## EARS syntax

Write requirements so they are unambiguous and testable:

| Pattern | Template | Example |
|---|---|---|
| Ubiquitous | THE SYSTEM SHALL ⟨response⟩ | THE SYSTEM SHALL store one interface language per user. |
| Event | WHEN ⟨trigger⟩, THE SYSTEM SHALL ⟨response⟩ | WHEN the user sends `/stats`, THE SYSTEM SHALL show mistakes per category. |
| State | WHILE ⟨state⟩, THE SYSTEM SHALL ⟨response⟩ | WHILE a writing task is open, THE SYSTEM SHALL treat the next text message as the answer. |
| Unwanted | IF ⟨condition⟩, THEN THE SYSTEM SHALL ⟨response⟩ | IF analysis fails, THEN THE SYSTEM SHALL log it without notifying the user. |
| Optional | WHERE ⟨feature/config⟩, THE SYSTEM SHALL ⟨response⟩ | WHERE `BOT_MODE=webhook`, THE SYSTEM SHALL … |

One behaviour per requirement. If you need "and" to describe two behaviours, write two requirements.

## Open questions

Every open question has a **working assumption** and a **Blocks** value: the requirement IDs that can't be implemented until it is answered (a milestone if no IDs exist yet), or `no` if the assumption can stand. A blocking question must be answered — and the spec updated — **before** implementation of what it blocks starts. That is where the most expensive rework comes from.

## Change discipline

- **Update the spec before the code.** If requirements change mid-work, edit the spec first, then re-run implementation against it. Chat history is not a source of truth.
- **Log why.** Every change adds a line to the spec's Changelog: date, what changed, why.
- **Don't spec while exploring.** If the direction is still unclear, explore first (a throwaway spike), then write the spec once it is.

## Specs and decisions (ADRs)

A spec says **what** the system does. An [ADR](../docs/adr/README.md) says **why** it is built a certain way, with the alternatives that were rejected. When answering an open question involves a real trade-off, record the choice as an ADR (`/adr`), then resolve the question in the spec and link the ADR. Easy-to-reverse choices only need a Changelog line.

## When a work package needs its own spec

A WP needs a feature spec (or an update to an existing one) when **any** of these is true:

- it adds or changes user-visible behaviour that is not already written as EARS requirements;
- it introduces or changes a data model, an API or another contract;
- it hits a blocking open question.

Infrastructure, chore and refactor WPs usually don't — the umbrella requirements and acceptance criteria are enough. If a feature spec for the area already exists, **update it** rather than creating a new one.
