# Init Mode — Setup Guide

**Version:** Squad v0.11.0 · **Estimated Duration:** 60 minutes facilitated or 30 minutes self-paced

**Audience:** Developers learning Squad for the first time  
**Purpose:** Hands-on walkthrough of Squad default (`local`) init, core capabilities, and labs

[→ Alternative Modes](./alternative-modes-guide.md) · [→ Supplementary (commit guidance, troubleshooting, concepts)](./modes/init/supplementary.md)

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
- **Internet access** (npm install must reach the registry)
- **A local Git repository** to work in (or run `git init my-lab; cd my-lab` to create one)
- **On Windows:** PowerShell 7+ is recommended. In PowerShell 5.x `&&` is not supported — chain commands with `;` instead.

*Note:* GitHub CLI (`gh`) is **optional** and only required for the optional GitHub extension lab.

---

## Setup Steps

### 1 — Create a New Git Repo

```powershell
mkdir squad-lab; cd squad-lab
git init
```

> **Expected:** `Initialized empty Git repository in .../squad-lab/.git/`

### 2 — Initialize Squad

```powershell
npx @bradygaster/squad-cli@latest init
```

> **Expected:** Squad prompts you interactively — confirm the package installation, name your team, select an agent universe, and optionally connect a `@copilot` agent. On completion:
> - `.squad\` directory created with `team.md` and `routing.md`
> - Coordinator file placed at `.github\agents\squad.agent.md`
> - `.copilot\mcp-config.json`, `.github\workflows\`, `.github\skills\`, and `.mcp.json` created
>
> CLI prints: `Squad initialized. Run copilot --agent squad and tell it what you're building.`

### 3 — Verify

```powershell
Get-Content .squad\team.md
```

> **Expected:** A markdown roster showing your coordinator and at least one member.

```powershell
Get-Content .squad\routing.md
```

> **Expected:** A routing table mapping work types to agent names.

```powershell
Get-Content .squad\config.json
```

> **Expected:** `{ "version": 1 }` or similar — no `stateBackend` key means the `local` backend is active.

---

## Labs

### Lab 1 — Inspect, Cast, and Routing *(~10 min)*

**Goal:** Read your team state and understand routing.

```powershell
Get-Content .squad\team.md
Get-Content .squad\routing.md
squad roles --search writer
```

**Reflect:** Which member handles documentation? Which handles testing? Where would a bug-fix issue route?

---

### Lab 2 — Coordinator Routing via Copilot CLI *(~10 min)*

**Goal:** Invoke the Squad custom agent and observe routing decisions.

1. In your project directory, start a Copilot CLI session:

```powershell
copilot --agent squad
```

2. At the session prompt, enter:

```
Task: We need to add unit tests for the auth module.
```

> **Expected:** The coordinator identifies this as a testing task, routes to the appropriate squad member per `routing.md`, and explains its reasoning.

**Note:** `copilot --agent squad` is the correct invocation — Squad is a custom agent through the standalone Copilot CLI, not a built-in slash command and not `gh copilot`.

---

### Lab 3 — Model Selection *(~5 min)*

**Goal:** Understand how to control which model Squad uses.

```powershell
squad status
```

> **Expected:** Shows squad configuration, state backend, and version.

```powershell
squad start --help
```

> **Expected:** Lists flags including `--model`, `--tunnel`, `--port`, `--command`.

```powershell
squad start --model gpt-4o
```

> **Expected:** Copilot CLI starts with Squad agent loaded using the specified model.

---

### Lab 4 — Monitoring, Memory, and Ceremonies *(~10 min)*

**Goal:** Explore session logs, memory, and ceremony patterns.

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
> **Expected (local-only repo with no `origin` remote):** Error — `Could not detect platform: No git remote "origin" found. Cannot create platform adapter.` This is expected. Add `git remote add origin <url>` to use triage.

```powershell
Get-Content .squad\decisions.md
```

> **Expected:** Active decisions or the governance template if none are recorded yet.

---

### Optional Extension — GitHub Issue to PR

> **Prerequisites:** `gh` CLI authenticated, a GitHub repo with Issues enabled, write permissions.

1. Create an issue with the `squad` label in your GitHub repo.
2. Start a triage loop:

```powershell
squad triage --execute --max-concurrent 1
```

> **Expected:** Squad detects the labeled issue, spawns the assigned agent, and the agent opens a draft PR.

3. Review the PR on GitHub. Comment or merge if satisfied.

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

---

**→ [Supplementary Guide](./supplementary.md)** — what to commit, conceptual background, troubleshooting, limitations, and responsible-use notes.
