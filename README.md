# SwarmCraft — Reproduce Vibe-Trading with a Multi-Agent Team

**SwarmCraft** is a hands-on course that teaches multi-agent collaboration by reproducing
an open-source quantitative trading project — **[HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading)** —
using JiuwenSwarm workflows. All facts in the SwarmCraft package are verified against the public
Vibe-Trading repository (v0.1.14, GitHub API + official README + repository analysis,
fetched 2026-09-03).

---

## What This Is

**SwarmCraft** is a **hands-on course demo** that teaches students how to reproduce an
open-source quantitative trading project — **[HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading)** —
**collaboratively**, using JiuwenSwarm multi-agent workflows.

The demo itself was produced by a multi-agent team, showcasing the full pipeline:

1. **Background research** — repository analysis of Vibe-Trading
2. **Reproduction plan** — a staged, milestone-based plan for rebuilding the project
3. **Course / demo design** — the teaching syllabus and hands-on exercises
4. **English documentation** — this integrated package
5. **GitHub backup** — the final push-ready repository

### What students will learn

- How to decompose a real open-source project into reproducible milestones
- How to assign roles and coordinate work in a multi-agent team
- How to run review loops and verification gates between agents
- How to reproduce an LLM-powered trading agent without using OpenAI models
  (see [LLM Provider Guidance](docs/llm-provider-guidance.md))

---

## Why This Project

Vibe-Trading is a strong teaching vehicle because it is:

- **Real and popular** — ~32.3k GitHub stars, 5.2k+ forks, active development
- **Multi-agent by design** — investment, quant, crypto, and risk agent teams
- **LLM-driven but model-flexible** — works with DeepSeek, Ollama, and other open-source providers
- **Reproducible on modest hardware** — no API keys required for market data; LLM can run locally via Ollama

This course treats the reproduction itself as the learning exercise: students practice
multi-agent collaboration by rebuilding a real project step by step, rather than working
through toy examples.

---

## What Is Vibe-Trading? (verified facts)

> "Vibe-Trading: Your Personal Trading Agent"
> — *One Command to Empower Your Agent with Comprehensive Trading Capabilities*

Per the project's own README:

> Vibe-Trading is an open-source research workspace for turning finance questions into
> runnable analysis. It connects natural-language prompts to market-data loaders, strategy
> generation, backtest engines, reports, exports, and persistent research memory.

**Key headline features** (from the official README):

| Feature | Highlights |
|---|---|
| Self-Improving Trading Agent | Natural-language market research, strategy drafts, memory-backed workflows |
| Multi-Agent Trading Teams | Investment, quant, crypto, and risk teams; streaming progress; persisted reports |
| Cross-Market Data & Backtesting | A/HK/US/Canada/UK/India/Korea equities, crypto, futures, forex; data fallback; composite backtests |
| Shadow Account | Broker-journal behavior diagnostics, rule-based comparisons, audit reports |

**Tech stack** (verified from official README / GitHub API / repository analysis):

- Language: **Python 3.11+**
- Backend: **FastAPI**
- Frontend: **React 19**
- PyPI package: **`vibe-trading-ai`** (CLI commands: `vibe-trading`, `vibe-trading serve`, `vibe-trading-mcp`)
- Homepage / docs: <https://vibetrading.wiki/>
- Topics: `ai-agent`, `algorithmic-trading`, `backtesting`, `fintech`, `llm`, `mcp`, `multi-agent`, `python`, `quantitative-finance`, `trading`
- License: **MIT**
- Version analyzed: **0.1.14** (per `pyproject.toml`)

**Scale of the codebase** (per [Repository Analysis](docs/repo-analysis.md)):

| Dimension | Figure |
|---|---|
| Test files | 555 (unit + integration) |
| Bundled skills | 90 (`SKILL.md` files, 9 categories) |
| Alpha zoo | 462 alphas (Qlib 158 + alpha101 + GTJA 191 + academic + fundamental) |
| Swarm team presets | 30 YAML teams (equity research, quant desk, crypto desk, risk committee, ...) |
| Backtest engines | 10+ (per-market + composite cross-market) |
| Market data loaders | 24, with automatic fallback chains |
| Tool modules | 78 (≈107 auto-discovered tools) |

**Classroom-friendly facts** (per [Repository Analysis](docs/repo-analysis.md)):

- No GPU required — LLM calls are remote; backtests are CPU-bound and fine on modest hardware.
- All major markets work **without any data API key** (Yahoo/yfinance, OKX, mootdx/AKShare fallbacks).
- Zero-cost classroom setup needs only **one DeepSeek API key** (or zero keys with local Ollama).
- The stack is provider-agnostic; DeepSeek and open-source models are first-class, and no OpenAI/GPT model is required anywhere.

For the full architecture map, module breakdown, and reproduction difficulty assessment,
see [Repository Analysis](docs/repo-analysis.md).

---

## SwarmCraft Package Structure

```
.
├── README.md                     # this file — project overview
├── VERIFICATION.md               # quick verification checklist
├── docs/
│   ├── repo-analysis.md          # consolidated: repository analysis report
│   ├── reproduction-plan.md      # consolidated: staged reproduction plan (M0–M6)
│   ├── course-design.md          # consolidated: course syllabus & demo design
│   └── llm-provider-guidance.md  # consolidated: DeepSeek / open-source LLM setup
└── github-backup.md              # backup manifest & push checklist
```

> **Note on document sources:** the four upstream deliverables also exist at the package
> root (`repo-analysis.md`, `reproduction-plan.md`, `course-design.md`,
> `llm-provider-guidance.md`) as the multi-agent team's original outputs. `docs/` holds the
> **canonical consolidated versions** — terminology aligned, cross-references fixed — and
> is what the rest of this package links to.

---

## How to Use This Course

This is a **2-hour hands-on workshop** (1.5 h and 2.5 h variants included) for 12–40
students with basic Python + git skills. Students work in mini-swarms of 4–5; each team
reproduces a small slice of Vibe-Trading and practices multi-agent collaboration end to
end. See [Course Design](docs/course-design.md) for the full syllabus.

**Course at a glance** (from [Course Design](docs/course-design.md)):

| Phase | Time | What happens |
|---|---|---|
| Welcome & Hook | 0:00–0:10 | Why multi-agent? This course itself was designed by a multi-agent swarm |
| Core Concepts | 0:10–0:30 | Swarm anatomy: leader / members / task board / DAG / review loops |
| Live Demo | 0:30–0:55 | Instructor builds a swarm live, narrates every command |
| Hands-On Mini-Swarms | 0:55–1:40 | Teams reproduce a Vibe-Trading slice (install → data → strategy → backtest → review) |
| Show & Debrief | 1:40–1:50 | Teams demo results and their task-board journey |
| Discussion & Reflection | 1:50–2:05 | Guided questions, exit ticket |

**Student exercise slice** (per team, 4–6 tasks with a mini-DAG):
*"Run one moving-average crossover backtest with Vibe-Trading's engine, and produce a
5-line report."* This maps to the repo's real pipeline — data loader → strategy →
backtest engine → metrics/report — and works with **zero data API keys**.

**The only credential a student needs: one DeepSeek API key** (or none, with local
Ollama) — see [LLM Provider Guidance](docs/llm-provider-guidance.md). No OpenAI/GPT
models are used anywhere.

**Suggested flow:**

1. Instructor: read [Repository Analysis](docs/repo-analysis.md) (understand the target)
   and [Course Design](docs/course-design.md) (syllabus, demo script, instructor checklist).
2. Instructor: follow the environment pre-checks in Course Design §6 and the
   [LLM Provider Guidance](docs/llm-provider-guidance.md) to configure DeepSeek/Ollama.
3. Students: follow the [Reproduction Plan](docs/reproduction-plan.md) milestone by milestone.
4. Run the course exercises; use [VERIFICATION.md](VERIFICATION.md) to confirm each
   milestone is complete.

---

## LLM Provider Note (course constraint)

This course **does not use OpenAI GPT models**. All LLM components of the reproduction and
the course run on **DeepSeek or open-source models** (e.g., local Ollama).

Vibe-Trading itself is provider-agnostic — it supports DeepSeek, Ollama (no API key needed
for local use), OpenRouter, Groq, DashScope/Qwen, and many others, and ships DeepSeek as a
default provider option. The [LLM Provider Guidance](docs/llm-provider-guidance.md) document
makes the "no OpenAI" constraint concrete: it covers the DeepSeek API setup (course default,
model `deepseek-chat`), the Ollama local fallback, and non-OpenAI alternatives, with a
decision table mapping classroom scenarios to providers.

Both the [Reproduction Plan](docs/reproduction-plan.md) (Milestone 1) and the
[Course Design](docs/course-design.md) reference this guidance wherever LLM configuration
is mentioned. The only credential a student needs is one DeepSeek API key — or none at all
with a local open-source model.

---

## Credits & License

**SwarmCraft** is an **educational re-implementation** built around the open-source project:

- **Project:** [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) — *Vibe-Trading:
  Your Personal Trading Agent*
- **Author organization:** HKUDS (Hong Kong University Data Science group)
- **License:** MIT License

**License note:** Vibe-Trading is released under the **MIT License**. SwarmCraft
references, documents, and re-implements its architecture for teaching purposes; any code
or documentation copied from Vibe-Trading retains the original MIT license terms and
attribution. See the [LICENSE](LICENSE) file of Vibe-Trading
(<https://github.com/HKUDS/Vibe-Trading/blob/main/LICENSE>) for details.

SwarmCraft is a **separate documentation project** — it is not affiliated with or
endorsed by HKUDS.
