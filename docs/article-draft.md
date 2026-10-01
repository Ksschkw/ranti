# How I gave my chatbot a hippocampus

Most "memory" in a chatbot is a few recalled strings stuffed into a prompt. I built one of those first. Then a real person used it, and it told them it had no long-term memory.

## Why the obvious build rots

The brief asks for a chatbot that remembers. The obvious build calls `remember()` after each turn and `recall()` before the next. Three things then go wrong, and none of them are Walrus Memory's fault. It stores memories and returns the closest ones. It never claimed to decide which ones deserve the context window.

Duplicates. The extractor pulls the same fact out of differently worded sentences, so "allergic to shellfish" ends up stored four times. All four are close matches, so all four get recalled and three waste tokens.

Stale values. Preferences change. The old one is still similar, so it is still recalled next to the new one, and the bot answers as if both are current.

Contradictions. "I do not eat meat" and "I had a great steak on Friday" can sit in the store forever. Append-only storage will not choose, and cosine distance does not know there is a conflict.

I checked before assuming. High-level `recall()` is top-K by distance with no recency or importance weighting. A `ScoringWeights` type with both exists, but it is only reachable through `recall_manual`, which returns blob ids and no text. The ranking machinery is there and the path most people take cannot reach it.

## What I built

Ranti is a memory-first assistant with one memory space and three independent surfaces: a Telegram bot, a terminal client, and a browser widget. Each writes to the same owner and namespace, and every memory records which surface produced it.

On top of Walrus Memory I added the layer I was missing. On ingest, extract candidate facts, compare each against its nearest neighbours, and classify it as same, update, contradict or new before writing anything. On recall, fetch wide, drop superseded entries, collapse near-duplicate restatements, re-rank by semantic distance plus recency plus importance, then cap to a budget.

## What a real user taught me

The first real person sent `/start`. The bot replied: "I don't have long-term memory."

That is the worst possible failure for this product, and no test caught it. My prompt's no-memories branch told the model to say plainly that it had no stored memories, and the model generalised that into denying memory across conversations entirely. The fix was one sentence of prompt and a regression test asserting both halves of the instruction. A demo would never have found it. A person typing the first thing that comes to mind found it immediately.

## Then I measured what I had assumed

Against the live relayer: reply generation 2.3 seconds, `recall()` 1.6 seconds, and `remember_and_wait()` 28.6 seconds. Turns took 81 seconds, because consolidation awaited one blocking write per fact.

So a turn no longer waits for persistence. It accepts each write, records a pending index row, and settles the real blob id in the background. Verified live: 81 seconds down to 18, with the placeholders resolving to real blob ids shortly after. The API reports "accepted, persisting" rather than claiming a memory is stored when it is not.

## The proof I did not expect to need

Walrus Memory cannot list memories, so the local index cannot be rebuilt by reading it back. I made the index self-describing: after each change the bot writes a compact snapshot of its index as a memory in a companion namespace.

Then deployment tested it by accident. My real users were on my laptop's instance and the hosted one started with an empty database. With zero memories in its local index, the hosted bot still recalled five of a returning user's memories, because recall runs against Walrus scoped by namespace rather than against application state. Rebuilding restored all six records, the four active and the two superseded, out of the snapshot. The memory survived the app being replaced, and so did the decisions made about it.

## Honest limitations

Namespaces are flat, so a per-user space is a naming convention. Recall has no default relevance floor. Snapshots are capped per write. Recency ranking depends on an `occurred_at` I store, because recall results carry no timestamp at all. And the numbers are honest rather than flattering: two real people are past ten stored memories, at 19 and 17, and a third has barely started.

## What to copy

The repo runs against an offline mock with no credentials, so you can watch the consolidation before you create a Walrus Memory account. <<FILL: links>>

If you are building anything that remembers, the thing worth copying is not the recall call. It is deciding what to forget.
