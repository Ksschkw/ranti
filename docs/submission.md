# Walrus Session 8 submission pack

Everything needed to file on Oct 9. Fill the marked values once real use is done.

## Airtable form answers

- LLM / runtime built with: Python 3.12, FastAPI, the `memwal` Python SDK 0.1.11 against the
  hosted Walrus Memory relayer. Primary model Groq `qwen3-32b`, failover to Google
  Gemini Flash-Lite and a local Ollama `qwen2.5:1.5b`. No OpenAI or Anthropic model is
  used as the primary, so this also enters the Beyond the Big Two category.
- Public GitHub repo: <<FILL: repo URL>>
- Bug or friction point and improvement idea: <<FILL: pick the strongest filed issue>>
  Short form: the Python SDK has no `forget` or `delete`, and `POST /api/forget` is
  unwrapped, so a Python integration cannot honour a deletion request without dropping to
  a wallet-authenticated Security Delete flow that only has a Node.js example. Second:
  `ScoringWeights` (recency and importance) is reachable only through `recall_manual`,
  which returns blob ids and no text, so the high-level `recall()` cannot rank by anything
  except semantic distance.
- Article link: <<FILL: Medium or dev.to URL>>
- X post link: <<FILL: post URL with #WalrusMemory and @WalrusProtocol>>

## Category entries

- Main Prizes: yes, the single submission.
- Beyond the Big Two: yes. Primary model is Qwen3-32B on Groq, with Gemini and a local
  Ollama model behind it. The article states the model and runtime explicitly and documents
  the integration friction hit on the Python SDK path.
- Best Article: yes.
- Bug Bounty: <<FILL: count>> issues filed at github.com/MystenLabs/MemWal/issues.
- Promo Prize: <<FILL: link to the out-of-ecosystem post>>. Target r/LocalLLaMA as the
  primary, with Show HN as a second. X, r/sui and Walrus or Sui channels do not count.

## X post draft

Most chatbots forget you when you close the tab, and the ones that remember only
remember you inside that one app.

We built Ranti: one memory, three apps. Telegram, terminal and browser share a single
Walrus Memory space, and a consolidation layer keeps it from rotting into duplicates and
contradictions.

You can wipe the whole app and the bot still knows you, because the index is itself a
memory.

Built with @WalrusProtocol Memory on the Python SDK, Qwen3 on Groq, zero dollars.
Full write-up: <<FILL: link>>
#WalrusMemory

## Promo post draft (r/LocalLLaMA, educational, not a launch)

Title: I built a consolidation layer for Walrus Memory because append-only recall rots

Body outline:
1. The setup: agent memory stores are append-only by design, and the high-level recall is
   cosine top-K. Neither is a bug until you run it for a week with real users.
2. What actually rots: near-duplicate restatements crowd out the context window; a changed
   preference never replaces the old one; two memories can contradict each other and
   nothing notices.
3. What I did: fetch a wide candidate set, drop superseded entries, collapse near-duplicate
   restatements, re-rank by semantic plus recency plus importance, then cap. On ingest,
   adjudicate a new fact against its nearest neighbours as same, update, contradict or new,
   and only then write.
4. The part I did not expect: Walrus Memory has no way to list memories, so the local index
   cannot be rebuilt by enumeration. I persist a compact index snapshot as a memory in a
   companion namespace, which makes the index portable by construction.
5. Numbers from real use: <<FILL: users, memories, duplicates skipped, contradictions
   flagged>>.
6. Code and the full write-up: <<FILL: links>>.

Post Tuesday to Thursday morning US Eastern, disclose authorship, answer every comment.

## Verification checklist before filing

- [ ] Repo is public and `README.md` quick start works from a clean clone.
- [ ] `.env` is not committed and no key appears anywhere in git history.
- [ ] At least 3 distinct users each hold 10 or more active memories on Walrus Memory.
- [ ] `/evidence/users` output is captured as a screenshot for the article.
- [ ] Article is published on Medium or dev.to and links the repo.
- [ ] X post is live and tagged.
- [ ] Bug issues are filed with reproduction steps and environment.
- [ ] Promo post is live outside the Walrus and Sui ecosystem.

## Current fills (2026-09-30)

- Public GitHub repo: https://github.com/Ksschkw/ranti
- Live deployment: https://ranti-gkn7.onrender.com (Render free, Docker,
  `memory: walrus`, `llm: groq + gemini`)
- Telegram bot: @Kosi_test_walrus1_bot, webhook mode, zero pending updates and no
  delivery errors
- LLM / runtime: Python 3.12, FastAPI, `memwal` 0.1.11 against the hosted
  relayer. Primary Groq `qwen/qwen3.8-27b`, failover Google
  `gemini-3.1-flash-lite`. No OpenAI or Anthropic model, so this also enters
  Beyond the Big Two.
- Bug bounty: four issues filed, https://github.com/MystenLabs/MemWal/issues/1061
  through 1064, plus a comment on the existing dedupe thread
  https://github.com/MystenLabs/MemWal/issues/1042#issuecomment-5920409813
- Article: not yet published. Draft at docs/article-draft.md, 794 words.
- X post: not yet published.
- Promo post: not yet published.

### Usage evidence, stated honestly

Three real Telegram identities have used the bot, holding 6, 1 and 1 memories.
Synthetic identities used for testing (telegram-9001, web-deploy-probe,
cli-live-probe, integration-test-user) are excluded and must never be counted.

The brief asks for three users with ten or more memories each. The user count is
met and the depth is not, and the two identities holding one memory each still
need confirming as distinct people rather than the account owner's own testing.

### Still required before filing

Publish the article, post on X with #WalrusMemory, post the promo outside the
Walrus and Sui ecosystem, then complete the Airtable form with those links.
