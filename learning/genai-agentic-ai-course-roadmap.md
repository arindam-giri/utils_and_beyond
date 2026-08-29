# GenAI + Basic Agentic AI Course Roadmap

A beginner-friendly path from **Generative AI fundamentals** to **building simple agentic systems**.  
Designed for someone who can write basic Python and wants to *build*, not just chat with models.

**Level:** Beginner → Early intermediate  
**Pace:** ~10–12 hours/week  
**Duration:** 10–12 weeks  
**Outcome:** You can explain how LLMs work, ship a RAG app, and build a tool-using agent with memory, evals, and a multi-agent prototype.

---

## How to use this roadmap

- Follow phases in order. Later phases assume earlier ones.
- Every phase ends with a **build**. If you skip the build, you did not finish the phase.
- Prefer **one model + one framework at a time**. Do not collect tools.
- Measure progress by artifacts (repos, notebooks, demos), not courses completed.
- Default stack for this roadmap:
  - Language: **Python 3.11+**
  - Models: OpenAI / Anthropic / Gemini APIs + **Ollama** for local
  - First agent framework: **LangGraph** (mechanics) + **CrewAI** (roles)
  - Observability: **LangSmith** (or equivalent traces)
  - Protocol to know: **MCP** (Model Context Protocol)

---

## Who this is for

- Developers, data folks, or technically curious builders
- People who already use ChatGPT/Claude and want to go past prompting
- Not for: training foundation models from scratch, or enterprise platform architecture (that is a later roadmap)

**Prerequisites (week 0 if missing):**

- Python: functions, classes, `async`, virtualenv, `pip`
- HTTP APIs, JSON, environment variables
- Git + GitHub
- Comfort reading official docs

If Python is weak, spend 1–2 weeks on functions, classes, APIs, and JSON before Phase 1.

---

## Big picture

```text
GenAI basics → Prompts & structured output
     → Embeddings + RAG
          → Tools / function calling
               → Single agent (ReAct loop)
                    → Memory + evals
                         → Multi-agent
                              → Deploy + observe
```

Mental model to keep:

| Thing | What it is |
|---|---|
| **LLM app** | One prompt in, one answer out |
| **Workflow** | Fixed multi-step pipeline you designed |
| **Agent** | Model decides *what to do next* (tools, loops, stop) |
| **Agentic system** | Agent(s) + tools + memory + evals + human checkpoints |

Most production systems in 2026 are **workflows with agentic steps**, not fully autonomous swarms.

---

## Phase 0 — Setup (1–2 days)

**Goal:** A working local lab.

- Python 3.11+, `uv` or `venv`, VS Code or Cursor
- Accounts: OpenAI *or* Anthropic *or* Gemini (pick one paid API)
- Install **Ollama** and pull a small local model (e.g. `llama3.2` or `qwen2.5`)
- Create a GitHub repo: `genai-agentic-lab`
- Learn to store keys in `.env` (never commit secrets)

**Build:** `hello_llm.py` that calls one cloud model and one local model and prints tokens + latency.

---

## Phase 1 — Generative AI foundations (Week 1–2)

**Goal:** Know what an LLM is doing so agents do not feel like magic.

### Topics

- Generative AI vs classical ML vs “just ChatGPT”
- Transformer intuition (attention, next-token prediction) — conceptual, not from-scratch math
- Tokens, context window, temperature, top-p
- Hallucinations: why they happen, how agents make them worse
- Model types: chat models, embedding models, vision/multimodal (overview)
- Cost: tokens in/out, caching, when to use small vs large models
- Safety basics: prompt injection as an idea (you will revisit this with agents)

### Skills

- Call a chat completions / messages API
- Count tokens (tiktoken or provider tokenizer)
- Compare two models on the same task (quality, cost, latency)

### Resources

- 3Blue1Brown *Attention in transformers* (visual intuition)
- Provider docs: OpenAI / Anthropic / Gemini “get started”
- Hugging Face *LLM Course* — chapters on transformers & tokenization (skim)
- Optional book-level: *Hands-On Large Language Models* (concepts only)

### Build

1. **Prompt playground notebook** — same task, 4 temperatures, log outputs.
2. **Cost calculator** — estimate cost of a 10k-token RAG answer vs a 200-token classifier.

**Checkpoint:** You can explain tokens, context limits, and why “just make the prompt longer” fails.

---

## Phase 2 — Prompting that agents actually use (Week 2–3)

**Goal:** Write prompts that are reliable enough to sit inside a loop.

### Topics

- System vs user vs tool messages
- Zero-shot, few-shot, role prompts
- Chain-of-Thought vs “don’t think out loud in production”
- Structured output: JSON schema / Pydantic
- Prompt injection and untrusted tool output
- Eval-driven prompting (change one thing, measure)

### Patterns to practice

- Classifier prompt (labels only)
- Extractor prompt (JSON)
- Planner prompt (numbered steps)
- Critic / reflection prompt (“what is wrong with this draft?”)

### Resources

- OpenAI / Anthropic prompt guides
- Andrew Ng — *ChatGPT Prompt Engineering for Developers* (DeepLearning.AI)
- Your provider’s structured-output docs

### Build

**JSON task router:** given a user message, output `{intent, entities, confidence}` validated by Pydantic. Reject invalid JSON and retry once.

**Checkpoint:** You never parse free-form text with regex when the model can return a schema.

---

## Phase 3 — Embeddings, RAG, and grounding (Week 3–4)

**Goal:** Connect a model to *your* data. RAG is the most common agent tool.

### Topics

- Embeddings and similarity search
- Chunking (size, overlap, by heading)
- Vector stores: Chroma (learn) → Pinecone / pgvector / Qdrant (later)
- Naive RAG pipeline: load → split → embed → retrieve → generate
- Failure modes: bad chunks, lost context, stale docs, citation lies
- When *not* to use RAG (small FAQ, structured DB, a single tool call)

### Skills

- Ingest a folder of markdown/PDFs
- Retrieve top-k chunks with scores
- Force the model to cite sources or say “I don’t know”

### Resources

- LlamaIndex or LangChain RAG tutorial (pick one)
- Pinecone / Chroma “RAG from scratch” guides
- Concept: “RAG is a retrieval tool, not an architecture”

### Build

**Personal knowledge bot** over this roadmap + 10 of your own notes.

Must have:

- Source citations
- “I don’t know” path
- Simple eval set: 15 questions with expected answers

**Checkpoint:** You can point to a wrong answer and say *retrieval failed* vs *generation failed*.

---

## Phase 4 — Tools and the agent loop (Week 5–6)

**Goal:** Leave single-shot chat. This is the start of agentic AI.

### Core idea

An agent is a loop:

```text
observe → reason → pick action (tool or answer) → execute → observe → ...
until stop condition
```

That loop is **ReAct** (Reason + Act). Almost every framework is this loop plus extra state.

### Topics

- Function / tool calling
- Tool schemas (name, description, arguments)
- Deterministic tools vs LLM tools
- Stop conditions: max steps, done flag, user interrupt
- Common tools: web search, calculator, code exec, HTTP GET, SQL, vector retrieve
- MCP at a glance: standard way to expose tools to many clients
- Andrew Ng’s four patterns (learn the *ideas* first, frameworks later):
  1. **Reflection** — critique and revise
  2. **Tool use** — call external functions
  3. **Planning** — break a goal into steps
  4. **Multi-agent** — specialized roles

### Build from scratch first

Write a **20–40 line ReAct loop in raw Python** (no LangGraph yet):

- Model returns either `{"tool": ..., "args": ...}` or `{"final": ...}`
- You execute the tool and append the observation
- Hard cap: 6 steps

Tools: calculator + web search (or a mock search) + current time.

### Then add

- Reflection pass: second model call critiques the draft
- Logging of every thought / action / observation

### Resources

- DeepLearning.AI — **Agentic AI (Andrew Ng)** — vendor-neutral, raw Python
- Provider function-calling docs
- MCP quickstart (read only; implement later)

### Build (project)

**Research mini-agent**

Input: a question  
Output: a 1-page brief with sources

Steps the agent may take: search → read snippet → notes → draft → critique → final.

**Checkpoint:** You can draw the loop on paper and debug a bad tool call from the trace.

---

## Phase 5 — First framework: LangGraph (Week 6–8)

**Goal:** Same loop, but with state, branches, and pause/resume.

Why LangGraph first: it shows the *mechanics* (state, nodes, edges, checkpoints). CrewAI hides that.

### Topics

- `StateGraph`: state schema, nodes, edges
- Conditional routing (“call tool” vs “finish”)
- Checkpointers / memory across turns
- Human-in-the-loop (`interrupt` / approve tool)
- Streaming tokens and node updates
- Time-travel / replay a run (if using LangSmith + checkpointer)

### Minimal graph to implement

```text
START → reason → (tool?) → tools → reason → ... → END
                 ↘ human_approve
```

### Resources

- Official LangGraph quickstart + “ReAct agent from scratch”
- LangSmith tracing tutorial
- Optional: LangChain only as a tool/model wrapper — do not start with LCEL chains

### Builds

1. Port your Phase 4 ReAct agent into LangGraph.
2. Add a **human approval node** before any write action (send email, delete file, run SQL write).
3. Persist thread state so a user can continue tomorrow.

**Checkpoint:** You can explain state, an edge, and why graphs beat linear chains when there are loops.

---

## Phase 6 — Memory, context, and reliability (Week 8)

**Goal:** Agents that do not forget, and do not silently fail.

### Topics

- Short-term: conversation / scratchpad
- Long-term: vector memory vs structured memory (user prefs in a DB)
- Context engineering: what goes in the window, what gets summarized
- Guardrails: max tools, allowlists, output validation
- Evals:
  - Golden questions
  - Trajectory evals (did it call the right tool?)
  - Online traces
- Error analysis: Andrew Ng’s point — this predicts whether you ship

### Build

Add to the research agent:

- Session memory (same user, next day)
- A 20-item eval set with pass/fail
- A one-page “error log”: 5 failures, root cause, fix

**Checkpoint:** You have numbers, not vibes. Example: “14/20 correct, 3 retrieval misses, 2 looped tools.”

---

## Phase 7 — Multi-agent basics (Week 9–10)

**Goal:** Know when a second agent is justified.

Rule: **do not add agents until a single agent with tools is failing for role/clarity reasons.**

### Patterns

- Supervisor + workers
- Sequential crew (researcher → writer → editor)
- Handoffs (OpenAI Agents SDK style)
- Shared state / blackboard (pass a structured object, not a novel)

### Framework #2: CrewAI

Use it to feel the *team metaphor*: role, goal, backstory, task.

Keep the crew small: 2–3 agents.

### Topics

- Role design (narrow beats generic)
- Task contracts (input schema, output schema)
- Debate vs pipeline (pipeline first)
- Cost explosion (N agents × M steps)

### Build

**Content crew**

- Researcher (search + RAG)
- Writer (draft)
- Editor (reflection + fact check)

Output: markdown article + source list.  
Log which agent produced which paragraph.

Optional stretch: same workflow in LangGraph supervisor pattern.

**Checkpoint:** You can argue *why* three agents beat one — or admit they don’t.

---

## Phase 8 — Ship something small (Week 10–12)

**Goal:** Out of the notebook.

### Topics

- FastAPI or Streamlit/Gradio UI
- Auth + rate limits (even if basic)
- Docker
- Secrets, logging, tracing
- Cost and latency dashboard (even a printed table)
- MCP: wrap one of your tools as an MCP server and call it from a client (Cursor / Claude Desktop / a tiny host)

### Production-lite checklist

- [ ] Max step limit
- [ ] Tool allowlist
- [ ] Structured outputs validated
- [ ] Human confirm on side effects
- [ ] Traces for every run
- [ ] Eval suite in CI (even 10 cases)
- [ ] “I don’t know” and fallback

### Capstone (pick one)

1. **Deep research agent** — question → plan → search/RAG → critique → report  
2. **Support agent** — FAQ RAG + ticket tool + escalate to human  
3. **Data analyst agent** — CSV/SQL tool + chart + reflection on wrong queries  
4. **Personal ops agent** — calendar/email *read-only* + draft replies (no auto-send)

Ship: GitHub repo + README + 2-minute demo video/gif + eval results.

---

## 12-week calendar

| Week | Focus | You should have |
|---|---|---|
| 0 | Setup, Python/API rust | `hello_llm.py` |
| 1 | LLM + tokens + cost | Comparison notebook |
| 2 | Prompting + JSON | Task router |
| 3 | Embeddings + chunking | Vector index of your notes |
| 4 | Naive RAG + evals | Cited Q&A bot |
| 5 | Raw Python ReAct | Loop with 3 tools |
| 6 | Reflection + planning | Research brief agent |
| 7 | LangGraph port | Stateful graph + HITL |
| 8 | Memory + eval suite | Scores on 20 cases |
| 9 | CrewAI 2–3 agents | Role-based pipeline |
| 10 | API + UI | Runnable demo |
| 11–12 | Capstone + polish | Public repo |

If you only have 6 weeks: do Phases 1–5 + a thin capstone. Skip CrewAI.

---

## Tooling map (2026, keep it small)

| Layer | Learn now | Learn later |
|---|---|---|
| Models | One cloud API + Ollama | Multi-provider routers |
| Orchestration | LangGraph | OpenAI Agents SDK, Google ADK, Microsoft Agent Framework |
| Multi-agent prototype | CrewAI | AG2 / AutoGen style conversations |
| Data | Chroma + your files | pgvector, hybrid search |
| Tools protocol | MCP concept + 1 server | Full marketplace / many servers |
| Observe | LangSmith (or print traces) | AgentOps, custom telemetry |
| UI | Streamlit or Gradio | Next.js / production frontend |
| Deploy | Docker + one host (Modal / Railway / Fly) | K8s, queues, long-running workers |

**Framework cheat sheet**

- **LangGraph** — state, loops, HITL, production workflows  
- **CrewAI** — fastest role-based prototype  
- **OpenAI Agents SDK** — smallest API if you live in that ecosystem  
- **Raw Python** — required once so frameworks don’t own your mental model  

---

## Recommended courses (use as supplements)

Do **not** binge all of these. Pick one per phase.

| Phase | Course / resource |
|---|---|
| GenAI + prompts | DeepLearning.AI *ChatGPT Prompt Engineering for Developers* |
| LLM intuition | Hugging Face LLM Course (selected chapters) |
| Agentic patterns | DeepLearning.AI **Agentic AI** (Andrew Ng) — best conceptual core |
| RAG | Any official LlamaIndex or LangChain RAG short course |
| Graphs | LangGraph Academy / official tutorials |
| Broader syllabus | Open lists such as *awesome-agentic-ai* style roadmaps (use as a catalog, not a todo) |

Ng’s course is vendor-neutral and uses raw Python. Take it during Phase 4–7.

---

## Project ladder (portfolio)

Build these in order. Each should be a separate folder in one repo.

1. Token + cost playground  
2. Structured JSON extractor  
3. RAG over your notes with citations  
4. ReAct agent from scratch (no framework)  
5. Same agent in LangGraph + human approve  
6. Three-agent research/write/edit crew  
7. Capstone with UI, traces, and eval table  

README for each project: problem, architecture diagram (even ASCII), tools, eval scores, cost per run, what broke.

---

## Habits that matter more than frameworks

1. **Evals before features.** If you cannot score it, you cannot improve it.
2. **Traces over print-debugging.** Save every thought/tool/observation.
3. **Small tools with sharp descriptions.** Bad tool docs = lost agents.
4. **Hard limits.** Max steps, max tokens, allowlisted URLs.
5. **Humans on side effects.** Draft ≠ send.
6. **One variable at a time.** New model *or* new prompt *or* new retriever.
7. **Prefer workflows.** Full autonomy is a product choice, not a default.

---

## What “done with basics” looks like

You are ready for an intermediate roadmap when you can:

- [ ] Explain tokens, context, and why agents hallucinate tool results
- [ ] Ship RAG that cites sources and fails closed
- [ ] Implement ReAct in raw Python
- [ ] Rebuild it as a LangGraph with checkpointing
- [ ] Add one human approval gate
- [ ] Run a 20-case eval and change the system based on error analysis
- [ ] Stand up a 2–3 agent crew *and* know when not to
- [ ] Wrap a tool as MCP or a clean HTTP tool
- [ ] Estimate cost of a run before you launch it

---

## After this roadmap (do not start yet)

- Agentic RAG and long-horizon tasks
- Browser / computer-use agents
- Durable workflows (days-long runs, queues)
- Policy, security, prompt-injection hardening
- Fine-tuning / GRPO-style agent improvement (advanced)
- Multi-user production AgentOps

---

## One-page starter stack

```text
Python 3.11 + uv
OpenAI or Anthropic API
Ollama (local)
LangGraph + LangSmith
Chroma
Pydantic
FastAPI or Streamlit
GitHub
```

Start Phase 0 today. Do not install five frameworks first.
