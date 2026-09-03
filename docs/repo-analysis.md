# Vibe-Trading Repository Analysis Report

**Repository:** HKUDS/Vibe-Trading (local checkout at `/Users/congwang/Documents/GitHub/Vibe-Trading`)
**Analyzed by:** Repository Analyst (repo-researcher)
**Date:** 2026-09-03
**Version analyzed:** `0.1.14` (per `pyproject.toml`)

---

## 1. Top-Level Structure and Module Responsibilities

The repository is a monorepo with a Python backend, a React web frontend, an optional Electron desktop shell, a static wiki, and CI/dev tooling.

```
Vibe-Trading/
├── agent/                          # Python backend + package source (installed as `vibe-trading-ai`)
│   ├── cli/                        # CLI package — interactive TUI + subcommands (entry: `cli:main`)
│   ├── api_server.py               # FastAPI server — runs, sessions, upload, swarm, SSE
│   ├── mcp_server.py               # MCP server — 74 research tools (stdio / SSE / HTTP)
│   ├── SKILL.md                    # Agent-facing skill doc describing how to use the repo
│   ├── requirements.txt            # Human-edited dependency source
│   ├── requirements-channels.txt   # IM channel SDK extras
│   ├── .env.example                # Canonical environment template (~17 KB, heavily commented)
│   ├── src/                        # Core application code (see §4)
│   ├── backtest/                   # Backtest engines + data loaders + portfolio optimizers
│   ├── scripts/                    # Benchmarks & maintenance scripts
│   ├── skills/                     # Extra bundled skill (ashare-mootdx)
│   └── tests/                      # 555 test files (unit + integration)
├── frontend/                       # Web UI — React 19 + Vite + TypeScript
│   ├── package.json                # Node >= 22.22 required
│   ├── src/                        # pages (Home, Agent, AlphaZoo, RunDetail, Compare, Correlation, Settings), components, stores (Zustand)
│   └── vite.config.ts              # dev proxy to API (port 5899 → 8899)
├── desktop/electron/               # Unofficial community Electron desktop build (Windows-focused)
│   ├── package.json                # electron 43.x, electron-builder
│   ├── src/                        # main/preload/renderer code
│   └── scripts/                    # build-backend.ps1, packaging, smoke tests
├── wiki/                           # Public documentation site (Cloudflare Pages + Wrangler)
│   ├── docs/, home/, research-lab/, tutorials/, alpha-library/
│   └── wrangler.toml               # CF Pages deployment config
├── scripts/dev                     # Local dev orchestration: up/stop/restart/status/logs/open
├── tools/                          # CI hygiene gates (not shipped)
│   ├── ci_env_var_gate.py          # blocks undeclared env vars in CI
│   ├── ci_grep_gates.sh            # blocks forbidden strings (e.g., hardcoded keys)
│   └── test_ci_env_var_gate.py
├── .github/workflows/              # test.yml, docker-build.yml, wiki-deploy.yml, wiki.yml, desktop-windows.yml
├── .devcontainer/devcontainer.json # VS Code dev container (python:3.11-bookworm + node 20)
├── pyproject.toml                  # Package manifest (PEP 621)
├── requirements-lock.txt           # uv-compiled, hash-pinned lock (~351 KB)
├── requirements-channels-lock.txt  # uv-compiled, hash-pinned lock for channel SDKs
├── Dockerfile                      # Multi-stage build (frontend + python wheels → slim runtime)
├── docker-compose.yml              # Backend (8899) + optional frontend (5899) services
├── README.md (+ ar/es/ja/ko/zh)    # Very large main docs (~268 KB English)
├── AGENT_CONTRIBUTOR_GUIDE.md      # Safety/verification expectations for AI-assisted contributors
├── CHANGELOG.md, CONTRIBUTING.md, SECURITY.md, LICENSE (MIT), NOTICE
└── .gitignore                      # Excludes .env, sessions/runs/uploads, broker live state
```

**Key responsibilities at a glance:**

| Path | Responsibility |
|---|---|
| `agent/` | All Python logic: CLI, REST API, MCP server, ReAct agent core, skills, tools, swarm multi-agent engine, backtests, data loaders, memory, live-trading connectors. |
| `frontend/` | Browser UI: chat, run cards, alpha zoo browser, correlation view, settings. Talks to the API over SSE/HTTP. |
| `desktop/electron/` | Unofficial community desktop app that bundles the Python backend as a runtime and embeds the frontend. Windows packaging focus. |
| `wiki/` | Public documentation site deployed to Cloudflare Pages; separate CI checks (wiki.yml). |
| `scripts/dev` | One-command local dev loop (backend + frontend dev servers with PID/log management). |
| `tools/` | CI-only gates that enforce environment-variable and secret hygiene; not part of the runtime package. |

---

## 2. Tech Stack

### 2.1 Python backend (`pyproject.toml`)

- **Python:** `requires-python = ">=3.11"` (classifiers list 3.11/3.12/3.13; CI/lock targets 3.11).
- **Package:** `vibe-trading-ai`, version `0.1.14`, MIT license, authors HKUDS.
- **Orchestration / LLM framework (LangChain 1.x):**
  - `langchain>=1.3.9,<2`, `langchain-core>=1.0.0,<2`, `langchain-openai>=1.0.0,<2`
  - `langgraph>=1.2.5,<1.3`, `langgraph-checkpoint>=2.1.0,<5`
  - Optional adapters: `langchain-deepseek` (extra `deepseek`), `langchain-anthropic` (extra `anthropic`)
- **Data & numerics:** `pandas>=2.0,<3.0`, `numpy>=1.24`, `scipy>=1.10`, `bottleneck`, `duckdb>=1.2`, `scikit-learn`, `joblib`; optional `statsmodels`/`arch` (extra `stats`).
- **Market data sources:** `tushare>=1.2.89`, `yfinance>=0.2.30`, `akshare>=1.12.0`, `ccxt>=4.5.71`, `aiohttp`, `cryptography`, `defusedxml`; optional `baostock` (A-shares), `pykrx` (Korea), `MetaTrader5` (Windows-only), plus lazy integrations for Futu, LongBridge, IBKR, Tickerall, OpenBB.
- **API / serving:** `fastapi>=0.104`, `uvicorn[standard]`, `websockets>=12`, `pydantic>=2`, `python-multipart`, `sse-starlette`.
- **MCP:** `fastmcp>=2.14.0,<4.0.0` (version-capped: 4.0 changed `MCPError` constructor).
- **Web search:** `ddgs>=6.0.0` (DuckDuckGo).
- **Document/PDF/reporting:** `openpyxl`, `python-docx`, `python-pptx`, `pypdfium2`, `Pillow`, `jinja2`, `matplotlib`, `weasyprint>=60`.
- **CLI UX:** `rich>=13`, `prompt_toolkit>=3`, `pyyaml`, `httpx`, `h2`, `oauth-cli-kit`.
- **Console scripts** (`[project.scripts]`):
  - `vibe-trading` → `cli:main`
  - `vibe-trading-mcp` → `mcp_server:main`
- **Extras:** `ibkr`, `longbridge`, `mt5`, `deepseek`, `copilot`, `anthropic`, `openbb`, `krx`, `stats`, `ashare`, `harmonic`, `smc`, `keyring`, and 15+ IM-channel extras (`dingtalk`, `discord`, `feishu`, `matrix`, `slack`, `telegram`, `wecom`, `whatsapp`, `qq`, `msteams`, `mochat`, `napcat`, `weixin`, `channels`, `dev`).

### 2.2 Frontend (`frontend/package.json`)

- **Runtime:** Node `>=22.22.0` (a hard requirement).
- **Framework:** React `^19.2`, TypeScript `^5.7`, Vite `^8.2`, TailwindCSS `^3.4`.
- **Libraries:** `echarts` (charts), `zustand` (state), `react-markdown` + `katex` + `highlight.js` (rendering), `i18next` (i18n), `sonner` (toasts), `lucide-react` (icons), `clsx`/`tailwind-merge`.
- **Tests:** Vitest + Testing Library + jsdom.

### 2.3 Desktop (`desktop/electron/package.json`)

- Electron `43.1.1`, electron-builder `26.15.3`, TypeScript.
- Ships the Python backend as an embedded runtime (`scripts/build-backend.ps1`, `requirements-windows-lock.txt`); Windows-only packaging pipeline (`pack:win`, signed installer scripts, SBOM via `npm sbom`).

### 2.4 Docker & CI

- **`Dockerfile`:** multi-stage — Node 22 builds the frontend; `python:3.11-slim` compiles hash-pinned wheels; slim runtime copies the prebuilt venv. Base images pinned by **SHA256 digests**. Runtime runs as non-root `vibe` user with `cap_drop: ALL`, `read_only: true`, `mem_limit: 4g`, `cpus: 2`, `pids_limit: 512`, plus a `vibe-sandbox` unprivileged user for LLM-generated-code subprocesses.
- **`docker-compose.yml`:** backend on `127.0.0.1:8899`, optional frontend profile on `5899`; named volumes for runs/sessions/home/swarm/uploads; Taiwan stock data mounted read-only from outside the checkout.
- **CI (`test.yml`, `docker-build.yml`):** pytest (network-blocked unit suite), frontend `npm ci && npm run build`, Docker image build.
- **Dependency pinning:** `requirements-lock.txt` is generated by `uv pip compile --universal --python-version 3.11 --generate-hashes` — **every transitive dependency is hash-pinned**; Docker installs with `--require-hashes`. Channel SDKs get a second constrained lock (`requirements-channels-lock.txt`).

## 3. Core Workflows and Entry Points

### 3.1 Entry points

| Entry point | Where | Purpose |
|---|---|---|
| `vibe-trading` | `agent/cli/main.py` (via `cli:main`) | Interactive TUI; `run -p "..."` one-shot research; `init` onboarding wizard |
| `vibe-trading serve` | `agent/api_server.py` | FastAPI REST + SSE server on port 8899; serves `frontend/dist` in production mode |
| `vibe-trading-mcp` | `agent/mcp_server.py` | MCP server for Claude Desktop / OpenClaw / Cursor etc. |
| `scripts/dev up` | `scripts/dev` | Starts backend (8899) + frontend dev (5899) with PID/log management |
| `docker compose up --build` | `docker-compose.yml` | All-in-one containerized stack (backend serves built frontend) |
| `vibe-trading alpha ...` | `agent/cli/commands/` | Browse/bench the 462-alpha zoo |
| `vibe-trading playbook list` | `agent/cli/commands/` | Scheduled-research templates |
| `vibe-trading provider doctor` | `agent/cli/commands/` | Redacted provider/proxy/package diagnostics |
| `vibe-trading channels status` | `agent/cli/commands/` | IM channel config inspection |

### 3.2 Core user workflows (from README Quick Start / Examples)

1. **Natural-language research:**
   ```bash
   pip install vibe-trading-ai
   vibe-trading init                 # interactive .env setup
   vibe-trading run -p "Backtest a BTC-USDT 20/50 moving-average strategy for 2024 and summarize return and drawdown"
   ```
2. **Web UI (Path A Docker):** clone → `cp agent/.env.example agent/.env` → edit LLM provider → `docker compose up --build` → open `http://localhost:8899`.
3. **Local install (Path B):** venv → `pip install -e .` → configure `.env` → `vibe-trading` TUI; optional web UI via `vibe-trading serve` + `frontend` dev server.
4. **MCP plugin (Path C):** point any MCP client at `agent/mcp_server.py`.
5. **Alpha zoo bench (one-liner):**
   ```bash
   vibe-trading alpha bench --zoo gtja191 --universe csi300 --period 2018-2025 --top 20
   ```
6. **Shadow-account review:** `vibe-trading --upload trades_export.csv` → behavioral diagnostics vs. a "shadow strategy".
7. **Swarm research teams:** launch preset multi-agent teams (bull/bear debate, quant strategy pipeline, crypto desk, global macro allocation) via natural language.
8. **Scheduled research:** cron/interval jobs fired by the API-server background scheduler (`VIBE_TRADING_ENABLE_SCHEDULER=1`).

### 3.3 Request flow (mental model)

Natural language → `ChatLLM` (provider abstraction) → ReAct `AgentLoop`
(`agent/src/agent/loop.py`) → tool registry (`agent/src/tools/`) → data loaders
(`agent/backtest/loaders/`) / backtest engines (`agent/backtest/engines/`) / skills
(`agent/src/skills/`) → run artifacts under `agent/runs` or `~/.vibe-trading`
→ results streamed back via CLI/SSE/MCP.

---

## 4. Agentic Components (Multi-Agent Trading System)

This is the heart of the project: a tool-heavy ReAct agent, a skill system, a persistent memory layer, and a **swarm multi-agent DAG engine** with 30 preset teams.

### 4.1 ReAct agent core — `agent/src/agent/`

- **`loop.py` — `AgentLoop`:** the central ReAct loop. Features **5-layer context management**: microcompact pruning → context_collapse (LLM-free folding) → auto_compact (LLM structured summary) → explicit `compact` tool → iterative update of prior summaries. Read/write tool batching runs consecutive read-only tools in parallel threads.
- **`context.py` — `ContextBuilder`:** system prompt assembly + auto-recall from persistent memory.
- **`tools.py` — `ToolRegistry`:** tool base class + discovery registry.
- **`skills.py` — `SkillsLoader`:** loads SKILL.md files (90 bundled + user-created via CRUD).
- **`memory.py` / `trace.py` / `frontmatter.py` / `progress.py` / `grounding.py`:** per-run workspace memory, execution trace writer, shared YAML frontmatter parser, heartbeat/progress events, and a grounding ledger for source citations.

### 4.2 Tool system — `agent/src/tools/` (78 modules; ~107 auto-discovered tools per README)

Categories: backtest & factor analysis (`backtest_tool.py`, `factor_analysis_tool.py`, `alpha_bench_tool.py`, `alpha_compare_tool.py`, `alpha_zoo_tool.py`), market data (`market_data_tool.py`, `get_fundamentals_tool.py`, `options_chain_tool.py`, `orderbook_depth_tool.py`, `financial_statements_tool.py`, ...), discovery & flow (iwencai, northbound, dragon-tiger, fund-flow, margin-trading, block-trades, shareholder-count, lockup-expiry), document/vision (`doc_reader_tool.py`, `image_vision_tool.py`, OCR), web (`web_search_tool.py`, `web_reader_tool.py`), agent-self-maintenance (`load_skill_tool.py`, `skill_writer_tool.py`, `remember_tool.py`, `session_search_tool.py`, `compact_tool.py`, `goal_tool.py`, `hypothesis_tool.py`), swarm (`swarm_tool.py`, `autopilot_tool.py`), sandboxed code execution (`bash_tool.py`, `edit_file_tool.py`, `write_file_tool.py` behind `_shell_safety.py` and `VIBE_TRADING_ENABLE_SHELL_TOOLS`), live trading (`trading_connector_tool.py`, `block_trades_tool.py`), and MCP client mode (`mcp.py`, `MCPRemoteTool`).

### 4.3 Swarm multi-agent engine — `agent/src/swarm/`

- **`models.py`:** Pydantic models (`SwarmAgentSpec`, `SwarmTask`, `SwarmRun`, `WorkerResult`, ...) with a whitelist of public LLM providers and credential-shaped-token redaction.
- **`runtime.py`:** DAG orchestrator — schedules workers by **topological layer**, parallel within a layer and serial between layers; runs in a background daemon thread with cancellation and event-callback support; capped exponential worker retries.
- **`worker.py`:** standalone worker execution engine with a lightweight ReAct loop (uses `ChatLLM.chat` directly, without instantiating `AgentLoop`).
- **`task_store.py`:** DAG validation + `topological_layers` + dependency resolution.
- **`presets/`:** **30 YAML team presets** (e.g., `equity_research_team.yaml`, `quant_strategy_desk.yaml`, `crypto_trading_desk.yaml`, `risk_committee.yaml`, `investment_committee.yaml`). Each preset declares agent roles (system prompt, allowed tools, bundled skills, iteration/timeout limits), a task DAG with `depends_on` edges and `input_from` context passing, plus templated variables (e.g., `{market}`, `{goal}`).

**Example — `equity_research_team.yaml`:** `macro_analyst` → `sector_analyst` → `stock_picker` → `aggregator`, a four-agent pipeline where each downstream agent receives upstream context and the final editor is restricted from using data tools (must synthesize only from `{upstream_context}`).

### 4.4 LLM provider abstraction — `agent/src/providers/`

- **`llm_providers.json`:** registry of ~20 providers with env-var names, default base URLs, and default models: OpenRouter (default model `deepseek/deepseek-v4-pro`), Requesty, OpenAI, Anthropic, OpenAI Codex (OAuth), DeepSeek, NVIDIA NIM, Gemini, Groq, DashScope/Qwen, Zhipu, Moonshot/Kimi, Kimi Coding, MiniMax, MIMO, Novita, iFlytek Spark, Z.ai, ModelScope, GitHub Copilot (SDK), **Ollama (local, no key)**.
- **`chat.py` (`ChatLLM`):** unified chat interface with streaming, retries, timeouts, and provider-specific transports; `llm.py` builds provider clients.
- **`capabilities.py` / `content_filter.py` / `copilot_auth.py` / `openai_codex.py`:** reasoning-effort flags, content-moderation warning ratio, Copilot auth, Codex OAuth.
- **Env config:** `LANGCHAIN_PROVIDER`, `LANGCHAIN_MODEL_NAME`, `<PROVIDER>_API_KEY`, `<PROVIDER>_BASE_URL`, `LANGCHAIN_TEMPERATURE`, `TIMEOUT_SECONDS`, `MAX_RETRIES`, `LANGCHAIN_REASONING_EFFORT`.

### 4.5 Skills library — `agent/src/skills/` (90 skills, 9 categories)

Every skill is a `SKILL.md` with YAML frontmatter (`name`, `description`, `category`) plus a workflow body and tool parameters. Categories: Data Source (10), Strategy (19), Analysis (23), Asset Class (9), Crypto (7), Flow (8), Tool (10), Research (3), Risk Analysis (1). Examples: `factor-research`, `strategy-generate`, `global-macro`, `onchain-analysis`, `report-generate`, `trade-journal`, `alpha-zoo`.

### 4.6 Alpha Zoo — `agent/src/factors/` (462 alphas)

- `zoo/`: qlib158 (154) + alpha101 (101) + gtja191 (191) + academic (12) + fundamental (4).
- `base.py`: 19 operators (rank/scale/ts_*/delta/decay_linear/safe_div/vwap); `registry.py`: AST-only metadata load + lazy compute + sanity gates; `bench_runner.py`: IC + alive/reversed/dead categorization.

### 4.7 Backtesting — `agent/backtest/`

- **`engines/`:** 8+ engines — `china_a`, `china_futures`, `crypto`, `forex`, `global_equity`, `global_futures`, `india_equity`, `korea_equity`, `options_portfolio`, `vietnam_equity`, plus `composite` cross-market engine.
- **`loaders/`:** 24 data loaders (`tushare`, `yfinance`, `akshare`, `baostock`, `tencent`, `mootdx`, `eastmoney`, `sina`, `stooq`, `yahoo`, `okx`, `binance`, `ccxt`, `futu`, `pykrx`, `longbridge`, `mt5`, `qveris`, `local`, `alphavantage`, `tiingo`, `fmp`, `finnhub`, `india_broker`) with a registry + automatic fallback chains.
- **`optimizers/`:** MVO, equal volatility, max diversification, risk parity, turnover-aware.

### 4.8 Other agentic modules

- `agent/src/memory/` — cross-session persistent memory (`persistent.py`, file-based under `~/.vibe-trading/memory/`); opt-in lifecycle (`VT_MEMORY=off|on|full`).
- `agent/src/session/` — multi-turn chat + FTS5 cross-session search (`sessions.db`).
- `agent/src/goal/` — research-goal tracking (context, policy, store) to steer long-running research.
- `agent/src/hypotheses/` — hypothesis registry for strategy exploration.
- `agent/src/live/` — broker/live-trading layer: mandate-gated, kill-switch-aware, fail-closed, audit-logged (per AGENT_CONTRIBUTOR_GUIDE.md and `SECURITY.md`).
- `agent/src/channels/` + `channelsui/` — IM channel runtime (WebSocket, Telegram, Slack, Discord, Matrix, WhatsApp, Signal, QQ/NapCat, WeChat/WeCom, Feishu/Lark, DingTalk, Teams, email, Mochat).
- `agent/src/scheduled_research/` — cron/interval research playbooks.
- `agent/src/security/` — secrets handling; `agent/src/portfolio/` — multi-broker portfolio; `agent/src/shadow_account/` — shadow-account analysis/reporting; `agent/src/strategy_discovery/` + `strategy_store/` — strategy generation pipelines.
- **`AGENT_CONTRIBUTOR_GUIDE.md`:** explicit agent-facing expectations — never commit `.env`/tokens/broker exports; order-affecting code is safety-critical; DCO `Signed-off-by:` required; `API_AUTH_KEY` needed for non-loopback deployments; external MCP servers are operator-trust surfaces; prefer sanitized fixtures over real financial data.

---

## 5. Reproduction Blockers & Classroom-Friendly Suggestions

### 5.1 Environment pinning (what is locked, what is not)

| Aspect | Status | Notes |
|---|---|---|
| Python version | Pinned to `>=3.11`; CI/lock target 3.11 | Use 3.11 (or 3.12) — 3.13 may have wheel gaps for some transitive deps. |
| Python deps | **Hash-pinned lock** (`requirements-lock.txt`, `uv pip compile --generate-hashes`) | Docker uses `--require-hashes`; a local `pip install -e .` resolves loosely against PyPI. |
| Node version | **Hard requirement** `>=22.22.0` (frontend + desktop) | Older Node will not run Vite 8 / React 19 toolchain. |
| Docker base images | SHA256 digest-pinned (`node:22-slim`, `python:3.11-slim`) | Reproducible builds. |
| System libs | Docker runtime installs weasyprint deps (Pango/HarfBuzz/Cairo/gdk-pixbuf, `fonts-dejavu-core`) | Local macOS/Windows installs need these too for PDF report rendering. |
| LangChain 1.x transition | 0.1.10 moved to LangChain 1.x; README warns a venv recreate may be needed on upgrade | Fresh installs unaffected. |

**Blocker:** frontend/desktop require Node ≥ 22.22 — a classroom laptop with an older
Node LTS must upgrade first (or use the Docker path, which sidesteps this entirely).

### 5.2 External API keys (LLM + data)

**LLM API key (required for most providers):** every provider except Ollama requires a key.
Per `.env.example` the supported list includes OpenRouter, DeepSeek, OpenAI, Anthropic,
Gemini, Groq, DashScope, Zhipu, Moonshot, MiniMax, NVIDIA NIM, ModelScope, Novita,
iFlytek, Z.ai, MIMO, Copilot (subscription), Codex (OAuth), Requesty.
**Course-relevant:**
- **DeepSeek official API** (`LANGCHAIN_PROVIDER=deepseek`, `DEEPSEEK_API_KEY`) — the
  `.env.example` default model is `deepseek-v4-pro`; the README "Recommended Models" table
  lists `deepseek-v4-pro` / `deepseek/deepseek-v4-pro` as the **sweet-spot default**.
- **OpenRouter** default is also `deepseek/deepseek-v4-pro` — one key unlocks many models.
- **Ollama (local, zero key):** `LANGCHAIN_PROVIDER=ollama`, `OLLAMA_BASE_URL=http://localhost:11434`,
  e.g. `qwen2.5:32b` — needs a decent local machine; smaller models work but degrade tool-calling.
- **No OpenAI/GPT requirement anywhere** — the stack is provider-agnostic; DeepSeek and
  open-source models (Qwen, Llama via Groq/NVIDIA/ModelScope, local Ollama) are first-class.

**Data-source keys (mostly optional):** the project is designed so all major markets work
**without any data API key** via automatic fallback:
- HK/US/UK equities: **yfinance/Yahoo — free, no key**
- Crypto: **OKX public API — free, no key**; CCXT adds 100+ exchanges
- A-shares: **mootdx (TCP-direct, no IP throttle) / AKShare / baostock — free, no key**;
  `TUSHARE_TOKEN` optional (adds a premium feed)
- Korea: `pykrx` extra; Taiwan: local SQLite snapshot (`VIBE_TW_STOCK_DB`)
- Optional paid/keys: Finnhub, AlphaVantage, Tiingo, FMP, Tickerall, FRED, iwencai, QVeris
  (paid routing), LongBridge, Futu OpenD, IBKR, MetaTrader 5 (Windows)

**So a zero-cost classroom setup needs only: one LLM key (DeepSeek) + no data keys.**

### 5.3 Data dependencies & network

- **Live network required** for market data (no bundled dataset). First backtest will download
  bars from Yahoo/OKX/AKShare; classrooms need reliable outbound HTTPS.
- Optional local cache: `VIBE_TRADING_DATA_CACHE=1` caches settled bars under
  `~/.vibe-trading/cache/loaders/` — useful to pre-warm a lab.
- Persisted state lives under `~/.vibe-trading/` (sessions.db, memory, runs, uploads) —
  in Docker these are named volumes; `docker compose down -v` deletes them.
- Timezone note: scheduled-research cron jobs evaluate in UTC unless an IANA `timezone` is set.

### 5.4 Hardware / GPU requirements

- **No GPU required** for API-based LLM usage — compute is remote. Backtests are CPU-bound
  (pandas/numpy) and fine on modest hardware; Docker compose sets `mem_limit: 4g`, `cpus: 2`,
  `pids_limit: 512` as generous self-hosted defaults.
- **Local Ollama** is the only path with meaningful hardware needs (RAM/VRAM scale with model
  size; `qwen2.5:32b` needs a beefy machine — use a 7B/8B model for labs).
- README warns small/`-nano` models produce unreliable tool-calling — avoid them in demos.

### 5.5 Recommended minimal classroom path (exact commands)

```bash
# Option 1 — Docker (zero local setup, ~2 min; recommended for labs)
git clone https://github.com/HKUDS/Vibe-Trading.git
cd Vibe-Trading
cp agent/.env.example agent/.env
# Edit agent/.env: set LANGCHAIN_PROVIDER=deepseek, LANGCHAIN_MODEL_NAME=deepseek-chat,
# DEEPSEEK_API_KEY=sk-..., and comment out the OpenRouter block.
# (the repo's own .env.example may read deepseek-v4-pro — either id works; see llm-provider-guidance.md §2.4)
docker compose up --build
# open http://localhost:8899

# Option 2 — Local install (dev-focused, ~5 min)
python -m venv .venv && source .venv/bin/activate
pip install -e .
cp agent/.env.example agent/.env   # same DeepSeek edit
vibe-trading                        # interactive TUI

# One-shot research (no UI needed) — the backtest CLI entry point:
vibe-trading run -p "Backtest a BTC-USDT 20/50 moving-average strategy for 2024 and summarize return and drawdown"
```

**No-key data guarantee:** crypto (OKX), US/HK (Yahoo/yfinance), A-shares (mootdx/AKShare)
all work with zero data keys, so the only credential a student needs is one DeepSeek API key.

### 5.6 Other reproducibility caveats

- **Safety defaults:** API binds loopback by default; non-local access requires `API_AUTH_KEY`.
  Shell-capable tools (`bash`, `write_file`, `edit_file`) are opt-in
  (`VIBE_TRADING_ENABLE_SHELL_TOOLS`) — good for classroom security.
- **Never live-trade in class:** broker connectors exist (`trading_connector_tool.py`,
  TAP mode for credential isolation) but are out of scope for a reproduction course;
  keep to research/backtest/shadow-account flows.
- **Tests:** 555 test files; `pytest --ignore=agent/tests/e2e_backtest ...` for a quick suite;
  frontend needs `npm ci && npm run build`.
- **Huge README:** main docs are ~268 KB with translated copies; the wiki (`wiki/`) is the
  better-structured docs surface for students.
- **Windows notes:** desktop build and MT5 connector are Windows-only; core stack is
  cross-platform (PowerShell `Activate.ps1` note in README).

---

*Report grounded in the actual checkout: `pyproject.toml`, `agent/requirements.txt`,
`requirements-lock.txt`, `frontend/package.json`, `desktop/electron/package.json`,
`Dockerfile`, `docker-compose.yml`, `agent/.env.example`, `agent/src/` (agent, tools, swarm,
skills, factors, providers, backtest), `AGENT_CONTRIBUTOR_GUIDE.md`, and `README.md`.*


