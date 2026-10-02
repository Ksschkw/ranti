# Cheta: Complete Hackathon Execution Plan
## Walrus Session 8: Chatbots That Remember (Deadline: October 9, 2026)

Budget constraint: $0.00. Zero paid APIs, zero credit card requirements, free tiers only (Groq, Gemini Flash-Lite, Ollama, Render free tier, Walrus devnet/testnet).

This document details every addition, fix, architectural change, UI revamp, and prize-qualifying asset planned and implemented for the final submission.

---

## 1. Product Positioning & Core Theme

### The Core Problem with Current Submissions
Most hackathon entrants build a simple chatbot wrapper around an LLM with `autoSave: true` on an append-only store. That approach fails two fundamental tests:
1. **Memory Rot**: Repeated conversations accumulate duplicate facts, outdated preferences persist alongside new ones, and contradictory statements sit unhandled in the top-K semantic search results.
2. **Surface Silos**: The bot remembers inside one tab, but the memory is trapped inside that single application database.

### Cheta's Differentiating Pitch
**Cheta is a portable, cross-surface agentic memory layer that follows a person across their tools, coupled with an assistant that acts on that memory.**
- **Four Connected Surfaces**: Terminal CLI, Telegram bot, Web application, and Browser Extension (Chromium & Firefox) all read and write to one shared Walrus Memory namespace per user.
- **Hygiene & Consolidation**: Automatic deduplication, superseding updates for changed attributes, contradiction detection, and index snapshots stored directly on Walrus for zero-loss redeployment.
- **Agentic Multi-Step Action**: Multi-tool reasoning across all surfaces (13 built-in tools) plus browser-native page tool planning capable of executing sequential multi-step actions on active web pages.

---

## 2. User Identity & Naming Fix ("Extension Visitor" Resolution)

### Root Cause
When the extension is installed, it generated a local random identifier `extension-<uuid>` and defaulted `state.displayName` to `"Extension visitor"`. Because there was no settings input or dedicated command to set the name, and the server greeted the user using the passed `display_name`, every response began with "Extension visitor, ...". Furthermore, users were not prompted to connect their existing Telegram or CLI memory space.

### Fixes & Additions
1. **Universal `/name <new_name>` Command**:
   - Add `/name` to `COMMANDS` in `src/services/conversation_service.py`.
   - When a user enters `/name <YourName>`, the backend immediately updates the user record's `display_name` in SQLite and returns a confirmation that updates the conversational context.
   - The extension intercepts `/name` to also persist the new name to `chrome.storage.local` under `ranti.extension.display_name`.
2. **Extension Profile & Settings UI**:
   - Add an inline Profile/Identity widget in `extension/sidepanel.html` allowing the user to view their current name, set or change it at any time, and view their user pairing status.
3. **Friendly Onboarding & Pairing Prompt**:
   - If `displayName` is blank or unset, prompt the user smoothly: "What should I call you? Or type `/pair <code>` to link your existing Telegram or CLI memory."
4. **Three Real Verified Telegram Users**:
   - Document the 3 real users:
     1. Kosisochukwu (Me) - Telegram ID `5527434923`
     2. K (My brother) - Telegram ID `6959647089`
     3. Kenzy / Nono (My girlfriend) - Telegram ID `6190892934`
   - Ensure all 3 user profiles hold at least 10 durable memories on the live Walrus Memory relayer to satisfy the hackathon qualification checklist.

---

## 3. Multi-Step Page-Tool Planner & Agentic Execution

### Problem
Previously, `plan_page_tool` and `PagePlanSchema` could only return a single tool call (`PagePlanSchema.tool` and `arguments`). If a user requested a compound action on a web page (e.g., "Add 2 pepperoni pizzas to cart and apply coupon CODE10"), the planner could only pick one action or failed entirely.

### Architecture & Implementation
1. **Schema Upgrade (`src/schemas/page_tool_schema.py`)**:
   - Introduce `PagePlanStepSchema` with `tool`, `arguments`, and `explanation`.
   - Update `PagePlanSchema` to include `steps: list[PagePlanStepSchema]`, while preserving backwards compatibility by populating top-level `tool` and `arguments` with the first step.
2. **Multi-Step Prompt Decomposition (`src/services/conversation_service.py`)**:
   - Instruct the LLM to analyze user instructions against the available page tools (DOM/MCP tools).
   - If the request requires sequential actions, return an ordered array of steps in JSON.
3. **Sequential Execution Engine (`extension/sidepanel.js`)**:
   - Upgrade `handlePlannedPageTool` to execute multi-step plans sequentially.
   - Provide visual feedback for each step ("Step 1/2: Running `add_to_cart`...", "Step 2/2: Running `apply_discount`...").
   - Safely pause and request user confirmation for state-changing or financial actions.
   - Feed back step outputs into the conversational transcript and send a final synthesis turn to Cheta.
4. **Native In-Browser Page Tools (`extension/sidepanel.js`)**:
   - Provide 5 universal native browser tools on any tab: `page_find_text`, `page_highlight_text`, `page_scroll_to`, `page_extract_links`, and `page_summarize`.
   - Execute directly in page DOM via `browserApi.executeScript` without requiring third-party WebMCP servers.
5. **Cross-Surface Agentic Tool Activity**:
   - Ensure tool execution across CLI, Telegram, Web, and Extension displays clear execution badges (`tool_activity`), keeping autonomous actions transparent.
   - Deterministic tool dispatch in `OfflineLlm.complete_with_tools` ensures full offline / zero-credential verification of arithmetic, weather, search, reminders, and calendar tools.
   - Web UI tools drawer with one-click interactive examples and local document attachment parsing.
   - CLI commands `cheta tools` and `cheta file <path> "<question>"` for terminal workflows.

---

## 4. Firefox Add-on Packaging & Installation Fix

### Problem
Firefox `about:debugging` -> "Load Temporary Add-on..." sometimes restricts or disables `.json` selection in native file dialogs on certain Linux/desktop environments, or fails when selecting unpacked directories without an archive.

### Solution
- Update `scripts/build_extension.py` to automatically package `extension-dist/cheta-firefox.zip` and `extension-dist/cheta-firefox.xpi` alongside the unpacked folders.
- In Firefox `about:debugging`, users can directly select `cheta-firefox.xpi` or `cheta-firefox.zip` with zero file chooser friction.
- Document this in `extension/README.md` and `extension-dist/firefox/README.md`.

---

## 5. Complete UI/UX & Visual Revamp

### Constraints & Rules
- **Retain the official ICON as-is** (do not alter icon assets).
- **Strict Monochrome Theme with Single Bright Accent**:
  - Pure obsidian dark backgrounds (`#0a0b0e`, `#111318`, `#161922`).
  - Crisp typography and borders (`#232734`, `#2e3446`).
  - Single vivid electric accent (`#00f0ff` or signature electric cyan `#35d0ba`).
  - Zero gradients.
  - Zero "egg white orange".
- **Eliminate Mediocre Buzzwords**:
  - Replace inflated tech jargon with concrete, action-oriented verbs ("Store fact", "Supersede preference", "Replay counterfactual", "Pair device").
- **Component Polish**:
  - Sharpen buttons, input areas, tool execution chips, and memory cards.
  - Distinctive visual states: active, focus, hover, degraded, and loading pulses.
  - Clear diff layout for the counterfactual replay ("With Memory" vs. "Without Memory").

---

## 6. Hackathon Prize Category Execution

### 1. Main Prizes: Best Chatbot ($500 / $250 / $150)
- End-to-end functionality demonstrated across all 4 surfaces.
- Genuine cross-device memory recall and self-healing index recovery from Walrus.
- 3 real users storing >= 10 memories each.

### 2. Beyond the Big Two: Open & Alternative Models ($150)
- Primary runtime: Groq `qwen3-32b` (or `qwen/qwen3.8-27b`) with fallback to Google Gemini Flash-Lite and local Ollama (`qwen2.5:1.5b`).
- Zero reliance on OpenAI or Anthropic as primary models.
- Explicit documentation of the open model runtime and SDK friction points.

### 3. Best Article ($100)
- Publication target: Medium or Inkray (dev.to mirror).
- Word count: 500-800 words.
- Strictly adheres to all 7 writing rules:
  - No em dashes (use parentheses, colons, or separate sentences).
  - No "It's not about X, it's about Y".
  - No rule-of-three obsession.
  - No Wikipedia voice.
  - No "tapestry", "landscape", or "in today's world" cliches.
  - No enthusiasm overload.
  - No setup-payoff dramatic structure.
  - Define every term/abbreviation on first mention.
  - Start broad and narrow down with honest technical clarity.

### 4. Bug Bounty ($100)
- File reproducible issues on MystenLabs/MemWal:
  1. Unique `blob_id` collision across namespaces when pairing or copying identical notes.
  2. Missing `forget` or `delete` method in the Python SDK.
  3. `ScoringWeights` (recency and importance) unreachable in high-level `recall()`.
  4. Recall responses omitting timestamps.
  5. Python SDK default server URL mismatch.
  6. `MemWalMock` lacking fact extraction parity with live relayer.

### 5. Promo Prize ($100)
- High-value technical post for r/LocalLLaMA and developer forums explaining the memory consolidation layer and index recovery design without marketing fluff.
