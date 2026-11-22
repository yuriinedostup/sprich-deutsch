# Spec: ⟨Feature name⟩

| | |
|---|---|
| **Status** | Draft |
| **Kind** | Feature |
| **Area code(s)** | ⟨e.g. WRT⟩ |
| **Refines** | [product.md](product.md) — ⟨umbrella requirement IDs, e.g. WRT-1…5⟩ |
| **Milestone** | ⟨e.g. v0.2⟩ |
| **Work packages** | ⟨#issue, #issue⟩ |
| **Last updated** | ⟨YYYY-MM-DD⟩ |

<!-- Public file. No secrets, prompt text, real learner data, personal details or infra URLs. -->

## Intent

⟨Why this exists, in two or three sentences. What problem for the learner, and what outcome.⟩

## Background

⟨Current state, relevant decisions (D-n) and principles from product.md, constraints. Link, don't repeat.⟩

## Requirements

<!-- EARS. One behaviour per line. Continue the area's numbering (next free ID). See specs/README.md#ears-syntax -->

- **⟨AREA-n⟩** — WHEN ⟨trigger⟩, THE SYSTEM SHALL ⟨response⟩.
- **⟨AREA-n⟩** — IF ⟨condition⟩, THEN THE SYSTEM SHALL ⟨response⟩.

## Out of scope

- ⟨What this spec deliberately does not cover, and where it is tracked instead.⟩

## Interfaces & contracts

<!-- Only the sections that apply. -->

### Bot flow

```
user: /command
bot:  ⟨message key or description⟩ [buttons]
user: ⟨input⟩
bot:  ⟨response⟩
```

### Data model

| Table | Column | Type | Notes |
|---|---|---|---|
| | | | |

### Message keys (i18n)

| Key | `en` meaning | Notes |
|---|---|---|
| | | Present in every shipped locale catalog |

### API / internal interfaces

⟨Function signatures, payload schemas, LLM structured-output models.⟩

## Acceptance criteria

<!-- Checkable. Each maps to requirement IDs. Validated by /wp-finish. -->

- [ ] ⟨Observable outcome⟩ — ⟨AREA-n⟩
- [ ] ⟨Observable outcome⟩ — ⟨AREA-n⟩

## Open questions

| # | Question | Assumption | Blocks |
|---|---|---|---|
| Q-1 | ⟨…⟩ | ⟨…⟩ | ⟨requirement IDs, or no⟩ |

## Changelog

- **⟨YYYY-MM-DD⟩** — Initial draft. ⟨Why.⟩
