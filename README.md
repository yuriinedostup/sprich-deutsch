# Sprich!

*AI German speaking coach that learns from your mistakes.*

Many people living in Germany understand German but can't **speak** it with confidence. Sprich! is a Telegram bot for short daily practice:

- **Write** — get a task at your level, answer in German, get a corrected version with explanations.
- **Learn from mistakes** — every mistake is recorded by type (cases, word order, adjective endings…), and practice is planned around your weak areas.
- **Talk & review** — conversation practice (text, later voice) and spaced-repetition vocabulary built from your own gaps.

Explanations come in **English, Ukrainian or Russian** — or in **German**, for confident learners who want full immersion. The language you practise is always German. Levels A2–B2.

## Status

**Pre-alpha.** Specs are written; v0.1 is next. Not usable yet.

## Roadmap

| Milestone | What |
|---|---|
| v0.1 Foundation | Local setup, `/start` onboarding in `en`/`uk`/`ru`/`de`, LLM wrapper, CI |
| v0.2 Writing + mistakes | `/write` with corrections, typed mistake capture, `/stats` |
| v0.3 Deploy & dogfood | Hosted 24/7 (EU), daily reminders, `/settings` |
| v0.4 Vocabulary | FSRS cards from mistakes and unknown words, `/vocab`, `/add` |
| v0.5 Conversation | `/talk` with a conversation partner, `/stop` summary |
| v0.6 Voice | Voice notes in, optional voice replies out |
| v0.7 Adaptive planner | Exercises chosen from your weak areas |
| v1.0 Public release | Multi-user limits, GDPR export/delete, docs |

Details: [specs/product.md](specs/product.md#milestones).

## Tech stack

Python · FastAPI · aiogram 3 · LangGraph · Pydantic · Postgres (SQLAlchemy, Alembic) · FSRS · docker-compose · Railway (EU) · Langfuse

## How it's built

Spec-driven, with [Claude Code](https://claude.com/claude-code):

1. Intent → a **spec** in [`specs/`](specs/) with testable requirements; significant choices recorded as **decision records** in [`docs/adr/`](docs/adr/).
2. Spec → **work packages** as GitHub issues.
3. Each work package → a reviewed **plan** → implementation → **PR** where every change traces back to the spec.

The full workflow, including how Claude Code is used in *Guide*, *Pair* and *Delegate* modes: [docs/workflow.md](docs/workflow.md).

## Running locally

Coming in v0.1 (docker-compose with the app and Postgres, bot in long-polling mode).

## Prompts & content

The code is open; the tuned prompts and curated learning content are private and loaded at runtime from `PROMPTS_DIR`. The repo will ship minimal default prompts so it runs without them.

## Contributing

Not accepting outside contributions yet. Issues are used as the project's own task tracker.

## License

[AGPL-3.0](LICENSE)
