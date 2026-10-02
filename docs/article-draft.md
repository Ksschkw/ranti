# The memory should outlive the app

Every assistant I have used forgets me when I close the window. The ones that
remember usually remember me inside one product. Open a second app and I am a
stranger again. That is a reasonable place to start if you are building a chat
assistant, and it is where I started.

Walrus Memory is a storage service from the Walrus project. It saves data as
blobs, which are immutable chunks of bytes addressed by a hash of their content,
so nothing is quietly overwritten. A namespace is the name you file those blobs
under. I used its Python SDK behind a FastAPI service, with an open-weight Qwen
model served through Groq and Google Gemini as failover. No Anthropic or OpenAI
model runs as the primary.

I built Cheta as a memory-first assistant with four clients: a Telegram bot, a
terminal program, a browser widget and a Chrome extension. The interesting part
is not that each one remembers. It is that they share one memory space per
person. Pair a new client and it reads what the others already know. That is the
claim Walrus makes, portability, and almost nobody demonstrates it.

## What actually rots

I stored first and stored naively. Call a remember function after a turn, call a
search function before the next one. Three failures show up within a week.

The same fact arrives in different words and is stored several times. A changed
preference sits beside the old value, and both are returned as if they are
current. Two statements that cannot both be true are stored together and nothing
notices.

None of that is Walrus Memory misbehaving. It stores memories and returns the
closest ones. It never claimed to decide which of them deserve the prompt.

I checked before I assumed. The high-level search ranks by distance alone. A
scoring type that also considers recency and importance exists, but the common
path cannot reach it. The machinery was there; the road most people take skips
it.

## The layer I added

Before a fact is written, I compare it with its nearest neighbours and label it
as new, a duplicate, an update or a contradiction. Only then does it get stored.
Before memory is placed in the prompt, I fetch a wide candidate set, drop
superseded entries, collapse restatements, rank by meaning plus recency plus
importance, and cap the result.

Those decisions are data, not hidden state. After each change the assistant
writes a compact snapshot of its own index into a companion namespace. A fresh
deployment with an empty database can rebuild from Walrus alone. That happened
by accident: real users were on one instance and the hosted one started empty,
and it still recalled five of a returning person's memories.

The assistant also acts. Thirteen tools run through one loop on all four
clients: search the web, fetch a page, check the weather, set a reminder, run a
calculation, read a document, and read or write memory directly. Each turn now
reports which tools it used, so the work is visible instead of being buried in a
polite sentence.

## What a real user found

The first real person typed `/start`. The bot answered that it had no long-term
memory.

That is the worst sentence this product could produce. The prompt's empty-memory
branch invited the model to say it had nothing stored yet, and the model widened
that into denying memory entirely. The fix was one instruction and a regression
test. A scripted demo would never have caught it. A person typing the first
thing on their mind found it in seconds.

A later traceback was worse and more useful. The local index declared blob
identifiers unique, but the relayer returns the same identifier when the same
note lands in a second namespace. A pairing copy then tried to insert a second
row and the write failed. I filed it with the reproduction steps. That kind of
bug only appears when you leave the demo path.

## Honest limits

Namespaces are flat, so a personal space is a naming convention. Recall has no
default relevance floor. Snapshots are size-capped. Recency ranking relies on a
timestamp I store myself, because search results carry none.

Three people have used it for real. Two are past ten stored memories, at
nineteen and seventeen. A third has barely started. Those numbers are small and
I am not dressing them up.

Code, a credentials-free mock, and the write-up: <<FILL: links>>.

If you take one idea from this, take the boring one. A chatbot that remembers is
easy. A chatbot that still remembers after you replace the chatbot is the part
worth building.
