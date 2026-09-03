# GitHub Backup Manifest & Push Checklist

**Task:** Prepare the SwarmCraft course-demo project for GitHub backup; push when the user provides
the repository URL.
**Status:** ✅ **PUSH COMPLETE.** Target repo:
`https://github.com/congw729/reproduce-repo-craft-demo` (SSH remote
`git@github.com:congw729/reproduce-repo-craft-demo.git`). The repository is named
`reproduce-repo-craft-demo` (user's choice) while the project inside is branded
**SwarmCraft** — both names are kept as-is.

**Pushed:** branch `main` · 3 commits (f157861 → 7eecd75 → 7a1dc97) · verified by
re-clone. Commit author uses GitHub noreply email
(`115451386+congw729@users.noreply.github.com`).

This document defines: (1) the file manifest to publish, (2) what must NOT be published
(and the `.gitignore` to enforce it), (3) the recommended commit sequence, and (4) the
step-by-step push checklist.

---

## 1. Backup Manifest (files to push)

The public repository contains the **polished SwarmCraft course-demo package only**. Canonical
documents live in `docs/`; upstream originals at the workspace root are internal working
copies and are **not** published.

| File | Purpose | Include |
|---|---|---|
| `README.md` | Project overview, verified facts, course usage, credits + MIT license note | ✅ |
| `VERIFICATION.md` | Env pre-checks, smoke test, M0–M6 milestone gates, safety checklist | ✅ |
| `docs/repo-analysis.md` | Consolidated repository analysis report | ✅ |
| `docs/reproduction-plan.md` | Consolidated reproduction plan (Milestones 0–6) | ✅ |
| `docs/course-design.md` | Consolidated course syllabus & demo design | ✅ |
| `docs/llm-provider-guidance.md` | Consolidated DeepSeek / open-source LLM setup (no OpenAI) | ✅ |
| `LICENSE` | Course-package license (suggested: MIT, with upstream attribution note) | ✅ recommended |
| `.gitignore` | Excludes internal / secret files (see §2) | ✅ |
| `github-backup.md` | This manifest + checklist (optional; useful as repo documentation) | ⚠️ optional |

**Total public payload: ~100 KB of Markdown** (README 10 KB + VERIFICATION 5.7 KB +
docs/ ~86 KB).

### 1.1 Files that must NOT be pushed

| Path (in team workspace) | Why excluded |
|---|---|
| `.team-meta/` | Internal team metadata / version history |
| `trajectories/` | Internal agent trajectory logs |
| `skills/` | Internal skill symlinks (skill-creator, swarmskill-creator) |
| `artifacts/` | Internal artifacts staging |
| Root-level `repo-analysis.md`, `reproduction-plan.md`, `course-design.md`, `llm-provider-guidance.md` | Upstream originals — superseded by the consolidated `docs/` versions |
| Any `.env`, `agent/.env`, `*.key`, `*token*` | Secrets — never publish (matches Vibe-Trading's own `.gitignore` hygiene) |
| Any broker exports, trade journals, run artifacts | Personal/real financial data — out of scope |

---

## 2. `.gitignore` Recommendations

Publish a `.gitignore` in the repo root. Recommended content:

```gitignore
# --- Secrets (never commit) ---
.env
agent/.env
*.key
*.pem
*token*
*credential*
secrets.*

# --- Python ---
__pycache__/
*.py[cod]
.venv/
venv/
*.egg-info/
.pytest_cache/

# --- Node ---
node_modules/
frontend/node_modules/
frontend/dist/

# --- Course data & journals ---
trades_export.csv
trades_export*.csv
*.broker-export.csv
runs/
~/.vibe-trading/

# --- Team-internal (if repo initialized inside the team workspace) ---
.team-meta/
trajectories/
skills/
artifacts/
workspaces/

# --- OS ---
.DS_Store
Thumbs.db
```

> **Recommended workflow:** initialize git in a **clean staging directory** that contains
> only the manifest files (copy README.md, VERIFICATION.md, docs/, LICENSE, .gitignore
> there), rather than inside the team workspace. This makes the exclusion list trivial and
> guarantees no internal file is ever staged.

---

## 3. Suggested Commit Sequence

Follow this order for a clean history (each commit is independently reviewable):

| # | Commit | Contents | Rationale |
|---|---|---|---|
| 1 | `chore: initial commit — SwarmCraft scaffolding` | `LICENSE`, `.gitignore`, `README.md` | Foundation: license + navigation first |
| 2 | `docs: add consolidated SwarmCraft docs` | `docs/` (4 documents) | The body of the course material |
| 3 | `docs: add verification checklist` | `VERIFICATION.md` | Quality gate for the reproduction milestones |
| 4 (optional) | `docs: add github backup manifest` | `github-backup.md` | Repo-level documentation |

Use conventional-commit style messages. **Do not** squash into one giant commit — the
multi-stage history mirrors the multi-agent workflow the course teaches.

---

## 4. Step-by-Step Push Checklist

Execute these steps **only when the GitHub repository URL arrives** (pending from user).

### Pre-flight (already done)
- [x] All four upstream deliverables consolidated into `docs/` (task-docs-integration)
- [x] README.md finalized with credits + MIT license note
- [x] VERIFICATION.md finalized
- [x] Manifest, `.gitignore`, commit sequence defined (this document)

### When URL arrives

1. **Create a clean staging directory** (do not push the whole team workspace):
   ```bash
   mkdir -p ~/swarmcraft-repo && cd ~/swarmcraft-repo
   ```
2. **Copy the public files:**
   ```bash
   cp .team/quant-swarm-course/README.md .team/quant-swarm-course/VERIFICATION.md .
   cp .team/quant-swarm-course/LICENSE .
   cp -r .team/quant-swarm-course/docs .
   cp .team/quant-swarm-course/github-backup.md .
   # add the .gitignore from §2
   ```
3. **Initialize git and sanity-check staged files:**
   ```bash
   git init
   git add -A
   git status            # MUST show only the manifest files — no .team/, no .env, no trajectories
   ```
4. **Commit in sequence (§3):**
   ```bash
   git commit -m "chore: initial commit — SwarmCraft scaffolding"
   git commit -m "docs: add consolidated SwarmCraft docs"          # after staging docs/ separately
   git commit -m "docs: add verification checklist"
   ```
   (Adjust `git add` per commit — add only the files each commit intends.)
5. **Add the remote and push:**
   ```bash
   git branch -M main
   git remote add origin https://github.com/congw729/reproduce-repo-craft-demo
   git push -u origin main
   ```
6. **Verify remotely:**
   - [ ] `git ls-remote origin` shows the pushed ref
   - [ ] Repo page renders README.md correctly
   - [ ] `docs/` links in README resolve (relative paths are preserved)
   - [ ] No `.env` / secret / internal files visible in the file browser
7. **Confirm to the leader** with the repo URL and a summary.

### Rollback / safety notes
- If `git remote add origin <URL>` was already set, use `git remote set-url origin <URL>`.
- To abort a mistaken push: `git reset --hard HEAD~1` before pushing further commits
  (never rewrite history after it is public without explicit agreement).
- Never force-push to a shared branch.

---

## 5. Open Items

| # | Item | Owner | Status |
|---|---|---|---|
| 1 | GitHub repository URL — `https://github.com/congw729/reproduce-repo-craft-demo` | User | ✅ **PROVIDED** |
| 2 | SwarmCraft `LICENSE` file (MIT + upstream attribution to HKUDS/Vibe-Trading) | doc-writer | ✅ Added |
| 3 | Push execution — branch `main`, HEAD `7a1dc97`, verified by re-clone | doc-writer | ✅ **COMPLETE** |

*This document was prepared by the Documentation Integrator (doc-writer) as part of
task-github-backup. All content is English; nothing has been pushed yet.*
