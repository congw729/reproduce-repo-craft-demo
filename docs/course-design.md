# SwarmCraft — Multi-Agent Collaboration with JiuwenSwarm
## A Hands-On Course Built on Reproducing Vibe-Trading

**Course ID:** swarmcraft-course-01
**Version:** 1.0 (2026-09-03)
**Target audience:** Students with basic Python + git skills; no prior swarm/agent experience required.
**Duration:** 2.0 hours core (1.5 h and 2.5 h variants included)
**Class size:** 12-40 students, working in mini-swarms of 4-5
**Hardware:** Modest classroom machines (4 GB+ RAM, any 64-bit OS); one instructor demo machine with projector.
**Software:** Python 3.11/3.12, git, a terminal, and **one DeepSeek API key** (instructor-provided shared key, or students bring their own).
**Constraint respected:** No OpenAI GPT models are used anywhere in the stack; DeepSeek (or open-source models via Ollama/Groq) are the LLM backends.

---

## 1. Learning Objectives

The PRIMARY goal is teaching **how to work with multi-agent systems** (JiuwenSwarm-style): the mechanics of decomposing a goal into tasks, assigning roles, spawning members, creating a task DAG, communicating between members, approving plans, and running review/verification loops. Reproducing a slice of Vibe-Trading is the **vehicle** that makes these concepts concrete and observable.

| # | Learning Objective (by the end, students can...) | Mapped Multi-Agent Workflow Concept |
|---|---|---|
| LO1 | Decompose a vague goal ("make Vibe-Trading run") into well-scoped, deliverable-shaped tasks | Task decomposition; deliverable-focused task writing |
| LO2 | Assign tasks to roles that match expertise (researcher vs developer vs reviewer) | Role assignment; capability matching |
| LO3 | Spawn/instantiate team members with a stated purpose and boundary | Spawning members (`spawn_teammate`) |
| LO4 | Build a task DAG with dependencies and verify ordering | Task DAG creation (`create_task` + blocked_by) |
| LO5 | Communicate deliverables via shared workspace + direct messages/broadcasts | Inter-member communication channels |
| LO6 | Approve a plan before execution and re-plan when blocked | Plan approval gate; escalation |
| LO7 | Run a review/verification loop: pass completes, fail returns for rework | Review/verification loops (`verify_task`) |
| LO8 | Reflect on where multi-agent collaboration adds value vs. friction | Meta-cognition; collaboration economics |

**Secondary (vehicle) objectives:** understand Vibe-Trading's architecture at a high level (data loaders → strategy → backtest engine → report) and run at least one real backtest on the student's own machine.

---

## 2. Session Timeline

### 2.1 Core schedule (2.0 h)

| Phase | Time | Content | Format |
|---|---|---|---|
| 0. Welcome & Hook | 0:00-0:10 | Why multi-agent? Real example: this very course was designed by a multi-agent swarm. Poll the class. | Lecture + poll |
| 1. Core Concepts | 0:10-0:30 | Swarm anatomy: leader / members / task board / DAG / messages / plan approval / review loops. Vocabulary table (Appendix A). | Interactive lecture |
| 2. Live Demo | 0:30-0:55 | Instructor runs a real swarm live: build_team → spawn → create_task → broadcast → review → complete. Screen-share, narrate every command. | Live demo |
| 3. Hands-On: Mini-Swarms | 0:55-1:40 | Students form mini-swarms (4-5), reproduce a small Vibe-Trading slice (see Section 4). | Group exercise |
| 4. Show & Debrief | 1:40-1:50 | 2-3 teams demo their backtest result and their task-board journey. | Team demos |
| 5. Discussion & Reflection | 1:50-2:05 | Guided questions (Section 5), exit ticket. | Discussion |
| Buffer | 2:05-2:10 | Overflow / troubleshooting / next steps. | — |

### 2.2 Variants

- **Short (1.5 h):** cut Phase 4 to quick verbal report-outs (no screen demo); trim Phase 1 vocabulary table to the 8 core terms; exercise scope drops the Scribe task (leader absorbs it).
- **Extended (2.5 h):** add a 20-min "second iteration" round where teams re-run their slice with a different strategy (SMA 10/30) and observe the DAG replay only the failed subgraph on error; add a 15-min cross-team artifact hand-off (two mini-swarms exchange results and critique each other's reports).

---

## 3. Live Demo Script (Instructor, Leader Perspective)

The instructor acts as **team leader**. All commands are shown on the projector; students copy the *pattern*, not the repo.

### 3.1 Setup (pre-class; full checklist in Section 6)

- Terminal 1: JiuwenSwarm CLI ready.
- Terminal 2: Vibe-Trading workspace ready at `~/vibe-demo` (clone exists, `.env` configured with DeepSeek, one demo backtest already run once as a fallback).

### 3.2 Script beats (narrate each)

**Beat A — build the team.** Explain: a swarm starts with a goal, not with code.

```
> build_team(display_name="Vibe-Repro-Swarm",
             goal="Reproduce a minimal runnable slice of Vibe-Trading")
```

Narrate: we gave the team a goal; the leader does not execute tasks — members do.

**Beat B — spawn members with boundaries.**

```
> spawn_teammate("repo-researcher",  role="analyze the repo; output facts only")
> spawn_teammate("quant-developer",  role="write and run a minimal backtest")
> spawn_teammate("qa-reviewer",      role="verify backtest outputs; may reject")
> spawn_teammate("doc-writer",       role="write a 10-line run summary")
```

Narrate: each spawn is a role with a boundary — QA cannot write code, doc-writer cannot change code. Boundaries are what make the review loop meaningful.

**Beat C — create the task DAG.**

```
> create_task(task-1 "Identify minimal run path" assignee=repo-researcher)
> create_task(task-2 "Write minimal backtest"    assignee=quant-developer blocked_by=[task-1])
> create_task(task-3 "Verify outputs"            assignee=qa-reviewer   blocked_by=[task-2])
> create_task(task-4 "Write run summary"         assignee=doc-writer    blocked_by=[task-3])
```

Narrate: DAG = dependencies; task-4 waits for task-3, which waits for task-2, which waits for task-1. Show the board.

**Beat D — broadcast kickoff.** Members only start when told:

```
> broadcast("Kickoff! Claim your tasks. Save artifacts under .team/ — messages carry paths, not paste.")
```

**Beat E — watch the state machine.** Show the board as tasks flow:
`pending → in_progress → in_review → completed`
Point out: when task-1 completes, task-2 unlocks; no polling needed — events push.

**Beat F — run the review loop (the money shot).**
- QA reviewer marks task-3 `fail` with feedback (e.g., "Backtest window starts before data begins — fix warmup").
- Show task-3 flip back to `in_progress`, quant-developer reworks, re-submits.
- QA marks `pass` → task-3 → completed, task-4 unlocks.

Narrate: *a review loop is a quality gate, not a formality — and the DAG means rework only re-runs the failed subgraph.*

**Beat G — wrap up.** Leader broadcasts completion, collects artifacts, shows final board.

**Beat H — the meta-insight (optional, 3 min).** Open Vibe-Trading's own swarm engine: `agent/src/swarm/presets/equity_research_team.yaml` (macro_analyst → sector_analyst → stock_picker → aggregator). Show that the very project we are reproducing has the same DAG+roles pattern — *JiuwenSwarm-style collaboration is a transferable skill, not a tool quirk.*

**Timing:** ~25 min including narration; allow pauses for questions.

---

## 4. Hands-On Exercise: Mini-Swarm Reproduces a Slice of Vibe-Trading

### 4.1 Setup
- Teams of 4-5: each student takes one role (Leader / Researcher / Developer / Reviewer / Scribe).
- Every student works on their **own machine** (modest hardware is fine; no GPU needed).
- **Pre-warmed data cache** and pre-installed packages (instructor checklist in Section 6) to survive slow classroom Wi-Fi.

### 4.2 The slice (scope tightly)
Goal: *"Run one moving-average crossover backtest with Vibe-Trading's engine, and produce a 5-line report."*

This maps to the repo's real pipeline: **data loader → strategy → backtest engine → metrics/report**, and it works with **zero data API keys** (crypto via OKX public API; US/HK equities via Yahoo/yfinance; A-shares via mootdx/AKShare — all free, no key).

### 4.3 Role cards & tasks (mini-DAG)

| Task | Owner | Deliverable | Notes |
|---|---|---|---|
| T1 | Researcher | Install `vibe-trading-ai` in a venv; verify `vibe-trading --version`; note which config keys are required | Facts to `.team/notes/install.md` |
| T2 | Data member | Confirm a free no-key data path (e.g., OKX for crypto or yfinance for equities); record the chosen tickers | `.team/data/plan.md` |
| T3 | Strategy member | Define SMA(20)/SMA(50) crossover parameters and the exact natural-language prompt | 1-page pseudo-code + prompt |
| T4 | Developer | Run the one-shot backtest via the CLI; export equity curve + metrics (return, drawdown, Sharpe) | `.team/results/backtest.json` |
| T5 | Reviewer | Check: data window vs evaluation window; metrics finite and sane; artifacts exist; reject with feedback if not | Pass/Fail + feedback |
| T6 | Scribe | Write a 5-line run summary; broadcast to team | `.team/summary.md` |

Dependencies: T2 ∥ T3 → T4 → T5 → T6. T1 is the team's first task and unlocks everything.

### 4.4 Exact commands (from repo-analysis.md Section 5.5 — verified)

Option A — one-shot CLI backtest (recommended for the exercise; no UI needed):
```bash
python -m venv .venv && source .venv/bin/activate
pip install vibe-trading-ai
vibe-trading init                  # interactive .env setup (DeepSeek key)
vibe-trading run -p "Backtest a BTC-USDT 20/50 moving-average strategy for 2024 and summarize return and drawdown"
```

Option B — Docker (only if student machines lack Python 3.11; sidesteps Node requirements):
```bash
git clone https://github.com/HKUDS/Vibe-Trading.git && cd Vibe-Trading
cp agent/.env.example agent/.env
# edit agent/.env: LANGCHAIN_PROVIDER=deepseek, LANGCHAIN_MODEL_NAME=deepseek-chat, DEEPSEEK_API_KEY=...
# (the repo's own .env.example may read deepseek-v4-pro — either id works; see llm-provider-guidance.md §2.4)
docker compose up --build          # then open http://localhost:8899
```

**Key facts to tell students:** the only credential needed is one DeepSeek API key; data sources are key-free; **no GPU required** (LLM compute is remote, backtests are CPU-bound).

### 4.5 Exercise protocol (25 min work + 10 min report)
1. **5 min:** Leader (student) builds the mini-swarm, spawns roles, creates the DAG, broadcasts kickoff.
2. **20 min:** Members claim tasks, communicate via the shared `.team/` workspace + messages, honor the DAG.
3. **10 min:** At least one full review loop must happen; teams that finish early run a second strategy variant (SMA 10/30) and compare.
4. Instructor roams; the **focus is on process**, not trading performance — remind teams that a "bad" backtest is a *successful exercise* if the review loop caught it.

### 4.6 Modest-hardware & classroom guardrails
- All heavy work is pre-cached; students never download large datasets at runtime (cache pre-warmed by instructor).
- If a machine can't run the engine, that student still participates as "observer-reporter"; role assignments are flexible.
- No live market data streaming beyond the one-shot download, **no brokerage connections, no real money anywhere** (project safety defaults: loopback-only binding; shell tools opt-in; live trading explicitly out of scope).
- If classroom Wi-Fi is slow, the instructor runs one shared demo machine and teams "borrow" its output for their review loop — the process lesson survives even if the compute doesn't.

---

## 5. Discussion & Reflection Questions

1. Where did task decomposition change *how* you thought about the problem?
2. What made a task description good vs. vague? What happened when a task was vague?
3. How is a review loop different from "the teacher checks your homework"? Who benefited from the fail → rework cycle?
4. When would you NOT want a swarm? (hint: tiny tasks, real-time latency, single-expert problems)
5. The leader never wrote code in the demo — did the leader matter? What did the leader actually do?
6. What would break if members ignored the DAG and all started at once?
7. Model-agnostic check: our swarm used DeepSeek/open models — what changes if members are humans, Claude, or local models? What stays the same?
8. **Meta:** we reproduced a project that *itself* contains a swarm DAG engine (`agent/src/swarm/presets/`, 30 teams). What does it mean that "multi-agent" is both our tool *and* our subject?
9. **Exit ticket (2 min, written):** name one workflow concept you'll apply next week.

---

## 6. Instructor Checklist

### Environment pre-checks (60 min before)
- [ ] Instructor machine: Python 3.11/3.12 (`python --version`), git, JiuwenSwarm CLI launches.
- [ ] Instructor machine: `vibe-trading-ai` installed in a demo venv; one backtest already run end-to-end once (screenshots ready as fallback).
- [ ] Projector + screen-sharing verified; fonts large enough for back-row.
- [ ] Classroom machines: Python 3.11+ verified on 3 random machines; venv creation works.
- [ ] Wi-Fi: verify `pip install` works at classroom speed; if slow, pre-bake wheels/offline packages and set `VIBE_TRADING_DATA_CACHE=1` with a pre-warmed cache (`~/.vibe-trading/cache/loaders/`).
- [ ] DeepSeek API key: instructor key configured; policy decided (shared classroom key vs. student keys). **No OpenAI keys — project constraint.**
- [ ] `.team/` shared workspace exists on each team's setup; write permission verified.
- [ ] Timebox alarms/timers visible in the classroom.
- [ ] Backup: printed copies of demo script (Section 3) and role cards (Section 4) in case screens fail.
- [ ] Verify `docker compose` path works if any student machine lacks Python 3.11 (Node >= 22.22 only matters for frontend dev — the CLI/backtest path does NOT need Node).

### During class
- [ ] Start recording or have a TA for the demo phase.
- [ ] Roam during the exercise; unblock stuck teams by asking *"what does your DAG say?"* — not by solving it for them.
- [ ] Watch for teams that skip the review loop; nudge them.

### After class
- [ ] Collect exit tickets; harvest 3 best team summaries for a follow-up email.
- [ ] Note hardware/network issues → feed back to this checklist.

---

## Appendix A — Vocabulary (Phase 1 handout)

| Term | Meaning |
|---|---|
| Leader | Orchestrator: defines goal, splits work, reviews outcomes. Does not execute tasks. |
| Member / Teammate | Executes assigned tasks within a role boundary. |
| Task board | Shared list of tasks with states (pending → in_progress → in_review → completed/cancelled). |
| Task DAG | Dependency graph: a task starts only when its `blocked_by` tasks complete. |
| Claim | A member self-assigns a pending task. |
| Spawn | Instantiate a new member with a named role/purpose. |
| Broadcast / DM | Channel-wide vs. point-to-point messaging between members. |
| Plan approval | Gate where the leader approves a plan before execution starts. |
| Review / Verify | A reviewer checks deliverables; pass completes the task, fail returns it for rework. |
| Shared workspace | A versioned folder (`.team/`) where artifacts live; messages carry paths, not content. |

## Appendix B — Vibe-Trading Facts Used in This Design (verified against the actual repo, v0.1.14)

- Python >= 3.11 (recommend 3.11/3.12); PyPI package `vibe-trading-ai`; MIT license; console entries `vibe-trading` (CLI, `cli:main`) and `vibe-trading-mcp` (MCP server).
- Backend: LangChain 1.x + LangGraph, FastAPI, pandas/numpy; frontend: React 19 + Node >= 22.22 (not needed for the CLI path); optional Electron desktop; Docker with hash-pinned lock files.
- Multi-market backtest engines (8+) and 24 data loaders; **key-free data paths**: OKX (crypto), Yahoo/yfinance (US/HK/UK), mootdx/AKShare/baostock (A-shares).
- LLM providers: DeepSeek is the README "sweet-spot default" (`deepseek-v4-pro`); Ollama gives a zero-key local option (use 7B/8B models in labs; avoid nano models — unreliable tool-calling). **No OpenAI/GPT requirement anywhere.**
- Swarm engine: `agent/src/swarm/` with 30 YAML team presets; DAG orchestrator runs workers in topological layers, retries failed subgraphs.
- Hardware: **no GPU required** (LLM compute is remote; backtests are CPU-bound); Docker compose default caps at 4 GB RAM / 2 CPUs.
- Safety: API binds loopback by default; shell-capable tools opt-in; live broker trading explicitly out of scope for this course.

## Appendix C — Trimming & Extension Options

- **Trim (1.5 h):** drop T6, merge Scribe into Leader; Phase 4 becomes verbal report-outs only.
- **Extend (2.5 h):** strategy-variant re-run (SMA 10/30) with DAG subgraph-replay observation; cross-team artifact hand-off with peer critique.
- **Advanced follow-up:** have students open `agent/src/swarm/presets/*.yaml` and redesign the `equity_research_team` DAG (add a risk-reviewer node, change context passing) — turning the reproduction exercise into a design exercise.
