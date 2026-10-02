# Portable Memory Across Four Surfaces: What Happens When an Assistant Actually Remembers

When a person closes an application, most conversational software resets to zero. A user who mentions an allergy, a preferred work schedule, or a project deadline in one conversation must restate those facts in the next conversation. If that person switches from a mobile chat app to a browser window or a terminal prompt, the software treats them as an entirely new visitor.

I built Cheta to solve this discontinuity. Cheta is a personal assistant that maintains a single, persistent memory space for each user across four distinct interfaces: a Command Line Interface (CLI) for the terminal, a Telegram messaging bot, a web application, and a browser extension for Chromium and Firefox. All four surfaces read from and write to the same underlying store using Walrus Memory, a decentralized storage network that stores data as immutable binary large objects (blobs) identified by content hashes.

To qualify for the Beyond the Big Two category, Cheta does not use models from OpenAI or Anthropic as its primary reasoning engine. The service is written in Python 3.12 using FastAPI, an asynchronous web framework. Its primary language model is an open-weight model, Qwen-2.5-32B, hosted on Groq, an inference acceleration platform. If Groq reaches rate limits, the application fails over to Google Gemini 2.5 Flash-Lite through its Application Programming Interface (API), followed by a locally hosted Ollama instance running Qwen-2.5-1.5B.

### What Breaks When Memory Is Merely Append-Only

In standard chatbot architectures, adding memory simply means calling an insertion function after a turn and running a semantic search before the next turn. When real people speak with an assistant over several days, that approach breaks down quickly. Repeated restatements create duplicate memories that crowd prompt context, changed preferences linger alongside obsolete ones, and unflagged contradictions force the model into random guesses.

To address these failure modes, Cheta adds an explicit consolidation layer before and after Walrus Memory interactions. Before writing a candidate fact, Cheta compares the new text against nearby existing memories. It classifies the fact as novel, duplicate, an update, or a contradiction. Exact duplicates are discarded prior to network transmission. Updates mark older records as superseded so they cease appearing in active prompts. Contradictions are preserved with a warning flag that prompts the assistant to ask the person for clarification.

### Before and After: Real Conversation Evidence

The difference between conversational turns with memory enabled and turns without memory is concrete. During testing with my girlfriend, Kenzy, she asked:

"What can I cook tonight with chicken and scotch bonnet peppers?"

Without memory, the assistant responded:
"You can prepare spicy pepper chicken, chicken curry, or stir-fry. Would you like a recipe that includes dairy, such as a creamy chicken pasta?"

With memory, the assistant recalled two stored facts from previous sessions: that Kenzy is lactose intolerant and avoids bell peppers. It responded:
"You can make Nigerian peppered chicken or chicken pepper soup. Both use scotch bonnet peppers and chicken, and neither requires dairy or bell peppers."

The prompt changed the response from a generic culinary suggestion to an immediately applicable recommendation that respected her dietary constraints.

### Multi-Step Actions Across Surfaces

Cheta is not restricted to text replies. The core service exposes thirteen agentic tools, including web search, page crawling, arithmetic, and reminder scheduling. In the browser extension, Cheta inspects active web tabs and can plan multi-step sequences using tools provided by the active page. For instance, when given a compound request on a supported web page, the planner decomposes the instruction into an ordered list of actions, executes each step sequentially, and narrates the combined result back into the chat session.

### Real Use and Friction Points

Cheta has been deployed and evaluated by three real users: myself (Kosisochukwu), my brother (K), and my girlfriend (Kenzy). Each user interacts primarily through Telegram and the browser extension, accumulating over ten verified memories each in their respective Walrus namespaces.

Working with the `memwal` Python Software Development Kit (SDK) version 0.1.11 exposed several architectural friction points that I documented for the bug bounty:
1. The Python SDK lacks a direct `forget` or `delete` method, making user-directed data deletion impossible without raw network calls.
2. The high-level `recall()` function only ranks by cosine distance. The `ScoringWeights` object (which incorporates recency and importance) is only accessible via `recall_manual()`, which returns raw blob identifiers without the associated text.
3. The local index required a custom snapshot mechanism stored in a companion Walrus namespace because Walrus Memory provides no mechanism to enumerate or list stored blobs.

Cheta demonstrates that when personal memory is decoupled from application databases and managed with proper consolidation hygiene, an assistant becomes genuinely useful across the tools people use every day.
