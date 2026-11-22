# ADR-0001: Record significant decisions as ADRs

| | |
|---|---|
| **Status** | Accepted |
| **Type** | Architecture |
| **Date** | 2026-11-22 |
| **Answers** | — |
| **Related** | [specs/README.md](../../specs/README.md), [docs/workflow.md](../workflow.md), [product.md → Key decisions](../../specs/product.md#key-decisions) |
| **Supersedes** | — |

## Context

Sprich! is built spec-first. Specs capture *what* the system does as testable requirements, but not *why* one approach was chosen over another. Several open questions in the product spec (LLM provider, i18n library, who may use the bot) will be settled by weighing alternatives. Without a record, the answer becomes a one-line spec changelog entry and the rejected options and their trade-offs are lost.

The project is also a learning project: understanding the trade-offs is as valuable as the code. And it's a public, AGPL repo where future readers (and Claude Code sessions) need the reasoning without access to past chats.

The product spec already holds a table of founding decisions D-1…D-11 from the project brief, with short rationales that were inferred rather than recorded.

## Options considered

### No records — spec changelog only

- **+** Nothing new to maintain.
- **−** Alternatives and trade-offs are lost; "why" lives in chat history.

### Decisions table in the product spec only

- **+** One place, already exists.
- **−** One-line rationales can't hold options, pros and cons; editing rows in place erases the earlier reasoning.

### ADRs in `docs/adr/`, product-spec table as the index of product-wide decisions

- **+** Each decision keeps its context, options and consequences; immutable once accepted, so history survives changes of mind.
- **+** Reviewable in PRs like code; links to requirement IDs keep it traceable.
- **−** More documents to write — mitigated by a clear bar for when an ADR is needed.

## Decision

We will record significant architecture and product decisions as ADRs in `docs/adr/`, using a short template (context, options with pros and cons, decision, consequences), because it keeps the reasoning and the rejected alternatives next to the code that depends on them.

**Forward-only:** founding decisions D-1…D-11 are not backfilled. Their rationales stay as one-liners in the product spec; revisiting one of them means writing a new ADR and linking it from its row.

## Consequences

- **Easier:** reviewing code against the reasoning; onboarding a future reader or Claude Code session; changing a decision without losing why it was made.
- **Harder:** answering an open question now sometimes means writing an ADR, not just editing the spec.
- **Follow-ups:** `/adr` skill; `/spec`, `/wp-start`, `/wp-review` and `/wp-finish` prompt for an ADR when a significant decision is made; ADR column in the product spec's Key decisions table.
- **Revisit when:** ADRs are routinely skipped or written for trivial choices — then the bar in [README.md](README.md#when-to-write-one) needs adjusting.
