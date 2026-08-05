# Squad Hands-On Guide (Init Mode)

**Version:** Squad v0.11.0 · **Estimated Duration:** 30 minutes self-paced

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
  - [Installation reference](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli#installing-with-winget-windows)


*Note:* GitHub CLI (`gh`) is **optional** and only required for the optional GitHub extension lab.

---

## Setup Steps

### 1 — Create a New Git Repo

```powershell
mkdir squad-lab; cd squad-lab
git init
```

### 2 — Initialize Squad

```powershell
npm install -g @bradygaster/squad-cli@latest
squad init
```

> **Expected:** 
> - `.squad\` directory created with `team.md` and `routing.md`
> - Coordinator file placed at `.github\agents\squad.agent.md`
> - `.copilot\mcp-config.json`, `.github\workflows\`, `.github\skills\`, and `.mcp.json` created
>
> CLI prints: `Squad initialized. Run copilot --agent squad and tell it what you're building.`

### 3 — Verify

```powershell
code .
```

In Visual Studio Code Explorer, open:

- `.squad\team.md` — confirm it contains a markdown roster with the coordinator and team members.
- `.squad\routing.md` — confirm it maps concrete work types to team members.
- `.squad\config.json` — confirm it contains `{ "version": 1 }` or similar. No `stateBackend` key means the default `local` backend is active.

> **Expected:** The generated Squad files are visible and readable in the repository.

---

## Labs

### Lab 1 — Inspect Team and Routing

**Goal:** Read your team state and understand routing.

In Visual Studio Code, inspect `.squad\team.md` and `.squad\routing.md`. Then explore the available writer roles:

```powershell
squad roles --search writer
```

**Reflect:** Which member handles documentation? Which handles testing? Where would a bug-fix issue route?

---

### Lab 2 — Coordinator Routing via Copilot CLI 

**Goal:** Invoke the Squad custom agent and observe routing decisions.

1. In your project directory, start a Copilot CLI session:

```powershell
copilot --agent squad
```

2. At the session prompt, enter you own prompt and ask squad to perform a task:

```
Task: Add a <> feature to this repo.
```

**Note:** `copilot --agent squad` is the correct invocation — Squad is a custom agent through the standalone Copilot CLI, not a built-in slash command and not `gh copilot`.

---

### Lab 3 — Model Selection

**Goal:** Understand how to control which model Squad uses.

```powershell
squad status
```

> **Expected:** Shows squad configuration, state backend, and version.

```powershell
squad start --model gpt-4o
```

> **Expected:** Copilot CLI starts with Squad agent loaded using the specified model.

---

### Lab 4 — Monitoring

**Goal:** Explore session logs, memory, and ceremony patterns.

```powershell
squad nap --dry-run
squad cost --all
```

---

### Optional Extension — GitHub Issue to PR

> **Prerequisites:** `gh` CLI authenticated, a GitHub repo with Issues enabled, write permissions.

1. Create an issue with the `squad` label in your personal GitHub repo (use your own sample repo for this exercise).
2. Start a triage loop:

```powershell
squad triage --execute --max-concurrent 1
```

> **Expected:** Squad detects the labeled issue, spawns the assigned agent, and the agent opens a draft PR.

3. Review the PR on GitHub. Comment or merge if satisfied.


---

**→ [Supplementary Guide](./modes/init/supplementary.md)** — what to commit, conceptual background, troubleshooting, limitations, and responsible-use notes.

**→ [Scenario 1 — Build a Codebase-Specialized Squad](./scenarios/scenario-1-specialized-team.md)** — clone your own repository, run Squad init, and ask Squad to create a team grounded in its codebase patterns.
