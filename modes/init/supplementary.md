# Init Mode — Supplementary Guide

**Version:** Squad v0.11.0

[← Default Hands-On Lab](../../Readme.md) · [← Alternative Modes Guide](../../alternative-modes-guide.md)

---

## When to Use This Mode

The default `local` init stores all Squad state — team roster, routing rules, decisions, and history — in the working tree of your repository, committed to main. Choose this when you want the simplest possible setup with no external dependencies.

> **Non-default modes:** For orphan/two-layer state backends, satellite mode, or external state, see **[alternative-modes-guide.md](../../alternative-modes-guide.md)**.

---

## What to Commit

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

> **Casting files (`.squad/casting/policy.json`, `.squad/casting/registry.json`):** Commit these when using the **default `local` backend**. Under `orphan` or `two-layer` backends the pre-commit hook may block them. Follow the specific mode guide.

### Mutable state — local backend (default for this lab)

These files are append-only, managed by runtime state tools, and live in your working tree with the `local` backend:

| Path | Notes |
|------|-------|
| `.squad/decisions.md` | Active decisions |
| `.squad/agents/{name}/history.md` | Per-agent session history |
| `.squad/rai/audit-trail.md`, `.squad/fact-checker/audit-trail.md` | Built-in audit trails |
| `.squad/casting/history.json` | Agent-to-name casting history |

> **`local` backend** (default): all state lives in your working tree, committed to main. Simplest setup; what this lab uses.
>
> For `orphan` or `two-layer` backends, see [alternative-modes-guide.md](../../alternative-modes-guide.md).

### Ignored by default

Squad adds the following to `.gitignore` at init. Entries may vary across CLI versions — verify your actual file:

```gitignore
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

## What Is Squad?

Squad is a lightweight AI-team framework. You define team members with specific roles — writer, engineer, tester, reviewer — and Squad routes work to the right agent based on what needs done. Each agent has a charter describing their domain. Named agents (cast from a movie universe such as Ocean's Eleven) appear consistently in logs, PR comments, and decisions.

Squad surfaces as a **custom Copilot agent** invoked through the GitHub Copilot CLI (the standalone `copilot` command). It is not a built-in slash command; you invoke it with `copilot --agent squad` after init, or through the `squad` CLI for setup commands.

### What You Will Build

By the end of this session you will have:

- A Squad installation in a local Git repo
- A team roster with routing rules
- Hands-on experience with four core capability areas
- An understanding of practical adoption patterns

---

## Core Capabilities Reference

### Inspect Your Team

```powershell
squad cast
```

> **Expected:** Lists current team members with names, roles, and charter paths.

> **Note (v0.11.0 known limitation):** `squad cast --name <Name> --role "<Role>"` launches an interactive wizard but does **not** write the new member to `team.md`. To modify the roster, edit `team.md` directly.

```powershell
Get-Content .squad\team.md
```

### Routing Rules

```powershell
Get-Content .squad\routing.md
```

> **Expected:** Routing table mapping work types and issue labels to agent names.

### Model Selection and Squad Start

```powershell
squad roles
```

> **Expected:** List of built-in role categories (coordinator, tester, writer, etc.).

```powershell
squad start --tunnel --model claude-sonnet-4
```

> **Expected:** Copilot CLI starts with the specified model and Squad agent loaded. Requires Copilot CLI installed and authenticated.

### Monitoring, Memory, and Ceremonies

```powershell
squad cost
```

> **Expected:** Token usage table per agent. A fresh install shows no data — this is normal. Data accumulates after agent sessions run.

```powershell
Get-ChildItem .squad\orchestration-log\
```

> **Expected:** Empty on a fresh install; log files appear after triage or loop sessions.

```powershell
squad nap --dry-run
```

> **Expected:** Preview of what would be pruned without making changes.

---

## v0.11.0 Known Limitations

- **`squad cast --name` is a stub:** The `--name`/`--role` flags launch an interactive wizard but do **not** write the new member to `team.md`. To add or modify members, edit `.squad\team.md` directly.
- **`squad sync --help` does not provide detailed flag help:** Use `squad help` instead.
- **`squad doctor` false warnings:** With non-local backends, doctor may report `decisions.md` not found — expected because decisions live on the state branch. Use `Get-Content .squad\config.json` and `squad status` as primary verification.
- **`squad cost` shows no data on fresh install:** Normal. Token data accumulates after agent sessions run.
- **`squad triage` hard-errors without a remote:** Requires `git remote add origin <url>` pointing to a GitHub repository.

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `squad: command not found` | CLI not in PATH | Run `npx @bradygaster/squad-cli@latest <command>` or `npm install -g @bradygaster/squad-cli` |
| npm install fails with proxy/network error | Corporate firewall | Set `npm config set proxy http://proxy:port`; verify registry access |
| `Not a git repository` error on `squad init` | No `.git` dir | Run `git init` first, then `squad init` |
| Permission denied on `.squad/` | File ACL issue | Check directory ownership; on Windows run PowerShell as user (not admin) |
| Windows PowerShell: `&&` not recognized | PowerShell 5.x does not support `&&` | Use `;` to chain commands (e.g., `mkdir lab; cd lab; git init`). PowerShell 7+ and cmd.exe support `&&`. |
| npm scripts blocked by execution policy | PowerShell script execution policy | Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` as a troubleshooting step only. |
| Squad agent not available in Copilot CLI | MCP bridge not configured | Check `.mcp.json` in project root; ensure `squad_state` server is configured |
| Canary token missing warning | `squad.agent.md` truncated or absent | Run `squad upgrade` to restore; restart Copilot CLI session |
| `squad cost` shows no data | No orchestration logs yet | Normal on fresh install; data appears after agent sessions run |
| `squad triage` hard-errors with "Cannot create platform adapter" | No git `origin` remote | Add `git remote add origin <url>`; triage is optional on local-only repos |

---

## Responsible-Use Notes

> **Data residency:** When using non-local state backends (orphan, two-layer), Squad state may be synchronized to a remote repository. Apply your organization's repository access and data-handling policies accordingly.

> **Orchestration/cost logs:** `squad cost` and the orchestration-log directory expose operational metadata (token counts, agent activity). Treat these like any other committed artifact and apply appropriate access controls.

---

## Useful Commands Reference

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

## Next Steps

1. Read `.squad\team.md`, `.squad\routing.md`, `.squad\decisions.md` — your team's source of truth.
2. Try the optional GitHub extension with a real issue in your own repo.
3. Run `squad nap` (without `--dry-run`) to archive stale state after active use.
4. Check `squad --help` for the full command reference.
5. To stay current: `squad upgrade` — preserves team state, updates Squad-owned files.

---

*[← Default Hands-On Lab](../../Readme.md) · [← Alternative Modes](../../alternative-modes-guide.md)*
