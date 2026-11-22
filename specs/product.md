# Spec: Sprich! — Product (umbrella)

*AI German speaking coach that learns from your mistakes.*

| | |
|---|---|
| **Status** | Draft |
| **Kind** | Umbrella (product-wide). Feature specs refine it — see [specs/README.md](README.md). |
| **Last updated** | 2026-11-22 |
| **How we work** | [docs/workflow.md](../docs/workflow.md) |
| **Decisions** | [docs/adr/](../docs/adr/README.md) |

This is the single source of truth for *what* Sprich! is and *why*. Work packages (GitHub issues) link here by requirement ID. Change this file **before** changing behaviour, and log every change in the [Changelog](#changelog).

## Intent

Foreigners living in Germany often **understand** German but can't **speak** it with confidence. The main gaps are **vocabulary** and **real speaking practice**.

Sprich! is a Telegram bot for daily practice through writing, vocabulary and conversation. Every mistake is recorded and grouped by type, and the bot plans the next exercises around the learner's weak areas (Dativ not sticking → more Dativ practice; vocabulary gaps → words added to spaced repetition).

**User zero is the project author**, who uses it daily from the first deployable version.

## Users & context

- Adults living in Germany, working or studying in another language.
- Their everyday language is **English, Ukrainian or Russian** (more later). All explanations, instructions and summaries come in that language; the language being learned is always **German**.
- More confident learners can switch the whole interface to **German** for immersion.
- Level **A2–B2**: past the basics, not yet fluent.
- They practise in short daily sessions on their phone.

## Key decisions

Founding decisions from the project brief. Decisions made since are recorded as ADRs in [docs/adr/](../docs/adr/README.md); an ADR that adds or changes a product-wide decision also adds or updates a row here and links it.

| ID | Area | Decision | Why | ADR |
|---|---|---|---|---|
| D-1 | Interface | Telegram bot first (aiogram 3). Web app possibly later. | Lives where users already chat daily; no UI to build. | — |
| D-2 | Stack | Python, FastAPI, LangGraph. Provider SDK (Anthropic/OpenAI) called directly inside graph nodes; no LangChain layer. Pydantic for structured output. | Fewer abstraction layers; learn the primitives. | — |
| D-3 | Database | Postgres (JSONB for flexible payloads, pgvector later, LangGraph Postgres checkpointer). SQLAlchemy + Alembic. | One store for app data and agent state. | — |
| D-4 | Spaced repetition | FSRS (`fsrs` Python package). | Modern scheduler, better retention than SM-2. | — |
| D-5 | Local dev | docker-compose (app + Postgres), bot in long-polling mode. | No public URL needed locally. | — |
| D-6 | Hosting | Railway, **EU region**, managed Postgres; webhook mode with secret-token check. Alternatives: Fly.io, Render, Hetzner. | Simple ops; EU data residency. | — |
| D-7 | Observability | Langfuse (or LangSmith). | LLM traces, latency and cost per call. | — |
| D-8 | Open source | Public repo, **AGPL-3.0**. Tuned prompts and curated content are private and loaded from `PROMPTS_DIR`. Secrets only in env vars. | Open code; the hosted product keeps its edge. | — |
| D-9 | Contributions | No outside contributions at first (or require a CLA). | Keeps licensing options open. | — |
| D-10 | Languages | Multilingual interface from v0.1: `en`, `uk`, `ru`, plus `de` for confident learners. German is the only learning target. | Most learners read explanations best in their own language; confident ones learn more by staying in German. | — |
| D-11 | Monetization | Later, optional: paid hosted bot (subscription); possible premium parts: voice, curriculum, exam prep. | Not a goal before v1.0. | — |

## Architecture

### Principles

- **Thin bot layer.** Handlers translate Telegram into calls to `core/`. `core/` never imports Telegram code.
- **Partner ≠ analyzer.** The conversation partner keeps the chat flowing. A separate analyzer extracts structured mistakes **in the background**, after the reply is sent.
- **Mistakes are events.** `mistakes` is an append-only log → rolled up into `skill_mastery` → used by the planner to choose exercises (raw → aggregated → used, like a medallion architecture).
- **Two languages per user.** *Target* language is German (fixed). *Interface* language is per user (`en`, `uk`, `ru`, `de`; ISO 639-1 — Ukrainian is `uk`, not `ua`).
  - Static bot text comes from per-locale message catalogs.
  - LLM-generated explanations, correction comments and summaries are produced in the interface language (passed to prompts as a parameter).
  - German learning content (tasks, corrected text, conversation) stays German. With the `de` interface, everything is German.
  - `core/` and the database store language-neutral codes, never translated labels.
  - Adding a language = add a catalog + register the locale. No other code changes.

### Module layout (target)

```
app/
  main.py            # FastAPI + bot (polling | webhook via BOT_MODE)
  config.py          # pydantic-settings
  bot/  handlers/ (start, write, vocab, talk, stats, settings), states.py, keyboards.py
  api/  webhook.py, health.py, v1/ (later)
  core/ conversation.py, writing.py, vocab.py, learner_model.py, speech.py
  agent/ graph.py, nodes/ (partner, analyzer, planner), prompts.py
  db/   models.py, migrations/
  locales/           # message catalogs per interface language
  scheduler.py       # daily nudges
```

### Core tables

`users`, `messages`, `mistakes`, `skill_mastery`, `vocab_cards`, `exercises`. Columns are defined in feature specs as each table is introduced.

### Mistake taxonomy

Stable, language-neutral codes. Display names are localized.

| Code | Category | Sub-types |
|---|---|---|
| `case` | Case | `nom`, `akk`, `dat`, `gen` |
| `article_gender` | Article / gender | |
| `adj_ending` | Adjective endings | |
| `verb_conjugation` | Verb conjugation | |
| `word_order` | Word order | `v2`, `verb_final` |
| `separable_verb` | Separable verbs | |
| `preposition` | Prepositions | |
| `tense_aux` | Tense / Perfekt auxiliary | |
| `vocabulary` | Vocabulary | `wrong_word`, `missing_word` |
| `spelling` | Spelling | |
| `other` | Anything that fits none of the above | |

`other` is a catch-all so the analyzer never has to force a wrong category; frequent `other` mistakes signal that the taxonomy needs a new code.

## Bot commands

| Command | Purpose | Milestone |
|---|---|---|
| `/start` | Onboarding: interface language, level (A2–B2), goals, daily reminder time | v0.1 |
| `/write` | Writing task → corrections + mistakes recorded | v0.2 |
| `/stats` | Weak areas and progress per category (v1: counts) | v0.2 |
| `/settings` | Interface language, level, reminder time (strictness from v0.7) | v0.3 |
| `/vocab` | Review due FSRS cards | v0.4 |
| `/add` | Add a word to vocabulary manually | v0.4 |
| `/talk` | Conversation mode (text, later voice notes) | v0.5 |
| `/stop` | End session → short summary of mistakes | v0.5 |

## Requirements

EARS format; IDs are stable and never reused (see [specs/README.md](README.md#requirement-ids)). Detailed for v0.1–v0.3. Later milestones are outlined under [Milestones](#milestones) and get requirements when we reach them.

### FND — Foundation (v0.1)

- **FND-1** — THE SYSTEM SHALL start the app and a Postgres database locally with a single `docker compose up`.
- **FND-2** — THE SYSTEM SHALL read all configuration from environment variables via pydantic-settings, and `.env.example` SHALL list every variable with placeholder values only.
- **FND-3** — IF a required setting is missing at startup, THEN THE SYSTEM SHALL exit with an error naming the missing setting, without printing any secret value.
- **FND-4** — THE SYSTEM SHALL change the database schema only through Alembic migrations.
- **FND-5** — WHERE `BOT_MODE=polling`, THE SYSTEM SHALL receive Telegram updates by long polling; WHERE `BOT_MODE=webhook`, THE SYSTEM SHALL receive them on a webhook endpoint.
- **FND-6** — WHEN the health endpoint is requested, THE SYSTEM SHALL respond `200` if the database is reachable and `503` otherwise.
- **FND-7** — THE SYSTEM SHALL call LLMs only through one client wrapper that returns Pydantic-validated structured output.
- **FND-8** — IF an LLM call fails or its output fails validation, THEN THE SYSTEM SHALL retry a bounded number of times and then send the user a localized error message.
- **FND-9** — WHEN a pull request is opened or updated, CI SHALL run lint and tests and report the result on the PR.
- **FND-10** — Code in `core/` SHALL NOT import Telegram or aiogram modules, and a test SHALL enforce this.

### I18N — Localization (v0.1)

- **I18N-1** — THE SYSTEM SHALL store one interface language per user, chosen from the supported locales (`en`, `uk`, `ru`, `de`).
- **I18N-2** — THE SYSTEM SHALL render all static user-facing text from per-locale message catalogs; handlers and `core/` SHALL NOT contain hard-coded user-facing text.
- **I18N-3** — IF a message key is missing from the user's locale catalog, THEN THE SYSTEM SHALL use the `en` text and log the missing key.
- **I18N-4** — THE SYSTEM SHALL generate explanations, correction comments and summaries in the user's interface language, and SHALL keep German learning content in German.
- **I18N-5** — THE SYSTEM SHALL store language-neutral codes (levels, goals, mistake categories) in the database, never translated labels.
- **I18N-6** — THE SYSTEM SHALL read the list of supported locales from one registry, so that adding a locale needs only a new catalog and a registry entry.

### ONB — Onboarding `/start` (v0.1)

- **ONB-1** — WHEN a user without a profile sends `/start`, THE SYSTEM SHALL first ask for the interface language, pre-selecting the user's Telegram `language_code` if it is supported and `en` otherwise.
- **ONB-2** — WHEN the interface language is chosen, THE SYSTEM SHALL ask, in that language and in order: level (A2, B1, B2), learning goals (multi-select from a fixed list), and daily reminder time (or no reminder).
- **ONB-3** — WHEN onboarding completes, THE SYSTEM SHALL save the profile and reply with a summary of the chosen settings.
- **ONB-4** — WHEN a user with a profile sends `/start`, THE SYSTEM SHALL show the current profile and offer to redo onboarding, without deleting any learning data.
- **ONB-5** — IF the user sends input that does not answer the current onboarding step, THEN THE SYSTEM SHALL repeat that step's question.

### WRT — Writing practice `/write` (v0.2)

- **WRT-1** — WHEN the user sends `/write`, THE SYSTEM SHALL send a German writing task suited to the user's level, with instructions in the interface language.
- **WRT-2** — WHEN the user answers an open writing task, THE SYSTEM SHALL reply with the corrected German text and an explanation of each correction in the interface language.
- **WRT-3** — IF the answer contains no mistakes, THEN THE SYSTEM SHALL say so and MAY suggest a more natural phrasing.
- **WRT-4** — THE SYSTEM SHALL store each task and the user's answer as messages linked to the user.
- **WRT-5** — IF the user sends another command while a writing task is open, THEN THE SYSTEM SHALL close the open task and handle the new command.

### MST — Mistake capture (v0.2)

- **MST-1** — WHEN the reply to a user's German text has been sent, THE SYSTEM SHALL analyze that text in the background and record each mistake as a separate event.
- **MST-2** — Each mistake event SHALL record: user, source message, original fragment, correction, category code, optional sub-type, explanation with its language code, and timestamp.
- **MST-3** — IF the analyzer returns a category outside the taxonomy, THEN THE SYSTEM SHALL record it as `other` and log the unknown value.
- **MST-4** — THE SYSTEM SHALL never update or delete mistake events, except when deleting a user's data at their request.
- **MST-5** — IF analysis fails, THEN THE SYSTEM SHALL log the failure without notifying the user or delaying any reply.

### STA — Stats `/stats` v1 (v0.2)

- **STA-1** — WHEN the user sends `/stats`, THE SYSTEM SHALL show the number of recorded mistakes per category, highest first, with category names in the interface language.
- **STA-2** — IF the user has no recorded mistakes, THEN THE SYSTEM SHALL reply with a hint to start with `/write`.

### OPS — Deploy & operations (v0.3)

- **OPS-1** — THE SYSTEM SHALL run on Railway in an EU region with managed Postgres.
- **OPS-2** — WHERE `BOT_MODE=webhook`, IF a webhook request lacks a valid Telegram secret-token header, THEN THE SYSTEM SHALL reject it without processing the update.
- **OPS-3** — THE SYSTEM SHALL send a trace of every LLM call (model, latency, tokens, cost) to Langfuse.
- **OPS-4** — WHEN a new version is deployed, THE SYSTEM SHALL apply pending migrations before handling updates.
- **OPS-5** *(proposed — see open questions)* — WHERE an allowlist of Telegram user IDs is configured, IF a user not on it messages the bot, THEN THE SYSTEM SHALL reply with a localized "private beta" message and SHALL NOT call any LLM.

### RMD — Daily reminder (v0.3)

- **RMD-1** — WHEN a user's reminder time arrives, THE SYSTEM SHALL send that user one localized practice reminder.
- **RMD-2** — THE SYSTEM SHALL send at most one reminder per user per day, including across restarts and redeploys.
- **RMD-3** — WHERE a user chose no reminder, THE SYSTEM SHALL send none.

### SET — Settings `/settings` (v0.3)

- **SET-1** — WHEN the user sends `/settings`, THE SYSTEM SHALL let them change interface language, level and reminder time.
- **SET-2** — WHEN a setting changes, THE SYSTEM SHALL apply it to all following messages and reminders.

### PRV — Privacy & secrets (all milestones)

- **PRV-1** — THE SYSTEM SHALL read secrets only from environment variables; no secret SHALL appear in the repository, logs, issues or PRs.
- **PRV-2** — THE SYSTEM SHALL load tuned prompts from `PROMPTS_DIR` when it is set, and otherwise use minimal default prompts shipped in the repository.
- **PRV-3** — THE SYSTEM SHALL NOT write learner message text or Telegram user IDs to application logs; logs use internal IDs.

## Milestones

Each milestone lists its requirements and a *Done when* checklist. Feature specs are written per work package when needed (see [specs/README.md](README.md#when-a-work-package-needs-its-own-spec)).

### v0.1 — Foundation (local)

Repo, license, README, `.env.example`, docker-compose, Postgres + Alembic, aiogram polling, i18n with `en`/`uk`/`ru`/`de`, `/start` onboarding, LLM client wrapper, CI (lint + tests).
**Requirements:** FND-1…10, I18N-1…6, ONB-1…5, PRV-1…3.
**Done when:**
- [ ] The bot answers `/start` on the author's laptop and saves the profile.
- [ ] Onboarding works end to end in each of `en`, `uk`, `ru`, `de`.
- [ ] CI is green on the PR that completes the milestone.

### v0.2 — Writing + mistake capture

`/write` produces a task at the user's level → user answers → corrected version + explanation. The analyzer stores typed mistakes. `/stats` v1 (counts per category).
**Requirements:** WRT-1…5, MST-1…5, STA-1…2.
**Done when:**
- [ ] 10 writing sessions produce a meaningful mistake breakdown in `/stats`.

### v0.3 — Deploy & dogfood

Railway (EU), webhook + secret token, managed Postgres, Langfuse, daily reminder, minimal `/settings`.
**Requirements:** OPS-1…5, RMD-1…3, SET-1…2.
**Done when:**
- [ ] The bot runs 24/7 and the author uses it daily from their phone.

### v0.4 — Vocabulary *(outline)*

Vocabulary mistakes and unknown words become FSRS cards; manual `/add`. `/vocab` review with keyboard buttons. Area code: `VOC`.
**Done when:** daily reviews come due and stick.

### v0.5 — Conversation mode, text *(outline)*

`/talk` with a LangGraph partner (topics: everyday Berlin life, work, Behörde, small talk), background analyzer, `/stop` summary. Area code: `TLK`.
**Done when:** a 10-minute text conversation ends with a `/stop` summary whose mistakes appear in `/stats`.

### v0.6 — Voice notes *(outline)*

Telegram voice → speech-to-text → partner; optional text-to-speech reply. Area code: `VOI`.
**Done when:** a voice-note conversation works end to end and its mistakes are recorded like text ones.

### v0.7 — Adaptive planner *(outline)*

`skill_mastery` rollup; planner node picks writing tasks, drills and conversation topics that target weak categories; correction strictness setting. Area code: `PLN`.
**Done when:** exercises visibly change as weak areas change.

### v1.0 — Public release *(outline)*

Multi-user hardening (rate limits, cost caps per user), GDPR basics (data export/delete), docs, first Medium post. Area codes: `HRD`, `GDPR`.
**Done when:** a new user can onboard without help, can't exceed their cost cap, and can export or delete their data.

### Later

Web app, realtime voice, exam prep (Goethe/telc), paid hosted tier.

## Out of scope

- Learning target languages other than German.
- Web app, realtime voice, exam prep, paid tier — all *Later*, not before v1.0.
- Outside contributions (see D-9).
- Native mobile apps.

## Open questions

Each has a working assumption. **Blocks** names the requirements that can't be implemented until the question is answered (and this spec updated) — `/wp-start` resolves it with the author when a WP touches them. `no` = the assumption stands unless someone objects.

| # | Question | Assumption | Blocks |
|---|---|---|---|
| Q-1 | Which LLM provider for v0.1 — Anthropic or OpenAI? | One provider behind the wrapper (FND-7); chosen when that WP starts. | FND-7 |
| Q-2 | What does a public clone run without the private prompts? | Minimal default prompts ship in the repo; `PROMPTS_DIR` overrides them (PRV-2). | PRV-2 |
| Q-3 | Which i18n library? | aiogram's built-in i18n with gettext catalogs; decided when that WP starts. | I18N-2 |
| Q-4 | Prompts per language or one prompt with a language parameter? | One German-focused prompt, interface language passed as a parameter (I18N-4). | I18N-4, WRT-2 |
| Q-5 | Does "more languages" ever mean more *learning* languages? | No — only more interface languages; other target languages are out of scope. | no |
| Q-6 | Learning goals list for onboarding (ONB-2)? | Fixed codes: `everyday`, `work`, `behoerde`, `small_talk`; more later. | ONB-2 |
| Q-7 | Reminder timezone? | `Europe/Berlin` for everyone until v1.0. | RMD-1 |
| Q-8 | Who may use the bot once it is public 24/7 at v0.3? Cost caps only come in v1.0. | Allowlist of Telegram user IDs from an env var (OPS-5). **Needs your decision.** | OPS-5 |
| Q-9 | `/settings` and `/stop` had no milestone in the brief. | Minimal `/settings` in v0.3, `/stop` in v0.5, strictness in v0.7. | no |
| Q-10 | For `/write`, does the correction call also emit structured mistakes, or does the separate analyzer always run? | Separate analyzer, reused by `/write` and `/talk` (partner ≠ analyzer). | MST-1 |
| Q-11 | Time window for `/stats` v1? | All time. | no |
| Q-12 | Should the reminder skip days the user already practised? | No in v0.3; revisit with the planner. | no |
| Q-13 | Langfuse or LangSmith? | Langfuse (EU cloud or self-hosted). | OPS-3 |
| Q-14 | Telegram username `@sprich_bot` available? | Fallback `@sprich_deutsch_bot`; check with BotFather. | no — check before creating the bot |
| Q-15 | Correction strictness levels — what are they? | Defined with the planner. | v0.7 |
| Q-16 | Speech-to-text / text-to-speech providers? | Chosen in v0.6. | v0.6 |
| Q-17 | `skill_mastery` formula? | Defined in v0.7. | v0.7 |
| Q-18 | GDPR export format and data retention? | Defined in v1.0. | v1.0 |
| Q-19 | Contributions: none, or allowed with a CLA? | None until v1.0. | v1.0 |
| Q-20 | With the `de` interface, should explanations be simplified to the learner's level? | Yes — plain German at or just below the user's level. | I18N-4 |

## Naming

- **Product:** Sprich! ("Speak!")
- **GitHub repo:** `sprich-deutsch`
- **Telegram bot:** `@sprich_bot`, or `@sprich_deutsch_bot` if taken
- **Repo description:** "AI German speaking coach that learns from your mistakes."

## Glossary

- **Work package (WP)** — one GitHub issue = one branch = one PR, sized to one milestone task.
- **Umbrella spec** — this file. **Feature spec** — `specs/<feature>.md`, refines one capability.
- **Requirement ID** — `AREA-n`, stable reference used by issues, plans, tests and PRs.
- **Mistake** — one recorded error event in a learner's German text, with a taxonomy category.
- **Skill area** — a taxonomy category as tracked in `skill_mastery` (not to be confused with Claude Code skills in `.claude/skills/`).
- **Card** — an FSRS vocabulary card.
- **Interface language** — the language of the bot's instructions and explanations (`en`, `uk`, `ru`, `de`). **Target language** — German.
- **ADR** — Architecture/product Decision Record in [docs/adr/](../docs/adr/README.md): why a significant choice was made, with the alternatives.
- **Mode** — how Claude Code works on a WP: Guide, Pair or Delegate (see [docs/workflow.md](../docs/workflow.md#working-modes)).

## Changelog

- **2026-11-22** — Added `de` interface language for confident learners (D-10, I18N-1, Q-20): immersion option requested by the author. Key decisions table gains an ADR column; new decisions are recorded as ADRs (ADR-0001), founding decisions are not backfilled.
- **2026-11-22** — Initial version from the project brief. Added: multilingual interface (`en`/`uk`/`ru`) as D-10 and `I18N` requirements; `other` mistake category; proposed allowlist OPS-5; open questions Q-1…Q-19.
