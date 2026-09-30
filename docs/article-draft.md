# How I gave my chatbot a hippocampus

Most "memory" in a chatbot is a prompt with a few recalled strings in it. I built one of
those first. It demoed beautifully and it rotted within a week of real use.

## Why the obvious build fails

The Session 8 brief asks for a chatbot that remembers. The obvious build is a support bot
that calls `remember()` after each turn and `recall()` before the next one.

Three things went wrong, and none of them are Walrus Memory's fault. Walrus Memory stores
memories and returns the semantically closest ones. It never claimed to decide which ones
deserve the context window.

Duplicates. The extractor pulls the same fact out of slightly different sentences, so "Ada
is allergic to peanuts" ends up stored four times. All four are close matches, so all four
get recalled and three are wasted tokens.

Stale values. Preferences change. The old one is still stored, still semantically similar,
still recalled next to the new one. The bot answers as if both are current.

Contradictions. "I do not eat meat" and "I had a great steak on Friday" can both sit in the
store forever. Append-only storage will not pick a winner, and cosine distance does not know
there is a conflict.

I checked the SDK before assuming. High-level `recall()` is top-K by distance with an
optional cutoff and no recency or importance weighting. A `ScoringWeights` type with
recency and importance exists, but it is only reachable through `recall_manual`, which
returns blob ids and no text. The ranking machinery is there; the path most people take
cannot reach it.

## What I built

Ranti is a memory-first assistant with one memory space and three independently deployed
surfaces: a Telegram bot, a terminal client, and a browser widget. Each writes to the same
`owner + namespace`, and every memory is stamped with the surface that produced it, so you
can watch a fact learned in the terminal get recalled in the browser.

On top of Walrus Memory I added the layer I was missing: the hippocampus.

On ingest, extract candidate facts, compare each against its nearest existing memories, and
classify it as same, update, contradict or new before writing anything. Only new and update
facts are written. An update marks the old memory superseded. A contradiction is surfaced
for a human instead of being silently appended.

On recall, fetch a wide candidate set, drop superseded entries, collapse near-duplicate
restatements, re-rank by semantic distance plus recency plus importance, then cap to a
budget.

On one real user that skipped <<FILL: duplicates>> duplicate writes and flagged
<<FILL: contradictions>> contradictions the naive build would have merged straight into the
context.

## The moment it mattered

A user mentioned in passing on Telegram that they had changed jobs. Three days later they
asked the web widget for advice about their work. The old job was still in the store, still
semantically close to the question. The consolidation layer had marked it superseded, so the
widget answered from the new one. Small thing. Also the difference between a demo and
something you keep using.

## Showing before and after honestly

I did not want to hand-pick screenshots, so the bot generates the comparison itself.
Every turn stores the memories recalled for it, and `/chat/counterfactual/{turn_id}`
re-runs the same prompt with memory off and diffs the answers. It also reports when memory
made no difference, which is most of the time. Memory only earns its latency when it
changes the answer.

## What broke

The Python SDK has no `forget` or `delete`. The relayer exposes `POST /api/forget`, but the
SDK does not wrap it, and the only true deletion path is a wallet-authenticated flow with a
Node.js example. A Python integration cannot honour a deletion request
without leaving the SDK.

Nothing lists memories either. `recall()` searches; nothing enumerates. Your local index cannot be
rebuilt by reading Walrus Memory back. So
I made the index self-describing: after each turn that changes something, the bot writes a
compact snapshot of its index as a memory in a companion namespace. Delete the local
database, restart, and the bot recalls the newest snapshot and rebuilds. The index is data,
and the data lives in Walrus.

## Honest limitations

Namespaces are flat, so a per-user space is a naming convention. Recall has no default
relevance floor, so a small space returns weak filler unless you filter it. Snapshots are
capped per write, so a large space needs trimming by importance. And
recency ranking relies on an `occurred_at` value the bot stores, because recall results carry
no timestamp.

## Try it

The repo runs against an offline mock with no credentials, so you can watch the
consolidation behaviour before creating an account. <<FILL: links>>

If you are building anything that remembers, the thing worth copying is not the recall call.
It is deciding what to forget.
