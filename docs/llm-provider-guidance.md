# SwarmCraft — LLM Provider Guidance for Vibe-Trading (No OpenAI — DeepSeek & Open-Source Only)

> **Purpose.** This document makes the project-wide constraint **"no OpenAI GPT models
> anywhere"** concrete for the Vibe-Trading reproduction and the SwarmCraft course demo. It is
> self-contained: a student can configure the entire LLM layer of Vibe-Trading without
> touching OpenAI — no OpenAI account, no OpenAI API key, no `gpt-*` model names.
>
> **Scope.** Everything below is grounded in the actual repository
> (`agent/.env.example`, `agent/src/providers/llm.py`, `agent/src/providers/capabilities.py`,
> `agent/src/providers/llm_providers.json`, `agent/src/config/env_schema.py`,
> `agent/requirements.txt`, `pyproject.toml`) and the upstream analysis
> (`repo-analysis.md`, §4.4 "LLM provider abstraction"). No OpenAI GPT configuration is
> documented anywhere in this file.

---

## 1. How Vibe-Trading Consumes an LLM Provider

### 1.1 One factory, many providers

Vibe-Trading talks to models exclusively through a **provider abstraction**, not through
hard-coded OpenAI calls:

```
LANGCHAIN_PROVIDER / LANGCHAIN_MODEL_NAME + <PROVIDER>_API_KEY / <PROVIDER>_BASE_URL
        │
        ▼
agent/src/providers/llm.py :: build_llm()          ← the single factory
        │
        ▼
ChatLLM (agent/src/providers/chat.py)               ← streaming, retries, timeouts
        │
        ├──► ReAct AgentLoop (agent/src/agent/loop.py)   ← main trading/research agent
        ├──► Swarm workers (agent/src/swarm/worker.py)   ← multi-agent DAG engine
        └──► Tools (web_search, image_vision, OCR, ...)   ← agent/src/tools/
```

The factory resolves the provider from `LANGCHAIN_PROVIDER`, looks up the credential and
endpoint env-var names in `agent/src/providers/capabilities.py` /
`llm_providers.json`, and builds a LangChain chat model. The **same env-var contract**
works for every supported provider (DeepSeek, Ollama, OpenRouter, SiliconFlow, Groq,
DashScope/Qwen, Zhipu, ModelScope, …) — only the values change.

### 1.2 Where the configuration lives

| Item | Location / env var |
|---|---|
| Provider selector | `LANGCHAIN_PROVIDER` (default `openai` in `env_schema.py` — **override it**) |
| Model name | `LANGCHAIN_MODEL_NAME` (required; runtime error if unset) |
| Temperature | `LANGCHAIN_TEMPERATURE` (default `0.0`) |
| Timeouts / retries | `TIMEOUT_SECONDS` (default 120), `MAX_RETRIES` (default 2) |
| Reasoning effort | `LANGCHAIN_REASONING_EFFORT` (`none` / `low` / `medium` / `high` / `max`) |
| Per-provider key | `<PROVIDER>_API_KEY`, e.g. `DEEPSEEK_API_KEY` |
| Per-provider endpoint | `<PROVIDER>_BASE_URL`, e.g. `DEEPSEEK_BASE_URL` |
| DeepSeek adapter mode | `VIBE_TRADING_DEEPSEEK_ADAPTER` (`auto` / `native` / `openai-compatible`) |
| Ignore HTTP(S)_PROXY for LLM calls | `VIBE_TRADING_DISABLE_HTTP_PROXY=1` |
| `.env` search order | `~/.vibe-trading/.env` → `agent/.env` → `$CWD/.env` (first match wins) |

> **Credential fallback chain** (`capabilities.py :: get_llm_credentials`): the provider's
> own `<PROVIDER>_API_KEY` is read first, then `OPENAI_API_KEY` as a **generic bearer-token
> fallback** (any OpenAI-compatible endpoint accepts an arbitrary key string — this does
> **not** mean OpenAI is used). The endpoint falls back in the order
> `<PROVIDER>_BASE_URL` → `OPENAI_BASE_URL` → `OPENAI_API_BASE` → the provider's default
> base URL from `llm_providers.json`.

### 1.3 Quick diagnostic command

The repo ships a redacted provider doctor. Run it after configuring to verify that
provider, model, key, endpoint, packages, and proxy state are correct:

```bash
vibe-trading provider doctor
```

It prints only redacted diagnostics (never the API key value). Also covered by the
preflight checks at startup.

---

## 2. Primary Provider: DeepSeek API

DeepSeek is the **recommended default** for the course: hosted (no local GPU needed),
cheap, and the repo has a first-class DeepSeek adapter with reasoning support.

### 2.1 Install the optional adapter (once)

DeepSeek has two adapters; the `auto` mode picks the best available:

```bash
pip install "vibe-trading-ai[deepseek]"      # installs langchain-deepseek>=1.0.0,<2
# or, inside a source checkout:
pip install -e ".[deepseek]"
```

`langchain-openai>=1.0.0,<2` is already a **base dependency** of the project — it is the
generic OpenAI-compatible transport and is required regardless of provider. It does not
make the project use OpenAI models.

### 2.2 Obtain an API key

1. Create an account at **https://platform.deepseek.com** (DeepSeek official platform).
2. Go to **API Keys** → create a key, copy it, and treat it like a password (never commit
   it to git; never paste it into shared course documents).

### 2.3 Configure `agent/.env`

Edit `agent/.env` (copy `agent/.env.example` first: `cp agent/.env.example agent/.env`)
and **uncomment / set only the DeepSeek block**:

```bash
# --- DeepSeek (primary) ---
LANGCHAIN_PROVIDER=deepseek
LANGCHAIN_MODEL_NAME=deepseek-chat        # see §2.4 for model names
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxx  # your real key
DEEPSEEK_BASE_URL=https://api.deepseek.com/v1

# Optional but recommended for the course:
LANGCHAIN_TEMPERATURE=0.0                 # deterministic, reproducible answers
TIMEOUT_SECONDS=120
MAX_RETRIES=2
# VIBE_TRADING_DEEPSEEK_ADAPTER=auto      # auto | native | openai-compatible (default auto)
```

> **Adapter modes** (`llm.py :: _deepseek_adapter_mode`):
> - `auto` (default) — use the native `langchain-deepseek` adapter when installed, else
>   fall back to the OpenAI-compatible path. Best for students.
> - `native` — require `langchain-deepseek`; raise a clear error if missing.
> - `openai-compatible` — force the generic `ChatOpenAI` path (works with only the base
>   `langchain-openai` dependency).

### 2.4 Model names

| Model id | Type | When to use |
|---|---|---|
| `deepseek-chat` | general chat / tool-calling | **default for the course** — fast, cheap, good at tool use (ReAct loops, swarm workers) |
| `deepseek-reasoner` | reasoning model | optional stretch exercise (slower, more tokens, better for hard analysis steps) |

If your installed repo version lists a different default model in
`agent/.env.example` / `llm_providers.json` (e.g. `deepseek-v4-pro`), the model string is
just a label sent to the API — use whichever model id your DeepSeek account actually
exposes via the **DeepSeek API docs**; `deepseek-chat` and `deepseek-reasoner` are the
stable public ids.

### 2.5 Verify

```bash
vibe-trading provider doctor        # provider=deepseek, key set, base URL https://api.deepseek.com
vibe-trading run -p "What is the 20-day moving average of AAPL? Use get_market_data."
```

A successful reply means the full agent stack (factory → ChatLLM → ReAct loop → tools) is
talking to DeepSeek.

---

## 3. Local Fallback: Ollama (Open-Weight Models, No API Key)

For classrooms with **no internet at runtime**, **no budget**, or **privacy constraints**,
run open-weight models locally with Ollama. The repo has a built-in Ollama provider
(`api_key_env = None` in `capabilities.py` — **no key required**).

### 3.1 Install and pull a model

```bash
# 1. Install Ollama from https://ollama.com (macOS / Linux / Windows)
# 2. Pull a classroom-friendly model:
ollama pull qwen2.5:7b          # ~4.7 GB, runs on 8 GB RAM laptops (CPU or small GPU)
# lighter options: qwen2.5:3b (~2 GB), llama3.2:3b (~2 GB)
```

> The repo's default Ollama model entry is `qwen2.5:32b` — **too heavy for typical student
> hardware**. Prefer `qwen2.5:7b` or `qwen2.5:3b` in the classroom (see §5 hardware table).

### 3.2 Configure `agent/.env`

```bash
# --- Ollama (local, no key) ---
LANGCHAIN_PROVIDER=ollama
LANGCHAIN_MODEL_NAME=qwen2.5:7b
OLLAMA_BASE_URL=http://localhost:11434

# Local models are slow — give the UI/SSE more patience:
VIBE_TRADING_SSE_TIMEOUT=300     # default 90s; raise for CPU-only inference
TIMEOUT_SECONDS=300
```

Notes:
- The factory normalizes the Ollama URL to its OpenAI-compatible root automatically
  (`http://localhost:11434` → `http://localhost:11434/v1`, `capabilities.py`).
- No API key is needed; the credential resolver supplies the placeholder key `ollama`.
- **Performance tip for students:** keep prompts/tasks short, prefer `3b/7b` models, and
  run only one agent task at a time on CPU-only machines.

### 3.3 Verify

```bash
ollama list                        # model present
vibe-trading provider doctor       # provider=ollama, api_key not required
vibe-trading run -p "Say hello and list three stock data sources you know."
```

### 3.4 Optional: vLLM server (advanced / shared lab server)

If the classroom has one GPU workstation, a shared vLLM server serves many students:

```bash
pip install vllm
vllm serve Qwen/Qwen2.5-7B-Instruct --port 8000
```

Then point Vibe-Trading at it (any OpenAI-compatible endpoint works):

```bash
LANGCHAIN_PROVIDER=openai          # generic OpenAI-compatible transport label
LANGCHAIN_MODEL_NAME=Qwen/Qwen2.5-7B-Instruct
OPENAI_API_KEY=not-needed          # any non-empty string; vLLM ignores it
OPENAI_BASE_URL=http://<server-ip>:8000/v1
```

> **Constraint note:** `LANGCHAIN_PROVIDER=openai` here is only the **transport label**
> (OpenAI-*compatible* wire format). The model is an open-weight Qwen served by vLLM —
> no OpenAI model is involved. Use this route only if you understand that distinction;
> otherwise prefer the named providers above.

---

## 4. Cloud Open-Source / Non-OpenAI Alternatives (With API Key)

If DeepSeek is unavailable in the classroom region, these OpenAI-compatible providers
also work **with the same env-var pattern** (all served via `llm_providers.json`):

| Provider | `LANGCHAIN_PROVIDER` | Model example | Endpoint | Key env var |
|---|---|---|---|---|
| OpenRouter (multi-model gateway) | `openrouter` | `deepseek/deepseek-chat` | `https://openrouter.ai/api/v1` | `OPENROUTER_API_KEY` |
| SiliconFlow (CN/Global) | `siliconflow-cn` / `siliconflow-global` | `deepseek-ai/DeepSeek-V3` | `https://api.siliconflow.cn/v1` (or `.com`) | `SILICONFLOW_API_KEY` |
| Groq (fast Llama hosting) | `groq` | `meta-llama/llama-3.3-70b-versatile` | `https://api.groq.com/openai/v1` | `GROQ_API_KEY` |
| DashScope / Qwen (Alibaba) | `dashscope` | `qwen-plus` | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `DASHSCOPE_API_KEY` |
| Zhipu GLM | `zhipu` | `glm-4-plus` | `https://open.bigmodel.cn/api/paas/v4` | `ZHIPU_API_KEY` |
| ModelScope | `modelscope` | `Qwen/Qwen2.5-7B-Instruct` | `https://api-inference.modelscope.cn/v1` | `MODELSCOPE_API_KEY` |

Example block for OpenRouter (a good secondary option; it can also serve DeepSeek models):

```bash
# --- OpenRouter (alternative gateway) ---
LANGCHAIN_PROVIDER=openrouter
LANGCHAIN_MODEL_NAME=deepseek/deepseek-chat
OPENROUTER_API_KEY=sk-or-xxxx
OPENROUTER_BASE_URL=https://openrouter.ai/api/v1
```

> All of these speak the OpenAI-compatible Chat Completions format, so they work with the
> generic `ChatOpenAI` transport. Check each provider's current model catalog; model id
> strings in this table are examples.

---

## 5. Decision Table: Scenario → Recommended Provider

| Classroom scenario | Recommended provider | Config summary | Hardware |
|---|---|---|---|
| Normal online class, small budget | **DeepSeek API** | `LANGCHAIN_PROVIDER=deepseek`, `LANGCHAIN_MODEL_NAME=deepseek-chat`, `DEEPSEEK_API_KEY`, `DEEPSEEK_BASE_URL=https://api.deepseek.com/v1` | Any laptop with internet; **no GPU** |
| Offline / no budget / privacy | **Ollama local** | `LANGCHAIN_PROVIDER=ollama`, `LANGCHAIN_MODEL_NAME=qwen2.5:7b`, `OLLAMA_BASE_URL=http://localhost:11434`, raise `VIBE_TRADING_SSE_TIMEOUT` | 8 GB RAM (7b); 4–6 GB (3b); CPU-only is OK but slow |
| Shared lab server with 1 GPU | **vLLM server** | `vllm serve Qwen/Qwen2.5-7B-Instruct`; point any OpenAI-compatible label at `http://<server>:8000/v1` | 1 GPU with ≥ 16 GB VRAM (7b) |
| DeepSeek blocked in region | **OpenRouter** or **SiliconFlow** | `openrouter` + `deepseek/deepseek-chat` (or `siliconflow-cn` + DeepSeek model) | Any laptop with internet |
| Students want the fastest demos | **Groq** (Llama hosted) | `LANGCHAIN_PROVIDER=groq`, `GROQ_API_KEY`, model `meta-llama/llama-3.3-70b-versatile` | Any laptop with internet; free tier available |
| Reasoning-heavy stretch exercise | **DeepSeek reasoner** | `LANGCHAIN_MODEL_NAME=deepseek-reasoner` (keep provider `deepseek`) | Any laptop with internet; higher token cost |

**Suggested course default: DeepSeek API (`deepseek-chat`)**, with **Ollama (`qwen2.5:7b`)**
as the documented offline fallback so the demo never depends on a single vendor.

---

## 6. Common Pitfalls and Fixes

| # | Symptom | Root cause | Fix |
|---|---|---|---|
| 1 | `RuntimeError: LANGCHAIN_MODEL_NAME is not set` at startup | Model env var missing | Set `LANGCHAIN_MODEL_NAME` (e.g. `deepseek-chat`) in `agent/.env` |
| 2 | 401 / `AuthenticationError` on first call | Missing or wrong API key, or key pasted with quotes/whitespace | Re-set `DEEPSEEK_API_KEY` (raw key, no quotes, no newline). The repo validates the key is ASCII with no control chars. Check `vibe-trading provider doctor` |
| 3 | `ModuleNotFoundError: langchain_deepseek` despite provider=deepseek | Native adapter not installed while `VIBE_TRADING_DEEPSEEK_ADAPTER=native` | `pip install "vibe-trading-ai[deepseek]"`, or set `VIBE_TRADING_DEEPSEEK_ADAPTER=auto` / `openai-compatible` |
| 4 | Requests hang or time out behind a proxy | `HTTP_PROXY`/`HTTPS_PROXY` intercepting API calls; or timeout too low | Set `VIBE_TRADING_DISABLE_HTTP_PROXY=1` (ignores proxy env for LLM calls only, still honors `SSL_CERT_FILE`), or raise `TIMEOUT_SECONDS` |
| 5 | Rate-limit / 429 errors in a large class | All students share one key, or bursts hit the provider's RPM/TPM limits | Use per-student or per-group keys; stagger exercise starts; lower `MAX_RETRIES` only after adding backoff; enable the repo's built-in retry (default `MAX_RETRIES=2`) |
| 6 | Unexpected cost spikes | Reasoning models or long agent loops burn tokens; temperature/reasoning settings | Default to `deepseek-chat` (not `deepseek-reasoner`); keep `LANGCHAIN_TEMPERATURE=0.0`; keep agent tasks small in the course; set per-account spending limits on the provider console |
| 7 | UI shows "Execution timed out" with local models | `VIBE_TRADING_SSE_TIMEOUT` (default 90 s) too short for CPU inference | Raise `VIBE_TRADING_SSE_TIMEOUT=300` (or more) for Ollama; prefer 3b/7b models |
| 8 | Accidental fallback to an OpenAI endpoint | `OPENAI_BASE_URL`/`OPENAI_API_KEY` set globally (e.g. in shell profile) and no `<PROVIDER>_BASE_URL` override | Set the provider's own `<PROVIDER>_BASE_URL`; check `vibe-trading provider doctor` shows the intended base URL; remove stray `OPENAI_BASE_URL` if it points at `api.openai.com` |
| 9 | Model responds but ignores tools | Model id not a tool-capable variant, or reasoning model quirks | Use `deepseek-chat` for tool-heavy ReAct/swarm tasks; keep `deepseek-reasoner` for analysis-only steps |
| 10 | `RuntimeError: langchain-openai is not installed` | Project base dependency missing (it is the generic transport, needed by every provider) | `pip install -e .` (or `pip install "vibe-trading-ai[deepseek]"` which pulls it) |

---

## 7. Cost & Security Notes for the Classroom

- **Never commit API keys.** `agent/.gitignore` covers `.env`; keep `agent/.env` out of any
  shared drive or git history. Rotate a leaked key immediately in the provider console.
- **Budget control:** DeepSeek and Groq have pay-as-you-go pricing; set a hard spending cap
  in the provider console before the class. Prefer `deepseek-chat` and short tasks.
- **Key sharing:** per-student or per-group keys isolate rate limits and cost; a single
  shared key is the fastest way to hit 429s.
- **No OpenAI dependency anywhere:** this document configures only DeepSeek, Ollama, vLLM
  (open-weight Qwen), and other non-OpenAI OpenAI-compatible gateways. If a student's
  environment still has `OPENAI_API_KEY`/`OPENAI_BASE_URL` exported, the doctor command
  (§1.3) will show it — override with the provider-specific vars listed in §2–§4.

---

*End of document. If upstream facts change (e.g. DeepSeek model ids or endpoints), update
§2.4 / §4 tables accordingly and re-run `vibe-trading provider doctor` to validate.*



