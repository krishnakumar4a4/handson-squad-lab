# Squad Hands-On Workshop Guide

**Version:** Squad v0.11.0 · **Estimated Duration:** 60 minutes facilitated or 30 minutes self-paced

**Audience:** Developers learning Squad for the first time  
**Purpose:** Hands-on walkthrough of Squad setup, core capabilities, and practical adoption patterns

---

## Prerequisites

Before you start, verify you have:

- **Node.js** ≥ 22.5.0 (`node --version`)
- **npm** ≥ 10 (`npm --version`)
- **Git** (`git --version`)
- **GitHub Copilot CLI** (the standalone `copilot` command) installed and authenticated (`copilot --version`)
  - This is the standalone command-line tool, **distinct from** the GitHub Copilot desktop app and from VS Code's built-in Copilot slash commands.
  - No official standalone package has been confirmed for automated install via `winget` or `npm`. **Facilitators must pre-install it on participant machines.** See the [official GitHub Copilot CLI installation documentation](https://docs.github.com/en/copilot/github-copilot-in-the-cli) for the authoritative install path.
  - After `squad init` completes, the verified invocation is `copilot --agent squad`.
- **Internet access** (for npm install to reach the registry)
- **A local Git repository** to work in (or run `git init my-lab; cd my-lab` to create one)
- **On Windows:** PowerShell 7+ is recommended. In PowerShell 5.x `&&` is not supported — chain commands with `;` instead.

*Note:* GitHub CLI (`gh`) is **optional** and only required if you plan to try the optional GitHub extension lab at the end.

---

## 1. Introduction

### What is Squad?

Squad is a lightweight AI-team framework. You define team members with specific roles — writer, engineer, tester, reviewer — and Squad routes work to the right agent based on what needs done. Each agent has a charter describing their domain. Named agents (cast from a movie universe such as Ocean's Eleven) appear consistently in logs, PR comments, and decisions.

Squad surfaces as a **custom Copilot agent** invoked through the GitHub Copilot CLI (the standalone `copilot` command). It is not a built-in slash command; you invoke it with `copilot --agent squad` after init, or through the `squad` CLI for setup commands.

### What You Will Build

By the end of this session you will have:

- A Squad installation in a local Git repo
- A team roster with routing rules
- Hands-on experience with four core capability areas
- An understanding of practical adoption patterns

---

## 2. Setup — Fresh Init (Default)

> **Non-default modes:** If you need to upgrade an existing installation, use an orphan/two-layer state backend, satellite mode, or external state, see **[alternative-modes-guide.md](./alternative-modes-guide.md)**.

1. Create and enter a new Git repo:

```powershell
mkdir squad-lab; cd squad-lab
git init
```

> **Expected:** `Initialized empty Git repository in .../squad-lab/.git/`

2. Initialize Squad:

```powershell
npx @bradygaster/squad-cli@latest init
```

> **Expected:** Squad will prompt you interactively — confirm the package installation, name your team, select an agent universe, and optionally connect a `@copilot` agent. On completion, `.squad\` directory is created with `team.md` and `routing.md`; the coordinator file is placed at `.github\agents\squad.agent.md`; `.copilot\mcp-config.json`, `.github\workflows\`, and `.github\skills\` are also created. When complete, the CLI prints:
> `Squad initialized. Run copilot --agent squad and tell it what you're building.`

3. Verify:

```powershell
Get-Content .squad\team.md
```

> **Expected:** A markdown roster showing your coordinator and at least one member.

---

## 2b. What to Commit

After running `squad init`, review `git status` before committing. Repository policy takes precedence over these defaults.

### Always commit — defines team behavior

| Path | Purpose |
|------|---------|
| `.github/agents/squad.agent.md` | Coordinator prompt (Squad-owned; overwritten by `upgrade`) |
| `.github/copilot-instructions.md` | Copilot context injected into every session |
| `.github/skills/` | Built-in skill library synced by Squad (Squad-owned; overwritten by `upgrade`) |
| `.github/workflows/` | Workflow files created at init |
| `.copilot/mcp-config.json` | MCP configuration for Copilot integration |
| `.squad/team.md` | Team roster |
| `.squad/routing.md` | Work-routing rules |
| `.squad/ceremonies.md` | Ceremony config |
| `.squad/config.json` | State-backend and version metadata |
| `.squad/agents/{name}/charter.md` | Per-agent role and boundaries |
| `.squad/rai/policy.md`, `.squad/fact-checker/policy.md` | Built-in policy files |
| `.squad/templates/` | Format guides (Squad-owned; overwritten by `upgrade`) |
| `.mcp.json` | MCP server bridge config (required for Copilot CLI integration) |
| `.gitattributes` | Union-merge declarations for append-only Squad files |

> **Casting files (`.squad/casting/policy.json`, `.squad/casting/registry.json`):** Commit these when using the **default `local` backend**. Under `orphan` or `two-layer` backends, the pre-commit hook may block them as runtime-owned state. Follow the specific mode guide for guidance.

### Mutable state — local backend (default for this lab)

These files are append-only and managed by runtime state tools. With the default `local` backend, they live in your working tree and commit to main:

| Path | Notes |
|------|-------|
| `.squad/decisions.md` | Active decisions |
| `.squad/agents/{name}/history.md` | Per-agent session history |
| `.squad/rai/audit-trail.md`, `.squad/fact-checker/audit-trail.md` | Built-in audit trails |
| `.squad/casting/history.json` | Agent-to-name casting history |

> **`local` backend** (default when `config.json` has no backend key): all state lives in your working tree, committed to main. This is the simplest setup and what this lab uses.
>
> For `orphan` or `two-layer` backends where state lives on a separate branch, see [alternative-modes-guide.md](./alternative-modes-guide.md).

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

The `squad cast` command displays the current roster:

```powershell
squad cast
```

> **Expected:** Lists current team members with names, roles, and charter paths.

> **Note (v0.11.0 known limitation):** Adding a new agent via `squad cast --name <Name> --role "<Role>"` launches an interactive wizard but does **not** write the new member to `team.md` in v0.11.0. This feature is in development. To view or modify the roster, read `team.md` directly:

```powershell
Get-Content .squad\team.md
```

> **Expected:** Markdown table with member names, roles, charter paths, and status.

### Routing

Routing rules live in `.squad\routing.md`. Read them:

```powershell
Get-Content .squad\routing.md
```

> **Expected:** A routing table mapping work types to agent names.

To understand routing decisions, inspect the table and note the label-based issue routing rules — when a GitHub issue gets a `squad:{member}` label, that member picks it up in their next session.

### Model Selection and Squad Start

To see which agent roles are built into Squad, run:

```powershell
squad roles
```

> **Expected:** List of built-in role categories (coordinator, tester, writer, etc.).

To run Squad with a specific model, pass model flags when invoking `squad start`:

```powershell
squad start --tunnel --model claude-sonnet-4
```

> **Expected:** Copilot CLI starts with the specified model and Squad agent loaded. Requires the Copilot CLI to be installed and authenticated.

### Monitoring, Memory, and Ceremonies

Check token usage from orchestration logs:

```powershell
squad cost
```

> **Expected:** Token usage table per agent. A fresh installation with no orchestration history will show no data — this is normal. Data accumulates after agent sessions run.

Inspect the orchestration log directory:

```powershell
Get-ChildItem .squad\orchestration-log\
```

> **Expected:** Empty on a fresh install; log files appear after triage or loop sessions.

Run context hygiene to prune/archive state:

```powershell
squad nap --dry-run
```

> **Expected:** Preview of what would be pruned without making changes.

---

## 4. Labs

### Lab 1 — Inspect, Cast, and Routing

**Goal:** Read your team state and understand routing. *(~10 min)*

```powershell
Get-Content .squad\team.md
Get-Content .squad\routing.md
squad roles --search writer
```

**Reflect:** Which member handles documentation? Which handles testing? Where would a bug-fix issue route?

### Lab 2 — Coordinator Routing via Copilot CLI

**Goal:** Invoke the Squad custom agent and observe routing decisions. *(~10 min)*

1. In your project directory, start a Copilot CLI session:

```powershell
copilot --agent squad
```

2. At the session prompt, enter the task:

```
Task: We need to add unit tests for the auth module.
```

> **Expected:** The coordinator identifies this as a testing task, routes to the appropriate squad member per `routing.md`, and explains its reasoning.

**Note:** `copilot --agent squad` is the correct invocation — the Squad agent is a custom agent loaded through the standalone Copilot CLI, not a built-in slash command and not `gh copilot`.

---

### Lab 3 — Model Selection

**Goal:** Understand how to control which model Squad uses. *(~5 min)*

Check the current configuration:

```powershell
squad status
```

> **Expected:** Shows squad configuration, state backend, and version. Active model display depends on whether a Copilot CLI session is running.

Review the `squad start` flags available:

```powershell
squad start --help
```

> **Expected:** Lists flags including `--model`, `--tunnel`, `--port`, `--command`.

To launch Squad with a custom model (requires Copilot CLI installed and authenticated):

```powershell
squad start --model gpt-4o
```

> **Expected:** Copilot CLI starts with Squad agent loaded using the specified model.

---

### Lab 4 — Monitoring, Memory, and Ceremonies

**Goal:** Explore session logs, memory, and ceremony patterns. *(~10 min)*

```powershell
squad nap --dry-run
squad cost --all
```

**Optional — triage dry-run** (requires a git remote named `origin`):

```powershell
squad triage
```

> **Expected (with GitHub remote configured):** Squad scans for actionable work and reports findings.
>
> **Expected (local-only repo with no `origin` remote):** Error — `Could not detect platform: No git remote "origin" found. Cannot create platform adapter.` This is expected behavior. To use triage, run `git remote add origin <url>` pointing to a GitHub repository, or skip this step.

Inspect decisions:

```powershell
Get-Content .squad\decisions.md
```

> **Expected:** Active decisions or the governance template if none are recorded yet.

---

### Optional Extension — GitHub Issue to PR

> **Prerequisites:** `gh` CLI authenticated, a GitHub repo with Issues enabled, write permissions to create labels and PRs.

Try this if you want to see Squad in action on a real GitHub issue:

1. Create an issue with the `squad` label in your GitHub repo.
2. The coordinator triages it and applies a `squad:{member}` label to route the work.
3. Start a triage loop:

```powershell
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
| `squad cast` | Display current team roster; `--name` flag is a v0.11.0 stub |
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
- [ ] Completed Lab 4 — monitoring, memory, and triage (triage optional if no GitHub remote)
- [ ] (Optional) Completed the GitHub issue-to-PR extension
- [ ] No unresolved errors in your session
