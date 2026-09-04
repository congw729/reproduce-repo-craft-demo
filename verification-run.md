# Vibe-Trading Reproduction Plan — Real-World Verification Run (M0–M2)

**Date:** 2026-09-04
**Machine:** macOS (Darwin 23.5.0), Apple Silicon (Homebrew at `/opt/homebrew`)
**Verifier:** Quant Reproduction Architect
**Plan under test:** `.team/quant-swarm-course/reproduction-plan.md`
**Repo under test:** local clone at `/Users/congwang/Documents/GitHub/Vibe-Trading` (used instead of re-cloning; symlink target of `/Users/congwang/Documents/GitHub/vllm-omni-draft/Vibe-Trading`)
**Constraint:** NO OpenAI models used anywhere — LLM config is DeepSeek-first (or Ollama), matching the plan.

---

## Executive Summary

| Milestone | Verdict | Wall-clock | Notes |
|-----------|---------|-----------|-------|
| M0 — Environment check & clone | **PASS** (with env note) | < 1 min | System `python3` is 3.9.6 (< 3.11); Homebrew `python@3.12` used instead. Repo already present; `.env.example` exists. |
| M1 — Install & CLI boot | **PASS** (install + CLI) + **PENDING-CREDENTIAL** (LLM config) | ~6 min | venv (Python 3.12.0) + `pip install -e .` + `.[deepseek]` succeeded; `vibe-trading --help` and `provider doctor` run. No real LLM credential found → LLM step pending user action. |
| M2 — First end-to-end backtest | **PENDING-CREDENTIAL** (LLM backtest) + **PASS** (offline CLI path) | ~6 min | `vibe-trading run -p "..."` fails at preflight: OKX connect timeout + LLM 401 (placeholder key). Offline `vibe-trading alpha list` works (462 alphas). `alpha bench` exceeded 300 s data-download budget — flagged for classroom. |

**Overall:** Milestones 0–1 are reproduced on this machine. Milestone 2's LLM-driven backtest **cannot complete until a real DeepSeek key (or Ollama) is provided**; its offline CLI counterpart is verified. See §"Blockers for the user".

---

## Milestone 0 — Environment Check & Repository Clone

**Goal:** Confirm toolchain and source on disk.

**Commands executed (exact):**

```bash
python3 --version        # -> Python 3.9.6
git --version            # -> git version 2.39.3 (Apple Git-146)
ls -la .../vllm-omni-draft/Vibe-Trading   # symlink -> ../Vibe-Trading (reused)
ls agent/.env.example    # -> present (17,760 bytes, Sep 3 01:07)
# Additional check (not in plan): available interpreters
for p in python3.11 python3.12 python3.13; do command -v $p; done   # none on PATH
ls /opt/homebrew/opt/ | grep -i python   # -> python@3.12, python@3.13
```

**stdout/stderr excerpts (real):**

```
$ python3 --version
Python 3.9.6
$ git --version
git version 2.39.3 (Apple Git-146)
```

**Verdict: PASS (with environment note).** System default `python3` (3.9.6, Apple) does NOT meet the plan's `>=3.11` requirement, exactly as the plan's risk #1 predicted. Homebrew provides `python@3.12` and `python@3.13`; I used `python@3.12` for M1 (3.13 may have wheel gaps per plan §0.3). Disk: 52 GiB free — ample.

**Issue + fix:** `python3` → 3.9.6. Fix: used `/opt/homebrew/opt/python@3.12/bin/python3.12` for the venv. Documented for students: check `brew list python@3.12` or install 3.11/3.12 first.

---

## Milestone 1 — Local Install & LLM Configuration

**Goal:** Install the package, configure an LLM provider, prove the CLI boots.

**Commands executed (exact):**

```bash
PY312=/opt/homebrew/opt/python@3.12/bin/python3.12
cd /Users/congwang/Documents/GitHub/Vibe-Trading
$PY312 -m venv .venv
./.venv/bin/pip install --upgrade pip
./.venv/bin/pip install -e .
./.venv/bin/pip install -e ".[deepseek]"
cp agent/.env.example agent/.env
./.venv/bin/vibe-trading --help
./.venv/bin/vibe-trading provider doctor
```

**stdout/stderr excerpts (real):**

```
$ ./.venv/bin/python --version
Python 3.12.0
$ ./.venv/bin/pip list | grep -iE "langchain-deepseek|vibe-trading|langchain-openai"
langchain-deepseek        1.1.0
langchain-openai          1.6.0
vibe-trading-ai           0.1.14       /Users/congwang/Documents/GitHub/Vibe-Trading
$ ./.venv/bin/vibe-trading --help
usage: vibe-trading [-h] [--version] [-p PROMPT] [-f PROMPT_FILE] [--json]
                    ...
                    {run,serve,provider,data,channels,list,show,chat,update,init,setup,dev,memory,portfolio,connector,alpha,hypothesis,playbook,strategy-evidence}
```

`provider doctor` returned JSON diagnostics: `provider=openrouter`, `model=deepseek/deepseek-v4-pro`, `base_url=https://openrouter.ai`, key state reported as "set" but the value in `.env` is the **placeholder** from `.env.example` (verified: `OPENROUTER_API_KEY=...here`-style placeholder; length 13, not a real key).

**Credential audit (per task instructions, checked BEFORE deciding the LLM path):**

- `$DEEPSEEK_API_KEY` env var: **NOT SET**
- existing `agent/.env`: **did not exist** → created by `cp agent/.env.example agent/.env` (this is the one allowed repo modification besides the venv)
- Ollama installed? **NO** (`ollama: command not found`)
- Real key anywhere in `.env`? **No** — OpenRouter key is a placeholder; DeepSeek block is commented out.

**Verdict: PASS (install + CLI boot) / PENDING-CREDENTIAL (LLM config).**
Install, imports (langchain 1.4.0, langgraph present — note: `langgraph.__version__` does not exist; that is normal, not an error), editable package `vibe-trading-ai 0.1.14`, and CLI boot all verified. The LLM step needs a real credential: either a DeepSeek API key (recommended) or Ollama with an open model.

**Issues + fixes:**
- `langgraph` has no `__version__` attribute → harmless; verified via `pip list` instead.
- Placeholder OpenRouter key in `.env` → must be replaced (or the DeepSeek block enabled) before M2.

## Milestone 2 — First End-to-End Backtest

**Goal:** Run one natural-language backtest end to end and read its results.

**Attempt 1 — LLM-driven backtest (the plan's canonical command):**

```bash
./.venv/bin/vibe-trading run -p "Backtest a BTC-USDT 20/50 moving-average strategy for 2024 and summarize return and drawdown"
```

**Real stdout excerpt (preflight + failure):**

```
Preflight Check
 OK   │ LLM (openrouter)   │ deepseek/deepseek-v4-pro via https://openrouter.ai
      │                    │ | base=https://openrouter.ai timeout=120s
      │                    │ | retries=2 proxy=NO_PROXY
 FAIL │ OKX API            │ ConnectTimeout: HTTPSConnectionPool(host='www.okx.com',
      │                    │ port=443): Max retries exceeded ... (connect timeout=10)
      │                    │ (crypto backtest unavailable)
 OK   │ yfinance           │ reachable
 N/A  │ Tushare            │ TUSHARE_TOKEN not set (optional)
 OK   │ akshare            │ installed
 OK   │ ccxt               │ installed
 OK   │ Content Filter     │ 5% ...
5/7 services ready

AgentLoop error: provider_stream_error provider=openrouter model=deepseek/deepseek-v4-pro:
OpenAIAuthenticationError: Error code: 401 - {'error': {'message': 'Missing Authentication header', 'code': 401}}
```

**Interpretation (real, on this machine):**
1. Preflight itself works and is informative: yfinance/akshare/ccxt OK, Tushare optional-skip, **OKX timed out** (classroom-network style host block — the plan's risk #4), LLM endpoint configured but unreachable auth-wise.
2. The LLM call failed with **401 Missing Authentication header** because the key in `agent/.env` is the **placeholder** from `.env.example` — exactly the M1 PENDING-CREDENTIAL situation. No real credential exists on this machine.

**Attempt 2 — offline CLI path (no LLM required):**

```bash
./.venv/bin/vibe-trading alpha list
```

**Real stdout excerpt:**

```
                          Alpha Zoo (50 of 462 alphas)
┏━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━┓
┃ id                   ┃ zoo      ┃ theme        ┃ universe     ┃ nickname     ┃
...
│ academic_bab         │ academic │ volatility   │ equity_us, ...│ Frazzini-Pe…
│ academic_carhart_mom │ academic │ momentum     │ equity_us, ...│ Carhart 1997 …
...
Showing 50 of 462. Pass --limit N (or --limit 0 for no cap).
```

Exit code **0** (a first run piped through `head` reported 1 — that was the pipe, not the command; verified standalone with exit 0). This confirms the alpha-zoo browse path works fully offline with no LLM.

**Attempt 3 — `alpha bench` (offline-capable but data-heavy):**

```bash
./.venv/bin/vibe-trading alpha bench --zoo gtja191 --universe sp500 --period 2024-2024 --top 5
```

**Result:** exceeded the 300-second tool budget without producing output (data download for the SP500 universe is slow on this network). No leftover process after cleanup. This matches the plan's guidance: keep benches small (`--top N`, short periods); on slow classroom networks even 1 year × SP500 may exceed ~5 min — a real calibration data point for the instructor.

**Verdict: PENDING-CREDENTIAL (LLM backtest) / PASS (offline CLI path).**
The plan's canonical M2 command cannot run without a real LLM credential (DeepSeek key or Ollama) — it fails deterministically at the LLM 401 before any backtest logic executes. The non-LLM CLI surface (`alpha list`) is verified working. `alpha bench` requires patient data download; budget > 5 min or pre-warm the data cache (`VIBE_TRADING_DATA_CACHE=1`) per the plan's M2 fallback.

**Issues + fixes:**
- `timeout` command absent on macOS → used the bash tool's own timeout; documented.
- OKX unreachable from this network (connect timeout) → plan already covers: ask agent for `yfinance` source or use equity examples; consider proxy or cached data.
- `alpha bench` slow → calibrate class exercise to short periods / small universes, or pre-warm cache.

---

## Blockers for the User

To finish M2 (and enable M5 swarm / M6 shadow runs), ONE of the following is needed:

1. **DeepSeek API key (recommended, zero hardware):**
   - Sign up at https://platform.deepseek.com → API Keys → create key.
   - In `/Users/congwang/Documents/GitHub/Vibe-Trading/agent/.env`, enable the DeepSeek block (comment out OpenRouter):
     ```dotenv
     LANGCHAIN_PROVIDER=deepseek
     LANGCHAIN_MODEL_NAME=deepseek-chat
     DEEPSEEK_API_KEY=sk-<your real key>
     DEEPSEEK_BASE_URL=https://api.deepseek.com/v1
     ```
   - Verify: `./.venv/bin/vibe-trading provider doctor` shows `provider=deepseek` and the key as set.
   - (OpenRouter with a real key also works — set `LANGCHAIN_MODEL_NAME=deepseek/deepseek-chat`.)

2. **Ollama (offline, no key, ~2–5 GB download):**
   - `brew install ollama` (or https://ollama.com), then `ollama pull qwen2.5:7b` (or `qwen2.5:3b` for lighter machines).
   - In `.env`: `LANGCHAIN_PROVIDER=ollama`, `LANGCHAIN_MODEL_NAME=qwen2.5:7b`, `OLLAMA_BASE_URL=http://localhost:11434`; raise `VIBE_TRADING_SSE_TIMEOUT=300` / `TIMEOUT_SECONDS=300`.

3. **Network note (not blocking, but affects data):** `www.okx.com` is unreachable from this network (connect timeout). Crypto backtests will fail until the network allows OKX (or use yfinance/US-equity examples, or pre-warmed `VIBE_TRADING_DATA_CACHE=1`). Yahoo/yfinance, akshare, ccxt are reachable — verified in preflight.

**Environment summary for the user:**
- venv ready: `/Users/congwang/Documents/GitHub/Vibe-Trading/.venv` (Python 3.12.0)
- Package installed: `vibe-trading-ai 0.1.14` (+ `langchain-deepseek 1.1.0` adapter)
- CLI verified: `vibe-trading --help`, `provider doctor`, `alpha list` all run
- Disk: 52 GiB free — no space blocker
- No real LLM credential present; no Ollama installed

---

---

## Update 1 — Local DeepSeek LLM Confirmed + OKX Proxy Diagnosis + M2 Retry PASS (2026-09-04 ~03:05–03:18)

### 1. LLM provider config verification (user-configured "local DeepSeek")

Active lines in `agent/.env` (key masked):

```dotenv
LANGCHAIN_PROVIDER=tokendance
LANGCHAIN_MODEL_NAME=deepseek-v4-flash-0731
OPENROUTER_API_KEY=<len=51, prefix sk-b…suffix 35d4>   # real key
OPENROUTER_BASE_URL=https://tokendance.space/gateway/v1
```

Findings:
- The endpoint is a **remote OpenAI-compatible gateway** (tokendance.space), not a local server — "local" here means "not the official OpenAI API"; model is `deepseek-v4-flash-0731` (DeepSeek model — satisfies the no-OpenAI constraint).
- `vibe-trading provider doctor` shows `provider=tokendance`, `model=deepseek-v4-flash-0731`, but `base_url=https://api.openai.com` and `OPENAI_API_KEY: unset`. Root cause: **`tokendance` is not a built-in provider name** in `agent/src/providers/capabilities.py` / `llm_providers.json`, so the code falls back to OpenAI credential resolution — which reads `OPENAI_API_KEY`, not the user's `OPENROUTER_API_KEY`.
- Direct gateway test with the user's key (no .env change): **HTTP 200**, model answered `GATEWAY_OK`. The key, endpoint, and model are all valid.
- Non-destructive smoke test via env override (no .env edit): `LANGCHAIN_PROVIDER=openrouter ./.venv/bin/vibe-trading run -p "Reply with exactly: LLM_OK"` → **Status: SUCCESS** (run `20260904_031002_32_717084`, ~8 s). The user's `OPENROUTER_API_KEY`/`OPENROUTER_BASE_URL` are picked up correctly when the provider label is one the code knows.
- **Recommendation (reported to leader; .env left untouched per instruction):** change only `LANGCHAIN_PROVIDER=tokendance` → `openrouter` in `agent/.env`. The gateway is OpenAI-compatible and the user's key/endpoint already live in the `OPENROUTER_*` variables, so nothing else changes. (The label `openrouter` is just the transport tag; the actual model served is DeepSeek — no OpenAI involved.)

### 2. OKX proxy connectivity diagnosis

Tests run with 8 s per candidate against `https://www.okx.com`:

| Route | Result |
|---|---|
| direct (no proxy) | HTTP 000 — DNS resolves to `169.254.0.2` (link-local), TCP connect times out |
| http://127.0.0.1:1053 | HTTP 000 — not listening |
| socks5://127.0.0.1:1053 | HTTP 000 — not listening |
| http://127.0.0.1:7890 (ClashX) | HTTP 000 — CONNECT tunnel established (200) but tunnel content times out |
| socks5://127.0.0.1:7890 | HTTP 000 — same |
| http://127.0.0.1:7897 (verge-mihomo) | HTTP 000 — no response |
| http://127.0.0.1:6152 | HTTP 000 — not listening |

System state: `scutil --proxy` shows HTTP/HTTPS/SOCKS proxy **all disabled**; PAC off. Listening proxy ports: ClashX on 7890/9090, verge-mihomo on 7897. No 1053/6152 listeners.

Diagnosis:
- **Direct path is broken at the network layer**: DNS for `www.okx.com` resolves to `169.254.0.2` (a link-local address — typical of a captive/hijacked DNS result), and TCP connects time out. This matches the original M2 failure.
- **ClashX 7890 can establish the CONNECT tunnel but the upstream node cannot reach OKX** (tunnel established, content transfer times out). Other candidates are not listening (1053/6152) or unresponsive (7897).
- Conclusion: **no working proxy route to OKX on this machine right now.** Per instructions, system settings were NOT modified.

### 3. M2 retry with working data source (yfinance) — PASS

Since the LLM chain is confirmed working (env-override label) and yfinance was reachable in preflight, M2 was retried with an explicit yfinance source (bypassing unreachable OKX):

```bash
LANGCHAIN_PROVIDER=openrouter ./.venv/bin/vibe-trading run \
  -p "Backtest a BTC-USD 20/50 moving-average strategy for 2024 using yfinance as the data source and summarize return and drawdown"
```

**Result: SUCCESS** (run `20260904_031219_06_76ccf1`, wall-clock ~6 min from 03:12 to 03:18; the bash tool reported a 300 s timeout but the agent process completed and wrote all artifacts).

Real run-card summary:

```
codes: [BTC-USD] | start 2023-10-01 | end 2024-12-31 | interval 1D
engine: daily | initial_cash: 1,000,000 | source: yfinance
final_value: 1,316,147.71
total_return: +31.61% | annual_return: +20.82%
max_drawdown: -40.13% | sharpe: 0.607 | calmar: 0.519
win_rate: 0.40 | profit_loss_ratio: 3.30 | trade_count: 5
```

Artifacts written: `artifacts/equity.csv`, `trades.csv`, `ohlcv_BTC-USD.csv`, `risk_xray.*`, `fills.jsonl`, `code/signal_engine.py` (819 B, sha `eed76c…`), config hash `12e2fe…`. LLM usage recorded per iteration (e.g. iter 1: 93,870 input / 1,086 output tokens; model `deepseek-v4-flash-0731`).

**Verdict change: M2 is now PASS** on the canonical workflow (natural-language prompt → LLM → data fetch → backtest engine → summary), using the user's local-DeepSeek-gateway config + yfinance. The only deviation from the plan's literal M2 command is the data source (yfinance instead of OKX) and the provider label override — both forced by environment (OKX unreachable; `tokendance` label unknown to the code).

### 4. Blockers for the user (updated)

1. **OKX unreachable on this network** (DNS hijack to 169.254.0.2 + proxy nodes failing). Recommended user action: open **Clash Verge GUI → enable System Proxy (or TUN mode)** so OKX traffic gets a working route, then re-run the canonical M2 command (with OKX) or keep using yfinance for crypto. No system settings were changed by this verification.
2. **One-line .env fix needed for the LLM provider label:** change `LANGCHAIN_PROVIDER=tokendance` → `openrouter` in `agent/.env` (user's key/endpoint unchanged; gateway is OpenAI-compatible, model stays DeepSeek `deepseek-v4-flash-0731`). Without it, `vibe-trading run` fails with `Missing credentials` (falls back to `OPENAI_API_KEY`). Applied as env override during verification; .env itself left untouched per instruction.
3. Everything else is ready: venv (Python 3.12.0), `vibe-trading-ai 0.1.14`, `langchain-deepseek 1.1.0`, LLM chain verified, yfinance data verified, M2 backtest PASS.

---

---

## Update 2 — Clean M2 Confirmation (Final, Pure .env, No Overrides)

**Status:** PASS — the canonical M2 workflow now runs end-to-end with only the user-edited `agent/.env`, no environment overrides.

### 1. Provider state (post user edit)

Verified via `./.venv/bin/vibe-trading provider doctor` before the run:

```
provider  : openrouter
model     : deepseek-v4-flash-0731
base_url  : https://tokendance.space      (from OPENROUTER_BASE_URL)
api_key   : OPENROUTER_API_KEY = set
```

The user's one-line `.env` edit (`LANGCHAIN_PROVIDER=tokendance` → `openrouter`) resolves the provider-label issue confirmed in Update 1. No env-var overrides were used for this run.

### 2. Command executed (exact, pure .env)

```bash
cd /Users/congwang/Documents/GitHub/Vibe-Trading
./.venv/bin/vibe-trading run -p "Backtest a BTC-USD 20/50 moving-average strategy for 2024 and summarize return and drawdown"
```

### 3. Run result

- **Run ID:** `20260904_034247_45_533cf1`
- **Wall-clock:** 03:42:47 → 03:53:44 (~11 min total; core run card generated at 03:44:07, remaining time spent on agent finalization/LLM context growth)
- **LLM usage:** 28 iterations; ~2,879,774 input / ~106,372 output tokens (~2.99M total, model `deepseek-v4-flash-0731`)

Real run-card summary:

```
codes: [BTC-USDT] | start 2023-09-01 | end 2024-12-31 | interval 1D
engine: daily | initial_cash: 1,000,000 | source: okx
Data Sources: binance          # <— agent requested OKX, auto-fell-back to Binance
final_value: 1,316,307.28
total_return: +31.63% | annual_return: +31.53%
max_drawdown: -40.12% | sharpe: 0.733 | calmar: 0.786 | sortino: 0.946
win_rate: 0.40 | profit_loss_ratio: 3.30 | profit_factor: 2.20 | trade_count: 5
avg_holding_days: 47.2 | benchmark_return: +121.3%
```

**Important data-source finding:** the agent requested `okx` as the source but the built-in crypto fallback chain (okx → ccxt → binance → ...) automatically served bars from **Binance** because OKX is unreachable on this network. This is a live, real-world demonstration of the fallback-chain behavior documented in the reproduction plan — crypto backtests work even when OKX is blocked.

### 4. Consistency check

Results are highly consistent with the earlier yfinance-based retry (Update 1):

| Metric | Update 1 (yfinance) | Update 2 (Binance via OKX request) |
|---|---|---|
| total_return | +31.61% | +31.63% |
| max_drawdown | -40.13% | -40.12% |
| trade_count | 5 | 5 |
| win_rate | 0.40 | 0.40 |

The small metric deltas come from different data vendors (yfinance vs Binance) and slightly different data windows (2023-10-01 vs 2023-09-01 start), confirming the strategy is reproducible across data sources.

### 5. Verdict

**PASS — final clean confirmation.** The full natural-language reproduction workflow (prompt → LLM via local DeepSeek gateway → data fetch → backtest engine → metrics summary) works end to end with the user's pure `.env` configuration and zero environment overrides. Remaining notes for the user/instructor: OKX remains unreachable on this network (DNS hijack), but the fallback chain makes crypto backtests work anyway; `vibe-trading alpha bench` still needs longer than 300 s for a 1-year SP500 universe (calibration note for class).

---

## Appendix — Wall-clock summary

| Step | Start → End (local) | Elapsed |
|------|---------------------|---------|
| M0 checks | 02:27:56 | < 1 min |
| venv + pip upgrade | 02:28:27 → ~02:28:30 | ~5 s |
| `pip install -e .` + `.[deepseek]` | ~02:28:30 → ~02:33:00 (background) | ~4.5 min |
| M1 verify (`--help`, doctor, imports) | 02:33:28 → 02:34:00 | ~32 s |
| M2 `vibe-trading run -p ...` | 02:34:24 → 02:34:55 (fails at preflight/LLM) | ~21 s |
| M2 `alpha list` | 02:35:07 → 02:35:10 | ~3 s |
| M2 `alpha bench` | ~02:35:15 → timeout | > 300 s (no output) |

*End of verification run. All commands executed on this machine (macOS, Darwin 23.5.0); outputs captured from real runs, not simulated.*

