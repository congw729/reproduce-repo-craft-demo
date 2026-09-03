# SwarmCraft — Vibe-Trading Reproduction Plan (Classroom Edition)

**Project:** HKUDS/Vibe-Trading — "Natural-language finance research AI agent with backtesting" (v0.1.14, MIT)
**Plan author:** Quant Reproduction Architect
**Inputs cross-checked against:** `repo-analysis.md` (Repository Analyst) and `llm-provider-guidance.md` (DeepSeek member)
**Audience:** A student with basic Python/terminal skills and **no prior knowledge of this repository**
**Goal:** Reproduce the core of Vibe-Trading step by step — install it, run backtests, browse its 462-factor alpha zoo, launch its web UI, drive its multi-agent swarm, and demo a shadow-account review — on a modest personal machine, in a classroom network, with **zero OpenAI GPT usage** (DeepSeek API or local open-source models only).

---

## 0. Plan Overview

### 0.1 What you will build

You will turn a plain Python environment into a working copy of Vibe-Trading, then use it to:

1. Run a natural-language backtest ("backtest BTC-USDT 20/50 MA and summarize returns") — end to end, from market-data download to a printed summary.
2. Browse and bench the built-in **462-factor alpha zoo** (qlib158 + alpha101 + gtja191 + academic + fundamental).
3. Interact with it through an interactive terminal UI and a browser-based web UI.
4. Launch **multi-agent swarm workflows** (e.g., an investment-committee debate) — the same collaboration pattern this course teaches with JiuwenSwarm.
5. Run a **shadow-account review** that profiles a trade journal and compares it against a rule-based strategy.

### 0.2 Milestone map

| # | Milestone | Outcome | Est. effort (solo) | Depends on |
|---|-----------|---------|-------------------|-----------|
| 0 | Environment check & repo clone | Verified Python/Git toolchain; repo on disk | 10–15 min | — |
| 1 | Local install & LLM config | `vibe-trading` CLI boots; DeepSeek (or Ollama) configured | 20–40 min | M0 |
| 2 | First end-to-end backtest | One natural-language backtest completes with a summary | 15–30 min | M1 |
| 3 | Alpha zoo browse & bench | `vibe-trading alpha list` works; a small bench completes | 20–40 min | M2 |
| 4 | Web UI (API server) | `http://localhost:8899` serves the app; a chat task produces a run card | 15–25 min | M2 |
| 5 | Multi-agent swarm | One swarm preset runs to completion | 20–40 min (token-dependent) | M2 |
| 6 | Shadow-account demo | Trade-journal profile + comparison report produced | 15–30 min | M2 |

**Total solo effort:** roughly 2–3 hours of hands-on time, excluding downloads/installs.

### 0.3 Global constraints (apply to every milestone)

- **No OpenAI GPT anywhere.** All LLM components use **DeepSeek API** (primary) or a **local open-source model via Ollama** (no-key fallback). OpenRouter is acceptable as a gateway but must point at `deepseek/*` or other open models. Details in M1 (sourced from `llm-provider-guidance.md`).
- **No paid data keys required.** All market data used here is free and keyless: OKX (crypto), Yahoo/yfinance (US/HK/UK equities), mootdx/AKShare (A-shares). Paid feeds (Tushare token, QVeris, Finnhub, etc.) are explicitly optional and not needed.
- **Modest hardware.** Everything is CPU-only; **no GPU required** (LLM compute is remote; backtests are pandas/numpy CPU work). Plan assumes >= 8 GB RAM and ~10 GB free disk.
- **Classroom network.** Reliable outbound HTTPS to GitHub/PyPI and the chosen LLM endpoint is required. Where a lab blocks hosts, use the per-milestone fallbacks.
- **No live trading.** Broker connectors and live orders are out of scope; all exercises use research/backtest/shadow flows only.
- **Environment pinning summary:** Python **3.11 or 3.12** (>=3.11 required; 3.13 may have wheel gaps for some transitive deps). Frontend dev requires **Node >= 22.22** but is optional (M4). Docker path pins base images by SHA256 and installs from hash-pinned `requirements-lock.txt`. Local installs resolve loosely against PyPI — acceptable for class, keep the venv fresh.

### 0.4 How each milestone is structured

Each milestone lists: **Goal -> Pre-flight -> Steps -> Verify (commands + expected output) -> Troubleshooting -> Fallback (if a step cannot be done in class)**. A milestone is complete only when its Verify block passes.

---

## Milestone 0 — Environment Check & Repository Clone

**Goal:** Confirm the toolchain and get the source on disk.

### Pre-flight

- A laptop/desktop with macOS, Linux, or Windows (PowerShell).
- Terminal access; ~1 GB free disk for clone + venv; outbound HTTPS.

### Steps

1. Check the toolchain:

   ```bash
   python3 --version        # want 3.11 or 3.12 (3.13 may have wheel gaps for some deps)
   git --version            # any recent git
   ```

   > If `python3` is absent or < 3.11, install Python 3.11/3.12 from python.org (macOS/Windows) or your OS package manager (Linux). Do not proceed with an old interpreter.

2. Clone the repository (shallow clone is enough for reproduction):

   ```bash
   git clone --depth 1 https://github.com/HKUDS/Vibe-Trading.git
   cd Vibe-Trading
   ```

3. Look at what you have:

   ```bash
   ls        # agent/  frontend/  desktop/  wiki/  Dockerfile  pyproject.toml  requirements-lock.txt ...
   ```

### Verify

```bash
python3 -c "import sys; assert sys.version_info >= (3,11); print('python OK', sys.version)"
ls agent/.env.example      # the environment template exists
```

Both commands succeed -> M0 complete.

### Troubleshooting

- **`python3` is old (e.g. 3.9):** install 3.11/3.12 and re-run the verify; do not rely on `pip` from the old interpreter.
- **Clone slow/fails:** use a GitHub mirror if throttled, or download the source ZIP via browser and unpack.

### Fallback (fully offline lab)

M2+ requires live data fetches, so an **offline lab needs a pre-baked image** (Docker image or VM snapshot) prepared by the instructor — repo cloned, deps installed, `VIBE_TRADING_DATA_CACHE=1` pre-warmed data (see M2 fallback). This plan otherwise assumes a live connection.

## Milestone 1 — Local Install & LLM Configuration

**Goal:** Install the package, configure an LLM provider (DeepSeek primary, Ollama fallback), and prove the CLI boots.

### Pre-flight

- M0 complete.
- One **DeepSeek API key** (https://platform.deepseek.com) — cheapest recommended option; OR willingness to run Ollama locally with an open model (no key, no account).

### Steps

**Path B — local install (recommended for this course; full CLI + tests):**

```bash
cd Vibe-Trading
python3 -m venv .venv
source .venv/bin/activate        # Linux/macOS
# Windows PowerShell: .venv\Scripts\Activate.ps1  (CMD: .venv\Scripts\activate.bat)
pip install -e .
# Optional but recommended — installs the native DeepSeek adapter (langchain-deepseek):
pip install -e ".[deepseek]"
```

> The project pins transitive deps with hashes in `requirements-lock.txt`, but a local `pip install -e .` resolves loosely against PyPI — fine for a classroom. (The Docker path installs from the locked file; see Fallback.)

**LLM configuration — DeepSeek (primary, no OpenAI):**

```bash
cp agent/.env.example agent/.env
```

Edit `agent/.env` so **exactly the DeepSeek block is active** and all other provider blocks are commented out:

```dotenv
# --- DeepSeek (primary) ---
LANGCHAIN_PROVIDER=deepseek
LANGCHAIN_MODEL_NAME=deepseek-chat        # stable public id; see note below
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxx  # your real key
DEEPSEEK_BASE_URL=https://api.deepseek.com/v1

# Optional but recommended for the course:
LANGCHAIN_TEMPERATURE=0.0                 # deterministic, reproducible answers
TIMEOUT_SECONDS=120
MAX_RETRIES=2
```

> **Model id note:** `deepseek-chat` (general chat / tool-calling) is the course default — fast, cheap, reliable for ReAct loops and swarm workers. `deepseek-reasoner` is an optional stretch (slower, more tokens). If a newer repo version lists a different default in `.env.example` (e.g. `deepseek-v4-pro`), the model string is just a label sent to the API — use whichever id your DeepSeek account actually exposes; `deepseek-chat` / `deepseek-reasoner` are the stable public ids.
>
> **Adapter mode:** default `auto` uses the native `langchain-deepseek` adapter when installed, else falls back to the OpenAI-compatible path — best for students. No change needed.

**No-key fallback — Ollama with an open-source model:**

```bash
# 1. Install Ollama from https://ollama.com (macOS / Linux / Windows)
# 2. Pull a classroom-friendly model:
ollama pull qwen2.5:7b          # ~4.7 GB; runs on 8 GB RAM laptops (CPU or small GPU)
# lighter: qwen2.5:3b (~2 GB), llama3.2:3b (~2 GB)
```

Then set in `agent/.env`:

```dotenv
# --- Ollama (local, no key) ---
LANGCHAIN_PROVIDER=ollama
LANGCHAIN_MODEL_NAME=qwen2.5:7b
OLLAMA_BASE_URL=http://localhost:11434

# Local models are slow — give the UI/SSE more patience:
VIBE_TRADING_SSE_TIMEOUT=300     # default 90s; raise for CPU-only inference
TIMEOUT_SECONDS=300
```

> The repo's default Ollama entry (`qwen2.5:32b`) is **too heavy for typical student hardware** — use 7b/3b in class. Do **not** use `*-nano`/`*-flash-lite`-class models: the README warns small models produce unreliable tool-calling (the agent "answers from memory" instead of running real backtests). 7B/8B open models are the smallest reliable class; DeepSeek API models are the cheapest reliable cloud option.

### Verify

```bash
vibe-trading --help          # shows CLI subcommands (no LLM call needed)
vibe-trading provider doctor # redacted diagnostics: provider, model, key state, base URL, packages, proxy
```

Both run without import errors -> M1 complete. (`provider doctor` may report "no key" if you use Ollama — expected and fine.)

### Troubleshooting

- **`vibe-trading: command not found`:** ensure the venv is active (`which vibe-trading`), or `pip install -e .` again.
- **`LANGCHAIN_MODEL_NAME is not set`:** set it in `agent/.env` (e.g. `deepseek-chat`); the value is required.
- **401/AuthenticationError on first call:** key missing, wrong, or pasted with quotes/whitespace. Re-set `DEEPSEEK_API_KEY` (raw, no quotes, no trailing newline), then `vibe-trading provider doctor`.
- **`ModuleNotFoundError: langchain_deepseek`:** the native adapter is missing while `VIBE_TRADING_DEEPSEEK_ADAPTER=native`; run `pip install -e ".[deepseek]"`, or set `VIBE_TRADING_DEEPSEEK_ADAPTER=auto` / `openai-compatible`.
- **Requests hang/time out behind a classroom proxy:** set `VIBE_TRADING_DISABLE_HTTP_PROXY=1` (ignores proxy env for LLM calls only, still honors `SSL_CERT_FILE`), or raise `TIMEOUT_SECONDS`.
- **Accidental OpenAI endpoint:** if `OPENAI_BASE_URL`/`OPENAI_API_KEY` are exported in the shell profile, the provider's own `<PROVIDER>_BASE_URL` wins; verify with `provider doctor` that the base URL is the intended DeepSeek/Ollama endpoint, and remove stray `OPENAI_BASE_URL` pointing at `api.openai.com`.
- **LangChain import errors after an upgrade:** create a **fresh venv** (`deactivate`, `rm -rf .venv`, repeat steps) — documented for the 0.1.10 LangChain 1.x transition.
- **PDF export fails:** PDF reports use `weasyprint`, which needs system libs (Pango/HarfBuzz/Cairo/Fontconfig). HTML reports still work — treat PDF as optional.

### Fallback

**Path A — Docker (zero local Python setup; ~2 min):** requires Docker Engine >= 20.10 / Compose v2:

```bash
git clone https://github.com/HKUDS/Vibe-Trading.git && cd Vibe-Trading
cp agent/.env.example agent/.env    # same DeepSeek edit as above
docker compose up --build           # compose sets mem_limit 4g, cpus 2; installs from hash-pinned lock
# open http://localhost:8899
```

Use Path A if a student machine has Docker but a broken Python toolchain, or for an instructor-prepared lab image. With Docker + Ollama, set `OLLAMA_BASE_URL=http://host.docker.internal:11434` (container -> host). Data persists in named volumes; `docker compose down -v` deletes them.

---

## Milestone 2 — First End-to-End Backtest

**Goal:** Run one natural-language backtest end to end and read its results.

### Pre-flight

- M1 complete (CLI boots; LLM provider configured).
- Outbound HTTPS to the LLM endpoint and to free market-data hosts (OKX, Yahoo).

### Steps

1. From the repo root with the venv active, run the canonical example:

   ```bash
   vibe-trading run -p "Backtest a BTC-USDT 20/50 moving-average strategy for 2024 and summarize return and drawdown"
   ```

   What happens: the agent reads your goal -> picks tools -> downloads BTC-USDT daily bars from OKX (free, no key) -> runs the 20/50 MA crossover backtest -> prints a summary with return and drawdown.

2. Try a second canonical example (equities, Yahoo feed — also keyless):

   ```bash
   vibe-trading run -p "Backtest a simple RSI mean-reversion strategy on AAPL for 2025"
   ```

3. Find where run artifacts live:

   ```bash
   ls ~/.vibe-trading/runs   # or ./agent/runs — each run id has its own folder
   ```

### Verify

- The first command exits successfully and its output contains **return** and **drawdown** figures (or an explicit "no trades" explanation — for 2024 BTC a MA crossover normally trades).
- The second command completes without a data-source error.
- At least one run directory exists under `~/.vibe-trading/runs` with artifacts (logs/JSON).

All three hold -> M2 complete.

### Troubleshooting

- **"No module named langchain" / adapter errors:** reinstall in the venv (`pip install -e .`), confirm the venv is active.
- **Data download failure for BTC-USDT:** OKX may be blocked on some networks. Ask the agent for an explicit source: `... use yfinance as the data source for BTC-USD ...`, or switch to the equity example (Yahoo). If both fail, the network blocks these hosts — see Fallback.
- **LLM timeout/errors:** check key/model in `agent/.env`; run `vibe-trading provider doctor`. With Ollama, confirm `ollama list` shows the model and `curl http://localhost:11434` answers.
- **"No trades" but no error:** the strategy legitimately may not trade in the window; ask for a different period (e.g. 2021) or strategy (e.g. 50/200 MA).

### Fallback

- **Hosted feeds blocked:** set `VIBE_TRADING_DATA_CACHE=1` to cache settled bars under `~/.vibe-trading/cache/loaders/`. An instructor can pre-warm this cache on a networked machine and copy it to lab machines; backtests then run offline against cached bars.
- **No LLM key and no Ollama:** M2 cannot complete — it is the first milestone that needs an LLM. Provide at minimum an instructor-shared DeepSeek key (small budget) or a pre-installed Ollama + qwen2.5:7b.

## Milestone 3 — Alpha Zoo: Browse & Bench

**Goal:** Use the built-in 462-factor alpha library and run a small bench.

### Pre-flight

- M2 complete.

### Steps

1. List the alpha zoo:

   ```bash
   vibe-trading alpha list
   ```

   Expect categories qlib158 (154), alpha101 (101), gtja191 (191), academic (12), fundamental (4) — 462 total.

2. Inspect one alpha family:

   ```bash
   vibe-trading alpha show gtja191    # or any family name from the list
   ```

3. Run a **small, quick** bench (keep it tiny for classroom speed):

   ```bash
   vibe-trading alpha bench --zoo gtja191 --universe csi300 --period 2023-2024 --top 10
   ```

   > Full-range benches (`--period 2018-2025`) download a lot of bars and take minutes on modest hardware; start with 1–2 years and a small `--top N` so the class sees results fast.

### Verify

- `vibe-trading alpha list` prints the expected families and counts (totals >= 400).
- The small bench completes and prints a table of alphas ranked by a metric (IC or similar), with alive/reversed/dead categories.

Both hold -> M3 complete.

### Troubleshooting

- **`alpha bench` fails on data for `csi300`:** the A-share chain is mootdx -> AKShare -> ... (keyless). If the host is blocked, use a US universe instead: `--universe sp500` (Yahoo) or `--universe nasdaq100`. Or an even shorter period (e.g. `--period 2024-2024`).
- **Bench too slow:** reduce `--top N` and/or the period; the point is the workflow, not a production result.

### Fallback

No network: use the pre-warmed data cache (M2 fallback) or skip `alpha bench` and demo `alpha list`/`alpha show` only (these read bundled YAML and need no network).

---

## Milestone 4 — Web UI via API Server

**Goal:** Serve the application and interact through the browser.

### Pre-flight

- M2 complete (backend is installed and configured).

### Steps

1. Start the API server:

   ```bash
   vibe-trading serve --port 8899
   ```

2. Open **http://localhost:8899** in a browser.

3. In the chat box, run a short task, e.g.:

   ```
   Backtest a 20/50 moving-average crossover on ETH-USDT for 2024 and show max drawdown.
   ```

4. (Optional) Full UI with dev frontend — requires Node >= 22.22:

   ```bash
   # Terminal 1: vibe-trading serve --port 8899
   # Terminal 2:
   cd frontend && npm install && npm run dev    # serves on http://localhost:5899, proxies API to 8899
   ```

### Verify

- `http://localhost:8899` responds (or, with the dev frontend, `http://localhost:5899` renders the app).
- The ETH-USDT task produces a run card with results.
- A health check from the terminal also passes:

  ```bash
  curl -s http://localhost:8899/live   # liveness endpoint; expect a healthy response
  ```

All hold -> M4 complete.

### Troubleshooting

- **API only reachable on the same machine (403 for remote):** by design the server is loopback-only unless `API_AUTH_KEY` is set. For a classroom, keep it loopback (students run their own instance) — do not expose the API on the LAN without setting a strong `API_AUTH_KEY`.
- **UI shows "Execution timed out" with local models:** `VIBE_TRADING_SSE_TIMEOUT` (default 90 s) is too short for CPU inference; raise it to 300+ (see M1 Ollama block) and prefer 3b/7b models.
- **Node too old for the dev frontend:** Node >= 22.22 is a hard requirement; upgrade Node or skip the dev frontend (the production build path `vibe-trading serve` serves `frontend/dist` if built, or the API-only page otherwise).

### Fallback

Frontend build fails or Node is unavailable: the API server itself is the deliverable — chat via `vibe-trading` TUI (M2) covers the same workflow without the browser.

## Milestone 5 — Multi-Agent Swarm Workflow

**Goal:** Launch one preset multi-agent team and watch it run to completion — this is the workflow this course teaches (task decomposition, role assignment, review loops), demonstrated with Vibe-Trading's own swarm engine.

### Pre-flight

- M2 complete.
- A provider with reliable tool-calling (DeepSeek `deepseek-chat` recommended; Ollama 7b/3b works but is slower). Swarms make many LLM calls — budget a few thousand tokens per run.

### Steps

1. List the swarm presets:

   ```bash
   vibe-trading --swarm-presets
   ```

   Expect ~30 presets (e.g. `investment_committee`, `equity_research_team`, `quant_strategy_desk`, `risk_committee`, `crypto_trading_desk`).

2. Run a small committee debate (keep the topic narrow so it finishes fast):

   ```bash
   vibe-trading --swarm-run investment_committee '{"topic": "Is BTC above its 200-day moving average a bullish signal for 2024?"}'
   ```

   Or drive it from the TUI with `/swarm`. Each preset is a YAML DAG: agent roles (system prompt, allowed tools), task dependencies (`depends_on`), and context passing (`input_from`) — the same shape as a JiuwenSwarm workflow.

3. Inspect a preset file to see the DAG structure:

   ```bash
   cat agent/src/swarm/presets/equity_research_team.yaml
   ```

### Verify

- `vibe-trading --swarm-presets` lists at least 10 presets.
- The swarm run completes (each worker finishes or errors explicitly; the final aggregator produces a summary).
- The preset YAML shows >= 2 roles and >= 1 `depends_on` edge (proves it is a DAG, not a single agent).

All hold -> M5 complete.

### Troubleshooting

- **Run is too slow / token-heavy:** pick a 2-agent preset or a narrower topic; use `deepseek-chat` (not `deepseek-reasoner`); set `LANGCHAIN_TEMPERATURE=0.0`.
- **Worker errors on tool calls:** with Ollama, smaller models mis-handle tools — switch to DeepSeek or a 7b+ model; with DeepSeek, confirm `deepseek-chat` (tool-capable) rather than a reasoning-only id.
- **Rate limits (429) in a large class:** stagger exercise starts; use per-group keys; keep `MAX_RETRIES=2` (built-in backoff).

### Fallback

No budget for a full swarm run: demo the preset YAMLs (architecture review) and run a **single-agent** task instead — the TUI `/swarm` help text plus one preset file inspection still teaches the DAG concept with zero tokens.

---

## Milestone 6 — Shadow-Account Demo

**Goal:** Profile a trade journal and compare it against a rule-based "shadow strategy" — the project's flagship self-review flow.

### Pre-flight

- M2 complete.

### Steps

1. Prepare a small trade-journal CSV (generic CSV format accepted). Create `trades_export.csv` with columns such as:

   ```csv
   symbol,date,side,price,quantity
   AAPL,2024-01-05,buy,181.00,10
   AAPL,2024-01-25,sell,195.00,10
   MSFT,2024-02-01,buy,410.00,5
   MSFT,2024-02-20,sell,405.00,5
   ```

   (Column names may vary — the agent parses common formats; a 4–10 row file is enough for a demo.)

2. Upload and analyze:

   ```bash
   vibe-trading --upload trades_export.csv
   vibe-trading run -p "Analyze my trading behavior, extract my shadow strategy, and compare it with my actual trades"
   ```

3. Inspect the report artifact (HTML by default; PDF optional if weasyprint system libs are present).

### Verify

- The run produces a behavioral profile (holding days, win rate, PnL ratio, drawdown, disposition effect / overtrading checks) and a shadow-strategy comparison.
- A report file (HTML) exists in the run directory.

Both hold -> M6 complete.

### Troubleshooting

- **Parser rejects the CSV:** check the format against the README's Shadow Account section or ask the agent what columns it expects; generic CSV with `symbol,date,side,price,quantity` is the documented baseline.
- **No report rendered:** HTML works everywhere; PDF needs weasyprint system libs (see M1 troubleshooting).

### Fallback

No journal file: the agent can synthesize a tiny sample journal for demo purposes (`vibe-trading run -p "Create a sample trade journal and run a shadow-account analysis on it"`) — the point is the workflow, not the data.

---

## 7. Risk List & Mitigation

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| 1 | Student machine has Python < 3.11 | Medium | High (blocks M1) | M0 pre-check; install 3.11/3.12 first; or Docker path |
| 2 | No DeepSeek key / account signup friction | Medium | High (blocks M2+) | Ollama local fallback (no key); instructor-shared key with budget cap |
| 3 | Classroom firewall blocks GitHub/PyPI | Medium | High | Mirror/ZIP download; instructor pre-baked Docker image |
| 4 | Firewall blocks market-data hosts (OKX/Yahoo/AKShare) | Medium | Medium | `VIBE_TRADING_DATA_CACHE=1` pre-warmed cache; explicit `source` override; equity vs crypto swap |
| 5 | LLM proxy/rate-limit/SSE timeout issues | High | Medium | `VIBE_TRADING_DISABLE_HTTP_PROXY=1`; per-group keys; raise `VIBE_TRADING_SSE_TIMEOUT`/`TIMEOUT_SECONDS` |
| 6 | Small model ignores tools ("answers from memory") | Medium | High | Use `deepseek-chat` or Ollama 7b/3b; never `*-nano`/`*-flash-lite` |
| 7 | Swarm runs cost too many tokens | Medium | Medium | Narrow topics; 2-agent presets; `LANGCHAIN_TEMPERATURE=0.0`; account spending cap |
| 8 | Node < 22.22 for frontend | Medium | Low | Frontend is optional; `vibe-trading serve` + TUI cover the same flow |
| 9 | Docker memory (4g limit) on small laptops | Low | Medium | Local venv path instead; compose limits are configurable |
| 10 | Stale model id in docs (e.g. `deepseek-v4-pro`) | High | Low | Use stable `deepseek-chat` / `deepseek-reasoner`; verify with `provider doctor` |
| 11 | Accidental OpenAI endpoint/env leakage | Low | High | Provider-specific `<PROVIDER>_BASE_URL` wins; `provider doctor` audit; never commit `.env` |
| 12 | Student pastes real broker data / secrets | Low | High | Use synthetic journals; keep `.env` out of git; AGENT_CONTRIBUTOR_GUIDE hygiene rules |

---

## 8. Instructor Notes (lab preparation)

- **Pre-warm plan (30–60 min, one networked machine):** clone repo, install in a venv (M1), run the M2 example twice to fill the data cache, copy the cache + venv (or a Docker image) to lab machines.
- **Keys:** request per-group DeepSeek keys ahead of class; set spending caps; stagger M2/M5 exercise starts to avoid shared-key 429s.
- **Offline lab:** distribute a pre-built Docker image (or VM snapshot) with repo + deps + warmed cache; students then run M2–M6 fully offline except LLM (Ollama pre-installed with qwen2.5:3b/7b).
- **Safety defaults (keep them):** API stays loopback-only; `VIBE_TRADING_ENABLE_SHELL_TOOLS` stays off; no broker/live-trading connectors in class.
- **Quick suite check (optional, instructor only):** `pytest --ignore=agent/tests/e2e_backtest` from the repo root runs the fast unit suite (network-blocked) — a good smoke test after install.

---

## 9. Quick-Reference Command Sheet

```bash
# M0
git clone --depth 1 https://github.com/HKUDS/Vibe-Trading.git && cd Vibe-Trading

# M1
python3 -m venv .venv && source .venv/bin/activate
pip install -e . && pip install -e ".[deepseek]"
cp agent/.env.example agent/.env    # set DeepSeek block only (see M1)
vibe-trading provider doctor

# M2
vibe-trading run -p "Backtest a BTC-USDT 20/50 moving-average strategy for 2024 and summarize return and drawdown"

# M3
vibe-trading alpha list
vibe-trading alpha bench --zoo gtja191 --universe csi300 --period 2023-2024 --top 10

# M4
vibe-trading serve --port 8899      # open http://localhost:8899

# M5
vibe-trading --swarm-presets
vibe-trading --swarm-run investment_committee '{"topic": "BTC 200-day MA signal for 2024"}'

# M6
vibe-trading --upload trades_export.csv
vibe-trading run -p "Analyze my trading behavior, extract my shadow strategy, and compare it with my actual trades"
```

*End of plan. Grounded in `repo-analysis.md` (repository facts), `llm-provider-guidance.md` (no-OpenAI LLM config), and a first-hand pass over `README.md`, `pyproject.toml`, `Dockerfile`, `docker-compose.yml`, `agent/.env.example`, and `agent/src/swarm/presets/`.*



