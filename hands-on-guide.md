# Squad Hands-On Workshop Guide

**Version:** Squad v0.11.0 · **Estimated Duration:** 60 minutes facilitated or 30 minutes self-paced

**Audience:** Developers learning Squad for the first time  
**Purpose:** Hands-on walkthrough of Squad setup, core capabilities, and practical adoption patterns

[→ For facilitators: See the `facilitator-guide.md`](./facilitator-guide.md)

---

## Prerequisites

Before you start, verify you have:

- **Node.js** ≥ 22.5.0 (`node --version`)
- **npm** ≥ 10 (`npm --version`)
- **Git** (`git --version`)
- **GitHub Copilot CLI** (standalone) installed and authenticated (`copilot --version`)
- **Internet access** (for npm install to reach the registry)
- **A local Git repository** to work in (or run `git init my-lab && cd my-lab` to create one)
- **On Windows:** PowerShell 7+ is recommended; legacy cmd.exe may have syntax differences

*Note:* GitHub CLI (`gh`) is **optional** and only required if you plan to try the optional GitHub extension lab at the end.

---

## 1. Introduction

### What is Squad?

Squad is a lightweight AI-team framework. You define team members with specific roles — writer, engineer, tester, reviewer — and Squad routes work to the right agent based on what needs done. Each agent has a charter describing their domain. Named agents (cast from a movie universe such as Ocean's Eleven) appear consistently in logs, PR comments, and decisions.

Squad surfaces as a **custom Copilot agent** invoked through GitHub Copilot CLI. It is not a built-in slash command; you invoke it through the Copilot CLI task tool or via the `squad` CLI for setup commands.

### What You Will Build

By the end of this session you will have:

- A Squad installation in a local Git repo
- A team roster with routing rules
- Hands-on experience with four core capability areas
- An understanding of practical adoption patterns

---

## 2. Setup Variants

### Variant A — Fresh Init (Recommended)

1. Create and enter a new Git repo:

```bash
mkdir squad-lab && cd squad-lab
git init
```

> **Expected:** `Initialized empty Git repository in .../squad-lab/.git/`

2. Initialize Squad:

```bash
npx @bradygaster/squad-cli@latest init
```

> **Expected:** Squad prompts you to name your team, select an agent universe, and confirms `.squad/` directory created with `team.md` and `routing.md`. The coordinator file is placed at `.github/agents/squad.agent.md`.

3. Verify:

```bash
cat .squad/team.md
```

> **Expected:** A markdown roster showing your coordinator and at least one member.

### Variant B — Explicit State Backend (orphan or two-layer)

Use this when you want Squad's state isolated on a separate Git branch (orphan) or split across local files and a remote branch (two-layer). This is common for shared repos or CI integration.

```bash
npx @bradygaster/squad-cli@latest init --state-backend orphan
```

> **Expected:** Init completes. Inspect `.squad/config.json` or run `squad doctor` to confirm the backend setting. Squad state will live on a separate orphan branch.

For two-layer (local files + remote sync):

```bash
npx @bradygaster/squad-cli@latest init --state-backend two-layer
```

To sync state with remote:

```bash
squad sync --push
```

> **Expected:** `Syncing squad-state branch...` then confirmation. No-op for local backend.

---

### Variant D — Satellite Mode (Shared Team Root)

Use this when your repository should inherit its team definition from a Squad installation in another repository. See the dedicated guide for full setup, portability notes, and state backend interaction:

**→ [Satellite Mode Guide](./satellite-mode-guide.md)**

---

### Variant C — Upgrade an Existing Squad Installation

If you already have Squad installed and want to update it:

```bash
squad upgrade
```

> **Expected:** Output lists files overwritten, then confirms completion.

**What upgrade preserves vs. overwrites** (from `squad --help`):

| Preserved | Overwritten |
|-----------|-------------|
| `.squad/` directory (your team state, charters, routing, decisions) | `squad.agent.md` |
| `.ai-team/` directory if present | `.squad/templates/` directory |

> **Note:** Upgrade does **not** touch your team configuration. Your roster, charters, and decisions are safe.

**Codespaces / dev containers:** The setup flow is identical to the Linux commands above. No separate product setup is needed; run the same `npx @bradygaster/squad-cli@latest init` inside the container terminal.

---

## 2b. What to Commit

After running `squad init` (or `squad upgrade`), review `git status` before committing. Repository policy takes precedence over these defaults.

### Always commit — defines team behavior

| Path | Purpose |
|------|---------|
| `.github/agents/squad.agent.md` | Coordinator prompt (Squad-owned; overwritten by `upgrade`) |
| `.github/copilot-instructions.md` | Copilot context injected into every session |
| `.squad/team.md` | Team roster |
| `.squad/routing.md` | Work-routing rules |
| `.squad/ceremonies.md` | Ceremony config |
| `.squad/config.json` | State-backend and version metadata |
| `.squad/casting/policy.json` | Universe allowlist and capacity |
| `.squad/casting/registry.json` | Persistent agent-to-name mappings |
| `.squad/agents/{name}/charter.md` | Per-agent role and boundaries |
| `.squad/rai/policy.md`, `.squad/fact-checker/policy.md` | Built-in policy files |
| `.squad/templates/` | Format guides (Squad-owned; overwritten by `upgrade`) |
| `.mcp.json` | MCP server bridge config (required for Copilot CLI integration) |
| `.gitattributes` | Union-merge declarations for append-only Squad files |

### Mutable state — commit behavior varies by backend

These files are append-only and managed by runtime state tools:

| Path | local backend | orphan / two-layer backend |
|------|--------------|---------------------------|
| `.squad/decisions.md` | Committed to main branch | Stored on a separate orphan/state branch |
| `.squad/agents/{name}/history.md` | Committed to main branch | Stored on state branch |
| `.squad/rai/audit-trail.md`, `.squad/fact-checker/audit-trail.md` | Committed to main branch | Stored on state branch |
| `.squad/casting/history.json` | Committed to main branch | Stored on state branch |

> **`local` backend** (default when `config.json` has no backend key): all state lives in your working tree, committed to main. This is the simplest setup and what this lab uses.
>
> **`orphan` backend**: runtime state is isolated on a separate orphan branch, never touching main. Use `squad sync` to push/pull.
>
> **`two-layer` backend**: static config on main; mutable state on a remote state branch, synced via `squad sync --push`.

### Ignored by default (do not commit)

Squad adds the following to `.gitignore` at init. Entries may vary across CLI versions — verify your actual file:

```
.squad/orchestration-log/
.squad/log/
.squad/decisions/inbox/
.squad/sessions/
.squad/.scratch/
.squad/.cache/
.squad-workstream
```

### Secrets — never commit

Do not commit credentials, API keys, `.env` files, or any local secret material to Squad files or anywhere in the repository.

---

## 3. Core Capabilities

### Inspect Your Team

The `squad cast` command displays the current roster. With flags it adds a new agent:

```bash
cat .squad/team.md
```

> **Expected:** Markdown table with member names, roles, charter paths, and status.

To add an agent interactively:

```bash
squad cast --name Riley --role "QA Engineer"
```

> **Expected:** Prompts for role details, then appends Riley to `.squad/team.md`.

### Routing

Routing rules live in `.squad/routing.md`. Read them:

```bash
cat .squad/routing.md
```

> **Expected:** A routing table mapping work types to agent names.

To understand routing decisions, inspect the table and note the label-based issue routing rules — when a GitHub issue gets a `squad:{member}` label, that member picks it up in their next session.

### Model and Reasoning Configuration

Check available roles and built-ins:

```bash
squad roles
```

> **Expected:** List of built-in role categories (coordinator, tester, writer, etc.).

To run Squad with a specific model, pass model flags through Copilot CLI when invoking the Squad agent. For example, using the `squad start` command:

```bash
squad start --tunnel --model claude-sonnet-4
```

> **Expected:** Copilot CLI starts with the specified model and Squad agent loaded.

### Monitoring, Memory, and Ceremonies

Check token usage from orchestration logs:

```bash
squad cost
```

> **Expected:** Token usage table per agent. A fresh installation with no orchestration history will show no data — this is normal. Data accumulates after agent sessions run.

Inspect the orchestration log directory:

```bash
ls .squad/orchestration-log/   # macOS/Linux
Get-ChildItem .squad\orchestration-log\   # Windows PowerShell
```

> **Expected:** Empty on a fresh install; log files appear after triage or loop sessions.

Run context hygiene to prune/archive state:

```bash
squad nap --dry-run
```

> **Expected:** Preview of what would be pruned without making changes.

---

## 4. Labs

### Lab 1 — Inspect, Cast, and Routing

**Goal:** Read your team state and understand routing. *(~10 min)*

```bash
cat .squad/team.md
cat .squad/routing.md
squad roles --search writer
```

**Reflect:** Which member handles documentation? Which handles testing? Where would a bug-fix issue route?

---

### Lab 2 — Coordinator Routing via Copilot CLI

**Goal:** Invoke the Squad custom agent and observe routing decisions. *(~10 min)*

1. Open a Copilot CLI session: run `copilot` (standalone CLI) in your project directory.
2. From the agent picker or session prompt, select or invoke the **Squad** custom agent.
3. Enter the prompt:

```
Task: "We need to add unit tests for the auth module."
```

> **Expected:** The coordinator identifies this as a testing task, routes to the appropriate squad member per `routing.md`, and explains its reasoning.

**Note:** The Squad agent is invoked as a custom agent through the standalone Copilot CLI — it is not a built-in slash command and does not require `gh copilot`.

---

### Lab 3 — Model and Reasoning Configuration

**Goal:** Understand how to control which model Squad uses. *(~5 min)*

Check the current status:

```bash
squad status
```

> **Expected:** Shows which squad is active, the state backend, and configuration summary.

Review the `squad start` flags available:

```bash
squad start --help
```

> **Expected:** Lists flags including `--model`, `--tunnel`, `--port`, `--command`.

To launch Squad with a custom model (requires Copilot CLI):

```bash
squad start --model gpt-4o
```

> **Expected:** Copilot CLI starts with Squad agent and the specified model active.

---

### Lab 4 — Monitoring, Memory, and Ceremonies

**Goal:** Explore session logs, memory, and ceremony patterns. *(~10 min)*

```bash
squad nap --dry-run
squad cost --all
```

Run a triage dry-run (no execution):

```bash
squad triage
```

> **Expected:** Squad scans for actionable work and reports findings. With no GitHub issues configured, it may report nothing to do — that is correct behavior.

Inspect decisions:

```bash
cat .squad/decisions.md
```

> **Expected:** Active decisions or the governance template if none are recorded yet.

---

### Optional Extension — GitHub Issue to PR

> **Prerequisites:** `gh` CLI authenticated, a GitHub repo with Issues enabled, write permissions to create labels and PRs.

Try this if you want to see Squad in action on a real GitHub issue:

1. Create an issue with the `squad` label in your GitHub repo.
2. The coordinator triages it and applies a `squad:{member}` label to route the work.
3. Start a triage loop:

```bash
squad triage --execute --max-concurrent 1
```

> **Expected:** Squad detects the labeled issue, spawns the assigned agent, and the agent opens a draft PR.

4. Review the PR on GitHub. Comment or merge if satisfied.

---

## 5. Next Steps

After this session:

1. Read `.squad/team.md`, `.squad/routing.md`, `.squad/decisions.md` — these are your team's source of truth.
2. Try the optional GitHub extension with a real issue in your own repo.
3. Run `squad nap` (without `--dry-run`) to archive stale state after active use.
4. Check `squad --help` for the full command reference — it reflects the installed version.
5. To stay current: `squad upgrade` — preserves your team state, updates Squad-owned files.

### Useful Commands Reference

| Command | Purpose |
|---------|---------|
| `squad init` | Initialize Squad in a repo |
| `squad upgrade` | Update Squad-owned files (preserves team state) |
| `squad status` | Show active squad and backend |
| `squad cast` | Add a new agent to the team |
| `squad roles` | List built-in roles |
| `squad cost` | Report token usage from logs |
| `squad triage` | Scan for work; `--execute` to act |
| `squad nap` | Prune and archive state |
| `squad sync` | Sync orphan/two-layer state with remote |

---

## Completion Checklist

Mark each item when done:

- [ ] Completed Variant A (fresh init) — `.squad/` directory exists
- [ ] Verified `team.md` and `routing.md` contents
- [ ] Completed Lab 1 — cast inspection and routing review
- [ ] Completed Lab 2 — coordinator routing via Copilot CLI
- [ ] Completed Lab 3 — model/status configuration
- [ ] Completed Lab 4 — monitoring, memory, and triage dry-run
- [ ] (Optional) Completed the GitHub issue-to-PR extension
- [ ] No unresolved errors in your session
