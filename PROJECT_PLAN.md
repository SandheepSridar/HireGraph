# HireGraph — Project Plan

**Goal:** Resume + JD matching pipeline. A multi-agent AI hiring intelligence platform that automates resume parsing, job matching, bias detection, and candidate ranking using structured LLM outputs and hybrid scoring logic. Each story is a learning milestone, not just a deliverable.

**Stack:** Python, LangGraph, LLM APIs
**Update the progress log in `instructions.md` as you complete stories.**

---

## Story 0 — Environment Setup

**Goal:** Get a local LLM reachable and confirm the LangGraph toolchain actually works before writing any real project code.

**You will learn:**
- How LM Studio's local server exposes an OpenAI-compatible API, and how to point a LangChain client at it
- The shape of a basic tool-calling LangGraph agent, by reading/running a template rather than writing one from scratch

**Tasks:**
- [x] Install `langchain-openai`, `langgraph`, `langgraph-cli[inmem]`
- [x] Load a model in LM Studio and confirm the local server is running
- [x] Run a template tool-calling agent (`scripts/main.py`, `scripts/state.py`) against the local model as a smoke test

**Success criteria:** You understand what each piece of the template does, and you know your local model is reachable — ready to build Story 1's graph from scratch.

---

## Story 1 — Hello Graph

**Goal:** Get a minimal LangGraph running end to end before any LLM or domain logic enters the picture.
**You will learn:**
- `StateGraph`, the state schema (TypedDict or Pydantic model), `add_node`, `add_edge`
- `START` / `END`, `compile()`, `invoke()`
- How state flows and merges between nodes (basic reducers)

**Tasks:**
- [ ] Define a state schema with a couple of fields (e.g. `raw_text: str`, `word_count: int`)
- [ ] Build a 2-3 node graph with plain Python functions (no LLM yet) — e.g. clean text -> count words -> uppercase
- [ ] Compile and invoke the graph on a sample string, print the resulting state
- [ ] Visualize the graph (LangGraph has a `.get_graph().draw_mermaid()` or similar) just to see the shape

**Success criteria:** You can explain, without looking it up, what a "node," an "edge," and "state" mean in LangGraph, and you have a working graph that runs on sample input.

---

## Story 2 — Resume Parser Agent (Structured Output)

**Goal:** Turn one graph node into a real agent: an LLM call that returns schema-constrained structured data instead of free text.

**You will learn:**
- Defining a Pydantic schema for structured output (e.g. `Resume(name, skills, years_experience, education)`)
- Wiring an LLM call with structured/schema-constrained output into a single LangGraph node
- Where parsing failures show up and how to inspect them

**Tasks:**
- [ ] Drop 2-3 sample resumes (plain text or PDF-to-text) into `raw_data/`
- [ ] Define a `Resume` Pydantic schema
- [ ] Write a single-node graph: raw resume text in -> structured `Resume` object out
- [ ] Run it on all sample resumes, sanity-check the structured output against what's actually in the text

**Success criteria:** Given a new resume text file, your graph reliably returns a valid `Resume` object with no manual string-wrangling.

---

## Story 3 — JD Parser + Branching

**Goal:** Add a second parsing agent (job descriptions) and turn the graph from a straight line into one with parallel branches that merge back together.

**You will learn:**
- Fan-out / fan-in: running independent branches (resume parsing, JD parsing) in parallel
- State reducers for merging results from multiple branches into shared state
- Basic conditional edges (routing based on a field in state)

**Tasks:**
- [ ] Define a `JobDescription` Pydantic schema (title, required skills, min years experience, etc.)
- [ ] Add a JD-parsing node using the same structured-output pattern from Story 2
- [ ] Restructure the graph so resume parsing and JD parsing run as parallel branches from a shared entry point, then merge into one state object
- [ ] Add one conditional edge (e.g. skip JD parsing if no JD text was provided)

**Success criteria:** One graph invocation takes a resume + a JD as input and produces a single merged state containing both structured objects.

---

## Story 4 — Hybrid Scoring Agent

**Goal:** Build the matching logic: combine deterministic rule-based signals (e.g. years-experience overlap, skill-set overlap) with an LLM judgment call into one hybrid score.

**You will learn:**
- Mixing plain Python logic and LLM calls within the same node (or as sibling nodes feeding a combiner)
- Conditional routing based on a computed score (e.g. route to "auto-reject" vs "needs review" paths)
- Tradeoffs between rule-based and LLM-based scoring (where each is more/less reliable)

**Tasks:**
- [ ] Write a rule-based scorer: skill overlap %, years-experience delta, etc. — pure Python, no LLM
- [ ] Write an LLM-judgment node: given the structured resume + JD, ask for a fit rating + rationale (structured output again)
- [ ] Combine both into a single hybrid score using a formula you choose (e.g. weighted average)
- [ ] Add a conditional edge that routes candidates below some threshold differently than those above it

**Success criteria:** For a batch of resume/JD pairs, you get a hybrid score plus a rationale, and you can explain why the score is what it is for at least one example.

---

## Story 5 — Bias Detection + Cycles

**Goal:** Add a bias-detection agent that can trigger a retry loop — your first cyclic graph.

**You will learn:**
- Conditional edges that loop back to an earlier node (cycles, not just DAGs)
- Guarding against infinite loops (max retry counts in state)
- Optionally: `interrupt` / human-in-the-loop patterns if you want a manual approval step

**Tasks:**
- [ ] Write a bias-detection node: given the LLM's fit rationale from Story 4, flag language that leans on protected-class signals (age, gender, name-based ethnicity guesses, etc.) instead of job-relevant criteria
- [ ] If bias is flagged, loop back to the scoring node and ask for a re-judgment with a note not to use flagged reasoning — cap retries (e.g. max 2) via a counter in state
- [ ] Log every retry attempt so you can see the loop working

**Success criteria:** You can force a biased-sounding rationale (test input) and watch the graph loop back and self-correct at least once, then stop looping via the retry cap.

---

## Story 6 — Ranking + Supervisor Orchestration

**Goal:** Tie parsing, scoring, and bias-detection agents together under a supervisor graph that orchestrates the whole pipeline across multiple candidates.

**You will learn:**
- The supervisor/orchestrator pattern — a top-level graph that calls into subgraphs (each story's graph becomes a subgraph or reusable node group)
- Running the pipeline over a batch of candidates and aggregating into a ranked list
- Checkpointing/persistence (e.g. `MemorySaver` or similar) so a run's state can be inspected or resumed

**Tasks:**
- [ ] Wrap the Story 2-5 graphs as subgraphs (or reusable node groups) inside one top-level supervisor graph
- [ ] Feed in a batch of resumes against one JD, run each through the full pipeline
- [ ] Aggregate results into a ranked list sorted by hybrid score, with bias flags visible
- [ ] Add checkpointing so you can inspect intermediate state for any candidate after the run

**Success criteria:** One invocation of the top-level graph takes N resumes + 1 JD and returns a ranked, bias-annotated candidate list.

---

## Final Deliverable — Tying It Together

**Goal:** A working end-to-end CLI pipeline: point it at a folder of resumes and a JD, get back a ranked, bias-flagged candidate report — and a clear mental model of LangGraph you can explain to someone else.

**Tasks:**
- [ ] Wire the Story 6 supervisor graph behind a simple CLI entry point in `main.py` (input: resumes folder + JD file; output: ranked report, printed or written to file)
- [ ] Run it against your own real job-search materials (your resume + a few JDs you're actually targeting) as the real-world test
- [ ] Write up, in your own words, the LangGraph concepts you used and where each showed up in this project (this is the actual learning artifact — keep it in `instructions.md` or a short doc)

**Success criteria:** You can run the pipeline on your own resume against a real JD, get a sensible ranked/scored/bias-checked output, and explain the graph architecture from memory without re-reading the code.
