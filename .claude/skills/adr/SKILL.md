---
name: adr
description: Use when a significant architecture or product decision for Sprich! is being made or revisited - an open question with real alternatives is answered, a provider, library, hosting, data-model or licensing choice is settled, a founding decision D-n is challenged, or the user asks to record a decision or write an ADR.
argument-hint: "[<decision topic> | Q-<n> | supersede <NNNN>]"
---

# Recording a decision (ADR)

Capture *why* a significant choice was made, with the alternatives, in **our** format in `docs/adr/`. **The author decides**; you lay out the options and write it up.

**Read first:** `docs/adr/README.md` (bar, lifecycle, conventions), the index of existing ADRs, and the spec sections or open question involved.

## 1. Is it ADR-worthy?

Apply the bar in `docs/adr/README.md#when-to-write-one`. If it doesn't meet it, say so and record the choice as a spec changelog line or in the commit message instead. Stop there.

If an existing ADR already covers the topic, this is a **supersede**, not a new ADR.

## 2. Lay out the options

- At least two **real** options. Each: one or two sentences, honest pros and cons tied to this project (spec requirements, D-n decisions, architecture principles, cost, the author's learning goals).
- Your recommendation and why.
- Ask the author to decide. One question at a time if something is unclear (constraints, what they already know).

**Guide and Pair mode:** this is a learning moment — explain the trade-offs and link official docs for each option so the author can judge. If superpowers `brainstorming` is available, you may use it to explore options; the output is still the ADR below, never `docs/superpowers/`.

## 3. Write

- Branch: on a WP branch, stay on it. On `master`, create `docs/adr-<slug>` first.
- Number: next free — `ls docs/adr | grep -E '^[0-9]{4}-' | sort | tail -1`, plus one, four digits.
- Copy `docs/adr/_template.md` to `docs/adr/NNNN-<kebab-title>.md`. Title = the decision as a statement. Fill every section. Status `Proposed`.

**Supersede:** the new ADR sets `Supersedes`; the old ADR's status becomes `Superseded by [ADR-NNNN](NNNN-<title>.md)` — the only edit ever made to an accepted ADR.

## 4. Link both ways

- `docs/adr/README.md` index: add the row (and update a superseded one's status).
- Answered open question → update the spec through the `/spec` rules: resolve the row, move the answer into requirements/background, link the ADR, Changelog line.
- Product-wide decision → add or update its row in `specs/product.md` Key decisions with the ADR link.

## 5. Privacy check

ADRs are public: no secrets, `PROMPTS_DIR` content, real learner data, personal details or infra URLs.

## 6. Review gate

Show the ADR and the linked spec changes. When the author approves, set Status `Accepted` and commit:

```
docs(adr): ADR-NNNN <title> (#<issue>)
```

## Common mistakes

- A straw-man second option — if there was no real alternative, it probably isn't ADR-worthy.
- Editing an accepted ADR's decision instead of superseding it.
- Recording the decision but leaving the spec's open question unresolved.
- Deciding for the author — recommend, then let them choose.
