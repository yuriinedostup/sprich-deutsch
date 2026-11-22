# Decision records (ADRs)

An ADR records **why** a significant architecture or product choice was made, which alternatives were weighed, and what follows from it. Specs say *what* the system does; ADRs say *why it is built this way*.

> **ADRs are public.** Same rule as specs and issues: no secrets, prompt text from `PROMPTS_DIR`, real learner data, personal details or infra URLs.

## Index

| ADR | Title | Type | Status |
|---|---|---|---|
| [0001](0001-record-decisions-as-adrs.md) | Record significant decisions as ADRs | Architecture | Accepted |

Add a row for every new ADR; update the status column when one is superseded.

## When to write one

Write an ADR when **any** of these is true:

- the decision is **hard or costly to reverse** (data model, provider, hosting, licensing, a core library);
- **real alternatives** were weighed and the trade-off matters;
- it **affects several modules** or sets a rule others must follow;
- it answers an open question whose answer involved a trade-off.

Don't write one for choices that are easy to undo and had no serious alternative (a helper's name, a small utility library, a log format) — a spec changelog line or a commit message is enough.

**Example:** adding German as a fourth interface language extended D-10 without weighing alternatives and is easy to reverse → spec changelog only. Choosing between aiogram i18n with gettext and Fluent (Q-3) → ADR.

## Lifecycle

`Proposed` → `Accepted` → (`Superseded by ADR-NNNN` | `Deprecated`)

- **Proposed** — drafted, under discussion in a PR.
- **Accepted** — the author approved it; set before the PR merges. Merging = the decision is in force.
- **Superseded** — a later ADR replaced it. The old ADR gets `Superseded by [ADR-NNNN](NNNN-<title>.md)`; the new one lists `Supersedes`.
- **Deprecated** — no longer relevant, nothing replaced it.

An accepted ADR is **immutable** except for its status and links. Changing your mind means writing a new ADR that supersedes it — the history of reasoning stays intact.

## Conventions

- File: `NNNN-<kebab-title>.md`, numbered in order, never renumbered or reused. Start from [_template.md](_template.md), or run `/adr`.
- Title: the decision as a short statement ("Use aiogram i18n with gettext catalogs"), not a question.
- **Type:** `Architecture` (how it's built) or `Product` (what it does, for whom, on what terms).
- At least two real options, each with honest pros and cons.
- Links both ways:
  - ADR → requirement IDs, specs, issues, the open question it answers.
  - Spec → ADR wherever the decision shapes requirements (Background section, resolved open question).
  - An ADR that adds or changes a product-wide decision also adds or updates its row in [product.md → Key decisions](../../specs/product.md#key-decisions).
- Founding decisions D-1…D-11 in the product spec were **not** backfilled (see [ADR-0001](0001-record-decisions-as-adrs.md)). Revisiting one means a new ADR, linked from its row.
