# CCA Architect Coach — study material

Reference material for the CCA-F study coach: curriculum, quiz bank, and
glossary. The slash commands that use this file live in `.claude/commands/`.
Coach behaviour rules live in `CLAUDE.md`.

---


## Curriculum map

**Foundations — How Claude Works**
- F1 What Claude is · F2 The Messages API · F3 Models & sampling · F4 Tokens & context · F5 Tool use · F6 What "agentic" means

**Domain 1 — Agentic Architecture & Orchestration (27%)**
- 1.1 Agents vs workflows · 1.2 The five patterns · 1.3 The agentic loop & stop_reason · 1.4 The Claude Agent SDK · 1.5 Subagents & multi-agent orchestration · 1.6 Reliability & guardrails

**Domain 2 — Claude Code Configuration & Workflows (20%)**
- 2.1 What Claude Code is · 2.2 CLAUDE.md hierarchy · 2.3 Rules & path-scoping · 2.4 Slash commands · 2.5 Plan mode · 2.6 Sessions: compact/resume/fork · 2.7 Hooks · 2.8 Subagents, permissions & headless/CI

**Domain 3 — Prompt Engineering & Structured Output (20%)**
- 3.1 Prompting principles · 3.2 Examples & few-shot · 3.3 Thinking & chain-of-thought · 3.4 Structured output with tool_use · 3.5 tool_choice & nullable fields · 3.6 Built-in tools & validation

**Domain 4 — Tool Design & MCP Integration (18%)**
- 4.1 What MCP is · 4.2 MCP primitives · 4.3 Configuring MCP (.mcp.json) · 4.4 Designing good tools · 4.5 Errors & isError

**Domain 5 — Context Management & Reliability (15%)**
- 5.1 The context window · 5.2 Managing context (summarise/RAG/caching) · 5.3 Sessions & forking · 5.4 The Message Batches API · 5.5 Error handling & escalation · 5.6 Evaluating reliability

---

## CURRICULUM

### F1 — What Claude is, in one breath
**Hook:** Claude is a large language model trained to predict the most plausible continuation of text — and it has no memory between conversations.
**Concept:** Every request carries ALL the context it uses; there's no server-side memory. Analogy: a brilliant contractor with amnesia between jobs — every call you hand over a folder with everything needed for THIS job. Assembling that folder well is "context engineering."
**Exam:** Every domain assumes this mental model.

### F2 — The Messages API
**Hook:** Every call is a list of messages with roles, plus an optional system prompt.
**Concept:** Roles are `user` and `assistant`. The system prompt frames every reply (persona, rules, format). Multi-turn = the list grows; you resend the whole history each time because Claude is stateless. KEY: state lives in the messages list, not on the server.
**Exam:** Tests whether you know state is client-side.

### F3 — Models & sampling
**Hook:** A model family (Opus, Sonnet, Haiku) plus knobs to steer output.
**Concept:** Opus = deepest reasoning; Sonnet = balanced daily driver; Haiku = fast and cheap. Routing = easy work to Haiku, hard to Sonnet/Opus. Knobs: `max_tokens` (required cap), `temperature` (creativity vs determinism), `stop_sequences`, and the system prompt (highest leverage).
**Exam:** Routing scenarios are common.

### F4 — Tokens & the context window
**Hook:** Text is processed in tokens (~¾ word). The context window is the total token budget for a call.
**Concept:** Input + output must fit the window. You pay per token (cost + latency). As conversations grow, you MUST manage context: summarise, retrieve relevant chunks (RAG), or cache.
**Exam:** Domain 5 builds on this.

### F5 — Tool use
**Hook:** By default Claude only produces text. Tool use lets it act.
**Concept:** You describe tools (name + description + JSON input schema). Claude returns a `tool_use` request — it does NOT run anything. YOUR code executes the tool and sends back a `tool_result`. Then Claude continues. This loop IS the agent.
**Exam:** Claude never executes tools itself — tested explicitly.

### F6 — What "agentic" means
**Hook:** An agentic system is one where Claude decides what to do next.
**Concept:** Non-agentic = you drive (call step 1, step 2, step 3). Agentic = Claude drives (a loop where Claude picks the next tool). The shift: Claude's output determines the next input. Power (flexibility) and risk (unpredictability, loops) both come from this.
**Exam:** Underpins all of Domain 1.

### 1.1 — Agents vs workflows
**Hook:** A workflow is you deciding the sequence. An agent is Claude deciding the sequence.
**Concept:** Workflow — you choreograph fixed steps; predictable, auditable, cheaper. Agent — Claude picks which tool, in what order, when to stop; flexible but needs guardrails. ACID TEST: "Can I write the exact sequence BEFORE the user talks to Claude?" Yes → workflow. No → agent. Trap: multiple steps or multiple tools ≠ agent. What matters is who controls the sequence.
**Exam:** Scenario questions — apply the acid test.

### 1.2 — The five agentic patterns
**Hook:** Agent loops follow one of five recognizable shapes.
**Concept:**
1. Simple loop — same action repeated until a goal is met.
2. Branching — reasoning about the input routes to a path.
3. Multi-step reasoning — chained DIFFERENT actions; each result shapes the next.
4. Validation & retry — attempt, check outcome, adapt and retry.
5. Delegation & aggregation — parallel/independent sub-tasks combined at the end.
Confusions: simple loop (same action) vs multi-step (different actions chained); branching (one decision point) vs multi-step (path has many steps).
**Exam:** Map a described process to a pattern.

### 1.3 — The agentic loop & stop_reason
**Hook:** `stop_reason` is your loop control — it tells you whether Claude is done or wants a tool.
**Concept:** Values: `end_turn` (done — extract text), `tool_use` (execute tool, loop), `max_tokens` (truncated — raise the cap). Loop shape: call Claude → if end_turn return text → if tool_use, execute, append Claude's FULL response.content AND the tool_result, loop. THREE GOTCHAS: (1) the tool_result's `tool_use_id` must match the id from Claude's tool_use block; (2) tool results go as role `user`, not `assistant`; (3) append the whole response.content, not just the tool_use block. Always cap iterations.
**Exam:** Loop shape + all three gotchas appear in code-reading questions.

### 1.4 — The Claude Agent SDK
**Hook:** A Python library that handles the agentic loop boilerplate for you.
**Concept:** Built on the Messages API. Manages the loop, tool dispatch, state, and hooks for logging/validation/errors. Use it for production agents wanting built-in reliability; use the raw API for simple scripts or full loop control. It doesn't change HOW Claude works — it wraps the same API.
**Exam:** Know when the SDK vs the raw API is right.

### 1.5 — Subagents & multi-agent orchestration
**Hook:** A subagent is Claude calling another Claude as a tool.
**Concept:** Patterns: orchestrator→subagent (delegate to a specialist); parallel subagents (analyse A, B, C at once); specialist routing (classify then route). Risks: context leakage, infinite delegation, trust boundaries (should a subagent run destructive actions?), cost explosion. Mitigate: iteration caps at every level, least-privilege permissions, log inter-agent communication.
**Exam:** Which pattern fits, and what are the risks?

### 1.6 — Reliability & guardrails for agents
**Hook:** More autonomy than workflows means more guardrails needed.
**Concept:** (1) Iteration limits — always cap the loop. (2) Least-privilege tool permissions. (3) Confirmation gates for destructive/irreversible actions. (4) Input validation before executing tool inputs. (5) Error handling & escalation — don't silently loop on failure. (6) Observability — log every tool call, result, stop_reason. (7) Idempotency — safe to retry (upsert, not insert).
**Exam:** "This agent did X bad thing — which guardrail prevents it?"

### 2.1 — What Claude Code is
**Hook:** Anthropic's CLI tool: an agentic coding assistant in your terminal.
**Concept:** The `claude` command. It reads files, runs bash, edits code — in a loop. Reads your project structure and CLAUDE.md files. It's task-oriented, not a chat window. Uses the same Messages API underneath — the agent loop is pre-built.
**Exam:** It's an agent, not a chat tool.

### 2.2 — CLAUDE.md memory hierarchy
**Hook:** Persistent, project-aware instructions organized by folder scope.
**Concept:** Order of scope: GLOBAL (`~/.claude/CLAUDE.md`, all projects) → PROJECT (`<root>/CLAUDE.md`) → LOCAL (`<subfolder>/CLAUDE.md`). More specific wins (like CSS specificity). Put architecture, conventions, files-not-to-touch, common commands, domain context. NEVER put secrets (may be committed to git).
**Exam:** Hierarchy order and what each level covers.

### 2.3 — Rules & path-scoping
**Hook:** Scope rules to specific paths so different folders get different instructions.
**Concept:** Rules in a subfolder's CLAUDE.md auto-apply to files in that folder. Example: a "never hardcode credentials" rule in `/src/ingestion/` doesn't wrongly apply to your database migration scripts elsewhere. Placement handles most path-scoping for you.
**Exam:** Given a structure, which CLAUDE.md governs a given file?

### 2.4 — Slash commands
**Hook:** Reusable prompts saved as Markdown files that you trigger with /command-name.
**Concept:** Custom slash commands are Markdown files placed in `.claude/commands/` — the FILENAME becomes the command name (`cca-start.md` → `/cca-start`). The file's CONTENT is the prompt Claude executes when invoked. Two scopes: project (`.claude/commands/`, shared via git) and personal (`~/.claude/commands/`, all your projects). Use `$ARGUMENTS` in the file to insert what the user typed after the command. Optional YAML frontmatter adds a description and can restrict allowed tools. NOTE: a `## Commands` section inside CLAUDE.md does NOT register commands — that's a common mistake; commands must be individual files in the commands directory. (Newer versions also support `.claude/skills/<name>/SKILL.md` as the recommended format; both work.)
**Exam:** Where command files live, how naming works, and $ARGUMENTS.

### 2.5 — Plan mode
**Hook:** Claude shows what it will do before doing it.
**Concept:** `claude --plan` or `/plan`. Claude reads context, outputs a step-by-step plan, and waits for approval before executing. Use before production changes, multi-file refactors, or when unsure. Same agent loop underneath — just a human gate at the start.
**Exam:** When to use plan mode vs just running.

### 2.6 — Sessions: compact, resume, fork
**Hook:** Manage long agent tasks across time.
**Concept:** COMPACT (`/compact`) summarises history to save tokens (loses detail). RESUME (`--resume <id>`) reloads a past session to continue. FORK (`--fork <id>`) branches from a point to try an alternative (like git branch for sessions). These let tasks span hours or days and support safe experimentation.
**Exam:** When to use each.

### 2.7 — Hooks
**Hook:** Functions that run automatically before/after Claude Code actions.
**Concept:** Points: before/after file write, before/after bash command. Uses: validate against standards, block dangerous commands (rm -rf, DROP TABLE), log for audit, transform output (auto-format). Hooks run YOUR code — the safety layer between Claude's intentions and your file system. Unlike CLAUDE.md instructions (which Claude can ignore), hooks are enforced.
**Exam:** "Claude Code did something dangerous — which hook prevents it?"

### 2.8 — Subagents, permissions & headless/CI
**Hook:** Claude Code can run with no human (headless) and spawn subagents.
**Concept:** Headless (`--headless "task"`) for CI/CD: automated reviews, nightly checks. Subagents parallelise (e.g. fix 10 tests with 10 subagents). No human means permissions MUST be explicit: which dirs, which commands, network, git. Least privilege: a test-fixing subagent shouldn't write production config.
**Exam:** Headless + permission design scenarios.

### 3.1 — Prompting principles
**Hook:** Clear prompts specify role, task, format, and constraints.
**Concept:** ROLE (who Claude is), TASK (what to do), FORMAT (output shape), CONSTRAINTS (the rules). Role + constraints live in the system prompt; task + format in the user message. Mistakes: assuming Claude knows your context, cramming too many asks into one prompt, not specifying format.
**Exam:** Given a bad prompt, spot the missing element.

### 3.2 — Examples & few-shot
**Hook:** Showing examples often beats describing.
**Concept:** Zero-shot = describe and hope. Few-shot = show 2–5 input→output pairs first. Few-shot wins when format is hard to describe, the task is domain-specific, or zero-shot keeps missing. Examples must be representative and correct — bad or contradictory examples teach the wrong thing.
**Exam:** When to use few-shot vs zero-shot.

### 3.3 — Thinking & chain-of-thought
**Hook:** Asking Claude to reason step-by-step improves accuracy on complex tasks.
**Concept:** CoT = "think step by step" before answering. Extended thinking (API) gives Claude a token budget to reason internally, returned as a separate block. Helps on multi-step logic, error-compounding tasks (math, debugging, data modelling). Doesn't help simple lookups or latency-critical tasks. Extended thinking costs extra tokens.
**Exam:** When is extended thinking worth the cost?

### 3.4 — Structured output with tool_use
**Hook:** The reliable way to get structured JSON is a tool schema, not a prompt.
**Concept:** Prompt-based ("return JSON") is unreliable — Claude may add text or markdown. Tool-use-based: define a tool with the exact schema, set `tool_choice` to force it. Claude returns a structured tool_use block guaranteed to match the schema (it validates before returning).
**Exam:** tool_use vs prompting for structured output — know why.

### 3.5 — tool_choice & nullable fields
**Hook:** `tool_choice` controls whether Claude must use a tool, any tool, or a specific one.
**Concept:** `auto` (Claude decides), `any` (must use some tool), `tool` + name (must use THIS tool). Nullable fields: if a field may lack a value, mark it `{"type": ["string", "null"]}` and remove from required — otherwise Claude hallucinates a value to fill a required field.
**Exam:** Which tool_choice setting fits a scenario?

### 3.6 — Built-in tools & validation
**Hook:** Claude has first-party tools; you can validate tool outputs before feeding them back.
**Concept:** Built-in tools include web_search and code_execution — enable by adding to the tools list. You don't have to blindly return raw tool output: validate/sanitise/transform before appending the tool_result. Malformed results can make Claude loop or hallucinate.
**Exam:** Which built-in tools exist; when validation matters.

### 4.1 — What MCP is
**Hook:** MCP (Model Context Protocol) is an open standard for connecting Claude to external tools and data.
**Concept:** Solves the one-off-integration problem — like HTTP for tool connections. Defines how tools expose themselves, how Claude discovers/calls them, how results return. Enables plug-and-play tools and a shared ecosystem. MCP is the PROTOCOL; an MCP server is the implementation. Both Claude Code and the API support it.
**Exam:** MCP is a protocol, not a product.

### 4.2 — MCP primitives
**Hook:** MCP exposes three things: tools, resources, prompts.
**Concept:** TOOLS (actions — Claude acts, side effects possible). RESOURCES (read-only data — schema, file tree; no side effects). PROMPTS (reusable templates the server provides). Pick by use case: read the schema → resource; run a query → tool; standard review template → prompt.
**Exam:** Tools vs resources vs prompts — when to use each.

### 4.3 — Configuring MCP (.mcp.json)
**Hook:** You configure which MCP servers Claude connects to in `.mcp.json`.
**Concept:** Under `mcpServers`, each entry has `command` (how to start the server), `args`, and `env` (use `${VAR}` to pull from your shell). Credentials go in env, NEVER hardcoded. The server runs as a separate process; Claude communicates via stdio. Multiple servers combine — Claude sees all their tools.
**Exam:** The .mcp.json structure and where credentials come from.

### 4.4 — Designing good tools
**Hook:** A good tool has a clear name, a WHEN-to-use description, and a schema that prevents hallucination.
**Concept:** (1) Verb-first self-describing name (query_snowflake, not sf). (2) Description says WHEN to use it, not just what it does. (3) Only mark truly-required fields required. (4) Use enum to constrain string params. (5) Single responsibility — many focused tools beat one mega-tool. Write descriptions from Claude's perspective ("use this when you need…").
**Exam:** Spot the poorly-designed tool and fix it.

### 4.5 — Errors & isError
**Hook:** On tool failure, return a tool_result with `is_error: true` — don't crash the loop.
**Concept:** Never raise and crash (Claude learns nothing) or return an empty string (Claude assumes success). Instead return a tool_result with `is_error: True` and a useful message. Claude then adapts: retry, escalate, or try another approach. Always return a tool_result, even for failures.
**Exam:** is_error is frequently tested — know what omitting it does.

### 5.1 — The context window
**Hook:** The context window is the total token budget for one call — input + output.
**Concept:** System prompt + all messages + tool definitions + tool results count as input. Exceed the window → API error. In agents, every loop iteration adds to history, so long agents can fill it and fail. Two-sided pressure: too little context = wrong answers; too much = hit the limit, cost/latency spike.
**Exam:** What fills the window; what happens when you exceed it.

### 5.2 — Managing context: summarise, RAG, caching
**Hook:** Three strategies to stay within the window.
**Concept:** SUMMARISE — replace old turns with a summary (loses detail; good for long chats). RAG — embed documents, retrieve only relevant chunks at query time (good for large corpora; retrieval quality is key). PROMPT CACHING — cache a repeated large block server-side; pay full price once, a fraction on hits (good for fixed shared context). Match: summarise→long chats; RAG→large varied corpora; caching→fixed shared context.
**Exam:** Which strategy fits a scenario, and the trade-offs.

### 5.3 — Sessions & forking
**Hook:** Resume long tasks; fork to try alternatives without losing progress.
**Concept:** A session is a persistent conversation with an ID. RESUME reloads full history to continue (multi-day tasks). FORK branches from a point — shared history up to the fork, then diverges (try two approaches). COMPACT reduces tokens before resuming. Essential for reliable, resumable, long-running agents.
**Exam:** Resume vs fork vs compact.

### 5.4 — The Message Batches API
**Hook:** Send thousands of requests asynchronously and cheaply.
**Concept:** Up to 10,000 requests per batch, run async, results returned when done. Cheaper than synchronous. For bulk tasks with no real-time need: classifying tickets, summarising documents, bulk QA. NOT for real-time UX, streaming, or inter-dependent requests. Great for bulk data enrichment/classification instead of row-by-row loops.
**Exam:** Batch vs synchronous Messages API.

### 5.5 — Error handling & escalation
**Hook:** Reliable agents retry where sensible and escalate when not.
**Concept:** Hierarchy: (1) transient errors (timeouts, 429) → retry with exponential backoff, capped. (2) input errors → return is_error, let Claude revise. (3) logic errors (looping, stuck) → escalate to a human. (4) irreversible actions → human gate BEFORE execution. Escalation signals: max iterations hit, same error 3+ times, Claude expresses uncertainty, irreversible action imminent.
**Exam:** Agent looping on an error — what should it do?

### 5.6 — Evaluating reliability
**Hook:** You can't trust an agent you haven't evaluated.
**Concept:** Dimensions: correctness, consistency, safety, efficiency. Methods: golden dataset (50–100 known cases, measure pass rate), regression testing (re-run after changes), adversarial testing (try to break it), observability (log everything), human-in-the-loop review for high-stakes. Evaluation is a discipline, not a one-time check.
**Exam:** Which evaluation approach catches a given problem.

---

## QUIZ BANK

Ask one at a time. Reveal the answer + explanation after each. Format: A/B/C/D.

**Q (1.1):** A payment flow runs verify → compliance → validate recipient → transfer, fixed order, no deviation. Agent or workflow?
A) Agent — adapts if a step fails · B) Workflow — sequence is predetermined · C) Agent — multiple tools · D) Workflow only if fast
**Answer: B.** The sequence is locked with no decision points for Claude. Multiple tools ≠ agent; who controls the sequence is what matters.

**Q (1.1):** A support bot handles varied requests (returns, subscriptions, resets, billing) and must reason which of four tools to call. Agent or workflow?
A) Workflow — map each intent to a tool · B) Agent — Claude interprets intent and picks the tool · C) Workflow — only four intents · D) Agent — all bots are agents
**Answer: B.** Intent is unpredictable; Claude must reason about which tool fits. A finite intent count doesn't make the sequence pre-scriptable.

**Q (1.2):** Search papers → evaluate results → fetch top three → synthesise. Each step informs the next. Which pattern?
A) Simple loop · B) Branching · C) Multi-step reasoning · D) Validation & retry
**Answer: C.** The action TYPE changes at each step and each result shapes the next — multi-step reasoning, not a repeated action (simple loop) or one decision point (branching).

**Q (1.2):** Charge card; on "insufficient funds" try smaller amount; on "declined" ask for new card. Which pattern?
A) Simple loop · B) Branching by card type · C) Validation & retry · D) Multi-step reasoning
**Answer: C.** Attempt → check outcome → adapt and retry. That's validation & retry.

**Q (1.2):** Receive brief → identify three independent workstreams → one subagent each → collect → synthesise. Which pattern?
A) Simple loop · B) Multi-step reasoning · C) Branching · D) Delegation & aggregation
**Answer: D.** Independent parallel subtasks combined at the end = delegation & aggregation.

**Q (1.3):** Response has stop_reason='tool_use'. Next step?
A) Return the text — done · B) Extract the tool_use block, execute, append both to messages, loop · C) Raise max_tokens and retry · D) Append only the tool_use block and loop
**Answer: B.** tool_use means execute and loop. Append Claude's FULL response.content AND the tool_result before looping.

**Q (1.3):** What role should a tool_result message use?
A) assistant · B) tool · C) user · D) system
**Answer: C.** Tool results go as role `user` — from Claude's view it's external input coming back.

**Q (1.3):** stop_reason='max_tokens' means what, and what do you do?
A) Done — return it · B) Truncated at the cap — raise max_tokens and retry · C) Wants a tool but ran out of room · D) Context full — start over
**Answer: B.** It was cut off at the cap you set. It's not done. Increase max_tokens.

**Q (1.3):** No iteration limit; a tool keeps erroring and Claude keeps retrying. Consequence?
A) Eventually succeeds · B) API stops it at 10 · C) Loops indefinitely, burning tokens until you kill it · D) Returns stop_reason='error'
**Answer: C.** No built-in loop limit exists — always set your own max_iterations.

**Q (2.2):** Global CLAUDE.md and project CLAUDE.md conflict on SQL rules for files in the project. Which wins?
A) Global · B) Project — more specific scope wins · C) Merged, no override · D) Most recently modified
**Answer: B.** More specific scope wins (like CSS specificity). Local > project > global.

**Q (2.7):** Prevent Claude Code from running any command containing 'DROP TABLE'. Where?
A) System prompt · B) A before_bash_command hook that blocks the pattern · C) CLAUDE.md rule · D) tool_choice
**Answer: B.** Hooks run your code and are enforced; instructions in prompts/CLAUDE.md can be ignored.

**Q (2.5):** Nervous about a 50-file refactor. Do what first?
A) Run normally, it'll confirm each file · B) Use plan mode to see the plan before any change · C) Lower max_tokens · D) Run headless
**Answer: B.** Plan mode is the human gate before a risky multi-file operation.

**Q (3.4):** Need reliable JSON with fixed keys; "return JSON" sometimes gives markdown. Best fix?
A) "Don't use markdown" in the prompt · B) Define a tool schema and set tool_choice to force it · C) stop_sequence of ``` · D) temperature=0
**Answer: B.** Tool schemas are enforced; prompt instructions are soft. Claude validates against the schema before returning.

**Q (3.5):** A required 'fix' field sometimes has no value, so Claude invents one. Fix?
A) Keep required, Claude leaves it blank · B) Make it ['string','null'] and remove from required · C) "Write N/A" in the description · D) Two separate tools
**Answer: B.** Nullable + not required lets Claude legitimately return null instead of hallucinating.

**Q (4.2):** Claude must READ the Snowflake schema but not modify it. Which MCP primitive?
A) Tool · B) Resource — read-only, no side effects · C) Prompt · D) Tool with read-only flag
**Answer: B.** Resources are read-only data sources; tools can cause side effects; there's no read-only flag on tools.

**Q (4.4):** A tool description reads "Does database stuff" and Claude misuses it. Root cause?
A) Name too short · B) Description doesn't say WHEN to use it · C) Too many optional fields · D) temperature too high
**Answer: B.** Descriptions should tell Claude when the tool is appropriate, not just what it does.

**Q (4.5):** Your tool raises an exception; you catch it. What do you return?
A) Re-raise and crash the loop · B) Log and return empty string · C) tool_result with is_error: True and a message · D) Retry three times silently
**Answer: C.** Always return a tool_result; is_error:True tells Claude to adapt. Crashing or empty strings mislead it.

**Q (5.2):** 10,000 documents, varied queries, docs don't change. Most cost-effective context strategy?
A) All docs in the system prompt · B) Summarise all into a paragraph · C) RAG — embed, retrieve top-5 per query · D) Cache the full set
**Answer: C.** RAG fits a large corpus with varied queries — retrieve only what's relevant.

**Q (5.4):** Classify 8,000 tickets overnight, results by 9 AM, not real-time. Best approach?
A) Synchronous loop · B) Message Batches API — async, cheaper, bulk · C) Streaming · D) Claude Code headless
**Answer: B.** Batch API is built for bulk async work with no real-time need.

**Q (5.5):** DB write fails with "connection timeout"; three immediate retries fail. Next?
A) Keep retrying forever · B) Exponential backoff, then escalate to a human · C) Raise max_tokens · D) Return stop_reason='error'
**Answer: B.** Timeouts are transient — backoff, then escalate if it persists.

---

## GLOSSARY

- **agent** — Claude drives the loop and decides what to do next.
- **workflow** — you decide the sequence; Claude executes each step.
- **stop_reason** — why Claude stopped: end_turn (done), tool_use (wants a tool), max_tokens (truncated).
- **tool_use** — Claude's structured request to call a function; your code runs it.
- **tool_result** — the tool's output sent back to Claude as a user-role message.
- **tool_use_id** — id on a tool_use block; your tool_result must reference the same id.
- **is_error** — set True in a tool_result to tell Claude the tool failed.
- **context_window** — max tokens (input + output) in one call.
- **token** — ~¾ of a word; you pay per token.
- **system_prompt** — top-level instructions setting role, rules, persona.
- **MCP** — Model Context Protocol; open standard for exposing tools/resources/prompts to Claude.
- **MCP server** — a process implementing MCP and exposing tools to Claude.
- **RAG** — retrieve only relevant document chunks at query time instead of loading everything.
- **prompt caching** — cache a repeated large block server-side to cut cost.
- **CLAUDE.md** — file Claude Code reads for persistent, project-scoped instructions.
- **plan mode** — Claude shows a plan and waits for approval before acting.
- **headless mode** — Claude Code running with no human, e.g. in CI/CD.
- **hook** — your code that runs before/after a Claude Code action for validation or safety.
- **subagent** — a Claude instance spawned by an orchestrating Claude instance.
- **Batch API** — Message Batches API for up to 10,000 async requests, cheaply.
- **few-shot** — showing 2–5 example input→output pairs before the real task.
- **chain-of-thought** — prompting Claude to reason step-by-step before answering.
- **tool_choice** — controls whether Claude must use a tool (any), a specific tool (tool), or decides (auto).

---

## Honesty notes

- Quiz questions are original study aids, not real exam questions.
- Exam logistics (fee, question count, access) may change — confirm on Anthropic's official Skilljar / claude.com/partners page.
- This is an independent study aid, not affiliated with or endorsed by Anthropic.
- Technical details are accurate to documented behaviour as of early 2026; confirm exact parameter names in the official docs (platform.claude.com/docs, code.claude.com/docs, modelcontextprotocol.io).
