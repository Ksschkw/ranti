# Walrus Session 8 — Winning Plan

Target: win Main Prizes (1st/2nd/3rd), Beyond the Big Two, Best Article, Bug Bounty, Promo.
Budget: $0. Deadline: Oct 9 2026. Today: Sep 30 2026 (9 days).

---

## 1. Read on the field

Public builds found: 3 real chatbots, 2 memory-infrastructure projects.

- Almost every entrant submits the same thing: a support bot wrapped with
  `withMemWal` + `autoSave: true`, memory used as a prompt-stuffing detail.
- "Verifiable receipts" is already owned by Carry. "Action trace" is owned by
  OneMem. MCP servers for coding agents are crowded. Do not headline those.
- No public entrant shows real user traction. Every one of them is a demo.
- The documented friction everyone hits: append-only writes with no dedupe,
  top-K recall with no relevance floor, no delete/forget in the SDK, flat
  namespaces, restore with no cursor.

## 2. The gap we take

Three lanes are open in every source reviewed:

1. Two independently deployed apps sharing one user memory space, side by side.
   Portability is Walrus Memory's headline claim and nobody demonstrates it.
2. Onboarding with an existing memory space ("bring your memory with you").
3. Memory that stays correct as it grows. Walrus Memory ships a cortex
   (permanent, encrypted, portable storage). It has no hippocampus
   (consolidation, salience, conflict resolution). Append-only plus
   auto-extraction means memory rots: duplicates, stale values, contradictions.

## 3. The product

One line: one mind, every app — portable memory that gets sharper, not
noisier, the longer you use it.

Four pillars:

1. Three independently deployed surfaces, one memory space.
   Telegram bot, CLI, and a web widget. Different delegate keys, same
   `owner + namespace`. Every memory records which surface produced it, and the
   UI shows provenance. This is the headline demo and the clearest open lane.
2. Memory Passport.
   Export a user's memory space to one signed portable bundle, import it into a
   fresh bot instance, and the first message is already personalized.
   Recovery demo: delete all local state, bring the bot up on a clean machine,
   and the memory returns from Walrus Memory alone.
3. Hippocampus.
   On ingest: extract candidate facts, then classify each against the existing
   space as new, duplicate, supersede, or contradiction, using embeddings plus
   LLM adjudication in the ambiguous band.
   On recall: fetch a wide candidate set, drop superseded and duplicate
   entries, re-rank by semantic plus recency plus importance, cap by a token
   budget, and return the memories that actually deserve the context window.
   The consolidation state is itself written back to Walrus Memory as compact
   index records, so the intelligence is portable and not trapped in SQLite.
4. Counterfactual Replay.
   Re-run any turn with memory disabled and diff the two answers. This turns the
   before/after evidence the judges ask for into generated, reproducible output
   instead of a cherry-picked screenshot.

Supporting: evidence dashboard (per-user memory counts via `list_namespaces`,
memory browser, contradiction inbox, receipts with blob ids).

## 4. Why this wins each prize

- Main: memory does measurable work, real users, reproducible repo, honest article.
- Beyond the Big Two: Python SDK (the less travelled path), Groq Llama or Gemini
  as primary, local Ollama as fallback, documented friction.
- Best Article: the cortex/hippocampus story plus honest failures.
- Bug Bounty: targeted, reproducible issues, see section 8.
- Promo: post outside the Walrus and Sui ecosystem.

## 5. Stack

- Language: Python 3.12 (Python SDK is the richer, less travelled surface:
  `ask`, `occurred_at`, OpenAI/LangChain middleware, sync client).
- Framework: FastAPI for the API and widget backend; python-telegram-bot for
  surface 1; a thin CLI for surface 2; a static widget for surface 3.
- Memory: `memwal` 0.1.11, hosted relayer `https://relayer.memory.walrus.xyz`.
- LLM: Groq (Llama) primary, Google Gemini free tier secondary, local Ollama
  fallback. All OpenAI-compatible where possible. Zero cost.
- Store: SQLite for the working index. Walrus Memory is the source of truth.
- Tests: pytest plus `MemWalMock` for deterministic memory tests.
- Arch check: import-linter.
- Hosting: always-on Linux box for the bot process; static widget on GitHub Pages.

## 6. Architecture (project_init.md compliant)

```
src/
  models/entities/   user_model, memory_model, memory_record_model,
                     conversation_model, message_model, receipt_model,
                     contradiction_model, passport_model, surface_model
  schemas/           <entity>_schema.py
  crud/              <entity>_crud.py            one entity each
  services/          chat_service, memory_ingest_service,
                     memory_recall_service, consolidation_service,
                     contradiction_service, passport_service,
                     counterfactual_service, evidence_service,
                     user_service, conversation_service
  routers/           telegram_router, web_router, cli_router, memory_router,
                     passport_router, evidence_router, health_router
  core/
    config.py  database.py  errors.py  resilience.py  container.py
    protocols.py     interfaces services depend on
    gateways/        memwal_gateway, llm_gateway, telegram_gateway
tests/               mirrors src/
arch-check.py
pyproject.toml
.env.example
README.md
```

Two notes to confirm:
- `core/protocols.py` and `core/gateways/` extend the skeleton so outbound
  clients are injectable and services never import a driver. No `utils`,
  `helpers`, or `common`.
- Breakers wrap only the gateways: MemWal, the LLM provider, Telegram.

## 7. Timeline

- Sep 30: decisions, credentials, scaffold, real MemWal round-trip proof.
- Oct 1: Telegram bot live with a naive memory baseline. Recruit users today.
- Oct 2: hippocampus ingest and recall. Evidence dashboard.
- Oct 3: CLI and web widget surfaces. Memory Passport.
- Oct 4-6: users accumulate memories. Counterfactual Replay. Contradiction
  inbox. File bug reports.
- Oct 7: article draft, screenshots, demo video, memory-count evidence export.
- Oct 8: publish on Medium and Inkray, X post with `#WalrusMemory`, promo post
  outside the ecosystem.
- Oct 9: Airtable submission. Final repo and README check.

Critical path: real users must start by Oct 2 to satisfy "used for a few days",
and 3 users must exceed 10 stored memories each.

## 8. Bug bounty targets

Only file what we reproduce. Already public, so not prizes: job latency,
empty-then-matching recall, job id versus blob id, timestamp ties.

Candidates from this codebase review:
1. No delete or forget in the Python SDK, while `POST /api/forget` exists server
   side and is unwrapped. A Python user has no in-SDK erasure path.
2. `ScoringWeights` (recency and importance) is only reachable through
   `recall_manual`, which returns blob ids and no text. The high-level `recall`
   has no recency or importance ranking at all.
3. `recall` results carry no timestamp, and `occurred_at` is documented as not
   stored in any server-readable column, so a client cannot build temporal
   reasoning without its own index.
4. `MemWalMock.analyze` does not extract facts, it stores the raw text as one
   fact, so no test can exercise the real extraction pipeline.
5. Python `DEFAULT_SERVER_URL` is `http://localhost:8000` while TypeScript
   defaults to the hosted relayer, and the docs claim the two SDKs mirror each
   other exactly. Copy-paste from the TS quick start silently fails.
6. No extract-only endpoint. `analyze` both extracts and stores, so a hygiene
   layer cannot dedupe before the write, only after.

## 9. Article angle

Working title: "I gave my chatbot a hippocampus: portable memory that survives
the app."

Beats: the problem, the naive build and why it rotted, wiring Walrus Memory,
the three-surface portability demo, the wipe-and-recover moment, the
counterfactual, what broke, and the immutable-storage insight (unrecallable is
not the same as erased). 500 to 800 words, honest over polished.

## 10. Non-negotiable rules

From project_init.md: layered one-file-per-entity, dependency inversion enforced
by arch-check, breakers only at outbound boundaries, no emojis anywhere in the
codebase, and no test that cannot fail.
