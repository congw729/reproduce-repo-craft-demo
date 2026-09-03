# Verification Checklist — SwarmCraft: Reproduce Vibe-Trading with a Multi-Agent Team

This checklist lets an instructor or student confirm the course environment is set up
correctly **before** and **during** the reproduction milestones. It is grounded in
`docs/repo-analysis.md` (v0.1.14 checkout), `docs/reproduction-plan.md` (Milestone 0–6
Verify blocks), `docs/course-design.md` (learning objectives), and
`docs/llm-provider-guidance.md` (no-OpenAI LLM setup).

---

## 1. Environment Prerequisites (pre-course)

| # | Check | How to verify | Source |
|---|---|---|---|
| 1.1 | Python 3.11 (or 3.12) available | `python --version` → `3.11.x` or `3.12.x` | repo-analysis §2.1, §5.1 |
| 1.2 | Node >= 22.22 available (frontend/desktop path only) | `node --version` → `>= 22.22.0` | repo-analysis §5.1 |
| 1.3 | Docker available (Path A fallback) | `docker --version` and `docker compose version` succeed | repo-analysis §5.5 |
| 1.4 | Outbound HTTPS works (market data needs live network) | `curl -sI https://query1.finance.yahoo.com` returns 200 | repo-analysis §5.3 |
| 1.5 | DeepSeek API key configured, **or** Ollama ready (no-key fallback) | `vibe-trading provider doctor` shows provider=deepseek (or ollama) | llm-provider-guidance §2, §3 |

> **Course default:** DeepSeek API (`LANGCHAIN_PROVIDER=deepseek`, `LANGCHAIN_MODEL_NAME=deepseek-chat`).
> **Offline fallback:** Ollama (`qwen2.5:7b`, no key). Never use OpenAI/GPT models — see
> llm-provider-guidance §5 decision table.

## 2. Install Verification (per chosen path)

| # | Path | Check | Source |
|---|---|---|---|
| 2.1 | Any | `pip install vibe-trading-ai` succeeds; `vibe-trading --help` prints usage | repo-analysis §3.1 |
| 2.2 | A. Docker | `docker compose up --build` starts; `http://localhost:8899` loads | repo-analysis §5.5 |
| 2.3 | B. Local | `python -m venv .venv && pip install -e .` succeeds; `vibe-trading` TUI launches | repo-analysis §5.5 |
| 2.4 | Both | `vibe-trading init` completes the interactive `.env` setup | repo-analysis §3.2 |
| 2.5 | LLM config | `agent/.env` sets `LANGCHAIN_PROVIDER=deepseek`, `LANGCHAIN_MODEL_NAME=deepseek-chat`, `DEEPSEEK_API_KEY`; **no OpenAI config active** | llm-provider-guidance §2 |

> **Model-name note:** the repo's own `.env.example` may read `deepseek-v4-pro` — either id
> works (the model string is just a label sent to the API). `deepseek-chat` is the stable
> public id recommended for the course; see llm-provider-guidance §2.4.

## 3. Smoke Test (one-shot research)

```bash
vibe-trading run -p "Backtest a BTC-USDT 20/50 moving-average strategy for 2024 and summarize return and drawdown"
```

| # | Expected result | Check |
|---|---|---|
| 3.1 | Agent completes without crashing | Command exits 0 |
| 3.2 | Strategy code generated, not fabricated from memory | Output shows a run artifact / code path |
| 3.3 | Return and drawdown metrics reported | Numbers present in output |
| 3.4 | No OpenAI/GPT calls made | `.env` uses DeepSeek/Ollama only |

## 4. Milestone Gates

Each milestone is complete **only when its Verify block passes** (reproduction-plan §0.4).
Quick gates:

| Milestone | Verify command(s) | Pass condition |
|---|---|---|
| M0 — Env & clone | `python3 -c "import sys; assert sys.version_info >= (3,11); print('python OK')"`; `ls agent/.env.example` | Both succeed |
| M1 — Install & LLM config | `vibe-trading --help`; `vibe-trading provider doctor` | Both run without import errors (doctor may say "no key" for Ollama — expected) |
| M2 — First backtest | `vibe-trading run -p "Backtest a BTC-USDT 20/50 MA strategy for 2024 and summarize return and drawdown"` | Exits 0; output has return + drawdown; a run dir exists under `~/.vibe-trading/runs` |
| M3 — Alpha zoo | `vibe-trading alpha list`; `vibe-trading alpha bench --zoo gtja191 --universe csi300 --period 2023-2024 --top 10` | Families/counts ≥ 400 total; bench prints ranked table (IC + alive/reversed/dead) |
| M4 — Web UI | `vibe-trading serve --port 8899`; `curl -s http://localhost:8899/live` | Page loads; health check passes; a chat task produces a run card |
| M5 — Multi-agent swarm | `vibe-trading --swarm-presets`; a small `--swarm-run` (e.g. `investment_committee`) | ≥ 10 presets listed; run completes; preset YAML shows ≥ 2 roles + ≥ 1 `depends_on` edge |
| M6 — Shadow account | `vibe-trading --upload trades_export.csv` + analysis prompt | Behavior profile + shadow-strategy comparison produced; HTML report exists |

**Learning-outcome gates** (from course-design §1): by course end students can decompose
goals into tasks (LO1), assign roles (LO2), spawn members (LO3), build a task DAG (LO4),
communicate via shared workspace + messages (LO5), approve plans / re-plan (LO6), and run
review/verification loops with pass/fail rework (LO7), and reflect on when multi-agent adds
value vs. friction (LO8).

## 5. Safety & Classroom Hygiene

| # | Check | Why |
|---|---|---|
| 5.1 | No live-trading connectors used in class | Out of scope for reproduction course (repo-analysis §5.6) |
| 5.2 | API stays loopback-only; no `API_AUTH_KEY` leaks | Security default (repo-analysis §5.6) |
| 5.3 | Shell-capable tools stay opt-in (`VIBE_TRADING_ENABLE_SHELL_TOOLS` off) | Classroom security (repo-analysis §5.6) |
| 5.4 | No `.env` / keys committed to git | Hygiene gate (repo-analysis §5.6, `.gitignore`) |
| 5.5 | No OpenAI/GPT models configured anywhere | Project-wide constraint (llm-provider-guidance) |

---

*Checklist grounded in `docs/repo-analysis.md` (v0.1.14 checkout), `docs/reproduction-plan.md`
(Milestone 0–6), `docs/course-design.md` (LO1–LO8), and `docs/llm-provider-guidance.md`.*
